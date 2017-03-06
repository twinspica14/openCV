# openCV
#Most of the codes are modified for learning and For my projects. Tested on CV 2.9 && 2.10 && 2.11
#using ip camera or any portable phone as camera

RaspberryPi is mostly cheap and good for nothing single board computer compared to its counterpart which are costly.

In our idea of implementation of locomative robot.at first we had no idea about locomation or any algorithm.Myself (NileshPandey) have worked on opencv from basic to advanced programming by my own,so Opencv wasn't problem. Problem is location of robot,It should be fast,Fast processing.
Using RPI which process image pretty slow,we came with idea of cloud computing here.
At first our approach was using webcam on rpi which will send data i.e. video to pc,which will return with result 

Our appraoch ,1st we could use VNC or RDP,

http://www.instructables.com/id/Raspberry-Pi-remote-webcam/

this process is fair and simple,it does what its name says,but problem is  we were unable to get data from this. For instance we will say it was our ####app1



Our approach 2,
http://www.instructables.com/id/The-RROP-RaspRobot-OpenCV-Project/?ALLSTEPS

so here motion isn't used instead,VLC or mplayer is used,with this we can get data.
Important thing to note here was  https://www.raspberrypi.org/forums/viewtopic.php?t=48597
the argument mentioned in above site about installing  mjpeg-streamer. This is most  imporatnt and crucial part. If this part fails,nothing is going work,Lets discuss why this method failed in our case.

The web came being used is QHMPL PC WEBCAM,

In above link,we first tried the topmost argument,it didn't worked.

MJPG Streamer Version: svn rev: 3:160M
i: Using V4L2 device.: /dev/video0
i: Desired Resolution: 640 x 480
i: Frames Per Second.: 5
i: Format…………: MJPEG
ERROR opening V4L interface: Is a directory
Init v4L2 failed !! exit fatal
i: init_VideoIn failed

if this error appears  after following instruction from https://wolfpaulus.com/journal/embedded/raspberrypi_webcam/

chances are webcam doesn't supports mjpg streamer!

http://jacobsalmela.com/raspberry-pi-webcam-using-mjpg-streamer-over-internet/

we again tried,it again failed to open webcam.
We went to 3rd approach, using netcat

net cat is mostly use in survelliance cameras,so we thought why not give a try,we could see it didn't required mjpeg streamer.
Again it failed,so lets see!
http://zacharybears.com/low-latency-raspberry-pi-video-streaming/

this is entirely to view from ubuntu,but i was on windows,

http://www.recantha.co.uk/blog/?p=4128    this one is for windows,we needed to edit folder and some words mentioned on site.

It was fruitless too. Before i go further,its important to note,( raspivid) is only for rpi cameras not for usb cameras. fswebcam 
is used for usb cameras,don't forget to replace raspivid with fswebcam.

After doing all this approach,we had no option but to optimize the opencv code.
I don't have link from where i downloaded the code,only possible way to speed up processing was to reduce captured frame size from HD to 160x240,with number of fps 5,it worked with great success but still there was lag,and reducing frame size ,made the program less optimum.

		resize(camera,camera,Size(160,240),0,0,INTER_LINEAR );

this is extra line added,its funny but its obvious .Most times being on pc,i don't prefer with above change,so without giving up i again started to try for cloud computing ,this time i had success .

We switched from USB camera to andoid /iphone camera,
1.Ip Camera app available on both android /iphone.
It is easy to setup
2. Open vlc under network streaming, type the address as http://192.168.0.2:8080/video

this will let you stream from android cam to vlc media player.
3.Using this data with Opencv C++ as well as Python code
It took much time to search but finally found ways,in Python its straight method 

example:-

import cv2
import urllib 
import numpy as np

stream=urllib.urlopen('http://192.168.0.2:8080/video')
bytes=''
while True:
    bytes+=stream.read(1024)
    a = bytes.find('\xff\xd8')
    b = bytes.find('\xff\xd9')
    if a!=-1 and b!=-1:
        jpg = bytes[a:b+2]
        bytes= bytes[b+2:]
        i = cv2.imdecode(np.fromstring(jpg, dtype=np.uint8),cv2.CV_LOAD_IMAGE_COLOR)
        cv2.imshow('i',i)
        if cv2.waitKey(1) ==27:
            exit(0)   

reffered from http://petrkout.com/electronics/low-latency-0-4-s-video-streaming-from-raspberry-pi-mjpeg-streamer-opencv/

 The problem i was facing here was,i never programmed in one language,i keep switching between Python and C++.

C++ doesn't have any library like “urlib”
But it does have it's alternative like “libcurl” it didn't worked for me,may be installation failed but basically i wasn't able to reslove problem,unless i found solution on 1 site,
http://answers.opencv.org/question/3664/ip-network-camera-access/


here is my code:-

#include <opencv2/opencv.hpp>
#include <opencv2/highgui/highgui.hpp>
#include <opencv2/imgproc/imgproc.hpp>
#include <iostream>
#include <stdio.h>

using namespace std;
using namespace cv;

int main()
{
    Mat frame;
    namedWindow("video", 1);


VideoCapture cap;
cap.open("http://192.168.0.2:8080/video?x.mjpg");

    while ( 1)
    {
        cap >> frame;
        if(frame.empty()) break;

        imshow("video", frame);
        waitKey(5);
    }
    return 0;
}


SUCCESS!!!!!.

We have Mat object in both above cases on which we can perform opencv operations.
