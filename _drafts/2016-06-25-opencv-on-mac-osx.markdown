---
layout: post
comments: true
title:  "OpenCV on Mac OSX"
excerpt: "Lets install OpenCV 3 with Python 3 bindings on Mac OSX"
date:   2016-06-25 12:45:00
mathjax: false
---

The aim of all this is to write my own motion detection and image processing code to run on the Raspberry Pi. However, doing this sort of thing on the Pi is a real pain. Much better to get it all working great on the latop and then move it to the Pi.
So in light of that, lets install OpenCV on my Mac OSX laptop.

# First we need Python
Python is the defacto standard for any sort of machine learning and computer vision tasks with some amazing libraries available - such as OpenCV for image processing. You might thing - "but python is a slow interpreted language".... An you're right. But the libraries we use are predominantly written in C and Fortran so that they can be blindingly fast. Essentially, Python provides the glue code between them.

I'm going to install Python 3 (Whatever the latest version is: 3.5.1 at time of writing).

Now, we don't *just* install python. In fact OSX even comes with it pre-installed (an old one though). What we should do is always use our own project specific python environments so that we don't end up with dependency hell or messing about with the standard system python.

There are a number of ways to handle this...
+ virtualenv & virtualenvwrapper
+ Anaconda
+ Docker

### Virtualenv
Virtualenv is a tried and tested way of creating python virtual environments. When you `pip install` a dependency it is installed into a quarantined area just for your project.
### Anaconda
Anaconda is like virtualenv on steroids. I like it myself as it comes pre-installed with allot of the most common data processing and scientific libraries. Additional to virtualenv it not only keeps python dependencies separate, but also system tools that may be already on Anaconda or are downloaded with apt-get.
Anaconda is my choice so I'll use it below.
### Docker
After doing all this I'm thinking that Docker is actually a better way to do things. I'll carry on with Anaconda for now for the learning experience. But after that I'm all docker, which I'll explain at the end of this post...

# Install OpenCV3 with Python3 bindings on MAC OSX
To install OpenCV3 on your mac is not as simple as most other online tutorials make out. I tried building from source as we did with the Pi above but couldn't get it to work - probably because I'm using the Anaconda package manager for python. I kept getting errors around HDF5 at various stages. I eventually got it to compile fine but when I `import cv2` into python it couldn't find the hp5f related libraries. I'm just not interested in fighting with that sort of nonsense when you can use `conda`!

Wtih Anaconda (which I highly recommend) you can install OpenCV 3 very easily with the following command:
```
conda install -c menpo opencv3=3.1.0
```
The menpo project on anaconda have allot Machine Learning and Computer Vision related libraries.
So if you can find the library you're looking for with `conda search opencv3` for example then search over at anacond.org (Ensure that the library is available for linux or OSX depending on where you're working).

Lets check that the installation was successful. Start the python interpreter and import cv2 (first make sure you are in your python3 virtual environment - I've use anaconda with python 3 so its in the correct virtual environment by default):
```
$ python
>>> import cv2
cv2.__version__
'3.1.0'
```
If you don't get any errors then everything is spot on!

# Testing out OpenCV
As has been discussed before the OpenCV cv2.VideoCapture function works with USB cameras (or the camera built into our laptop) but not the Raspberry Pi camera. For the Raspberry Pi we need the picamera python module.

Luckily there is a VideoStream class in the imutils module that abstracts over these camera types so that we can run the same OpenCV code against either.
Install imutils:
```
$ pip install imutils
```
or upgrade an existing installation with:
```
$ pip install --upgrade imutils
```

Lets test out the VideoStream class with a small example program from the [PyImageSearch](http://www.pyimagesearch.com/2016/01/04/unifying-picamera-and-cv2-videocapture-into-a-single-class-with-opencv/) blog:

```
# import the necessary packages
from imutils.video import VideoStream
import datetime
import argparse
import imutils
import time
import cv2

# construct the argument parse and parse the arguments
ap = argparse.ArgumentParser()
ap.add_argument("-p", "--picamera", type=int, default=-1,
	help="whether or not the Raspberry Pi camera should be used")
args = vars(ap.parse_args())

# initialize the video stream and allow the cammera sensor to warmup
vs = VideoStream(usePiCamera=args["picamera"] > 0).start()
time.sleep(2.0)


# loop over the frames from the video stream
while True:
	# grab the frame from the threaded video stream and resize it
	# to have a maximum width of 400 pixels
	frame = vs.read()
	frame = imutils.resize(frame, width=400)

	# draw the timestamp on the frame
	timestamp = datetime.datetime.now()
	ts = timestamp.strftime("%A %d %B %Y %I:%M:%S%p")
	cv2.putText(frame, ts, (10, frame.shape[0] - 10), cv2.FONT_HERSHEY_SIMPLEX,
		0.35, (0, 0, 255), 1)

	# show the frame
	cv2.imshow("Frame", frame)
	key = cv2.waitKey(1) & 0xFF

	# if the `q` key was pressed, break from the loop
	if key == ord("q"):
		break

# do a bit of cleanup
cv2.destroyAllWindows()
vs.stop()
```

Run the test:
```
$ python videostream_demo.py
```
Or, if you copy the program across to the Raspberry Pi:
```
$ python videostream_demo.py --picamera 1
```

![Videostream output](/assets/mac_opencv/videostream_demo.png)

# Docker
Now - even though I've gone on about installing OpenCV on my Mac above, it was mainly a learning exercise. I actually think its better to use docker containers where you can spin up different environment willy-nilly.
Check out the docker images at
+ https://github.com/Kaixhin/dockerfiles
+ https://github.com/jupyter/docker-stacks

These are both great repositories of docker files (they auto build onto docker hub as well).

The Kaixhin docker files are more aimed at machine learning tasks. The [Jupyter](http://jupyter.org/) docker files are more aimed at general purpose scientific python and include a running Jupyter server. Jupyter notebooks if you haven't seen them are a beautiful mesh of Python and web pages - you effectively code in a notebook which can easily be shared. Almost like a presentation.

![Jupyter notebook](/assets/pi/jupyterpreview.png "Jupyter Notebook")

# To Do
I intend to use these as a base for building my own soon...!?!@#$

I like the way Jupyter setup there docker files and they take care of some issues relating to user access, but I also am more used to using Ubuntu instead of Debian; so I might transfer their docker file onto Ubuntu then install OpenCV (via anaconda as I did above but the linux variant) and BAM! We have a nice docker image to play with OpenCV3.
