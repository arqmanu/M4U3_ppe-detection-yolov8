# AECO Governance Checklist

## 1. Data Provenance

- **Source:** Photographs taken by the project author at their own workplace (industrial assembly and warehouse areas), specifically for this academic project. No third-party or client images are used.
- **Date of collection:** September 2026. The photo metadata shows 22 Sep 2026. The photos were uploaded and annotated in Roboflow on 23 Sep 2026.
- **Owner:** The project author.
- **Annotation:** Roboflow Astra Auto Label proposals, followed by full manual review and correction against [`class_definitions.md`](class_definitions.md).
- **Frozen version:** Roboflow version 3: 134 images, 107 train and 27 validation. It is published as a GitHub Release asset with SHA-256 verification (see the README).
- **Classes:** `head_protection` and `high_visibility_clothing`.

## 2. PII Handling (Privacy and Consent)

- **Faces:** People appear in the images. Faces were obscured before publication in most images. **Some faces remain partially visible** (profile views or partly covered faces, e.g. `IMG_3409`).
- **Consent:** Every person shown signed a written consent that covers the use and public release of these images, including the images where the face remains partly visible.
- **Other PII:** No licence plates, names, badges with readable names, screens or documents are intentionally included. Company logos on clothing remain visible.
- **Protection strategy:**
  1. Written consent was collected before publication.
  2. Faces were manually obscured wherever practical.
  3. The model detects **objects (PPE), not people**. It performs no face recognition, identification or tracking.
- **Evidence pack:** The example images in `results/evidence/annotations/` were chosen so that no face is visible.

## 3. Data Minimisation

- Only the visual information needed to detect the two PPE classes is kept. The labels contain only class IDs and box coordinates.
- No identity, names, employee IDs, location tracking, audio or biometric data are processed or stored.
- Footwear and safety glasses were deliberately excluded. They were too small or too rarely visible to be labelled reliably, and unreliable labels would lead to unreliable conclusions.
- Images were resized to at most 640 × 640 px, which reduces the level of detail that is stored and published.

## 4. Intended Use and Limitations

**Intended use:** academic prototype for **preliminary screening** of workplace photographs. A competent person reviews every result.

**Do not use this model:**
- as the sole method of safety enforcement, or to certify compliance;
- for automated disciplinary decisions, worker surveillance or identification;
- on crowded scenes, distant workers (more than about 10 m) or top-down camera views, where most observed errors occur;
- on PPE types or sites that are not represented in the dataset (e.g. boots, glasses, harnesses, other companies' uniforms, outdoor construction sites, night or rain);
- as if it detected *non-compliance*. It detects *valid PPE*. Absence of a detection does **not** prove that PPE is missing. A follow-up iteration with explicit `person_no_helmet` / `person_no_vest` classes is planned to address this.

**Known dataset limits:**
- 134 images from a single site.
- 27 validation images, so metrics move by several points per error.
- White helmets appear in only one image.
- Two annotation inconsistencies were found and documented in [`error_analysis.md`](error_analysis.md).

## 5. Risk Statement

- **High-impact false negative:** valid PPE is present but not detected. Or, in the planned follow-up, a worker **without** PPE is not flagged. The consequence is a missed safety issue and false reassurance. This is the error type that the success criteria prioritise (recall ≥ 0.85 per class). Observed causes: small, distant and top-down heads.
- **High-impact false positive:** the model "sees" PPE that is not there, for example floor hazard stripes or a red T-shirt detected as a vest. In a compliance context, this could hide a worker who is not wearing PPE. In the current screening setup, it mostly costs reviewer time.
- **Trade-off:** For this use case, **false negatives are treated as more consequential**. If the model is used, the confidence threshold should favour recall, and the extra false alarms should be absorbed by human review.

## 6. Human-in-the-Loop

- **Review process:** The model is for screening only. The site safety technician (or another competent person) reviews **every** image and every detection, and makes the final decision.
- The reviewer checks that detected items are valid PPE, inspects uncertain or missed cases, and applies site-specific rules.

## 7. Project-Specific Label Rules

- **Open high-visibility vests are not labelled.** This is a deliberate academic decision that distinguishes *wearing a garment* from *wearing it correctly*. It is not a general legal rule.
- The **red visitor vest with reflective stripes** is valid high-visibility clothing (clarified after the error analysis; see the change log in [`class_definitions.md`](class_definitions.md)).

## 8. License

- **Code and notebooks:** MIT License ([`LICENSE`](../LICENSE)).
- **Dataset (images and labels):** CC BY 4.0, the same license as on Roboflow Universe. The author owns the images and holds written consent from the people shown. This allows public redistribution.
- **Model weights (`best.pt`):** published in the GitHub Release for academic reproducibility. Ultralytics YOLOv8, which is used for training and inference, is licensed under AGPL-3.0.

## 9. Reproducibility and Bus Factor

- The notebook runs in a fresh Google Colab session with **no credentials**: no Roboflow key, no Colab Secrets, no Google Drive, no local paths and no manual uploads.
- Dataset and weights are downloaded from the public GitHub Release and verified with SHA-256. The new test images are cloned from the public repository.
- A third party can reproduce the results without contacting the author. See the reproducibility proof in the README.

**This model is an assistive tool for preliminary screening only. It produces false negatives and false positives. It must not be used as the sole verifier for life-safety decisions.**
