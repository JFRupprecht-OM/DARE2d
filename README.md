<div align="center">

# DARE2D: Division Axis Recognition in 2D

<a href="https://www.tensorflow.org/"><img alt="TensorFlow" src="https://img.shields.io/badge/TensorFlow-FF6F00?logo=tensorflow&logoColor=white"></a> <a href="https://keras.io/"><img alt="Keras" src="https://img.shields.io/badge/Keras-D00000?logo=keras&logoColor=white"></a> <a href="https://hydra.cc/"><img alt="Config: Hydra" src="https://img.shields.io/badge/Config-Hydra-89b8cd"></a>

</div>

---

## Overview

**DARE2D** is a deep learning framework for detecting cell divisions in 2D time-lapse microscopy images and estimating their key attributes. This Python implementation leverages TensorFlow/Keras and Hydra for configuration management to provide a flexible, reproducible pipeline for:

1. **Segmentation**: Detect the center (barycenter) of cell divisions
2. **Regression**: Estimate division attributes such as:
   - Orientation angle
   - Division axis length
   - Other morphological parameters

The framework supports **inference on new data** using pre-trained model ensembles with a multi-model consensus approach for robust detection.

---

## Quick Start

The easiest way to get started is with the **included Jupyter notebook** and pre-trained model checkpoints:

```bash
# 1. Clone the repository
git clone https://github.com/qazi05/DARE2d
cd DARE2d

# 2. Create and activate conda environment
conda create -n dare2d python=3.9
conda activate dare2d

# 3. Install dependencies
pip install -r requirements.txt

# 4. Install DARE2D in development mode
pip install -e .

# 5. Run the prediction notebook
# Open and run: main2d.ipynb
```

> **Note**: Always activate the `dare2d` environment before running any scripts or notebooks.

---

## Installation

### Prerequisites
- Python 3.9
- TensorFlow 2.x (GPU recommended but CPU supported)
- Conda (recommended)

### Step-by-Step Installation

```bash
# 1. Clone the repository
git clone <repository-url>
cd DARE2d_main

# 2. Initialize your shell for conda (optional but recommended)
conda init powershell  # Windows PowerShell
# OR
conda init bash        # Linux/macOS

# 3. Create a fresh conda environment
conda create -n dare2d python=3.9
conda activate dare2d
```

#### Install Dependencies

```bash
# Install TensorFlow and other dependencies
pip install -r requirements.txt

# Install DARE2D in development mode
pip install -e .
```

---

## Project Structure

```
dare2d/                          # Main package
├── __init__.py
├── io.py                        # Input/output utilities
├── typing.py                    # Type definitions
│
├── callbacks/                   # Training callbacks
│   ├── display_callback.py
│   ├── regression_callback.py
│   ├── savelast_callback.py
│   └── tap2d_callback.py
│
├── datamodule/                  # Data loading and preprocessing
│   ├── datamodule.py           # Base datamodule
│   ├── regression2d.py         # Regression data
│   ├── segmentation2d.py       # Segmentation data
│   ├── tap2d.py                # Multi-task data
│   ├── augmentation/           # Data augmentation
│   └── generator/              # Data generators
│
├── evaluation/                  # Evaluation metrics and utilities
│
├── losses/                      # Loss functions
│
├── model/                       # Neural network architectures
│
├── prediction/                  # Inference utilities
│
└── trainer/                     # Training utilities

config/                          # Hydra configuration files
├── train.yaml                  # Training config template
├── datamodule/                 # Data config
├── model/                      # Model architectures config
│   ├── losses/                 # Loss functions config
│   ├── metrics/                # Metrics config
│   └── optimizer/              # Optimizer config
├── trainer/                    # Trainer config
├── callbacks/                  # Callback config
├── experiment/                 # Experiment presets
└── hparams_search/             # Hyperparameter search configs

scripts/                        # Utility scripts
├── all_model_inference.py     # Batch inference runner
├── inference/                  # Inference scripts
│   └── multistage_detection2d.py
├── postprocessing/             # Result postprocessing
├── evaluation/                 # Evaluation scripts
├── train/                      # Training scripts
└── tools/                      # Utility tools

notebooks/                      # Jupyter notebooks
├── main2d.ipynb               # Main prediction notebook (START HERE!)
├── data_analysis.ipynb        # Data exploration
└── train_data_display.ipynb   # Training data visualization

input/                          # Input data directory (you'll populate this)
                                # Place your .tif image stacks here

output/                         # Inference outputs (auto-generated)
├── {image_name}_1/            # Results from model set 1
├── {image_name}_2/            # Results from model set 2
└── ...                        # Results from model sets 3-8

regression_checkpoints/         # Regression model weights
├── checkpoints_set_1_all_but_target/
│   └── best.h5
├── checkpoints_set_2_all_but_target/
└── ...                        # Sets 3-8

segmentation_checkpoints/       # Segmentation model weights
├── checkpoints_set_1_all_but_target/
│   └── best.h5
├── checkpoints_set_2_all_but_target/
└── ...                        # Sets 3-8

postprocessed_results/          # Consensus outputs (auto-generated)
└── {image_name}/
    ├── chosen_divisions/       # Final consensus detections
    ├── post_processed_{image_name}.tiff
    └── post_processed_{image_name}_summary.csv

requirements.txt               # Python dependencies
setup.py                       # Package setup configuration
README.md                      # This file
README.backup.md              # Original README (for reference)
```

---

## Model Checkpoints

DARE2D uses an **ensemble of 8 model sets** for robust predictions:

- `regression_checkpoints/checkpoints_set_1_all_but_target/` through `checkpoints_set_8_all_but_target/`
- `segmentation_checkpoints/checkpoints_set_1_all_but_target/` through `checkpoints_set_8_all_but_target/`

Each set contains a `best.h5` file with the trained model weights. The ensemble approach improves detection reliability through multi-model consensus.

---

## Data Format

### Input Images
- **Format**: TIFF stacks (.tif)
- **Dimensions**: (T, Y, X) where T is time, Y/X are spatial dimensions
- **Type**: 8-bit or 16-bit grayscale images
- **Location**: Place input files in the `input/` directory

### Output Structure

After running inference, results are organized as:

```
output/
└── {image_name}_1/             # Results from model set 1
    └── {image_name}/
        ├── division_position1.npy   # Raw detections per frame
        └── {image_name}_result.tiff # Visualization

postprocessed_results/
└── {image_name}/
    ├── chosen_divisions/        # Final consensus detections
    ├── post_processed_{image_name}.tiff
    └── post_processed_{image_name}_summary.csv  # Statistics
```

---

## Inference (Prediction)

### Quickest Path: Use the Notebook

The easiest way to run inference is the **Jupyter notebook**:

```bash
conda activate dare2d
jupyter notebook main2d.ipynb
```

This notebook:
- Guides you through path setup
- Runs inference with all 8 model sets
- Generates consensus detections
- Produces visualization outputs

### Batch Inference (Command-Line)

For processing multiple images programmatically:

**Script**: `scripts/all_model_inference.py`

#### Setup

1. Edit the script and set `BASE_DIR` to your project root:
   ```python
   BASE_DIR = r"C:\Users\YourName\Documents\DARE2d"
   ```

2. Ensure model checkpoints exist in:
   - `regression_checkpoints/checkpoints_set_1_all_but_target/` through `checkpoints_set_8_all_but_target/`
   - `segmentation_checkpoints/checkpoints_set_1_all_but_target/` through `checkpoints_set_8_all_but_target/`

3. Place input `.tif` files in `input/`

#### Run Batch Inference

```powershell
conda activate dare2d
python scripts/all_model_inference.py
```

The script will:
- Automatically discover `.tif` files in `input/`
- Process each image with all 8 model sets
- Save results under `output/{image_name}_{n}/`
- Generate comprehensive logs

### Single Image Inference

For processing a single image with a specific model set:

```powershell
conda activate dare2d
python -m scripts.inference.multistage_detection2d \
  --regression regression_checkpoints/checkpoints_set_1_all_but_target/best.h5 \
  --segmentation segmentation_checkpoints/checkpoints_set_1_all_but_target/best.h5 \
  --img input/my_stack.tif \
  --output output/my_stack
```

**Parameters**:
- `--regression`: Path to regression model checkpoint
- `--segmentation`: Path to segmentation model checkpoint
- `--img`: Path to input TIFF stack
- `--output`: Output directory for results

---

## Postprocessing & Consensus


After inference, results from multiple models are aggregated to generate consensus detections:

### Consensus Generation

The postprocessing pipeline:
1. **Spatial Clustering**: Groups detections based on spatial proximity, taking into account the size of each cell. This ensures that clustering reflects the actual cell dimensions, so detections within the same cell area are grouped together.
2. **Temporal De-duplication**: Within each spatial cluster, duplicate detections across frames are removed to ensure that a single cell is not detected multiple times in the same area.
3. **Consensus Calculation**: Computes median positions, angles, and uncertainty statistics for each cell cluster.

### Parameters

Key postprocessing parameters (configurable in notebook or scripts):
- `eps = 10`: Spatial clustering distance threshold (pixels), typically set based on cell size
- `min_models = 6`: Minimum number of models required for consensus
- `num_models = 8`: Total number of models in ensemble
- `angle_mode = "auto"`: Angle selection strategy

### Running Postprocessing

Postprocessing is integrated in the notebook workflow. For standalone use:

```powershell
python scripts/postprocessing/main.py \
  --output_root output \
  --image_name my_image \
  --image_stack input/my_image.tif \
  --save_dir postprocessed_results/my_image \
  --eps 10 \
  --min_models 6 \
  --num_models 8
```

---

## License & Attribution

This work was granted access to the HPC resources of IDRIS under the allocation AD010314339 made by GENCI.

Authors: Romain Karpinski, Marc Karnat, Alice Gros, Qazi Saaheelur Rahaman, Jules Vanaret, Mehdi Saadaoui, Sham Tlili, and Jean-Francois Rupprecht

---

## Troubleshooting

### Common Issues

**Missing Checkpoints**
- Verify that all 8 model sets exist in both `regression_checkpoints/` and `segmentation_checkpoints/`
- Check that each set contains a `best.h5` file

**Import Errors**
- Ensure `conda activate dare2d` is active
- Verify all dependencies: `pip install -r requirements.txt`

**Path Errors**
- Update `BASE_DIR` in `scripts/all_model_inference.py` to match your project location
- Use absolute paths when possible

**Memory Errors**
- Reduce batch sizes in configuration files
- Process fewer images simultaneously
- Consider using CPU-only mode for large images

**No Input Images Found**
- Verify `.tif` files are in the `input/` directory
- Check file extensions (must be `.tif`, not `.tiff`)

### Performance Tips

- Use GPU for faster inference (TensorFlow GPU support)
- Process images in smaller batches for memory management
- Use SSD storage for faster I/O operations

---

## Related Projects

- **DARE3D**: 3D version of this framework for volumetric data
  - GitHub: https://github.com/JFRupprecht-OM/DARE3d/tree/main

---

**Questions?** Check the demo notebook or open an issue on the repository.

