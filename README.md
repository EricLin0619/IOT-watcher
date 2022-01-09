# **I-Watcher — I’m watching you**
![image](https://github.com/EricLin0619/I-Watcher/blob/main/image/15391447786702.jpg)
### [My Demo Video](https://www.youtube.com/watch?v=CPVSYrHYn-8&ab_channel=%E6%9E%97%E5%86%A0%E5%BB%B7)


Made by 資管3A林冠廷

# Overview
### Why I want to do this project ?
It is quite troublesome to take out the key from the bag to open the door every time I go home. Especially when I just came home from shopping. My hands were full but I still need to find out the key from my bag.  I wish the door could open by itself every time. So i want to make a machine that can open the door just by my face. Then I’ll never have to find the key to open the door anymore.

On the other hand, if relatives and friends visit my home, the host can also control the door lock directly through Telegram bot without opening the door personally.

### How to use this machine ?
1. First of all, you have to download telegram, a communication APP. Then establish a connection with I-Watcher , the bot on telegram.
2. Second, type in “Add new face” in the chat box,  and the camera will turn on to try to capture your face. Please use it in a place with sufficient light sources, otherwise the machine can’t find the face. After capturing the face, I-Watcher will tell you that your face has been stored.
3. Third, type in "Train" in the chat box to train a new face recognition model. After training, I-Watcher will tell you that your face has been trained.
4. Now, we can start using facial recognition to unlock the door! Type in “Turn on the camera” to turn on the pi camera, then camera will turn on and start to recognize face. 
    1. If I-Watcher recognizes your face, I-Watcher will unlock the door for five seconds and play a welcome home sound effect. Simultaneously，send a welcome home message in telegram. 
    2. If it cannot recognize your face, It will play a sound effect to warn the stranger. And take a photo of stranger and send it to the owner's mobile phone. Finally, the camera will be turned off automatically.

### Commands you can use in I-Watcher

- Start : to check if your bot wakeup
    - Reply : “Ok,sir!”
- Open the door : to open the door lock
    - Reply : “Door’s been opened.”
- Close the door : to close the door
    - Reply : “Door’s been closed.”
- Add new face :
    - Reply1 : Please face the camera with your front face.
    - Reply2 : I've gotten your face!
- Train :
    - Reply1 : Ok, I'm training now. Wait a minute please.
    - Reply2 : I can recognize your face now!
- Turn on the camera :
    - Reply1 : Welcome home,my lord.
    - Reply2 : There is a stranger outside your door!!
    - Reply3 : The camera will be turned off in few seconds.

⚠️All the commands can only be used when camera is off.

### Feature list

1. Open the solenoid lock.
2. Close the solenoid lock.
3. Use pi camera to get new faces.
4. Train the facial recognition model.
5. Recognize the face in the database.
6. Make sound effect whether it detect the stranger or someone it know.
7. Send the stranger’s profile to the host’s mobile phone.
8. Do all the features above in telegram.

# Preparation
### Hardware

- Raspberry Pi 4 Model B
- Jumper Wire
- Solenoid lock (DC12V/0.8A )
- Relay 5V
- Speaker、audio cable
- Power supply (input 100~240V , output 12V )
- Pi camera

### Software

- Python 3.9.2
- Opencv 4.5.1
- Python Telegram Bot 12.7
- Pygame
- Telegram App
# Installation
### Install opencv on your raspberrypi

There are several ways can install opencv on your raspberrypi. After my several attempts, this is the best way to install opencv.

Other ways : [樹莓派4b安裝OpenCV 新手推薦 | IT人 (iter01.com)](https://iter01.com/577788.html)

```python
sudo apt-get install -y libopencv-dev python3-opencv
```

⚠️If you want to download all packages and libraries, and compile them by yourself, it will take a lot of time and you are likely to get fail, please think twice.

### Install Python telegram Bot

```python
sudo pip install telepot
```

### Install pygame

To make your speaker can work. Type in this command on your terminal then you’ll hear the speaker’s sound.

```python
speaker-test -c2 -twav -17
```

Install command

```python
sudo apt-get install python3-pygame
```

### Circuit diagram

# Start

## Facial recognition


### Test your camera

```python
import numpy as np
import cv2
cap = cv2.VideoCapture(0)
cap.set(3,640) # set Width
cap.set(4,480) # set Height
while(True):
    ret, frame = cap.read()
    frame = cv2.flip(frame, -1) # Flip camera vertically
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    
    cv2.imshow('frame', frame)
    cv2.imshow('gray', gray)
    
    k = cv2.waitKey(30) & 0xff
    if k == 27: # press 'ESC' to quit
        break
cap.release()
cv2.destroyAllWindows()
```

The above code will capture the video stream that will be generated by your PiCam, displaying both, in BGR color and Gray mode.

To finish the program, you must press the key [ESC] on your keyboard. Click your mouse on the video window, before pressing [ESC].

![image](https://github.com/EricLin0619/I-Watcher/blob/main/image/%E8%9E%A2%E5%B9%95%E6%93%B7%E5%8F%96%E7%95%AB%E9%9D%A2%202022-01-09%20191643.png)

### Facial detection

```python
import numpy as np
import cv2
faceCascade = cv2.CascadeClassifier('Cascades/haarcascade_frontalface_default.xml')
cap = cv2.VideoCapture(0)
cap.set(3,640) # set Width
cap.set(4,480) # set Height
while True:
    ret, img = cap.read()
    img = cv2.flip(img, -1)
    gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    faces = faceCascade.detectMultiScale(
        gray,     
        scaleFactor=1.2,
        minNeighbors=5,     
        minSize=(20, 20)
    )
    for (x,y,w,h) in faces:
        cv2.rectangle(img,(x,y),(x+w,y+h),(255,0,0),2)
        roi_gray = gray[y:y+h, x:x+w]
        roi_color = img[y:y+h, x:x+w]  
    cv2.imshow('video',img)
    k = cv2.waitKey(30) & 0xff
    if k == 27: # press 'ESC' to quit
        break
cap.release()
cv2.destroyAllWindows()
```

The above code are all you need to detect a face, using Python and OpenCV.

```python
faceCascade = cv2.CascadeClassifier('Cascades/haarcascade_frontalface_default.xml')
```

Remember the path is the absolute path of “haarcascade_frontalface_default.xml”.

![螢幕擷取畫面 2022-01-09 192523.png](https://github.com/EricLin0619/I-Watcher/blob/main/image/%E8%9E%A2%E5%B9%95%E6%93%B7%E5%8F%96%E7%95%AB%E9%9D%A2%202022-01-09%20192523.png)

### Step 1 : Get data

```python
import cv2
import os
cam = cv2.VideoCapture(0)
cam.set(3, 640) # set video width
cam.set(4, 480) # set video height
face_detector = cv2.CascadeClassifier('haarcascade_frontalface_default.xml')
# For each person, enter one numeric face id
face_id = input('\n enter user id end press <return> ==>  ')
print("\n [INFO] Initializing face capture. Look the camera and wait ...")
# Initialize individual sampling face count
count = 0
while(True):
    ret, img = cam.read()
    img = cv2.flip(img, -1) # flip video image vertically
    gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    faces = face_detector.detectMultiScale(gray, 1.3, 5)
    for (x,y,w,h) in faces:
        cv2.rectangle(img, (x,y), (x+w,y+h), (255,0,0), 2)     
        count += 1
        # Save the captured image into the datasets folder
        cv2.imwrite("dataset/User." + str(face_id) + '.' + str(count) + ".jpg", gray[y:y+h,x:x+w])
        cv2.imshow('image', img)
    k = cv2.waitKey(100) & 0xff # Press 'ESC' for exiting video
    if k == 27:
        break
    elif count >= 60: # Take 30 face sample and stop video
         break
# Do a bit of cleanup
print("\n [INFO] Exiting Program and cleanup stuff")
cam.release()
cv2.destroyAllWindows()
```

Use this code you can get 30 photos in 10 seconds

![螢幕擷取畫面 2022-01-09 193515.png](https://github.com/EricLin0619/I-Watcher/blob/main/image/%E8%9E%A2%E5%B9%95%E6%93%B7%E5%8F%96%E7%95%AB%E9%9D%A2%202022-01-09%20193515.png)

![螢幕擷取畫面 2022-01-09 193541.png](https://github.com/EricLin0619/I-Watcher/blob/main/image/%E8%9E%A2%E5%B9%95%E6%93%B7%E5%8F%96%E7%95%AB%E9%9D%A2%202022-01-09%20193541.png)
### Step 2 : Train face model

```python
import cv2
import numpy as np
from PIL import Image
import os
# Path for face image database
path = 'dataset'
recognizer = cv2.face.LBPHFaceRecognizer_create()
detector = cv2.CascadeClassifier("haarcascade_frontalface_default.xml");
# function to get the images and label data
def getImagesAndLabels(path):
    imagePaths = [os.path.join(path,f) for f in os.listdir(path)]     
    faceSamples=[]
    ids = []
    for imagePath in imagePaths:
        PIL_img = Image.open(imagePath).convert('L') # convert it to grayscale
        img_numpy = np.array(PIL_img,'uint8')
        id = int(os.path.split(imagePath)[-1].split(".")[1])
        faces = detector.detectMultiScale(img_numpy)
        for (x,y,w,h) in faces:
            faceSamples.append(img_numpy[y:y+h,x:x+w])
            ids.append(id)
    return faceSamples,ids
print ("\n [INFO] Training faces. It will take a few seconds. Wait ...")
faces,ids = getImagesAndLabels(path)
recognizer.train(faces, np.array(ids))
# Save the model into trainer/trainer.yml
recognizer.write('trainer/trainer.yml') # recognizer.save() worked on Mac, but not on Pi
# Print the numer of faces trained and end program
print("\n [INFO] {0} faces trained. Exiting Program".format(len(np.unique(ids))))
```

Use above code you can get the recognition model, trainer.yml.

Remember to change the path if you have your own directory.

### Step 3 : Start to recognize face

```python
import cv2
import numpy as np
import os 
recognizer = cv2.face.LBPHFaceRecognizer_create()
recognizer.read('trainer/trainer.yml')
cascadePath = "haarcascade_frontalface_default.xml"
faceCascade = cv2.CascadeClassifier(cascadePath);
font = cv2.FONT_HERSHEY_SIMPLEX
#iniciate id counter
id = 0
# names related to ids: example ==> Marcelo: id=1,  etc
names = ['None', 'Eric', 'Paula', 'Ilza', 'Z', 'W'] 
# Initialize and start realtime video capture
cam = cv2.VideoCapture(0)
cam.set(3, 640) # set video widht
cam.set(4, 480) # set video height
# Define min window size to be recognized as a face
minW = 0.1*cam.get(3)
minH = 0.1*cam.get(4)
while True:
    ret, img =cam.read()
    img = cv2.flip(img, -1) # Flip vertically
    gray = cv2.cvtColor(img,cv2.COLOR_BGR2GRAY)
    
    faces = faceCascade.detectMultiScale( 
        gray,
        scaleFactor = 1.2,
        minNeighbors = 5,
        minSize = (int(minW), int(minH)),
       )
    for(x,y,w,h) in faces:
        cv2.rectangle(img, (x,y), (x+w,y+h), (0,255,0), 2)
        id, confidence = recognizer.predict(gray[y:y+h,x:x+w])
        # Check if confidence is less them 100 ==> "0" is perfect match 
        if (confidence < 100):
            id = names[id]
            confidence = "  {0}%".format(round(100 - confidence))
        else:
            id = "unknown"
            confidence = "  {0}%".format(round(100 - confidence))
        
        cv2.putText(img, str(id), (x+5,y-5), font, 1, (255,255,255), 2)
        cv2.putText(img, str(confidence), (x+5,y+h-5), font, 1, (255,255,0), 1)  
    
    cv2.imshow('camera',img) 
    k = cv2.waitKey(10) & 0xff # Press 'ESC' for exiting video
    if k == 27:
        break
# Do a bit of cleanup
print("\n [INFO] Exiting Program and cleanup stuff")
cam.release()
cv2.destroyAllWindows()
```

Use this to recognize your face.

![螢幕擷取畫面 2022-01-09 194537.png](https://github.com/EricLin0619/I-Watcher/blob/main/image/%E8%9E%A2%E5%B9%95%E6%93%B7%E5%8F%96%E7%95%AB%E9%9D%A2%202022-01-09%20194537.png)

## Solenoid

![螢幕擷取畫面 2022-01-09 194818.png](https://github.com/EricLin0619/I-Watcher/blob/main/image/%E8%9E%A2%E5%B9%95%E6%93%B7%E5%8F%96%E7%95%AB%E9%9D%A2%202022-01-09%20194818.png)

The circuit diagram for Raspberry Pi Solenoid Door Lock is very simple as you only need to connect the solenoid door lock to Raspberry Pi. Solenoid Lock needs 9V-12V to operate and Raspberry Pi GPIO pins can supply only 3.3V, so a 12V external power source is used to trigger the lock with the help of a relay.

```python
import RPi.GPIO as GPIO
from time import sleep

GPIO.setwarnings(False)
GPIO.setmode(GPIO.BCM)

GPIO.setup(18,GPIO.OUT)

try:
    while (True):
        GPIO.output(18,0)
        sleep(1)
        GPIO.output(18,1)
        sleep(1)
        
except KeyboardInterrupt:
    print("stop!")
    
finally:
    GPIO.cleanup()
```

This few lines code can cycle between locking and unlocking. And you can press “ctrl+c” to stop it.

![0c5de62e-62ef-4022-9680-cbe17642a7df_AdobeCreativeCloudExpress.gif](https://github.com/EricLin0619/I-Watcher/blob/main/image/0c5de62e-62ef-4022-9680-cbe17642a7df_AdobeCreativeCloudExpress.gif)

# Reference

- [Real-Time Face Recognition: An End-to-End Project - Hackster.io](https://www.hackster.io/mjrobot/real-time-face-recognition-an-end-to-end-project-a10826?fbclid=IwAR26TFQwNHnk91X4uxgUNaJWY299Sygu-l5GjkkfYkbAj8jVK3oH_sjPLlY)
- [IoT Based Solenoid Door Lock using Raspberry Pi 4 (iotdesignpro.com)](https://iotdesignpro.com/projects/iot-based-solenoid-door-lock-using-raspberry-pi-4)
- [How To Control A Solenoid With A Raspberry Pi Using a Relay - YouTube](https://www.youtube.com/watch?v=BVMeVGET_Ak&t=34s&ab_channel=CoreElectronics)
- [Controlling Raspberry Pi using Telegram - YouTube](https://www.youtube.com/watch?v=rRJ6H5gxaNA&ab_channel=MSDGurukul)
- [CircuitPython School - Playing Sound (wav or mp3) with PyGame on a Raspberry Pi - YouTube](https://www.youtube.com/watch?v=5F9cl4ZCqQ8&ab_channel=JohnGallaugher)
