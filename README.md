# Fast Tracking-by-Detection
This repository is the fast tracking-by-detection module in ADAS (Advanced Driver Assistance System).   

<p align="center">
<img src="https://user-images.githubusercontent.com/37034031/49984460-89baa080-ffab-11e8-8b5c-50007524d5bd.png" width=800>
</p>

## Pros of the Tracking-by-Detection
- Remove false detections
- Handle unstable detections
- Give object IDs
- Correct category of the object by voting accroding to time series
- Some post-processings are included in tracker (e.g. ignore too small objects and delete some overlapped boxes)  

## Requirements
- tensorflow 1.10.0
- opencv 3.3.1
- numpy 1.15.2
- filterpy 1.4.5
- sklearn 0.20.0

## Tracking-by-Detection Demo
Three videos are set by different intervals (1, 2, and 3) of the detector. Big interval value gives faster processing time but low accuracy. The interval is hyper-parameter, so it should be set by trading off between speed and accuracy.   

- Detector interval == 1.
<p align = 'center'>
  <a href = 'https://www.youtube.com/watch?v=EJkdIyk8JxY'>
    <img src = 'https://user-images.githubusercontent.com/37034031/49987504-d35cb880-ffb6-11e8-9ea2-4c7d5130c84b.gif'>
  </a>
</p>

- Detector interval == 2.
<p align = 'center'>
  <a href = 'https://www.youtube.com/watch?v=e1ig3GEzuJo&t=9s'>
    <img src = 'https://user-images.githubusercontent.com/37034031/49987601-2df61480-ffb7-11e8-9de9-0d43e16a2553.gif'>
  </a>
</p>

- Detector interval == 3.
<p align = 'center'>
  <a href = 'https://www.youtube.com/watch?v=Cinq8BE-eqY&feature=youtu.be'>
    <img src = 'https://user-images.githubusercontent.com/37034031/49987818-0489b880-ffb8-11e8-99bd-c4863f09e5e4.gif'>
  </a>
</p>  

**Note:** left is tracking-by-detection, right is detection only.  

- [Click to go to the full demo on YouTube! Tracking-by-Detection interval 1](https://www.youtube.com/watch?v=EJkdIyk8JxY)  
- [Click to go to the full demo on YouTube! Tracking-by-detection interval 2](https://www.youtube.com/watch?v=e1ig3GEzuJo&t=9s)  
- [Click to go to the full demo on YouTube! Tracking-by-detection nterval 3](https://www.youtube.com/watch?v=Cinq8BE-eqY&feature=youtu.be) 

In PC environment (CPU: Intel(R) Core(TM) i7-4770 CPU @ 3.40GHz, GPU: GTX 1080, RAM: 32GB)  

| interval | Detection only | Tracking-by-Detection |
|  :---:   |      :---:     |         :---:         |
|    1     |     40 FPS     |        37 FPS         |
|    2     |     79 FPS     |        70 FPS         |
|    3     |    119 FPS     |       102 FPS         |

## Implementation Details
- Detector uses Inception-SSD ([Single-Shot Multibox Detector](https://arxiv.org/pdf/1512.02325.pdf)) from the [Tensorflow Object Detection API](https://github.com/tensorflow/models/tree/master/research/object_detection). You can replace the detector by your own detector. Good detector means more powerful tracking-by-detection results.  

- Tracker uses Kalman filter with hungarian data-association algorithm.  

- The upper part of the input frame is sky, so we set RoI on bottom part, and detect objects in RoI region only. The input frame size is (720, 1280), RoI size is (480, 1280) and the input RoI in the pb file is automatically resied to (300, 300) to do forward-pass. You can't change the input placeholder size of the detector directly. The possible way for changing is to retrain the detector model by setting the different input size in the configuration file, and compile it.

## Documentation
### Directory Hierarchy
``` 
.
│   Fast-tracking-by-detection
│   ├── data (video data is not included in repository)
│   │   ├── video_01.avi 
│   ├── src
│   │   ├── cropper.py
│   │   ├── drawer.py
│   │   ├── main.py
│   │   ├── model.py
│   │   ├── tracker.py
│   │   │   ├── API
│   │   │   │   ├── drawer.py
│   │   │   │   ├── model.py
│   │   │   │   ├── recorder.py
│   │   │   │   ├── test_main.py
│   │   │   │   └── tracker.py
│   │   │   ├── models (detector pb file is not included in repository)
│   │   │   │    └── frozen_inference_graph.pb (compiled tensorflow pb file)
│   │   │   ├── utils
│   │   │   │   ├── readxml.py
│   │   │   │   └── video2frame.py
```  
**src**: source codes of the fast-tracking-by-detection  
**API**: the codes in `API` is the **naive version of the tracking-by-detection**. All of the following demo videos are used naive version. And the advanced version is under progress.

### Test Tracking-by-detection  
Run `test_main.py` in the `API` folder.  

```
python test_main.py --data=/give/adress/of/the/input/video
```  
- `--gpu_index`: gpu index, default: `0`  
- `--data`: test video path, default: `../data/video_01.avi`  
- `--interval`: detector interval between tracker, default: `2`  
- `--delay`: interval between two frames when showing, default: `1`  
- `--is_record`: recording video, default: `False`  

### Replace Your own Detector
Please refer to the [Tensorflow Object Detection API](https://github.com/tensorflow/models/tree/master/research/object_detection) tutorial to train your own model on your dataset. After training you should compile the model to the `pb` file and replace the `frozen_inference_graph.pb` in the folder `models`.  

For our Inception-SSD pb file, you can download from the [link](https://www.dropbox.com/sh/76pr9usl8jyq6ni/AAARnYKFI-zqaK0TzGSE6fNMa?dl=0). Make a new folder `models` in the correct place (check the above `Directory Hierarchy` structure) and copy the `pb` file in the folder.

You also need to consider about the `labels` of the object. Each model the label indexs are different. In our model, index ordering is "0: Background", "1: Car", "2: Pedestrain", "3: Cyclist", "4: Motorcyclist", "5: Truck", "6: Bus", and "7: Van". Refer to the file `drawer.py` in the `API` folder.  
(**Note:** `drawer.py` in `API` and `drawer` in `src` folder are different. You need to revise `drawer.py` in `API` folder. `drawer.py` in `src` is for advanced version of the tracking-by-detection)

### TO DO List
- Naive version of the tracking-by-detection (Finished)
- Advanced version of the tracking-by-detection (Under Progress)

### Citation
```
  @misc{chengbinjin2018fasttrackingbydetection,
    author = {Cheng-Bin Jin},
    title = {Fast-Tracking-by-Detection},
    year = {2018},
    howpublished = {\url{https://github.com/ChengBinJin/Fast-Tracking-by-Detection}},
    note = {commit xxxxxxx}
  }
```  

## License
Copyright (c) 2018 Cheng-Bin Jin. Contact me for commercial use (or rather any use that is not academic research) (email: sbkim0407@gmail.com). Free for research use, as long as proper attribution is given and this copyright notice is retained.
