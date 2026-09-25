# Error Analysis

## 1. Method

Errors were identified in two ways:

1. **Validation split (27 images).** Section 10 of the notebook compares every prediction (confidence ≥ 0.25) with the ground-truth labels. A prediction and a label match when they have the same class and a box overlap (IoU) of at least 0.5. Unmatched predictions are **false-positive candidates**, and unmatched labels are **false-negative candidates**. Every candidate was then reviewed by eye.
2. **New images kept outside Roboflow.** Predictions were reviewed visually.

Automatic matching on the validation split (verification run, 25 Sep 2026):

| True positives | FP candidates | FN candidates | Images with at least one error |
|---:|---:|---:|---:|
| 60 | 10 | 10 | 11 of 27 |

Not every candidate is a real model error. The visual review separated the candidates into four groups:

- **Real false positives.**
- **Real false negatives.**
- **Localisation errors:** the object is found, but the box is too loose or too tight to reach IoU 0.5. Examples are `IMG_3480` and `IMG_3499`. These count once as an FP and once as an FN.
- **Annotation inconsistencies:** the label is wrong and the model is right.

Side-by-side evidence images (left: labels, right: predictions, red = error) are in [`results/evidence/error_examples/`](../results/evidence/error_examples/). The full table is [`results/evidence/validation_error_table.csv`](../results/evidence/validation_error_table.csv).

## 2. False Positives

### FP1: Yellow and black floor markings detected as `high_visibility_clothing` (`IMG_3481`, confidence 0.57)
**What:** The model drew a clothing box over the yellow and black hazard stripes painted on the floor.
**Why (hypothesis):** Floor markings share the two features the model relies on most: saturated safety yellow and parallel stripes. The dataset contains very few floor areas without a label that show this pattern. The model therefore has not learned that the pattern must also appear on a human torso.

### FP2: Ordinary red T-shirt detected as `high_visibility_clothing` (`IMG_3495`, confidence 0.35)
**What:** A worker's plain red T-shirt, without reflective stripes, was boxed as high-visibility clothing.
**Why (hypothesis):** Many workers in the dataset wear red company T-shirts, often next to valid vests. The low confidence suggests that the model partly associates "upper body of a worker" with the class. It has not yet learned that reflective stripes are the deciding feature. Examples of red shirts without any label (hard negatives) are under-represented.

### FP3: Shadow next to a worker's head detected as `head_protection` (`IMG_3488`, confidence 0.37)
**What:** A second `head_protection` box appeared on the dark area beside a correctly detected cap.
**Why (hypothesis):** The valid company cap is **dark blue**. At a distance, a dark blob at head height looks like that cap. The class is defined largely by colour and position, so dark shadows near heads are a natural source of confusion.

## 3. False Negatives

### FN1: Distant workers' head protection missed (`IMG_3433`, 2 missed)
**What:** Two workers in the background had labelled head protection that was not detected. The worker in the foreground was detected correctly.
**Why (hypothesis):** At the 640 px training resolution, these caps are only a few pixels wide. Most `head_protection` examples in the training set are close-up, so the model sees few small instances.

### FN2: Crowded scene, several heads missed (`IMG_3493`, 3 missed)
**What:** In a scene with several workers at different distances, three labelled heads were missed. Two overlapping, low-confidence vest boxes (0.40–0.41) also appeared.
**Why (hypothesis):** This scene combines small objects, overlapping people and partial occlusion. It is the same failure pattern observed on the external image `IMG_3446`.

### FN3: Cap seen from above missed (`IMG_3418`)
**What:** A worker bending over a table shows only the top of the cap. The model did not detect it.
**Why (hypothesis):** Almost all training examples show the cap from the front or side. A top-down view changes the shape of the object (a round dark disc rather than a cap with a visor).

## 4. Annotation inconsistencies found

The review revealed two cases where the **label**, not the model, was wrong. These cases lower the reported metrics, so the real performance is slightly better than the numbers suggest. They also show that the class definitions need to be clarified.

| Image | Label | Correct according to the project rules | Effect on metrics |
|---|---|---|---|
| `IMG_3471` | Open vest labelled as `high_visibility_clothing` | Open vests must **not** be labelled | Counted as a false negative although the model was right |
| `IMG_3474` | Red visitor vest with reflective stripes left unlabelled | Confirmed by the project owner as **valid** high-visibility clothing | Counted as a false positive although the model was right |

The red visitor vest (`VISITAS / VISITORS`) was not mentioned in the original class definitions. [`class_definitions.md`](class_definitions.md) has been updated. The frozen dataset (Roboflow version 3) is **not** modified, because the reported results depend on it. The correction is planned for the next dataset version (see Priority 1 below).

## 5. New images kept outside Roboflow

Predictions on the new images are in [`results/evidence/new_images_predictions/`](../results/evidence/new_images_predictions/). Single-worker images at short or medium distance were detected correctly, with high confidence (0.89–0.98).

The weakest result was `IMG_3446`, a crowded scene with four workers at medium distance. The model produced several overlapping and duplicate boxes (`head_protection` 0.43–0.68 and `high_visibility_clothing` 0.49–0.89). The heads of workers looking down were not reliably detected. This is consistent with FN1 and FN2 on the validation split.

## 6. Key findings

- The model works well for **single, close or medium-distance workers**: precision 0.83, recall 0.90 and mAP50 0.92 on the original validation run.
- Errors concentrate on **small, distant, top-down or overlapping PPE**, mainly for `head_protection`. Recall for this class is 0.85–0.86, compared with 0.92–0.93 for clothing.
- **Colour and stripe patterns** drive the false positives: floor markings, red shirts and dark shadows.
- **Two labelling inconsistencies** were found. Label quality therefore limits the measurable performance as much as the model does.
- With 27 validation images, each error moves recall by several points. These metrics are indicative only and do not demonstrate production readiness.

## 7. Prioritized data improvements

### Priority 1: Fix the label contract and relabel (quick, high impact)
- Add the red visitor vest to the class definitions (done) and label it in all images.
- Audit all `high_visibility_clothing` labels and remove the labels on open vests (`IMG_3471` and any similar cases).
- Publish the result as a new Roboflow version and a new GitHub Release. Keep version 3 frozen as the baseline.

*Tied to:* Section 4. Label noise directly distorts both training and evaluation.

### Priority 2: Add small, distant, crowded and top-down PPE examples
- Take 30–50 new photos with **3 or more workers** at 5–15 m, including people bending over or seen from above.
- Label every visible valid cap and vest, even small ones, so that the model learns small instances.

*Tied to:* FN1, FN2, FN3 and `IMG_3446`. `head_protection` recall is the weakest metric.

### Priority 3: Add hard negative examples (images left unlabelled on purpose)
- Floor hazard markings, yellow or orange equipment, forklifts and pallets.
- Red company T-shirts and other clothing without reflective stripes.
- Dark shadows and ordinary caps at head height.

*Tied to:* FP1, FP2 and FP3.

After these changes, the model should be retrained with the same parameters (YOLOv8n, 30 epochs, imgsz 640, batch 16) and compared on the same validation and external images. The goal is to check whether false negatives decrease without an increase in false positives.

## 8. Safety interpretation

For PPE screening, a **false negative** (PPE present but not detected) is treated as more consequential than a false positive. It produces misleading safety information, and in this project's error profile it is the more frequent error for `head_protection`. False positives mainly cost reviewer time.

**This model is an assistive tool for preliminary screening only. It produces false negatives and false positives. It must not be used as the sole verifier for life-safety decisions.**
