# AECO Governance Checklist

## 1. Data Provenance

- **Source:** Workplace photographs taken specifically for this academic project.
- **Dataset owner:** The project author.
- **Annotation platform:** Roboflow.
- **Frozen dataset version:** Roboflow Version 3.
- **Dataset split:** 107 training images and 27 validation images.
- **Classes:** `head_protection` and `high_visibility_clothing`.
- **Dataset license:** CC BY 4.0.
- **Public dataset archive:** Published as a frozen GitHub Release asset with SHA256 verification.

## 2. Privacy and Consent

- People appear in the original workplace photographs.
- Written consent was obtained from the photographed participants.
- Faces were manually obscured before the images were uploaded and published.
- The dataset does not attempt to identify, recognize, or track individuals.
- Personal names and employee identifiers are not used as model inputs or labels.
- Images were reviewed to avoid publishing visible credentials, documents, screens, or other unnecessary sensitive information.

## 3. Data Minimization

Only visual information required to detect the defined PPE classes was retained.

The project does not process:

- Personal identity.
- Facial recognition information.
- Employee performance data.
- Location tracking information.
- Audio.
- Biometric identification.

Footwear and safety glasses were excluded because their visual variability and limited representation could result in unreliable labels and conclusions.

## 4. Intended Use

The model is intended as an academic prototype for preliminary PPE screening in workplace photographs.

The model may help identify:

- Valid head protection.
- Valid high-visibility clothing.

A human safety professional must review all detections and make the final decision.

## 5. Limitations and Prohibited Uses

This model must not be used:

- As the sole method for workplace safety enforcement.
- To certify that a worker or workplace complies with safety regulations.
- For automated disciplinary decisions.
- For employee surveillance or identification.
- In operational safety systems without additional testing and human oversight.
- With image conditions or PPE types that are not represented in the dataset.

**This model is an assistive tool for preliminary screening only. It produces false negatives and false positives. It must not be used as the sole verifier for life-safety decisions.**

## 6. Project-Specific Label Rules

The following annotation rule is a deliberate project decision:

- Open high-visibility vests are treated as invalid use and are not labelled as `high_visibility_clothing`.

This rule makes the exercise more demanding by distinguishing between the presence of a garment and its defined valid use. It does not establish a general legal or workplace safety rule.

The white construction helmet is valid head protection, but it is represented by only one image. The model must therefore not be assumed to generalize reliably to white helmets.

## 7. Risk Statement

### False Negatives

A false negative occurs when valid PPE is present but the model does not detect it.

This is the highest-impact error because a missed detection could incorrectly suggest that required protection is absent or could fail to identify valid protection during a safety review.

### False Positives

A false positive occurs when the model detects valid PPE where none exists.

Examples may include ordinary coloured clothing, distant objects, reflections, or partially visible garments.

False positives create unnecessary manual reviews, but they are generally less dangerous than relying on a false negative in a life-safety context.

## 8. Human-in-the-Loop

Every prediction must be reviewed by a competent person.

The model provides preliminary visual screening only. The human reviewer remains responsible for:

- Confirming whether the detected item is valid PPE.
- Considering site-specific rules and context.
- Inspecting uncertain or missed cases.
- Making any operational or safety-related decision.

## 9. Reproducibility and Bus Factor

The final workflow avoids dependencies on temporary Colab files, local computer paths, personal Roboflow credentials, and manual uploads.

The frozen dataset is downloaded from a public GitHub Release without credentials and verified using a SHA256 checksum.

The notebook downloads the fixed external test images from the public repository. This allows another person to reproduce the workflow in a fresh Google Colab session without access to the original author's computer or accounts.

## 10. Licensing

- **Repository software and documentation:** MIT License.
- **Dataset:** CC BY 4.0.
- **Model weights:** Distributed through the project GitHub Release for academic reproducibility.
- The photographs were created for this project and published only after the required consent and privacy measures were applied.
