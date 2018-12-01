Install OpenCV (3.4.2 or later), and CUDA support and cudNN. If on the Jetson TX2 (strongly recommended), this is not as simple as using a package installer, you may have to build from source. Install the GPU drivers.

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
>>> ./darknet detector demo DICs-model/DICS.data DICS-model/DICS.cfg DICS-model/DICS.weights input/test1.mp4
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
This forwards the frame stream from the camera (YUV format) from /dev/video0 to /dev/video2 (RBG format) at 30 fps.
Now finally run:
```
>>> python3 yolo_camera.py
```
or
```
./darknet detector demo DICs-model/DICS.data DICS-model/DICS.cfg DICS-model/DICS.weights -c 2
```
Use your phone or a printed out image of a stop sign or traffic light and show it to the camera to see if it picks it up!
It is likely that the performance of a camera-based trial of the algorithm will not perform well due to the nature of the training data used (field images from a dashcam).
