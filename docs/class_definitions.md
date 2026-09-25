# Class Definitions and Annotation Rules

## 1. head_protection

Valid head protection includes:

- The company dark-blue protective work cap.
- A valid construction safety helmet, including the white helmet.

Do not label ordinary caps or other non-protective headwear.

## 2. high_visibility_clothing

Valid high-visibility clothing includes:

- Yellow or orange safety vests with reflective stripes.
- Yellow-and-grey safety fleece with reflective stripes.
- Red visitor vest (`VISITAS / VISITORS`) with reflective stripes, **when closed**. *(Added after the error analysis; see the change log.)*

The garment must be worn correctly and closed.

Do not label:

- Open safety vests.
- Ordinary shirts or clothing without reflective stripes.
- Red shirts (e.g. the company T-shirt) or other clothing without reflective stripes.

## Bounding Box Rules

- Create one bounding box for each valid PPE item.
- Keep the box close to the visible item.
- Do not create duplicate or overlapping boxes for the same item.
- Label valid PPE whether worn by a person or shown separately.
- Partially visible PPE should only be labelled when it can be identified confidently.


## Change log

- **25 Sep 2026:** The error analysis found that the red visitor vest with reflective stripes was not covered by these rules. The project owner confirmed that it is valid high-visibility clothing when closed. (On re-check, the vest in `IMG_3474` is worn open, so leaving it unlabelled was correct.) It also found one open vest that had been labelled by mistake (`IMG_3471`). The frozen dataset (Roboflow version 3) used for the reported results is **not** modified. The corrections were applied in the iteration-2 dataset (Roboflow version 5). See [error_analysis.md](error_analysis.md).

## Iteration 2: compliance classes (Roboflow version 5)

Two violation classes were added. Every visible person gets one head label and one torso label:

| Class | Box on | Rule |
|---|---|---|
| `no_head_protection` | head | Head visible and not wearing valid head protection (bare head or ordinary cap) |
| `no_high_visibility_clothing` | torso | Torso visible and no valid garment: ordinary clothing, red company T-shirt, **open** vest or jacket |

If a person is too distant, cut off or occluded to decide, no label is drawn. Details: [iteration2.md](iteration2.md).
