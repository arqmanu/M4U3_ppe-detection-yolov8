# Iteration 2: From PPE Detection to Compliance Detection

## 1. Why a second iteration

Iteration 1 detects **valid PPE**, but *"no helmet detected"* does not prove that a worker is **not** wearing one. The model may simply have missed it (see the false negatives in [`error_analysis.md`](error_analysis.md)). A compliance use case needs the **violation itself** as a class. The course feedback recommended explicit negative classes over geometric post-processing (person box + helmet box + overlap rules), because negative classes are more direct and less fragile.

## 2. Class design (project decision)

| Class | Box on | Rule |
|---|---|---|
| `head_protection` | head | Unchanged from iteration 1 |
| `no_head_protection` | head | Head visible and **not** wearing valid head protection (bare head or ordinary cap) |
| `high_visibility_clothing` | torso | Unchanged: **closed** valid garment |
| `no_high_visibility_clothing` | torso | Torso visible and **no** valid garment: ordinary clothing, red company T-shirt, **open** vest or jacket |

- Every visible person gets **one head label and one torso label**.
- Persons that are too distant, cut off or occluded to decide are **not** labelled.
- The course feedback suggested boxes around the whole person (`person_with_helmet`, …). Head and torso boxes were chosen instead, for two reasons:
  1. All 308 boxes of iteration 1 remain valid, so only the new cases had to be labelled (about 1 hour instead of a full relabel).
  2. The box stays on the body part that carries the evidence.
- The open-vest rule of iteration 1 now becomes a real class: an open vest is a **violation** (`no_high_visibility_clothing`), not just "no label".

## 3. Labelling workflow and SAM 3 exploration

- New boxes were drawn manually in the Roboflow editor, following the rules above.
- **SAM 3** (Roboflow *Find Objects with AI*, model *SAM 3 (Masks)*) was tested first. See [`sam_exploration.md`](sam_exploration.md). SAM 3 could segment people well, but it cannot express *"without a vest"*: it also selected the worker who **was** wearing a vest. It was therefore not used for the final labels.
- A consistency check after export (one head label and one torso label per person) found 6 images with missing or wrong torso labels. All of them were **open garments**. They were fixed before training (Roboflow version 5). One known gap remains: `IMG_3475` has no torso label.

## 4. Dataset: Roboflow version 5

| Item | Value |
|---|---|
| Roboflow | [m4u3-ppe-detection, **version 5**](https://universe.roboflow.com/arq-manuelgo-gmail-com/m4u3-ppe-detection/dataset/5) (`ppe-detection-v5-compliance`) |
| Images and split | Same 134 images and same 107 / 27 split as iteration 1 |
| Frozen copy | [`m4u3-ppe-v5-yolo11.zip`](https://github.com/arqmanu/M4U3_ppe-detection-yolov8/releases/download/v2.0/m4u3-ppe-v5-yolo11.zip) (Release v2.0) |
| SHA-256 | `e700df6a36a4deb8fe6ee831c68f3de07c359696dbff0c5bbcc99458205e95b3` |

| Split | `head_protection` | `high_visibility_clothing` | `no_head_protection` | `no_high_visibility_clothing` |
|---|---:|---:|---:|---:|
| Train (107) | 137 | 101 | 20 | 48 |
| Validation (27) | 40 | 29 | **3** | 16 |

**Class imbalance:** most people at this workplace wear their PPE, so violations are rare. Only 23 `no_head_protection` examples exist in total, and 3 of them are in validation. **Metrics for that class are not statistically meaningful.**

## 5. Training

The configuration is identical to iteration 1: YOLOv8n from `yolov8n.pt`, 30 epochs, imgsz 640, batch 16, seed 0, Ultralytics 8.2.103. It was trained on 25 Sep 2026 on a Google Colab **CPU**, because the free GPU quota was exhausted. Training took 30.1 min.

- Notebook: [`notebooks/02_Iteration2_Compliance.ipynb`](../notebooks/02_Iteration2_Compliance.ipynb)
- Weights: [`best.pt` (Release v2.0)](https://github.com/arqmanu/M4U3_ppe-detection-yolov8/releases/download/v2.0/best.pt), SHA-256 `dc70d91ca09e974c4ff3c62d1796f1929d077cc310c1b29ba12080112d37881f`
- Curves and confusion matrix: [`results/iteration2/training/`](../results/iteration2/training/)

## 6. Results (validation, 27 images)

| Class | Precision | Recall | mAP50 | mAP50-95 |
|---|---:|---:|---:|---:|
| **all** | **0.809** | **0.782** | **0.905** | **0.650** |
| `head_protection` | 0.934 | 0.713 | 0.851 | 0.610 |
| `high_visibility_clothing` | 0.589 | 0.897 | 0.862 | 0.761 |
| `no_head_protection` (3 examples) | 1.000 | 0.645 | 0.995 | 0.642 |
| `no_high_visibility_clothing` | 0.711 | 0.875 | 0.912 | 0.588 |

**Comparison with iteration 1**, on the two classes the iterations share:

| Class | Recall (it. 1 → it. 2) | mAP50 (it. 1 → it. 2) |
|---|---|---|
| `head_protection` | 0.860 → **0.713** | 0.898 → 0.851 |
| `high_visibility_clothing` | 0.933 → 0.897 | 0.947 → 0.862 |

With the same small dataset, adding two rare classes cost some performance on the original classes. The model now has to split its capacity, and part of the "vest" appearance moved to "no vest" when the vest is open.

## 7. New images: iteration 1 vs iteration 2

Side-by-side images: [`results/iteration2/evidence/iteration1_vs_iteration2/`](../results/iteration2/evidence/iteration1_vs_iteration2/).

| Image | Iteration 1 | Iteration 2 | Verdict |
|---|---|---|---|
| `IMG_3431`: forklift driver, red T-shirt, cap | Cap only | Cap + **`no_high_visibility_clothing` 0.66** | ✅ The violation is now flagged, which iteration 1 could not express |
| `IMG_3443`: worker with cap and vest | Cap + vest | Cap + vest | ✅ Same |
| `IMG_3450`: cap and vest (mirror) | Cap + vest | Vest only (cap missed) | ❌ Regression on `head_protection` |
| `NEW_open_vest`: **open** vest, bare head | **Vest 0.52** (wrong: open vest accepted as valid) | Nothing | ⚠️ Half right: it no longer accepts the open vest, but it does not flag `no_high_visibility_clothing` / `no_head_protection` |
| `IMG_3446`: crowded, medium distance | Several overlapping boxes | 1 low-confidence box | ❌ Both iterations fail in crowded scenes; iteration 2 fails more |

## 8. Error analysis (validation)

Automatic matching (IoU ≥ 0.5, confidence ≥ 0.25) found 56 TP, 5 FP candidates and 32 FN candidates. The iteration is much more **conservative**: fewer false alarms, many more misses. Evidence: [`results/iteration2/evidence/error_examples/`](../results/iteration2/evidence/error_examples/).

**False positives**
1. **`IMG_3471`: open vest detected as valid `high_visibility_clothing` (0.90).** Open vests are only a small fraction of the 48 training examples of `no_high_visibility_clothing` (many of them are red T-shirts). That is not enough to override the strong "yellow + stripes" pattern.
2. **`IMG_3419`: one vest box covering two neighbouring workers (0.91).** A localisation error: the two people are standing shoulder to shoulder.
3. **`IMG_3474`: open red visitor vest detected as valid (0.31).** Same cause as FP1, with a rare colour.

**False negatives**
1. **`IMG_3495`, `IMG_3492`, `IMG_3513`: `no_high_visibility_clothing` missed.** Red T-shirts in unusual poses and an open vest seen from the side. The negative class is still under-represented.
2. **`IMG_3462`: `no_head_protection` missed on a bare head.** Only 20 training examples.
3. **`IMG_3419`, `IMG_3444`, `IMG_3493`: `head_protection` missed.** This includes the **white helmet**, which appears in only one image, and small distant heads. Recall for this class dropped from 0.86 to 0.71.

## 9. Conclusions and next steps

- **Iteration 2 answers the right question.** It can flag a worker without a vest (`IMG_3431`), which iteration 1 cannot do by design.
- **It is not reliable yet.** With 23 `no_head_protection` and 64 `no_high_visibility_clothing` examples, the violation classes are too rare. The shared classes also lost recall. **Iteration 1 remains the better PPE detector. Iteration 2 is a proof of concept for compliance detection.**
- **Next data improvements, prioritised:**
  1. Stage 30–50 photos of **deliberate violations** (bare heads, open vests, missing vests) with consenting colleagues. This is active curation, as the course feedback recommends, instead of waiting for violations to appear.
  2. Add more **open-vest** examples from several angles.
  3. Add more **white-helmet and distant-worker** examples.
- For compliance use, the confidence threshold should favour recall on the `no_*` classes. A missed violation is the costly error.

**This model is an assistive tool for preliminary screening only. It produces false negatives and false positives. It must not be used as the sole verifier for life-safety decisions.**
