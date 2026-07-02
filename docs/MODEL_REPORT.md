# Model Report — Credit Cat

**Pillar 2 · Unsupervised ML ("Find Your Cluster")** · Team: Emmet & Prithvi
Dataset: UCI Credit Approval — 689 applicants, 14 anonymized features (A1–A14), a historical approve/deny `Class`, and `CustomerID`.

---

## 1. Approach

We drop `CustomerID` (an identifier) and **hold out `Class` entirely** — it is never used for clustering, only later for validation and context. The 14 features are bucketed and preprocessed, then clustered with **K-Means**, with **PCA** used purely to visualize the result in 2-D.

Pipeline: bucket features → one-hot encode categoricals → log-transform heavily right-skewed continuous features → `StandardScaler` → K-Means. Final feature matrix: **38 columns**. All randomness fixed with `random_state=42` for reproducibility.

## 2. Feature Choices and Trade-offs

| Bucket | Features | Treatment & rationale |
|---|---|---|
| Identifier | CustomerID | **Dropped** — no signal. |
| Label | Class | **Held out** of clustering; used only for validation/context (this is unsupervised). |
| Binary flags | A1, A8, A9, A11 | Kept as 0/1 — already one unit apart. |
| Categorical | A4, A5, A6, A12 | **One-hot encoded** — the integer codes are unordered labels, so leaving them numeric would falsely imply "category 8 > category 1." |
| Continuous | A2, A3, A7, A10, A13, A14 | **Scaled** so no single feature (e.g., A14, which reaches 100,001) dominates distance. |

**A10 — continuous, not categorical.** A10 is an **ordered count** (more genuinely means more), so we treat it as a continuous quantity and scale it rather than creating dozens of meaningless one-hot columns. We also observed A10's distribution is **heavily right-skewed**, an observation that — together with A14 — informed our log-transform decision below.

## 3. Key Decisions

**Chose k=4 for the product, while reporting k=2 as the strongest split.** *"We let the data lead. Silhouette clearly favored k=2 — the dominant structure in credit applications is a clean two-way split, which our validation check shows aligns with historical approve/deny patterns. But a self-discovery tool that only says 'you're Type A or B' isn't useful to a real applicant, so for the product we deliberately chose k=4 to give users richer, more navigable groups — trading a bit of statistical separation for human usefulness. We made that trade-off knowingly."*

**Log-transformed skewed continuous features to fix an outlier-driven cluster.** *"At k=4 our first attempt produced a one-person cluster. We traced it to extreme right-skew in A14 (skewness ≈ 13), applied a log transform (`log1p`), and re-ran selection. On the corrected data, k=4 yields four well-sized groups (68–364 applicants) and a higher silhouette than k=3 — so we use four clusters for the product, while reporting that the single strongest split in the data is k=2."*

**Treated 98% serving/training agreement as a feature, not a bug.** 676 of 690 applicants get the identical cluster whether processed one-at-a-time (the app's path) or all-at-once (training). The 14 that differ sit right on a boundary between two clusters — genuinely "between groups." For a self-discovery tool that's honest and correct; it simply means a borderline user could reasonably belong to either neighbor.

**Shipped the paywall as a disabled demonstration.** We include a paywall interstitial as a realistic monetization surface, but it is **off by default** ("Continue for free" always works). Charging applicants to see information about themselves raises a genuine fairness concern, so we surface it openly and acknowledge the tension rather than hide it.

**Separated logic from UI; recommendations are actionable-only.** All model logic lives in `backend.py`; `app.py` is presentation only. The "next move" recommendation engine uses the data to find which *changeable* features separate a user from the next segment (employment, tenure, income) and never suggests immutable traits (age, gender, marital status, citizenship).

## 4. Validation

- **Class alignment (primary insight).** The model never saw `Class`, yet the four segments separate cleanly by historical approval rate — **~20% / ~40% / ~80% / ~90%** — strong evidence the clusters capture something real.
- **Silhouette.** k=2 = 0.186 (strongest); k=4 = 0.158 (chosen, all groups well-sized: 364/176/82/68); k=3 = 0.148.
- **Reproducibility.** Fixed seeds; 98% one-at-a-time vs batch agreement (above).

## 5. Limitations

- Features are anonymized; friendly interpretations are inferred, not confirmed (see §7). One flag (A9) behaves opposite to a "prior default" reading, which we flag rather than trust.
- The 2-D PCA map captures ~39% of total variance (25.6% + 13.1%) — a simplified view; clusters that overlap in 2-D may separate in full space.
- `Class` reflects past human lending decisions and may carry historical bias; we use it only for validation/context, never as a target.
- Small dataset (689 rows); segments are descriptive, not predictive.

## 6. Responsible AI

See `RESPONSIBLE_AI.md` (≤200-word statement). In brief: never a prediction or score; limit figures are general guidance, not offers; label used only for validation; recommendations limited to legitimate, changeable behaviors; no data stored.

## 7. Column Name Assignments

Our inferred, plain-language interpretations of A1–A14, based on distribution shape, approval-rate correlation, and domain knowledge of credit applications (full working in `notebooks/Variable_Justification.ipynb`). These are **evidence-based hypotheses, not confirmed definitions** — the dataset is anonymized and we have no ground truth to check against.

| Code | Inferred name | Confidence | Rationale |
|---|---|---|---|
| A1 | Gender (0=Male, 1=Female) | Medium | Binary, 2 values. Approval-rate gap is essentially zero (−0.013), consistent with a protected characteristic that shouldn't legitimately drive a lending decision. |
| A2 | Age | High | Continuous, 350 unique values, range 13.75–80.25; decimal precision fits self-reported age. Approved applicants average ~3.85 years older, consistent with longer credit histories being viewed favorably. |
| A3 | Years Employed | Medium | Continuous, 0–28, decimal precision suggests tenure. Approved applicants average 2.07 more years employed. *Caveat:* range overlaps closely with A7 below, so a swap between the two is plausible — we kept the pairing that produced the cleaner "employment vs. residence" split. |
| A4 | Marital Status (1=Single, 2=Married, 3=Divorced/Widowed) | Medium | Exactly 3 categories, weak approval gap (+0.168), matches a field commonly collected on credit applications. |
| A5 | Creditworthiness Tier | Low (uncertain) | 14 categories with a strong ordinal-looking approval gradient, but we deliberately one-hot encoded it as *unordered* per our preprocessing spec — in tension with a "tier" reading. We surface this contradiction rather than resolve it with false confidence. |
| A6 | Occupation Type | Medium | 8 categories; one catch-all bucket dominates (408/690). Approval pattern is non-ordinal and messier than A5's, consistent with a self-reported nominal field like occupation. |
| A7 | Years at Current Address | Medium | Continuous, 0–28.5, decimal precision. Approved applicants average 2.17 more years at address. Same A3/A7 swap caveat noted above. |
| A8 | Currently Employed | High | Binary. Largest approval-rate gap of any column in the dataset (+0.717) — unemployed applicants are denied 93% of the time. The strongest, cleanest signal we found. |
| A9 | Prior Default | Medium | Binary, second-largest approval gap (+0.460). *Polarity note:* unlike our other binary flags (where 1 = "yes"), the data only makes sense if A9=0 means "has a prior default." We use that reading here; anyone reusing this column elsewhere should double check the polarity rather than assume 1=true. |
| A10 | Debt (Thousands) | Medium | Integer, 0–67, heavily zero-inflated (395/690 at 0). One high-debt outlier (67, i.e. ~$67k) was still approved — explained by cross-referencing high income (A14), consistent with a debt-to-income story. |
| A11 | Driver's License | Medium | Binary, smallest approval-rate gap of any column (+0.032) — essentially a non-financial identifier collected on the application, not a credit signal. We initially hypothesized this as a second default flag; the near-zero correlation ruled that out. |
| A12 | Citizenship Status (1=Citizen, 2=Permanent Resident, 3=Non-Immigrant) | Medium | Exactly 3 categories, weak approval gap (+0.069), maps cleanly onto categories collected on U.S. financial applications. |
| A13 | Credit Balance | High | Continuous, 0–2,000, heavily right-skewed (justifying the log-transform), negatively correlated with approval (−$34.60 avg. gap) — consistent with an outstanding-balance figure, not a geographic code. **Cleanup note:** the saved model artifact previously carried a stale, less-scrutinized label of "Zip Code (uncertain)" for this column, left over from an earlier iteration and never reconciled with this analysis. We traced the inconsistency and corrected it — see `CHANGELOG.md`. |
| A14 | Annual Income | High | 240 unique values, range $1–$100,001 — the largest approval gap of any column (+$1,840 avg.). The $100,001 ceiling is a well-known artifact of this dataset family and rules out competing reads like credit score (tops out far lower) or income-in-thousands (implausibly large at that ceiling). |
