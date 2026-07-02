# Why I revisited this

*(Draft — this is written in your voice as a starting point. Edit freely; swap in whatever actually feels true to you.)*

Credit Cat started as a CSULB datathon submission with Emmet Gingerich — not our first time pairing up; we'd already written a Rubik's Cube group theory paper together, so when the "Unsupervised ML" pillar came up, teaming up again was easy. He drove most of the model build, the three UI iterations, and the branding/recommendation-engine work. My side was the dataset cleanup, the full column-by-column variable justification (`notebooks/Variable_Justification.ipynb`), and the cluster naming and interpretation that ended up in `Clustering_Story.ipynb`, plus a pass on `app.py` near the deadline.

The gap between "rushed" and "finished" is smaller than it looks. I'd already worked through the reasoning for all 14 anonymized columns — confidence levels, cross-checked against approval rates, caveats and all — but it never made it out of the notebook and into `MODEL_REPORT.md` before submission. Section 7 just said "intentionally left blank." That's a pretty honest artifact of what a datathon deadline actually does to a project: the thinking was there, the write-up wasn't.

Coming back to it, the fixes weren't glamorous — porting the analysis into the report, deduplicating some copy-pasted notebook cells, tracing down a labeling bug where a leftover field called A13 "Zip Code (uncertain)" when the data (and our own earlier analysis) clearly said "Credit Balance." None of that changes the model. It's just the difference between work that's *done* and work that's *legible* — and I care about that gap more than I used to.

What I'd still want to do if I keep extending this: resolve the A5 "Creditworthiness Tier" ambiguity properly (right now it's honestly flagged as uncertain, not solved), and maybe test whether the k=2 vs. k=4 tradeoff holds up with a different clustering algorithm.

— Prithvi
