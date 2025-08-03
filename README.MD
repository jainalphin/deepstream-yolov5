# DeepStream YOLOv5 in Docker

This guide explains how to build and run a Docker container for NVIDIA DeepStream with YOLOv5 integration.

## **1. Build the Docker Image**

```bash
docker build -t deep_stream .
```

## **2. Run the Docker Container**

First, allow Docker to display on your host system:

```bash
xhost +
```

Then, run the container:

```bash
docker run --gpus all -it --rm \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -e DISPLAY=$DISPLAY \
    -v ~/Bhavika_Kanani/SoftmaxAI/Alphin/DeepStream-SDK-Docker/deep_stream5:/opt/nvidia/deepstream/deepstream/sources/project \
    -w /opt/nvidia/deepstream/deepstream/sources/project \
    deepstream-app3
```

## **3. Inside the Docker Container**

### **Step 1: Clone Required Repositories**

```bash
git clone https://github.com/marcoslucianops/DeepStream-Yolo
git clone https://github.com/ultralytics/yolov5
```

### **Step 2: Prepare YOLOv5 for WTS/CFG Generation**

1. Copy the weight conversion script:

   ```bash
   cp DeepStream-Yolo/utils/gen_wts_yoloV5.py yolov5
   ```

2. Navigate to YOLOv5:

   ```bash
   cd yolov5
   ```

3. **Modify requirements.txt**
   Comment out the following lines:

   ```
   torch
   tensorflow
   pillow
   opencv
   thop
   ```

4. Install dependencies:

   ```bash
   pip3 install -r requirements.txt
   ```

5. Generate `.wts` and `.cfg` files from `.pt` and `.yaml`:

   ```bash
   python3 gen_wts_yoloV5.py -w yolov5s.pt -c yolov5s.yaml
   ```

6. Copy the generated files to DeepStream-Yolo:

   ```bash
   cp yolov5/yolov5s.wts DeepStream-Yolo
   cp yolov5/yolov5s.cfg DeepStream-Yolo
   ```

> **Note:** Make sure you update the number of classes in your `.yaml` file (e.g., set `nc: 2` if you have two classes).

### **Step 3: Build YOLO Plugin for DeepStream**

Inside `DeepStream-Yolo`:

```bash
CUDA_VER=11.4 make -C nvdsinfer_custom_impl_Yolo
```

### **Step 4: Configure DeepStream**

1. Edit **`config_infer_primary_yoloV5.txt`** for your YOLOv5 model (e.g., yolov5s).

2. Update **`deepstream_app_config.txt`** to match your setup.


### **Step 5: Run DeepStream**

```bash
deepstream-app -c deepstream_app_config.txt
```
