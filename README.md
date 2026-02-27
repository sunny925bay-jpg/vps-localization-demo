# 🗺️ Visual Positioning System — Lightweight Localization Demo

> A complete VPS pipeline implementing feature-based image localization with geometric verification.  
> Built as a portfolio piece for the **Niantic Spatial Applied CV Internship** (Summer 2026).

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/sunny925bay-jpg/vps-localization-demo/blob/main/visual_positioning_demo.ipynb)

---

## What This Is

This notebook implements the core stages of a **Visual Positioning System (VPS)** — the same kind of pipeline used in production localization systems (Niantic VPS, Google Street View, ARCore). Given a query image and a small map database, the system estimates which map location the query was captured from and how confident that estimate is.

```
Map Images ──► Feature Extraction ──► Feature Database
                                              │
Query Image ──► Feature Extraction ──► Descriptor Matching (+ Ratio Test)
                                              │
                                       Geometric Verification (RANSAC)
                                              │
                                       Pose Estimate + Confidence Score
```

## Pipeline Stages

| Stage | Method | Why |
|-------|--------|-----|
| Feature detection | ORB (1000 kps, 8 octaves) | Fast, patent-free, binary descriptors |
| Descriptor matching | BFMatcher + Hamming distance | Correct norm for binary descriptors |
| Outlier rejection | Lowe's ratio test (thresh=0.75) | Removes ~70% false matches |
| Geometric verification | RANSAC homography (5px threshold) | Robust to remaining outliers |
| Confidence scoring | Inlier ratio + absolute inlier count | `HIGH / MEDIUM / LOW / FAILED` |

## Results

- **Inlier ratio:** ~65–80% on synthetic viewpoint changes (rotation ±15°, scale ±10%)
- **Retrieval accuracy:** Correct top-1 map view in all tested scenarios
- **Ablation:** Ratio threshold 0.70–0.75 optimal — maximizes inlier ratio while retaining enough matches

## Quickstart

```bash
git clone https://github.com/sunnyanand/vps-localization-demo
cd vps-localization-demo
pip install -r requirements.txt
jupyter notebook visual_positioning_demo.ipynb
```

Or just click the **Open in Colab** badge above — no setup needed.

## Requirements

```
opencv-python-headless>=4.8
numpy>=1.24
matplotlib>=3.7
scikit-learn>=1.3
```

## Limitations & What I'd Build Next

**Current limitations:**
- ORB fails under large viewpoint changes (>30°) or illumination shifts
- 2D homography assumes planar scene — breaks for real 3D environments
- Exhaustive matching is O(N) in map size — impractical for large maps

**Next steps toward a production VPS:**
1. Replace ORB with **SuperPoint** (learned, more robust keypoints)
2. Use **COLMAP** to build a real 3D point cloud map from multi-view images
3. Switch from homography to **PnP + RANSAC** for full 6-DoF pose estimation
4. Index map descriptors with **FAISS** for sub-linear retrieval at scale
5. Deploy inference as a **Go HTTP microservice** with centimeter-level accuracy targets

---

*Sunny Anand · UCLA Computer Science · [sunnyincalif@g.ucla.edu](mailto:sunnyincalif@g.ucla.edu)*
