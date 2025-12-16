<div align="center">

# dare2d on TensorFlow

<a href="https://www.tensorflow.org/"><img alt="TensorFlow" src="https://img.shields.io/badge/TensorFlow-FF6F00?logo=tensorflow&logoColor=white"></a> <a href="https://keras.io/"><img alt="Keras" src="https://img.shields.io/badge/Keras-D00000?logo=keras&logoColor=white"></a> <a href="https://hydra.cc/"><img alt="Config: Hydra" src="https://img.shields.io/badge/Config-Hydra-89b8cd"></a> <a href="https://scikit-learn.org/"><img alt="scikit-learn" src="https://img.shields.io/badge/scikit--learn-F7931E?logo=scikit-learn&logoColor=white"></a><br>

</div>

## Description

**D**ivision **A**xis **RE**cognition in **2D** tissues

This repository contains a complete Python implementation for detecting cell divisions and their attributes (position, angle, and length) in 2D time-lapse microscopy images using a multi-model consensus framework.

## Overview

The DARE2D framework implements a robust two-stage pipeline for cell division detection:

1. **Multi-Model Inference**: Semantic segmentation to detect division centers using multiple trained models
2. **Consensus Generation**: Spatial clustering and temporal deduplication to create reliable consensus detections
3. **Quality Assessment**: Statistical analysis and uncertainty quantification of detection results

The pipeline processes 2D+t image stacks (time series of 2D images) by analyzing triplets of consecutive frames to capture temporal context for accurate division detection.

## Key Features

- **Multi-Model Consensus**: Combines predictions from multiple trained models for improved robustness
- **Uncertainty Quantification**: Provides confidence scores and uncertainty measures for each detection
- **Batch Processing**: Automated pipeline for processing multiple images with comprehensive result organization
- **Quality Metrics**: Statistical analysis and visualization of detection reliability
- **Jupyter Notebook Interface**: User-friendly notebook for complete pipeline execution

## Installation

### Prerequisites
- Python 3.8+
- Conda (recommended for environment management)

### Setup Steps

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd dare2d
   ```

2. **Create conda environment**
   ```bash
   conda create --name dare2d python=3.9
   conda activate dare2d
   ```

3. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

4. **Install the package**
   ```bash
   pip install -e .
   ```

## Quick Start

### Batch Inference Pipeline

The easiest way to run the complete pipeline is using the provided Jupyter notebook:

```bash
jupyter notebook main2d.ipynb
```

This notebook provides a complete workflow:
1. Setup and path configuration
2. Multi-model inference on multiple images
3. Consensus generation and postprocessing
4. Quality analysis and visualization

### Command Line Inference

For single image processing, use the inference script:

```bash
python scripts/inference/multistage_detection2d.py \
    --regression=<path-to-regression-weights> \
    --segmentation=<path-to-segmentation-weights> \
    --img=<path-to-tiff-stack> \
    --output=<output-directory>
```

### Batch Processing

For processing multiple images with multiple models:

```bash
python scripts/all_model_inference.py
```

This script automatically discovers images and runs inference with 8 different model pairs, followed by postprocessing for consensus results.

## Pipeline Details

### Stage 1: Multi-Model Inference

The pipeline processes each input image with multiple trained model pairs (segmentation + regression). Each model processes 3-frame windows [t-1, t, t+1] to capture temporal context.

**Segmentation Model**: Identifies potential division centers using U-Net architecture
**Regression Model**: Predicts division angle and length for each detected center

### Stage 2: Consensus Generation

Raw detections from multiple models are aggregated using:
- **Spatial Clustering**: HDBSCAN/DBSCAN to group nearby detections
- **Temporal Deduplication**: Removes duplicate detections across consecutive frames
- **Consensus Calculation**: Median positions, chosen angles, uncertainty statistics

### Stage 3: Quality Assessment

Postprocessing generates:
- Distribution plots for detection metrics
- Quality scores based on model agreement
- Uncertainty visualizations
- Comprehensive statistics CSV

## Output Structure

```
project_root/
├── output/                          # Raw inference results
│   ├── {image_name}_1/{image_name}/
│   │   ├── division_position1.npy   # Raw detections per frame
│   │   └── {image_name}_result.tiff # Inference visualization
│   └── {image_name}_2/...           # Results for each model
├── postprocessed_results/           # Consensus results
│   └── {image_name}/
│       ├── chosen_divisions/        # Final consensus detections
│       ├── post_processed_{image_name}.tiff  # Consensus visualization
│       └── post_processed_{image_name}_summary.csv  # Statistics
└── analysis_plots/                  # Quality analysis
    └── {image_name}/
        ├── distribution_plots.png
        ├── quality_scores.png
        └── metrics.csv
```

## Training

### Available Experiments

Train individual components using Hydra configuration:

```bash
# Segmentation model for division center detection
python scripts/train.py experiment=segmentation2d

# Regression model for angle/length prediction
python scripts/train.py experiment=regression2d

# TAP (Task Affinity Prediction) model
python scripts/train.py experiment=tap2d
```

### Configuration

Experiment configurations are located in `config/experiment/`. Modify parameters in the corresponding YAML files to customize training.

## Data Format

### Input Images
- **Format**: TIFF stacks (.tif)
- **Dimensions**: (T, Y, X) where T is time, Y/X are spatial dimensions
- **Type**: 8-bit or 16-bit grayscale images

### Ground Truth (for training)
- Division centers: (x, y) coordinates
- Division attributes: angle (degrees/radians), length (pixels)

## HPC Usage (Jean Zay)

### Build Singularity Image
```bash
sudo singularity build --nv dare2d.sif singularity.def
```

### Run on Jean Zay
```bash
# Reserve GPU node
srun --pty --ntasks=1 --gres=gpu:1 --hint=nomultithread bash

# Load singularity
module load singularity

# Run container with data binding
singularity shell --nv --bind $WORK/dare2d:/dare2d --bind $ALL_CCFRWORK/data:/dare2d/data $SINGULARITY_ALLOWED_DIR/dare2d.sif

# Install package and run
cd /dare2d && pip install -e .
python scripts/train.py experiment=segmentation2d
```

`scp dare2d.sif <user>@jean-zay.idris.fr:<path to $ALL_CCFRWORK>`

After that you can login on JeanZay and register the image
`idrcontmgr cp dare2d.sif`
You can list the registered images with
`idrcontmgr ls`
And remove and old one with
`idrcontmgr rm dare2d.sif`

### Run the container on JZ

**First do a reservation** no time specified = 10 minutes
`srun --pty --ntasks=1 --gres=gpu:1 --hint=nomultithread bash`

Load singularity
`module load singularity`

With a registered singularity image

`singularity shell --nv $SINGULARITY_ALLOWED_DIR/dare2d.sif`

However you won't have acces to the data nor the source code.
Assuming the source code is located in `$WORK/dare2d` and the data in `$ALL_CCFRWORK/data` you can bind these folders to the singularity image as follow:
`singularity shell --nv --bind $WORK/dare2d:/dare2d --bind $ALL_CCFRWORK/data:/dare2d/data $SINGULARITY_ALLOWED_DIR/dare2d.sif`

Once you have the shell you can check that everything is in order in `/dare2d`

Install the dare2d package locally: `cd /dare2d && pip install -e .`

You can now run your experiment

### Troubleshoot

If you have issue with /tmp `no space left on device` you can set `SINGULARITY_TMPDIR` and/or `SINGULARITY_CACHEDIR` to a different folder:

```
sudo SINGULARITY_TMPDIR=<path/tmp> singularity build --nv dare2d.sif singularity.def
```

## Troubleshooting

### Common Issues

**Memory Errors**: Reduce batch sizes or process fewer images simultaneously
**Missing Dependencies**: Ensure all packages in `requirements.txt` are installed
**CUDA Errors**: Check GPU memory and driver compatibility
**Singularity /tmp Issues**: Set `SINGULARITY_TMPDIR` to a different location

### Performance Tips

- Use SSD storage for faster I/O operations
- Process images in smaller batches for memory management
- Monitor GPU utilization during training/inference

## Citation

If you use DARE2D in your research, please cite:

```
[Your Name et al. "DARE2D: Multi-Model Consensus Framework for Robust 2D Cell Division Detection." Journal Name, Year.]
```

## License

[Specify license information]

## Contributing

[Guidelines for contributing to the project]
