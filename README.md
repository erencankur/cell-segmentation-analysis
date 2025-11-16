# Cell Segmentation in Pathology Biopsy Images

A comprehensive image processing pipeline for automated nuclei detection and segmentation in H&E stained pathology biopsy images using computer vision and machine learning techniques.

## 📋 Overview

This project implements an end-to-end pipeline for analyzing pathology biopsy images, specifically designed to segment and quantify cell nuclei in tissue samples. The images were sourced from the lecture material "Cellular Adaptation Mechanisms and Intracellular Accumulations" (Dr. Toros Taşkın). The system uses advanced image processing algorithms including color deconvolution, adaptive thresholding, and marker-based watershed segmentation to accurately identify and separate individual nuclei, even when they are touching or overlapping.

### Key Features

- **H&E Color Deconvolution**: Separates Hematoxylin and Eosin staining channels for optimal nuclei contrast
- **CLAHE Enhancement**: Contrast Limited Adaptive Histogram Equalization for improved local contrast
- **Multiple Segmentation Methods**: Implements Otsu, Adaptive, and Watershed algorithms for comparison
- **Marker-Based Watershed**: Advanced segmentation to separate touching nuclei accurately
- **Automated Quantification**: Counts nuclei and measures their morphological properties
- **Comprehensive Visualization**: Side-by-side comparison of all processing steps
- **Batch Processing**: Processes entire datasets automatically
- **CSV Export**: Detailed measurements exported for further analysis

## 🛠️ Technologies Used

- **Python 3.8+**
- **OpenCV**: Image preprocessing and manipulation
- **scikit-image**: Segmentation algorithms and morphological operations
- **NumPy**: Numerical computations
- **Pandas**: Data handling and CSV export
- **Matplotlib**: Visualization and result plotting
- **SciPy**: Distance transform and morphological operations

## 📁 Project Structure

```
cell-segmentation-analysis/
├── cell_segmentation.ipynb                     # Main Jupyter notebook with complete pipeline
├── README.md                                   # Project documentation
├── requirements.txt                            # Python dependencies
├── dataset/                                    # Input pathology biopsy images (.png)
└── output/                                     # Generated results
    ├── *_measurements.csv                      # Individual image measurements
    ├── *_comparison.png                        # Step-by-step visualization
    ├── _all_images_measurements.csv            # Summary of all images
    └── _combined_original_vs_segmentation.png  # Combined comparison
```

## 🚀 Installation

### Prerequisites

- Python 3.8 or higher
- pip package manager
- Jupyter Notebook or JupyterLab

### Setup Instructions

1. **Clone or download this project:**
   ```bash
   cd cell-segmentation-analysis
   ```

2. **Create a virtual environment (recommended):**
   ```bash
   python -m venv venv
   source venv/bin/activate  # On macOS/Linux
   # or
   venv\Scripts\activate     # On Windows
   ```

3. **Install required packages:**
   ```bash
   pip install -r requirements.txt
   ```

4. **Launch Jupyter Notebook:**
   ```bash
   jupyter notebook cell_segmentation.ipynb
   ```

## 📊 Usage

### Preparing Your Data

1. Place your H&E stained pathology biopsy images (`.png` format) in the `dataset/` directory
2. The images should be RGB color images with standard H&E staining

### Running the Pipeline

1. Open `cell_segmentation.ipynb` in Jupyter Notebook
2. Run all cells sequentially (Cell → Run All)
3. The pipeline will automatically:
   - Load all images from `dataset/`
   - Preprocess using H&E deconvolution and CLAHE
   - Apply three segmentation methods (Otsu, Adaptive, Watershed)
   - Count and measure nuclei properties
   - Generate visualizations
   - Export results to `output/` directory

### Understanding the Output

**For each image, the pipeline generates:**

- `{filename}_measurements.csv`: Detailed measurements for each detected nucleus
  - Columns: `filename`, `cell_name`, `cell_area`
  
- `{filename}_comparison.png`: Six-panel visualization showing:
  1. Original image
  2. Preprocessed image (Hematoxylin channel)
  3. Otsu threshold result
  4. Adaptive threshold result
  5. Watershed binary mask
  6. Final labeled nuclei (color-coded)

**Summary files:**

- `_all_images_measurements.csv`: Aggregated statistics for all images
  - Columns: `filename`, `nucleus_count`, `mean_nucleus_area`, `total_cell_area`
  
- `_combined_original_vs_segmentation.png`: Side-by-side comparison of all images

## 🔬 Methodology

### 1. Preprocessing Pipeline

```
RGB Image → H&E Deconvolution → Hematoxylin Channel → 
CLAHE Enhancement → Gaussian Blur → Preprocessed Image
```

- **H&E Color Deconvolution**: Images were converted to the HED color space using `skimage.color.rgb2hed` to separate the Hematoxylin (stains nuclei) and Eosin (stains cytoplasm) components. Only the Hematoxylin channel was selected for nucleus detection.
- **CLAHE**: The `skimage.exposure.equalize_adapthist` (CLAHE) filter was applied to enhance local contrast while preventing over-amplification, making the nuclei more distinct.
- **Gaussian Blur**: Reduces noise for cleaner segmentation

### 2. Segmentation Methods

The pipeline implements three algorithms for comparison:

1. **Otsu's Global Thresholding**: Analyzes the image histogram to determine a single, global threshold value, partitioning the image into foreground (nuclei) and background.

2. **Adaptive Local Thresholding**: Calculates a local threshold for each pixel based on its neighborhood to adapt to varying illumination conditions across the image.

3. **Marker-Based Watershed**: This was the most advanced method used, specifically to separate touching and overlapping nuclei. The centers of the nuclei were found by calculating the `distance_transform_edt` of the image. These centers were designated as "markers." The Watershed algorithm then treated the cell boundaries (found using a Sobel filter) as "ridges" and successfully separated adjacent nuclei by expanding from the markers.

### 3. Analysis and Measurement

The output from the Watershed segmentation, which provided the most robust results, was used for analysis. The `skimage.measure.regionprops_table` function was used to measure the following properties for each labeled nucleus:

- **Total nucleus count**: Number of detected nuclei
- **Mean nucleus area**: Average area of detected nuclei
- **Total cell area**: Sum of all detected nuclei areas

### 4. Watershed Algorithm Details

The watershed segmentation uses several optimized parameters for balanced detection:

- **Threshold Multiplier**: 0.75 × Otsu threshold for increased sensitivity
- **Min Distance**: 9 pixels between nucleus centers
- **Threshold Relative**: 0.25 for peak detection in distance transform
- **Min Size**: 10 pixels to filter noise objects

These parameters are tuned to balance sensitivity (detecting faint nuclei) and specificity (avoiding over-segmentation).

## 📈 Performance Optimization

The pipeline includes several optimization strategies:

- **Morphological Operations**: Opening/closing to refine masks
- **Small Object Removal**: Filters noise while preserving real nuclei
- **Hole Filling**: Completes partially detected nuclei
- **Distance Transform**: Accurately identifies nucleus centers
- **Gradient-Based Watershed**: Uses Sobel gradient for precise boundaries

## 🔧 Customization

### Adjusting Segmentation Sensitivity

In the `segment_watershed()` function, you can tune these parameters:

```python
thresh *= 0.75           # Lower = more sensitive (detects fainter nuclei)
min_distance=9           # Lower = detects smaller/closer nuclei
threshold_rel=0.25       # Lower = more nucleus markers detected
min_size=10              # Lower = keeps smaller objects
```

### Modifying Preprocessing

In the `preprocess()` function:

```python
clip_limit=0.05          # CLAHE contrast enhancement strength
GaussianBlur((3, 3), 0)  # Blur kernel size for noise reduction
```

## 📝 Results

The workflow generates the following outputs for each source image, saved to the `output/` directory:

1. **Output Images (.png)**: Six-panel side-by-side plots showing:
   - Original image
   - Preprocessed image
   - Otsu segmentation result
   - Adaptive segmentation result
   - Watershed binary mask
   - Final labeled nuclei (color-coded)

2. **Measurement Reports (.csv)**: Detailed CSV files containing a list of all detected nuclei and their area measurements for each image.

3. **Summary Report**: `_all_images_measurements.csv` containing aggregated statistics for all processed images.

4. **Combined Comparison**: `_combined_original_vs_segmentation.png` showing all images side-by-side.

### Typical Results

The pipeline successfully processes various tissue types including:

- Lymphoma samples
- Bone marrow biopsies
- Malignant melanoma
- Breast tissue
- Liver tissue
- Gastric mucosa
- Brain glioblastoma
- Kidney glomerulus

Performance metrics:
- **Nucleus Count**: 50-300+ per image (depending on tissue density)
- **Mean Nucleus Area**: 100-500 pixels² (varies by tissue type)
- **Processing Time**: 2-5 seconds per image

## 🤝 Contributing

Contributions are welcome! Potential improvements:

- Support for additional image formats (TIFF, JPEG)
- Deep learning-based segmentation (U-Net, Mask R-CNN)
- 3D volume segmentation for z-stack images
- Interactive parameter tuning GUI
- Additional cell property measurements (circularity, eccentricity)

## 📄 License

This project is provided as-is for educational and research purposes.

## 👤 Author

Developed as part of a computer vision and medical image analysis project.

## 🙏 Acknowledgments

- Dr. Toros Taşkın for the lecture material and dataset
- scikit-image documentation and examples
- OpenCV community tutorials
- Pathology biopsy image processing research papers

## 📞 Support

For questions or issues:
1. Check that all dependencies are correctly installed
2. Verify input images are RGB .png files
3. Ensure sufficient disk space for output files
4. Review the Jupyter notebook output for error messages

---

**Note**: This tool is designed for research and educational purposes. For clinical applications, please consult with medical professionals and follow appropriate validation procedures.