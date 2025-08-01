# Electrical Symbol Detection with YOLO OBB

This project trains a YOLO v11 OBB (Oriented Bounding Box) model to detect electrical symbols in circuit diagrams. The model can identify 4 types of electrical components: Transformers, Circuit Breakers, Switches, and MV Lines.

## Project Overview

- **Model**: YOLO v11 OBB (Oriented Bounding Box)
- **Classes**: 4 electrical symbols
- **Input**: Circuit diagram images (1024x1024)
- **Output**: Oriented bounding boxes with class predictions

## 📁 Project Structure
```
firstDataset/
├── README.md
├── data.yaml                 # Dataset configuration
├── best.pt                   # Trained model weights
├── process_image.py          # Image generation script
├── augment_images.py         # Image augmentation script
├── images/                   # Source images
│   ├── frame_0_delay-0.01s.jpg
│   ├── single-line-diagram-substation-two-parallel-transformers.png
│   └── substation-with-single-transformer.png
├── labels/                   # Source labels (YOLO OBB format)
│   └── train/
├── generated_images/         # Generated training images
├── generated_labels/         # Generated training labels
├── augmented_images/         # Augmented images
├── runs/                     # Training outputs
└── test/                     # Test images
```

##  Quick Start

### 1. Generate Training Data

```bash
# Generate 250 synthetic images with electrical symbols
python process_image.py
```

This creates:
- `generated_images/` - 250 synthetic circuit diagrams
- `generated_labels/` - Corresponding YOLO OBB labels

### 2. Organize Dataset

```bash
# Create proper YOLO directory structure
mkdir -p images/train images/val labels/train labels/val

# Split generated data 80/20 (train/val)
# Copy 200 images to train, 50 to val
# Copy corresponding labels
```

### 3. Train the Model

```bash
# Train YOLO OBB model
yolo obb train model=yolo11m-obb.pt data=data.yaml epochs=100 imgsz=1024 batch=8 project=runs/train name=electrical_obb_v1
```

### 4. Predict on New Images

```bash
# Predict on a single image
yolo obb predict model=runs/train/electrical_obb_v1/weights/best.pt source=test_image.png

# Predict on a directory
yolo obb predict model=runs/train/electrical_obb_v1/weights/best.pt source=test_images/

# Predict with confidence threshold
yolo obb predict model=runs/train/electrical_obb_v1/weights/best.pt source=test_image.png conf=0.5
```

##  Training Details

### Model Configuration
- **Model**: YOLO v11 Medium OBB (`yolo11m-obb.pt`)
- **Input Size**: 1024x1024 pixels
- **Batch Size**: 8 (adjust based on GPU memory)
- **Epochs**: 100

### Dataset Information
- **Training Images**: 200 synthetic circuit diagrams
- **Validation Images**: 50 synthetic circuit diagrams
- **Classes**: 4 electrical symbols
- **Label Format**: YOLO OBB (8 coordinates per detection)

### Class Mapping
```
0: Transformer
1: Circuit Breaker  
2: Switch
3: MV Line
```

## 🔧 Data Generation Process

### 1. Symbol Extraction
The `process_image.py` script:
- Extracts electrical symbols from source images
- Organizes symbols by class type
- Ensures even distribution across classes

### 2. Synthetic Image Generation
For each generated image:
- Randomly selects symbols from each class
- Places symbols with pathfinding connections
- Adds circuit lines between symbols
- Generates YOLO OBB labels

### 3. Data Augmentation
The `augment_images.py` script:
- Applies random transformations
- Creates additional training samples
- Maintains label consistency

##  Training Metrics

The model is evaluated using:
- **mAP50-95**: Primary metric for model selection
- **mAP50**: Mean Average Precision at IoU=0.50
- **Precision**: Detection precision
- **Recall**: Detection recall

##  Prediction Results

After training, the model can:
- Detect electrical symbols in circuit diagrams
- Predict oriented bounding boxes
- Provide confidence scores
- Handle rotated and scaled symbols

## 📁 Output Files

### Training Outputs
- `runs/train/electrical_obb_v1/weights/best.pt` - Best model weights
- `runs/train/electrical_obb_v1/weights/last.pt` - Last epoch weights
- `runs/train/electrical_obb_v1/results.png` - Training curves
- `runs/train/electrical_obb_v1/confusion_matrix.png` - Confusion matrix

## 🛠️ Customization

### Adjust Training Parameters
```bash
# Different model sizes
yolo train model=yolo11n-obb.pt data=data.yaml  # Nano (fastest)
yolo train model=yolo11s-obb.pt data=data.yaml  # Small
yolo train model=yolo11m-obb.pt data=data.yaml  # Medium
yolo train model=yolo11l-obb.pt data=data.yaml  # Large
yolo train model=yolo11x-obb.pt data=data.yaml  # Extra Large

# Different training parameters
yolo train model=yolo11m-obb.pt data=data.yaml epochs=100 imgsz=640 batch=16 patience=50
```

### Modify Data Generation
Edit `process_image.py`:
- Change `num_images_to_generate` for different dataset sizes
- Adjust symbol placement parameters
- Modify line drawing settings

## 🔍 Troubleshooting

### Common Issues

1. **Out of Memory**
   ```bash
   # Reduce batch size
   yolo train model=yolo11m-obb.pt data=data.yaml batch=4
   ```

2. **Slow Training**
   ```bash
   # Use smaller model
   yolo train model=yolo11n-obb.pt data=data.yaml
   ```

3. **Poor Performance**
   - Increase dataset size
   - Add more data augmentation
   - Train for more epochs

4. **Cache Issues**
   ```bash
   # Clear validation cache
   rm runs/val/*/val.cache
   ```

## 📝 Requirements

- Python 3.8+
- PyTorch
- Ultralytics YOLO
- OpenCV
- NumPy

##  Contributing

Feel free to submit issues and enhancement requests!