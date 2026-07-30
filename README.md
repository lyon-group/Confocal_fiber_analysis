[![DOI](https://zenodo.org/badge/1317742850.svg)](https://doi.org/10.5281/zenodo.21711387)

# Confocal Microscopy Fiber Analysis

A Python-based image-analysis workflow for quantifying fiber-like structures in confocal microscopy images.

The notebook processes multiple experimental groups, segments fibers using Otsu thresholding, removes small or non-fiber objects, calculates structural and intensity-based measurements, and compares groups statistically.

## Features

- Processes individual images and multi-frame image stacks
- Supports grayscale, RGB, TIFF, PNG, and JPEG images
- Performs automatic segmentation using Otsu thresholding
- Removes small background objects and image noise
- Filters detected objects by minimum aspect ratio
- Generates labeled fiber overlays
- Calculates fiber count and fiber density
- Estimates fiber width using skeletonization and distance transforms
- Calculates signal-to-background ratio
- Compares experimental groups using statistical tests
- Exports results to an Excel workbook

## Measurements

For each image or image-stack frame, the notebook reports:

- **Signal-to-background ratio (SBR)**
- **Fiber count**
- **Fiber density (%)**
- **Mean fiber width (pixels)**
- **Median fiber width (pixels)**

Fiber widths are reported in pixels. A spatial calibration must be applied separately to convert these values into micrometers.

## Statistical analysis

The statistical test is selected automatically according to the number of experimental groups:

- **Two groups:** Welch’s independent-samples t-test
- **Three or more groups:** One-way ANOVA
- **Significant ANOVA results:** Tukey’s HSD post-hoc comparisons

Tukey post-hoc testing requires the optional `statsmodels` package.

## Requirements

- Python 3.9 or newer
- Jupyter Notebook or JupyterLab

Install the required packages with:

```bash
pip install numpy pandas matplotlib scipy scikit-image openpyxl statsmodels jupyter
```

## Supported image formats

The workflow searches each dataset folder for:

```text
.jpg
.jpeg
.png
.tif
.tiff
```

Uppercase versions of these extensions are also supported.

The notebook can process:

- Two-dimensional grayscale images
- Two-dimensional RGB or RGBA images
- Three-dimensional grayscale stacks
- Four-dimensional RGB or RGBA stacks

## Repository contents

```text
Confocal_Fiber_Analysis_v2_0.ipynb
```

The notebook contains the complete image-processing, measurement, visualization, statistical-analysis, and export workflow.

## Usage

### 1. Organize the images

Place the images for each experimental condition in a separate folder. For example:

```text
project-data/
├── Control/
│   ├── control_01.tif
│   ├── control_02.tif
│   └── control_03.tif
├── Treatment_A/
│   ├── treatment_a_01.tif
│   ├── treatment_a_02.tif
│   └── treatment_a_03.tif
└── Treatment_B/
    ├── treatment_b_01.tif
    ├── treatment_b_02.tif
    └── treatment_b_03.tif
```

Do not place previously generated result images in the main image folder because the notebook searches that folder for supported image files.

### 2. Configure the dataset paths

Open `Confocal_Fiber_Analysis_v2_0.ipynb` and edit the `DATASET_PATHS` dictionary:

```python
DATASET_PATHS = {
    "Control": r"/path/to/project-data/Control",
    "Treatment A": r"/path/to/project-data/Treatment_A",
    "Treatment B": r"/path/to/project-data/Treatment_B",
}
```

The dictionary keys become the group labels used in figures and statistical comparisons.

Windows paths can be written as raw strings:

```python
DATASET_PATHS = {
    "Control": r"C:\Users\username\project-data\Control",
    "Treatment": r"C:\Users\username\project-data\Treatment",
}
```

### 3. Adjust the segmentation settings

The primary filtering parameters are:

```python
MIN_THICKNESS = 1
MIN_ASPECT_RATIO = 1.5
MIN_AREA = 5
```

#### `MIN_THICKNESS`

Minimum approximate fiber thickness in pixels.

A morphological opening operation is applied when this value is greater than zero. Increasing it can remove thin structures and noise, but excessively high values may remove real fibers.

```python
MIN_THICKNESS = 1
```

Set it to zero to disable thickness-based morphological filtering.

#### `MIN_ASPECT_RATIO`

Minimum ratio between an object’s major-axis and minor-axis lengths:

```text
aspect ratio = major axis length / minor axis length
```

Objects below this value are excluded. Increasing the threshold favors elongated fiber-like objects and rejects more rounded structures.

```python
MIN_ASPECT_RATIO = 1.5
```

#### `MIN_AREA`

Minimum connected-object area in pixels.

Objects smaller than this value are removed as potential noise or speckles.

```python
MIN_AREA = 5
```

Set it to zero to disable area filtering.

### 4. Run the notebook

Run the main code cell in Jupyter Notebook or JupyterLab.

The notebook will:

1. Load the images from every configured folder.
2. Convert RGB images to grayscale.
3. Separate image stacks into individual frames.
4. Normalize image intensities.
5. Segment bright structures using Otsu thresholding.
6. Clean and filter the binary mask.
7. Identify connected fiber-like objects.
8. Calculate image-level measurements.
9. Save labeled overlays and summary figures.
10. Compare experimental groups.
11. Export the results to Excel.

## Image-processing workflow

```text
Input image
    ↓
Grayscale conversion
    ↓
Intensity normalization
    ↓
Otsu thresholding
    ↓
Morphological opening
    ↓
Small-object removal
    ↓
Connected-component labeling
    ↓
Aspect-ratio filtering
    ↓
Skeletonization and width estimation
    ↓
Measurement and statistical analysis
```

### Segmentation

Otsu’s method automatically selects an intensity threshold that separates foreground and background pixels.

Pixels brighter than the threshold are classified as foreground:

```python
mask = gray_img > threshold
```

This approach assumes that the fibers appear brighter than the surrounding background.

### Fiber identification

Connected foreground regions are labeled as separate objects. Objects are retained when:

```text
major-axis length / minor-axis length ≥ MIN_ASPECT_RATIO
```

The resulting object count is reported as the fiber count.

Connected or overlapping fibers may be counted as one object. Therefore, fiber count should be interpreted as the number of segmented connected regions rather than necessarily the number of individual physical fibers.

### Fiber-width estimation

The binary fiber mask is skeletonized to obtain the approximate centerline of each structure.

A Euclidean distance transform measures the distance from every foreground pixel to the nearest background pixel. Width at each skeleton point is estimated as:

```text
local width = 2 × distance to the nearest background pixel
```

The notebook reports the mean and median of these local width measurements.

### Signal-to-background ratio

The signal-to-background ratio is calculated as:

```text
SBR = mean foreground intensity / mean background intensity
```

The foreground is defined by the final filtered fiber mask.

## Output files

A folder named `Results_Otsu` is created inside each dataset directory.

```text
Control/
├── control_01.tif
├── control_02.tif
└── Results_Otsu/
    ├── control_01_labeled.png
    └── control_01_summary.png
```

### Labeled image

The labeled image contains the original grayscale image with segmented objects displayed as a semitransparent overlay:

```text
<image-name>_labeled.png
```

### Image summary

The image summary contains:

- The final binary mask
- The detected object count
- The fiber-width distribution

```text
<image-name>_summary.png
```

For image stacks, the frame number is included in the output filename:

```text
sample_frame_001_labeled.png
sample_frame_001_summary.png
```

### Excel summary

A timestamped Excel workbook is saved in the first folder listed in `DATASET_PATHS`:

```text
Otsu_Analysis_Summary_YYYYMMDD_HHMMSS.xlsx
```

Depending on the analysis, the workbook may contain:

- **Raw Data** — measurements for every image or frame
- **Summary Statistics** — statistical-test type and p-value for each measurement
- **Post-Hoc Details** — Tukey HSD pairwise comparisons when applicable

## Important considerations

### Pixel calibration

The notebook reports width and area-related settings in pixels. Images should have the same spatial resolution when comparing groups.

To obtain measurements in micrometers, use the microscope calibration:

```text
width (µm) = width (pixels) × pixel size (µm/pixel)
```

### Image acquisition consistency

For meaningful comparisons, images should ideally be acquired using consistent:

- Objective magnification
- Pixel dimensions
- Laser power
- Detector gain
- Exposure settings
- Pinhole settings
- Bit depth
- Image-processing settings

Differences in acquisition conditions can affect segmentation, SBR, density, and width measurements.

### Statistical independence

Each image or frame is treated as an observation. Frames collected from the same sample may not be statistically independent.

For biological experiments, consider summarizing measurements at the biological-replicate level or using a statistical model that accounts for repeated measurements or nested data.

### Thresholding limitations

Global Otsu thresholding may perform poorly when:

- Background illumination is uneven
- Signal intensity varies strongly across the image
- Fibers are darker than the background
- Images contain strong autofluorescence
- Fibers overlap extensively
- Noise intensity is similar to fiber intensity

The segmentation parameters should be validated visually using the labeled overlays before interpreting quantitative results.

## Troubleshooting

### No images found

Confirm that:

- The dataset path is correct.
- The files use one of the supported extensions.
- The images are directly inside the specified folder.
- The folder is accessible from the Python environment.

### Too many small objects are detected

Increase:

```python
MIN_AREA
```

You may also increase:

```python
MIN_THICKNESS
```

### Rounded objects are counted as fibers

Increase:

```python
MIN_ASPECT_RATIO
```

### Real thin fibers are missing

Reduce:

```python
MIN_THICKNESS
```

or disable thickness filtering:

```python
MIN_THICKNESS = 0
```

### Post-hoc tests are skipped

Install `statsmodels`:

```bash
pip install statsmodels
```

Restart the Jupyter kernel and run the notebook again.

### Excel output cannot be saved

Install `openpyxl`:

```bash
pip install openpyxl
```

Also confirm that the destination folder is writable and that an existing workbook with the same name is not open in another program.

## Citation

When using this workflow in published research, cite the repository and the software packages used in the analysis. A repository citation can be added after the project receives a permanent release or DOI.

Suggested repository citation format:

```text
Author(s). Confocal Microscopy Fiber Analysis. Version 2.0.
GitHub repository, year. Repository URL.
```

## License

This project is licensed under the MIT License. See the [`LICENSE`](LICENSE) file for details.

## Disclaimer

This workflow is intended for research and exploratory image analysis. Results should be validated against representative images, appropriate controls, and independent analysis methods before being used for scientific or clinical conclusions.
