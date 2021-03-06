---
id: 522
title: Python Live Video Streaming Example
date: 2014-12-21T08:52:31+00:00
author: lo
layout: post
guid: /?p=522
permalink: /python-live-video-streaming-example/
dsq_thread_id:
  - 4752580561
categories:
  - Python
  - Virtual Reality
tags:
  - Code
  - MJPEG
  - Python
  - Video Streaming
---
Miguel Grinberg has written an excellent video streaming tutorial in Python <a href="http://blog.miguelgrinberg.com/post/video-streaming-with-flask" target="_blank">here</a>. I highly recommend it.

In short, you stream live video to clients using <a href="http://en.wikipedia.org/wiki/Motion_JPEG" target="_blank">Motion JPEG</a>, which just sends JPEG frames successively.

I modified the example code slightly to enable video streaming from a webcam using OpenCV. OpenCV uses VideoCapture returns raw images bytes which is not JPEG, so you need to do an extra step of encoding the image bytes to JPEG, then everything will work.

<pre># camera.py

import cv2

class VideoCamera(object):
    def __init__(self):
        # Using OpenCV to capture from device 0. If you have trouble capturing
        # from a webcam, comment the line below out and use a video file
        # instead.
        self.video = cv2.VideoCapture(0)
        # If you decide to use video.mp4, you must have this file in the folder
        # as the main.py.
        # self.video = cv2.VideoCapture('video.mp4')
    
    def __del__(self):
        self.video.release()
    
    def get_frame(self):
        success, image = self.video.read()
        # We are using Motion JPEG, but OpenCV defaults to capture raw images,
        # so we must encode it into JPEG in order to correctly display the
        # video stream.
        ret, jpeg = cv2.imencode('.jpg', image)
        return jpeg.tobytes()</pre>

<pre># main.py

from flask import Flask, render_template, Response
from camera import VideoCamera

app = Flask(__name__)

@app.route('/')
def index():
    return render_template('index.html')

def gen(camera):
    while True:
        frame = camera.get_frame()
        yield (b'--frame\r\n'
               b'Content-Type: image/jpeg\r\n\r\n' + frame + b'\r\n\r\n')

@app.route('/video_feed')
def video_feed():
    return Response(gen(VideoCamera()),
                    mimetype='multipart/x-mixed-replace; boundary=frame')

if __name__ == '__main__':
    app.run(host='0.0.0.0', debug=True)</pre>

The full Github code can be found <a href="https://github.com/log0/video_streaming_with_flask_example" target="_blank">here</a>.