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

**SHA256:**

```text
8bea1dbc79b0493f1ece9394428ad44ab9c38ca5b4341fa078814313693b5d52
