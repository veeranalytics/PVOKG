# Part–Verb–Object Knowledge Graphs: An Interpretable Intermediate Representation for Micro-Action Recognition

**Author:** Veer Abhimanyu Singh
**Status:** manuscript under review

> **The code and data source will be made available on request.**

---

## Overview

Micro-actions are brief, low-amplitude movements such as touching the neck, nodding or shifting in a chair.
This work recognises them through an explicit, human-readable representation instead of an opaque network.

- Each clip is converted into **Part–Verb–Object (PVO) primitives**: a body part, an atomic movement verb that fires only when stated kinematic criteria hold, an optional contacted body part, and a time interval (e.g. `right_hand – raise – head`).
- Temporal and anatomical edges link the primitives into a small **knowledge graph**.
- That graph is the **only input** to a graph-transformer classifier. Every explanation is therefore a subset of the evidence the model actually used, and its faithfulness can be tested.
- One fixed ontology covers **10 body parts and 20 verbs** (13 body, 7 facial) and is shared across three skeleton formats.

## Main results

| Dataset | Protocol | PVO-KG top-1 (%) | Mean-class (%) |
|---|---|---|---|
| MA-52 | official train / val | 33.65 ± 0.02 | 25.77 ± 0.33 |
| NTU RGB+D 120 | cross-subject | 53.55 ± 0.18 | 56.09 ± 0.17 |
| NW-UCLA | cross-view (v1+v2 → v3) | 75.29 ± 0.73 | 75.50 ± 0.73 |

Mean ± standard deviation over three seeds. On MA-52, an ST-GCN trained on the same poses reaches 51.30 %.

- **Price of the bottleneck:** a variant with a raw-feature skip path is no more accurate on any dataset.
- **Faithfulness:** deleting primitives in attention order lowers the predicted-class probability significantly faster than random deletion, on every dataset and seed.
- **Transfer:** five-shot NW-UCLA accuracy rises from 28.5 % to 54.9 % after pre-training on NTU RGB+D 120.
- **Compositional generalisation:** unseen part–verb pairs keep 58–84 % of random-split accuracy.

---

## Datasets

Five public datasets are used: three recognition benchmarks and two facial-calibration datasets.
**None of them is redistributed here.** Each must be obtained from its original authors under its own licence.

| | MA-52 | NTU RGB+D 120 | NW-UCLA | SAMM | DISFA |
|---|---|---|---|---|---|
| Use in this work | Benchmark | Benchmark | Benchmark | Facial calibration | Facial calibration |
| Clips / sequences | 22,422 | 114,480 | 1,494 | 159 | 27 videos |
| Classes | 52 (7 groups) | 120 | 10 | — | 12 AUs |
| Subjects | 205 | 106 | 10 | 32 | 27 |
| Capture | RGB video, 30 fps | Kinect v2, 3 views | Kinect v1, 3 views | video, 200 fps | video |
| Input used | OpenPose 28 keypoints, 2D | Kinect 25 joints, 3D | Kinect 20 joints, 3D | face landmarks | 66 landmarks |
| Protocol | official train / val | cross-subject | cross-view | — | — |
| Train / test used | 11,250 / 5,586 | 63,026 / 50,919 | 1,020 / 464 | — | 130,814 frames |

### MA-52 (micro-action benchmark)

- **Content:** 22,422 clips of spontaneous micro-actions from 205 participants, recorded during psychological interviews at 1920 × 1080 and 30 fps. Clips last 1–7 s (mean 1.9 s).
- **Labels:** 52 fine classes in 7 coarse body-part groups: body, head, upper limb, lower limb, body–hand, head–hand and leg–hand.
- **Input used:** the 28-keypoint 2D OpenPose skeletons distributed with the dataset (18 body joints + 10 fingertips).
- **Split:** the official split is 11,250 / 5,586 / 5,586 (train / val / test). Test labels are not public, so we train on the official training split (10,125 train, 1,125 for model selection) and report on the official validation split.
- **Class balance:** strongly long-tailed. Nodding, shaking head and turning head together are 30.6 % of training clips; the rarest classes have fewer than ten.
- **Keypoint quality:** fingertips are undetected in 21.5 % of frames, other joints in 0.6 %. Undetected joints are filled by linear interpolation over time.
- **Reference:** D. Guo, K. Li, B. Hu, Y. Zhang, M. Wang, *Benchmarking micro-action recognition: dataset, methods, and applications*, IEEE TCSVT 34 (7) (2024) 6238–6252. [arXiv:2403.05234](https://arxiv.org/abs/2403.05234)

| Group | Classes | Train / test clips | Sample classes |
|---|---|---|---|
| Body | 5 | 644 / 315 | shaking body, shrugging, turning around, rising up |
| Head | 6 | 4,473 / 2,233 | nodding, shaking head, turning head, tilting head |
| Upper limb | 13 | 2,626 / 1,304 | illustrative gestures, waving, clenching fist, spreading hands |
| Lower limb | 8 | 1,441 / 717 | shaking legs, crossing legs, tiptoe, stretching feet |
| Body–hand | 6 | 317 / 152 | touching neck, touching chest, arms akimbo, crossing arms |
| Head–hand | 10 | 719 / 353 | playing with hair, touching nose, covering mouth, pushing glasses |
| Leg–hand | 4 | 1,030 / 512 | touching legs, patting legs, scratching legs, scratching feet |

### NTU RGB+D 120

- **Content:** 114,480 clips of 120 daily, mutual and health-related actions by 106 subjects, recorded with three Kinect v2 cameras in 32 set-ups.
- **Input used:** 25-joint 3D skeletons of the primary body. The 535 samples with missing skeletons are excluded (113,945 remain), using the official list in the [NTURGB-D repository](https://github.com/shahroudy/NTURGB-D).
- **Split:** cross-subject (X-Sub): 63,026 training and 50,919 test clips.
- **Access:** by request from the [ROSE Lab](https://rose1.ntu.edu.sg/dataset/actionRecognition/).
- **References:** A. Shahroudy et al., CVPR 2016, pp. 1010–1019; J. Liu et al., IEEE TPAMI 42 (10) (2020) 2684–2701.

### NW-UCLA (Northwestern-UCLA Multiview Action 3D)

- **Content:** 1,494 clips of 10 actions by 10 subjects, each captured at the same time by three Kinect v1 cameras. 1,484 clips have skeletons in the `all_sqe` release.
- **Input used:** 20-joint 3D skeletons.
- **Split:** standard cross-view: views 1 and 2 for training (1,020 clips), view 3 for testing (464 clips).
- **Reference:** J. Wang, X. Nie, Y. Xia, Y. Wu, S.-C. Zhu, *Cross-view action modeling, learning and recognition*, CVPR 2014, pp. 2649–2656. [arXiv:1405.2941](https://arxiv.org/abs/1405.2941)

### SAMM (facial calibration, temporal)

- **Content:** 159 spontaneous micro-movements from 32 participants, filmed at 200 fps. Each has onset, apex and offset frames and FACS action-unit codes.
- **Use:** checks whether the facial verbs fire on expert-annotated micro-expressions. Not used for recognition.
- **Reference:** A.K. Davison, C. Lansley, N. Costen, K. Tan, M.H. Yap, *SAMM: a spontaneous micro-facial movement dataset*, IEEE Trans. Affective Computing 9 (1) (2018) 116–129. Available from the authors at Manchester Metropolitan University.

### DISFA (facial calibration, intensity)

- **Content:** spontaneous facial video of 27 subjects (130,814 frames), with 66 landmarks per frame and frame-level intensity (0–5) for 12 action units.
- **Use:** checks whether each facial verb's geometric signal tracks expert-graded intensity. Not used for recognition.
- **References:** S.M. Mavadati et al., IEEE Trans. Affective Computing 4 (2) (2013) 151–160; S.M. Mavadati, P. Sanger, M.H. Mahoor, CVPRW 2016, pp. 1452–1459. Available from the authors on request.

---

## Data and code availability

**The code and data source will be made available on request.**

## Citation

If you use this work, please cite:

```
V. A. Singh, Part–Verb–Object Knowledge Graphs: An Interpretable Intermediate Representation
for Micro-Action Recognition, under review, 2026.
```

## Acknowledgements

We thank the authors of MA-52, NTU RGB+D 120, NW-UCLA, SAMM and DISFA for making their datasets
available to the research community.

