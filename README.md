# M4U3 PPE Detection with YOLOv8

YOLOv8 object detection model for preliminary detection of valid personal protective equipment in workplace images.

> **Safety disclaimer:** This model is an assistive tool for preliminary screening only. It produces false negatives and false positives. It must not be used as the sole verifier for life-safety decisions.

## Project Overview

This academic AECO computer vision project detects two PPE classes:

- `head_protection`
- `high_visibility_clothing`

The objective is to demonstrate a reproducible cloud-based workflow that another person can run in Google Colab without local installation, personal credentials, or access to the author's computer.


## PPE Classes

### `head_protection`

Valid examples:

- Dark-blue protective work cap.
- Valid construction safety helmet, including the white helmet.

Ordinary non-protective caps are not labelled.

### `high_visibility_clothing`

Valid examples:

- Yellow safety vest with reflective stripes.
- Orange safety vest with reflective stripes.
- Yellow-and-grey fleece with reflective stripes.

Project-specific rule:

- Open high-visibility vests are treated as invalid use and are not labelled.

This is a deliberate project decision for the academic exercise. It does not establish a general workplace safety rule.

Detailed annotation rules are available in docs/class_definitions.md.


## Dataset

- **Source:** Workplace photographs collected specifically for this academic project.
- **Roboflow project version:** 3
- **Total images:** 134
- **Training images:** 107, approximately 80%
- **Validation images:** 27, approximately 20%
- **Test split:** 0 images
- **External test images:** 5 images kept outside training and validation
- **Preprocessing:** Auto-orient and fit within 640 × 640
- **Roboflow augmentations:** Disabled
- **Dataset license:** CC BY 4.0

### Frozen Public Dataset

The exact dataset used by the reproducible notebook is published as a keyless GitHub Release asset:

[Download m4u3-ppe-v3-yolo11.zip](https://github.com/arqmanu/M4U3_ppe-detection-yolov8/releases/download/v1.0/m4u3-ppe-v3-yolo11.zip)

**Package SHA-256:**

```text
8bea1dbc79b0493f1ece9394428ad44ab9c38ca5b4341fa078814313693b5d52
```

## Model Configuration

- **Model:** YOLOv8n
- **Ultralytics version:** 8.2.103
- **Epochs:** 30
- **Image size:** 640
- **Batch size:** 16
- **Pretrained weights:** `yolov8n.pt`
- **Optimizer:** Auto, resolved to AdamW during training
- **Random seed:** 0
- **Training environment:** Google Colab
- **GPU used:** Tesla T4

Naming note: The published package filename includes yolo11, but the included model was trained from yolov8n.pt. The filename is retained to preserve the existing release URL and package SHA-256.

## Results

### Overall Validation Metrics

| Metric | Result |
|---|---:|
| Precision | 0.833 |
| Recall | 0.897 |
| mAP50 | 0.923 |
| mAP50-95 | 0.717 |

### Results by Class

| Class | Precision | Recall | mAP50 | mAP50-95 |
|---|---:|---:|---:|---:|
| `head_protection` | 0.920 | 0.860 | 0.898 | 0.646 |
| `high_visibility_clothing` | 0.746 | 0.933 | 0.947 | 0.789 |

### Key Findings

- High-visibility clothing achieved the highest recall and mAP.
- Head protection was more difficult to detect at longer distances.
- Four of the five external test images produced correct detections.
- The complex multi-person image `IMG_3446.jpg` produced missed detections and one incorrect clothing detection.
- The validation set contains only 27 images, so these metrics do not demonstrate production readiness.

Training curves, confusion matrices and prediction examples are available in the results/ folder.


## Reproducibility

The notebook uses the frozen GitHub Release dataset as its primary data source.

The workflow does not require:

- A Roboflow account.
- A Roboflow API key.
- Google Colab Secrets.
- Local file paths.
- Manual dataset uploads.
- Access to the author's computer.

### Reproducibility Checklist

- [x] Public GitHub repository
- [x] Frozen dataset version
- [x] Public dataset URL without credentials
- [x] SHA256 checksum verification
- [x] Documented 80/20 split
- [x] Fixed model variant: YOLOv8n
- [x] Documented epochs, batch size and image size
- [x] Pinned Ultralytics version
- [x] Fixed external test images
- [x] Training outputs and prediction evidence
- [x] Public model weights
- [ ] Final fresh-runtime `Run all` test

### Reproducibility Proof

- **Last successful training run:** 24 September 2026
- **Hardware:** Google Colab with Tesla T4 GPU
- **Recorded training time:** Approximately 2 minutes for 30 epochs
- **Expected GPU runtime:** Approximately 2 to 10 minutes, depending on Colab availability
- **CPU fallback:** Possible but significantly slower

The final fresh-runtime `Run all` test is pending because Google Colab temporarily reported that the free GPU usage limit had been reached.


## Model Weights

The trained `best.pt` weights are available in the public GitHub Release:

- [Download `best.pt` and view Release v1.0](https://github.com/arqmanu/M4U3_ppe-detection-yolov8/releases/tag/v1.0)

The weights allow inference without repeating the complete training process.

## Evidence

The repository provides visible evidence so that results can be reviewed without executing the notebook.

### Training Results

Training outputs and metrics:

https://github.com/arqmanu/M4U3_ppe-detection-yolov8/tree/main/results/training

Training curves:

https://github.com/arqmanu/M4U3_ppe-detection-yolov8/blob/main/results/training/results.png

Confusion matrix:

https://github.com/arqmanu/M4U3_ppe-detection-yolov8/blob/main/results/training/confusion_matrix.png

Validation ground-truth montage:

https://github.com/arqmanu/M4U3_ppe-detection-yolov8/blob/main/results/training/val_batch0_labels.jpg

Validation prediction montage:

https://github.com/arqmanu/M4U3_ppe-detection-yolov8/blob/main/results/training/val_batch0_pred.jpg

### External Image Evidence

The following five images were kept outside the training and validation sets.

External test images:

https://github.com/arqmanu/M4U3_ppe-detection-yolov8/tree/main/results/03_new_test_images

Predictions on external test images:

https://github.com/arqmanu/M4U3_ppe-detection-yolov8/tree/main/results/evidence/new_images_predictions

### Supporting Documentation

Class definitions and annotation rules:

https://github.com/arqmanu/M4U3_ppe-detection-yolov8/blob/main/docs/class_definitions.md

Error analysis:

https://github.com/arqmanu/M4U3_ppe-detection-yolov8/blob/main/docs/error_analysis.md

Governance checklist:

https://github.com/arqmanu/M4U3_ppe-detection-yolov8/blob/main/docs/governance_checklist.md

## Error Analysis Summary

Four of the five external test images produced correct results:

- `IMG_3431.jpg`
- `IMG_3443.jpg`
- `IMG_3450.jpg`
- `IMG_3502.jpg`

The main external failure occurred in `IMG_3446.jpg`, a more complex scene containing several people at different distances:

- Two missed `head_protection` detections.
- One missed `high_visibility_clothing` detection.
- One incorrect `high_visibility_clothing` detection.

Two additional unmatched predictions were identified by comparing the validation prediction montage with the corresponding ground-truth montage.

The analysis indicates that the most difficult conditions were:

- Crowded scenes.
- Small or distant PPE.
- Partial visibility.
- Overlapping people.
- Colours or shapes similar to valid PPE.

The prioritized improvement plan includes collecting more crowded scenes, adding hard negative examples, and increasing the representation of underrepresented PPE types.

See the [complete error analysis](docs/error_analysis.md) for detailedses and proposed improvements.


