# AI-Based Mitral Valve Annuloplasty Suture Analysis

A deep learning and computer vision framework for automated analysis of suture placement during mitral valve annuloplasty. The project combines YOLO-based suture landmark detection, mitral annulus segmentation, and OpenCV-based geometric analysis to estimate suture count, spacing, annular coverage, and comparatively large gaps.

## Project Overview

This project was developed as part of an MEng research project at Western University in collaboration with the Robarts Research Institute and cardiac surgery researchers.

The system focuses on the stage of mitral valve repair where sutures are placed around the mitral annulus before the annuloplasty ring is implanted. At this stage, the suture entry and exit points are visible and can provide useful information about suture distribution.

The final framework consists of three main components:

1. **Suture landmark detection** – detects entry and exit points using YOLO26.
2. **Mitral annulus segmentation** – identifies the visible annulus using YOLO26 segmentation.
3. **Geometric post-processing** – combines the model predictions using OpenCV to analyse suture placement.

## Workflow

```text
Surgical Videos
      ↓
Frame Extraction
      ↓
Image Annotation
      ↓
┌─────────────────────────────┐
│ YOLO26 Entry/Exit Detection │
└─────────────────────────────┘
      ↓
 Entry / Exit Landmarks

┌─────────────────────────────┐
│ YOLO26 Annulus Segmentation │
└─────────────────────────────┘
      ↓
   Annulus Mask
      ↓
┌─────────────────────────────┐
│ OpenCV Post-Processing      │
│ • Duplicate removal         │
│ • Landmark projection       │
│ • Suture ordering           │
│ • Spacing analysis          │
│ • Coverage estimation       │
│ • Gap analysis              │
└─────────────────────────────┘
      ↓
Quantitative Suture Analysis
```

## Models

### YOLO26 Entry/Exit Point Detection

The final detection model uses YOLO26 to detect two classes:

* `entry_point`
* `exit_point`

The detected bounding-box centres are used as landmark coordinates for subsequent geometric analysis.

The final detection experiment used a case-based split:

* **Training:** V1–V11
* **Validation:** V12–V13
* **Testing:** V14–V15

This case-based separation helps prevent highly similar frames from the same surgical procedure appearing across different subsets.

### YOLO26 Annulus Segmentation

A separate YOLO26 segmentation model was developed to automatically identify the visible mitral annulus.

Annulus annotations were available for V7–V15. For the final segmentation experiment:

* **Development cases:** V7–V14
* **Independent test case:** V15
* V7–V14 were divided into training and validation subsets.

The segmentation output is converted into a centreline representation that is used as the geometric reference for suture analysis.

## Post-Processing

OpenCV-based processing combines the outputs from the detection and segmentation models.

The main processing steps include:

* Removal of duplicate entry and exit detections
* Estimation of visible suture count
* Projection of detected landmarks onto the annulus
* Ordering of landmarks along the annular curve
* Entry–exit landmark pairing
* Measurement of neighbouring suture spacing
* Estimation of annular coverage
* Identification of comparatively large gaps
* Visualization of the final results

The final-to-first annulus connection is not treated as a suture gap because the visible annulus is handled as an open curve.

Gap measurements are relative to the spacing observed within an image and should **not** be interpreted as clinically validated thresholds.

## Dataset

The dataset consists of surgical images extracted from mitral valve annuloplasty videos.

The original data are organized by surgical case:

```text
V1
V2
V3
...
V15
```

The dataset includes:

* Original surgical videos
* Extracted image frames
* YOLO entry/exit point annotations
* CVAT annulus annotations
* YOLO segmentation labels
* Training/validation/test datasets
* Model evaluation outputs

### Annotation Classes

For object detection:

```text
0: entry_point
1: exit_point
```

For segmentation:

```text
0: mitral_annulus
```

### Data Privacy

The dataset contains surgical research data. The original videos and patient-related images should only be stored and shared through approved research or institutional storage locations.

The surgical dataset should **not be uploaded to a public GitHub repository** unless appropriate permissions and data-sharing approvals have been obtained.

## Results

### Entry/Exit Detection

The final YOLO26 detection model achieved:

| Metric       | Result |
| ------------ | -----: |
| Precision    | 0.9656 |
| Recall       | 0.9392 |
| F1 Score     | 0.9491 |
| mAP@0.5      | 0.9844 |
| mAP@0.75     | 0.9292 |
| mAP@0.5:0.95 | 0.8065 |

These results were obtained on the independent V14–V15 test cases.

### Annulus Segmentation

Performance on the V7–V14 validation data:

| Metric       | Result |
| ------------ | -----: |
| Precision    | 0.9698 |
| Recall       | 0.9655 |
| F1 Score     | 0.9676 |
| mAP@0.5      | 0.9591 |
| mAP@0.75     | 0.9292 |
| mAP@0.5:0.95 | 0.5819 |

The independent V15 test case produced zero reported segmentation metrics. This indicates that the segmentation model did not generalize well to the unseen surgical case and remains the main limitation of the current framework.

## Project Structure

A simplified project structure is shown below:

```text
project/
│
├── Suture/
│   └── Original surgical videos
│
├── frames/
│   ├── v1/
│   ├── v2/
│   ├── ...
│   └── v15/
│
├── combined_dataset/
│   ├── all_images/
│   ├── all_labels/
│   ├── entry_exit_detection/
│   ├── annulus_segmentation_80_20/
│   ├── yolo_training/
│   └── yolo_evaluation/
│
├── combinedold_dataset/
│   └── Stage II YOLOv8 dataset and results
│
├── Suture counting.ipynb
├── README.md
└── Dataset_Guide.docx
```

Some folders contain intermediate or corrected versions created during dataset preparation and model development.

## Main Technologies

* Python
* YOLO26
* YOLOv8
* Ultralytics
* PyTorch
* OpenCV
* NumPy
* SciPy
* scikit-image
* Matplotlib
* Pandas
* CVAT
* Google Colab

## Running the Project

The main development workflow is contained in:

```text
Suture counting.ipynb
```

The notebook includes code for:

1. Preparing the surgical image dataset
2. Extracting and organizing frames
3. Preparing YOLO annotations
4. Training YOLO detection models
5. Training YOLO segmentation models
6. Evaluating model performance
7. Processing predicted annulus masks
8. Projecting suture landmarks onto the annulus
9. Calculating suture spacing and coverage
10. Visualizing the final analysis

The original notebook was developed primarily in Google Colab and uses Google Drive paths for storing the datasets and model outputs. These paths may need to be modified when running the notebook in another environment.

## Limitations

The current framework has several limitations:

* The number of independent surgical cases is limited.
* Annulus segmentation does not generalize well to the independent V15 test case.
* The quality of geometric analysis depends on both landmark detection and annulus segmentation.
* Partial visibility of the annulus can affect coverage measurements.
* Large-gap detection is based on relative image geometry rather than clinically validated measurements.
* The current system was evaluated using retrospective surgical images rather than during live surgery.
* Pixel-based measurements have not yet been converted into physical millimetres through camera calibration.

## Future Work

Future development could include:

* Increasing the number and diversity of annotated surgical cases
* Improving annulus segmentation generalization
* Using temporal information from surgical videos
* Camera calibration for physical measurements
* Establishing clinically meaningful suture-spacing thresholds
* Testing on larger external datasets
* Improving real-time processing
* Integrating the system into an intraoperative workflow

## Intended Use

This repository documents a research prototype developed for academic and research purposes. The system is **not intended for clinical diagnosis, treatment decisions, or direct surgical guidance**.

## Acknowledgements

This project was completed as part of the MEng program in Electrical and Computer Engineering at Western University.

The project used surgical data provided through collaboration with the Robarts Research Institute and cardiac surgery researchers.

## Author

**Ashwini Dalve**

MEng Electrical and Computer Engineering
Western University, London, Ontario, Canada
