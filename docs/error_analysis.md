# Error Analysis

## 1. Evaluation Scope

The model was evaluated using two sources:

1. The validation split from the frozen Roboflow dataset.
2. Five new images that were kept outside the training and validation sets.

The five external images were:

- `IMG_3431.jpg`
- `IMG_3443.jpg`
- `IMG_3446.jpg`
- `IMG_3450.jpg`
- `IMG_3502.jpg`

Four external images produced correct results:

- `IMG_3431.jpg`
- `IMG_3443.jpg`
- `IMG_3450.jpg`
- `IMG_3502.jpg`

The main external failure case was `IMG_3446.jpg`, a more complex scene containing five people at different positions and distances.

## 2. False Positives

### False Positive 1: Incorrect high-visibility clothing detection in `IMG_3446.jpg`

The model detected one instance of `high_visibility_clothing` where no valid high-visibility garment was present.

**Possible cause:** The scene contains several people, partially visible garments, overlapping objects, and visually similar colours. These conditions may have caused the model to associate part of the scene with the colour or shape of valid high-visibility clothing.

### False Positive 2: Unmatched detection in the validation predictions

A bounding box appeared in `val_batch0_pred.jpg` without a corresponding ground-truth box in `val_batch0_labels.jpg`.

**Possible cause:** The object may have shared visual characteristics with one of the PPE classes, such as colour, shape, reflective-looking areas, or partial visibility. The validation montage is too compressed to determine the exact object confidently.

### False Positive 3: Second unmatched detection in the validation predictions

A second predicted bounding box appeared in `val_batch0_pred.jpg` without a corresponding ground-truth annotation in `val_batch0_labels.jpg`.

**Possible cause:** The limited dataset size and the presence of small or distant objects may have encouraged the model to generalize from incomplete visual features. A larger individual-image inspection would be required to identify the precise visual trigger.

## 3. False Negatives

All three documented false negatives occurred in `IMG_3446.jpg`.

### False Negative 1: First missed head protection

One valid item of head protection was visible but not detected.

**Possible cause:** The PPE appeared relatively small within a scene containing several people. Distance and limited pixel detail may have reduced detection confidence.

### False Negative 2: Second missed head protection

A second valid item of head protection was visible but not detected.

**Possible cause:** Partial visibility, overlap with other people, viewing angle, or similarity between the dark-blue protective cap and the surrounding background may have made the object difficult to distinguish.

### False Negative 3: Missed high-visibility clothing

One valid high-visibility garment was visible but not detected.

**Possible cause:** The garment was relatively distant or partially visible. The model may not have received enough similar multi-person and long-distance examples during training.

## 4. Key Findings

- The model performed correctly on four of the five external test images.
- The main difficulties appeared in a crowded scene containing several people and relatively small PPE objects.
- Head protection was more difficult to detect reliably at a distance.
- High-visibility clothing generally performed well, but one valid garment was missed and one incorrect garment detection occurred.
- The validation comparison also revealed two unmatched predictions.
- The validation set contains only 27 images, so the reported metrics should not be interpreted as proof of production readiness.

## 5. Prioritized Data Improvements

### Priority 1: Add more crowded and distant scenes

Collect and annotate more images containing:

- Four or more people.
- People at different distances.
- Partially overlapping people.
- Small PPE objects.
- Different camera angles.

This directly targets the failures observed in `IMG_3446.jpg`.

### Priority 2: Add hard negative examples

Add more images containing visually similar but invalid objects, including:

- Ordinary coloured clothing.
- Yellow or orange objects without reflective stripes.
- Open high-visibility vests.
- Ordinary caps.
- Background objects with colours or shapes similar to PPE.

These images should remain unlabelled so that the model learns not to classify them as valid PPE.

### Priority 3: Improve class balance and PPE variety

Collect additional examples of underrepresented valid PPE, especially:

- White construction helmets.
- Dark-blue protective caps at longer distances.
- Orange high-visibility clothing.
- Yellow-and-grey reflective fleece garments.
- Valid garments viewed from the side and back.

The white construction helmet is represented by only one image, so the current model cannot be assumed to generalize reliably to that PPE type.

## 6. Iteration Plan

A future dataset version should:

1. Preserve the current dataset as a frozen baseline.
2. Add targeted images based on the documented failures.
3. Maintain consistent annotation rules.
4. Keep external test images separate from training and validation.
5. Train a new model using the same principal parameters.
6. Compare the new model against the current baseline using the same evaluation images.
7. Report whether false negatives decrease without creating an unacceptable increase in false positives.

## 7. Safety Interpretation

For this PPE-screening use case, false negatives are considered more consequential than false positives because missed PPE may produce misleading safety information.

However, the model must not make safety or disciplinary decisions automatically.

**This model is an assistive tool for preliminary screening only. It produces false negatives and false positives. It must not be used as the sole verifier for life-safety decisions.**
