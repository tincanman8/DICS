# DICS: Driver-Intersection Caution System
###### For the Jetson TX2

[Install JetPack](https://developer.nvidia.com/embedded/jetpack) on the Jetson TX2. [Install OpenCV](https://www.hackster.io/wilson-wang/jetson-tx2-tensorflow-opencv-keras-install-b74e40) (3.4.2 or later). Note, due to the ARM architecture of the Cortex CPUs on-board the Jetson TX2, you will not be able to use pip to install OpenCV. You must follow the instructions in the link above.

If any other dependencies are needed, install them as they are requested.

Once you have done that, clone this repository and download the model weights and test videos:
```
>>> git clone https://github.com/tincanman8/DICS.git
>>> cd DICS
>>> make
>>> wget --no-check-certificate 'https://drive.google.com/uc?export=download&id=1QmcaRFGYadec-TSXpe8iVqCc0Oir55aM' -O DICS-model/DICS.weights
>>> wget --no-check-certificate 'https://drive.google.com/uc?export=download&id=1dKJPAjPlpfaJ1m_syuaKL4tCBsN-6PPj' -O input/test1.mp4
>>> wget --no-check-certificate 'https://drive.google.com/uc?export=download&id=1nMUq6vVCmXsSYykq4BakIAcq-3qCmnHu' -O input/test2.mp4
```


Before testing the scripts, you will want to set your Jetson to Max-N mode and overclock:
```
>>> sudo nvpmodel -m 0
>>> sudo ~/jetson_clocks.sh
```
This will cause the fan to turn on as well, to cool the Jetson.


To test real-time detection on a video (as in, process frame by frame and output to screen immediately), run the following command:
```
>>> ./darknet detector demo DICS-model/DICS.data DICS-model/DICS.cfg DICS-model/DICS.weights input/test1.mp4 -thresh 0.85
```
Replace 'test1.mp4' in the command above with the path to a video of your choice, or leave as is to test functionality on a provided test video from YouTube.


To analyze an entire video and write to an output video file, run the following command:
```
>>> python3 yolo_video.py --input input/test1.mp4 --output output/test1_output.mp4 --yolo DICS-model
```


To test operation of the object detector on the Jetson TX2 camera, open a new terminal and run the following:
```
>>> gst-launch-1.0 nvcamerasrc ! 'video/x-raw(memory:NVMM), width=(int)1920, height=(int)1080, format=(string)I420, framerate=(fraction)30/1' ! nvtee ! nvvidconv ! 'video/x-raw, format=(string)I420, framerate=(fraction)30/1' ! tee ! v4l2sink device=/dev/video2
```
This forwards the frame stream from the camera (YUV format) from /dev/video0 to /dev/video2 (RBG format) at 30 fps. It is a process that keeps running until you Ctrl+C out of it.
Now finally, back in the original terminal run:
```
>>> python3 yolo_camera.py
```
or
```
./darknet detector demo DICs-model/DICS.data DICS-model/DICS.cfg DICS-model/DICS.weights -c 2
```
Use your phone or a printed out image of a stop sign or traffic light and show it to the camera to see if it picks it up!
It is likely that the performance of a camera-based trial of the algorithm will not perform well due to the nature of the training data used (field images from a dashcam).
