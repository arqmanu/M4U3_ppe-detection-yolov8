# M4U3 PPE Detection with YOLOv8

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/arqmanu/M4U3_ppe-detection-yolov8/blob/main/notebooks/01_Training_Evaluation.ipynb)

A YOLOv8 object-detection model that pre-screens workplace photographs for **valid personal protective equipment (PPE)**. It is an academic AECO project (Master's programme, Module 4 Unit 3: Computer Vision).

> **Safety disclaimer:** This model is an assistive tool for preliminary screening only. It produces false negatives and false positives. It must not be used as the sole verifier for life-safety decisions.

| What is this? | Does it work? | How do I run it? |
|---|---|---|
| A 2-class PPE detector (head protection and high-visibility clothing) trained on 134 photos from a real workplace | **mAP50 0.92, precision 0.83, recall 0.90** on the validation split. Main weaknesses: small, distant and crowded workers | Click the Colab badge → `Runtime → Run all`. No account, key or upload is needed ([details](#how-to-reproduce)) |

---

## 1. Problem and success criteria

**AECO problem.** Safety staff on industrial and construction sites check PPE by eye. Checking every worker in every photo is slow and inconsistent. A detector that highlights *valid* PPE can pre-screen photos, so the human reviewer can focus on the doubtful cases.

**Success criteria** *(project decision, set before evaluation)*:

| Criterion | Target | Result | Met? |
|---|---|---|---|
| Recall for each class on the validation split (missed PPE is the costly error) | ≥ 0.85 | 0.86 (head) / 0.93 (clothing) | ✅ |
| mAP50, all classes | ≥ 0.80 | 0.92 | ✅ |
| Correct detections on single workers in new, unseen photos | Qualitative | Yes (confidence 0.89–0.98) | ✅ |
| Reliable on crowded or distant scenes | Qualitative | **No** (see [error analysis](docs/error_analysis.md)) | ❌ |

## 2. Classes and label rules

| Class | Label (valid PPE) | Do **not** label |
|---|---|---|
| `head_protection` | Company dark-blue protective cap; construction safety helmet (including white) | Ordinary caps |
| `high_visibility_clothing` | **Closed** yellow or orange vest with reflective stripes; yellow-grey reflective fleece; red visitor vest with reflective stripes* | **Open vests** (project rule), red company T-shirts, clothing without reflective stripes |

*Added after the error analysis (see the change log in the class definitions).

The open-vest rule is a deliberate academic decision. It makes the model distinguish between *wearing a garment* and *wearing it correctly*. It is not a general legal rule. Full contract: [`docs/class_definitions.md`](docs/class_definitions.md).

## 3. Dataset

| Item | Value |
|---|---|
| Source | Workplace photographs taken by the author for this project |
| Annotation | Roboflow **Astra Auto Label**, then full manual review and correction |
| Roboflow project | [m4u3-ppe-detection, **version 3**](https://universe.roboflow.com/arq-manuelgo-gmail-com/m4u3-ppe-detection/dataset/3) (version note: *"corrected annotation rules; open safety vests are left unlabelled"*) |
| Split | **107 train (80%) / 27 validation (20%)**, no test split |
| Labelled objects | Train: 137 `head_protection`, 101 `high_visibility_clothing`. Validation: 40 and 30 |
| Preprocessing | Auto-orient; resize to fit within 640 × 640 |
| Augmentation | None |
| New test images | 5 photos never uploaded to Roboflow: [`results/03_new_test_images/`](results/03_new_test_images/) |
| License | CC BY 4.0 (see [Licensing](#9-licensing-and-data-rights)) |

**Frozen, keyless copy.** The notebook does not download from Roboflow. It uses the exact export of version 3, published as a GitHub Release asset and verified with SHA-256:

- File: [`m4u3-ppe-v3-yolo11.zip`](https://github.com/arqmanu/M4U3_ppe-detection-yolov8/releases/download/v1.0/m4u3-ppe-v3-yolo11.zip)
- SHA-256: `8bea1dbc79b0493f1ece9394428ad44ab9c38ca5b4341fa078814313693b5d52`

*Naming note:* `yolo11` in the file name refers to Roboflow's **export format** (YOLOv11 label format, which is compatible with YOLOv8). The model trained in this project is **YOLOv8n**. The naming is summarised below:

| Item | Value |
|---|---|
| Roboflow dataset version | 3 |
| Release asset name | `m4u3-ppe-v3-yolo11.zip` |
| Repository release tag | `v1.0` |

## 4. Model and training

| Parameter | Value |
|---|---|
| Model | YOLOv8n, fine-tuned from COCO-pretrained `yolov8n.pt` |
| Epochs / image size / batch | 30 / 640 / 16 |
| Optimizer / seed | Auto (resolved to AdamW) / 0 (deterministic) |
| Framework | Ultralytics 8.2.103 |
| Original training | 24 Sep 2026, Google Colab, Tesla T4 GPU, about 2 min |

**Trained weights:** [`best.pt`](https://github.com/arqmanu/M4U3_ppe-detection-yolov8/releases/download/v1.0/best.pt) from [Release v1.0](https://github.com/arqmanu/M4U3_ppe-detection-yolov8/releases/tag/v1.0). SHA-256: `e72ead7d46303af3798b12a28af7d6dde4e222588761fbc63d7d6cd668ee6bc2`.

**Baseline.** The generic COCO model (`yolov8n.pt`, no custom training) was run on the same new images. It finds `person`, but it has **no PPE classes**, and it also produced unrelated detections ("clock", "sink"). A generic detector can locate workers, but it cannot tell whether they wear valid PPE. A custom dataset and model are needed for that.

## 5. Results

Validation split (27 images, 70 labelled objects), original training run:

| Class | Precision | Recall | mAP50 | mAP50-95 |
|---|---:|---:|---:|---:|
| **all** | **0.833** | **0.897** | **0.923** | **0.717** |
| `head_protection` | 0.920 | 0.860 | 0.898 | 0.646 |
| `high_visibility_clothing` | 0.746 | 0.933 | 0.947 | 0.789 |

**Key takeaways**

1. **Clothing is found reliably (recall 0.93) but over-detected (precision 0.75).** Floor hazard stripes and red shirts are mistaken for vests.
2. **Head protection is precise (0.92) but missed more often (recall 0.86)**, especially small, distant or top-down caps. This is the safety-relevant weakness.
3. **Label quality matters as much as the model.** The error review found two annotation inconsistencies where the model was right and the label was wrong. With only 27 validation images, every single error moves the metrics by several points.

**Evidence** (viewable without running anything):

| Evidence | Location |
|---|---|
| Training curves, confusion matrix, PR/F1 curves | [`results/training/`](results/training/) ([curves](results/training/results.png), [confusion matrix](results/training/confusion_matrix.png)) |
| Annotation examples (4) | [`results/evidence/annotations/`](results/evidence/annotations/) |
| Validation predictions (10) | [`results/evidence/validation_predictions/`](results/evidence/validation_predictions/) |
| New-image predictions (5) | [`results/evidence/new_images_predictions/`](results/evidence/new_images_predictions/) |
| Error examples (labels vs predictions) | [`results/evidence/error_examples/`](results/evidence/error_examples/) |
| Error analysis | [`docs/error_analysis.md`](docs/error_analysis.md) |

## 6. Error analysis (summary)

| Type | Examples | Probable cause |
|---|---|---|
| False positives | Floor hazard stripes, red T-shirt, shadow beside a head | Colour and stripe patterns without the context of a person; few hard negatives |
| False negatives | Distant heads (×5), cap seen from above | Few small or top-down examples at 640 px |
| Label issues | Open vest labelled; red visitor vest not labelled | Class contract was incomplete, so it was updated |

**Next data improvements (prioritised):**
1. Fix the label contract and relabel.
2. Add small, distant and crowded workers.
3. Add hard negatives (floor markings, red shirts, shadows).

Details: [`docs/error_analysis.md`](docs/error_analysis.md).

## 7. How to reproduce

1. Open [`notebooks/01_Training_Evaluation.ipynb`](notebooks/01_Training_Evaluation.ipynb) with the **Open in Colab** badge.
2. *(Optional)* To re-train, select `Runtime → Change runtime type → T4 GPU`.
3. Select `Runtime → Disconnect and delete runtime`, then `Runtime → Run all`. Choose *Connect without GPU* if Colab offers it.
4. Expected outputs:
   - Dataset checksum verified.
   - Class/split table (107/27).
   - Baseline detections.
   - Metrics table.
   - Curves and confusion matrix.
   - 10 validation predictions and 5 new-image predictions.
   - Error table.
   - Reproducibility summary.

   Everything is saved to `/content/outputs` (zipped as `M4U3_outputs.zip`).

| Mode | When | Expected runtime |
|---|---|---|
| **Full training run** | A GPU is available | About 2–10 min |
| **Verification run** (downloads `best.pt` from the Release, checks its SHA-256, then evaluates) | No GPU available | About 2–5 min on CPU |

No Roboflow account, API key, Colab Secrets, Google Drive or manual upload is required.

### Reproducibility checklist
- [x] Public repository and notebook that opens from GitHub in Colab
- [x] Dataset: Roboflow version 3, frozen as a GitHub Release asset with SHA-256 check
- [x] Split documented: 107 / 27 (80/20)
- [x] Model variant: YOLOv8n (`yolov8n.pt`)
- [x] Epochs 30, batch 16, imgsz 640, seed 0
- [x] Ultralytics pinned: `8.2.103` (installed with `--no-deps` to keep Colab's NumPy 2 working)
- [x] Weights published with SHA-256 (`best.pt`, Release v1.0)
- [x] Fresh-runtime `Run all` completed (see proof below)

### Reproducibility proof
Last successful fresh-runtime `Run all` from GitHub:

- **Run date (UTC):** 2026-09-25 10:29
- **Mode:** verification run (published weights loaded, no training). The Colab free-tier GPU quota was exhausted.
- **Hardware:** Google Colab CPU (x86_64)
- **Python / PyTorch / Ultralytics:** 3.13.15 / 2.11.0+cpu / 8.2.103
- **Total notebook runtime:** 1.6 min
- **Validation metrics:** P 0.835 · R 0.883 · mAP50 0.922 · mAP50-95 0.716

These values differ from the training-time values by at most 0.014. That is expected: a stand-alone validation batches images differently from the validation that runs at the end of training, and it runs on CPU instead of GPU.

## 8. Governance and limitations

- **Intended use:** preliminary screening of photos, always followed by human review.
- **Do not use** for automated enforcement or disciplinary decisions, for worker surveillance or identification, for crowded or distant scenes, or for PPE types that are not in the dataset (e.g. boots, glasses).
- **Risk:** false negatives (missed PPE) are the more consequential error.
- Full checklist, covering privacy, consent, data minimisation, limitations and risk: [`docs/governance_checklist.md`](docs/governance_checklist.md).

## 9. Licensing and data rights

- **Code and notebooks:** [MIT License](LICENSE).
- **Dataset images and labels:** CC BY 4.0, the same license as on Roboflow Universe. The photos were taken by the author at their workplace. The people shown gave written consent, and faces were obscured before publication.
- **Model weights:** released for academic reproducibility. Note that Ultralytics YOLOv8 itself is licensed under AGPL-3.0.

## 10. Repository structure

```text
├── README.md
├── LICENSE
├── notebooks/
│   └── 01_Training_Evaluation.ipynb   # baseline, training/verification, evaluation, error evidence
├── docs/
│   ├── class_definitions.md           # label contract + change log
│   ├── error_analysis.md              # 3 FP, 3 FN, label issues, prioritised improvements
│   └── governance_checklist.md        # privacy, minimisation, limitations, risk
├── results/
│   ├── training/                      # curves, confusion matrix, results.csv, args.yaml
│   ├── 03_new_test_images/            # 5 external images (inputs)
│   └── evidence/                      # annotations, validation and new-image predictions, error examples
└── reports/                           # slides and mini report (PDF)
```

*The notebook is adapted from the Roboflow "train YOLOv8 on a custom dataset" template. Its generic tutorial cells were removed.*
