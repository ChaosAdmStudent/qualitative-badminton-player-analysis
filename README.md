# Badminton Shot Prediction

> **Status:** On-Going

In this group project carried out with Anannya Popat (@Anannyap7), the aim is to take a professional badminton match video as an input and predict the most probable space on the court where the shot will be hit by the player on the near side of the court. This is done by considering the positions of the players relative to each other and their body pose at the point-of-contact with the shuttle. Efficient use of Computer Vision ensures that this model is immune to subtle camera angle changes that are prominent across different tournaments with slight change of parameters in the setup stage. (More on this in the process explanation further!)

# Why this project? 
The inspiration for the project stems from my deep interest in badminton. Unfortunately, not a lot of technological support can be seen in badminton to the extent seen in sports like tennis and cricket. With this project, we have two major goals: 

  1. Helping aspiring badminton players to use the model as a guide for their own matches to get a most-likely shot placement that would have been done by a professional badminton player at the point-of-contact with the shuttle in a particular position on the court. 
  
  2. Enhancing user viewing experience. Greater inclusion of AI tools in sports makes it more appealing for the audience watching which subsequently helps the sport to grow. 

# Process Walkthrough 

## 1. Court Line Detection 

The task here is to detect the boundaries of the court to get the court corner coordinates. This is required so that all the processing and prediction going forward is with respect to relative positions of the players on the court, and not the frame. This is required because the differences in camera angles across different tournaments will change the players' position even if they are standing at the exact same position on the court, which is bound to give inaccurate results. 

1. **Image Binarization**: The video frame is binarized using Binary Threshold to highlight the white lines of the court and remove as much of the surroundings as possible. 

2. **Drawing Hough Lines**: Next, Hough Lines are drawn over all the edges. The vertical and horizontal ones are filtered and we craft a function to extract the lines we need using length of the lines as the constraint. This is the parameter that has to be tweaked for a new tournament match in order to extract the border lines for new tournament matches. 

<p align="center"> 
  <img src="https://user-images.githubusercontent.com/53689018/202634282-2b7ee279-0fa7-4529-89c4-9ba711b8a1db.png" width="500">
</p> 
  
3. **Finding Intersection of Boundary Lines**: Using determinants, all the intersection points of the filtered lines are found out. 

<p align="center"> 
  <img src="https://user-images.githubusercontent.com/53689018/202634363-9f669a53-6b5e-4ff1-89d0-41c729bae77c.png" width="500")
</p>  
       
4. **Grouping points together**: Lastly, K-Means clustering is done to form 4 clusters for the 4 corners of the court. This is done to address the issue of multiple overlapping boundary lines which passed through the aforementioned Hough Lines filter. 

<p align = "center">
  <img src="https://user-images.githubusercontent.com/53689018/202634418-2c7a6db5-e75f-4800-9579-e43291753680.png" width ="500")
</p>

## 2. Player Detection 

It is crucial to detect the player positions on the court as it's one of the most important parameters for predicting shot direction in most cases. For this purpose, we tried using three methods for object detection: 
- Particle Filter (Best trade off between speed and accuracy) 
- YoloV3 (Very accurate, but very slow)  
- Video Frame Difference (Detecting the smallest of movements, even outside the court)  

### 2.1 Automatic Color Detection 

Particle Filter works by tracking RGB values of a target pixel. To ensure that the script picks up the target color for any particular match, we automated the color detection of the target by the following means: 

- Took separate frames of top and bottom half of the court to get information for top and bottom player separately
- Masked the outer regions of the court using the court boundary coordinates 
- Extracted most widely used colors in the two sub-frames extracted and filtered out black (the region outside the court), green (the color of the court mat) and the court boundary color. 

<p align = "center">
  <img src="https://user-images.githubusercontent.com/53689018/203215591-b1cdfbb7-8997-4f14-8b2a-c5072c8b6cb8.png" width="600">
</p>  

### 2.2 Replay Detection using CNN 

While making our particle filter script, we noticed the script completely crashing whenever a close-up replay frame was encountered. We're also only interested in the over-the-court camera angle rather than the close-ups. For this purpose, we developed a CNN model to classify a frame into a "play" or "non" play frame. The frame IDs for a non-play frame are stored in a textfile which is referenced by the algorithm in section 2.3. 

### 2.3 Particle Filter 

We won't be talking about how this algorithm works, but rather the configuration unique to our implementation used for particle filter: 

- Two sets of particles, one for each player 
- Particles for bottom player spread out in the trapezium-like shape (as seen in the image in the previous section), while particles for upper player spread out in a rectangular region extending slightly outside the court (because the actual body of the player appears outside the court coordinates) 

- For each frame in the video, draw pure black lines on the vertical court boundaries to avoid particles from converging on the court boundaries in case the player jersey color is similar to that of the court boundaries. 

- For each frame, check if that frame id has been identified as a replay frame (done in section 2.2). If yes, re-initialize particles and pause object tracking till a "play" frame is encountered again. 

<p align ="center">
  <img src="https://github.com/ChaosAdmStudent/badminton-shot-prediction/blob/main/demos/ParticleFilterDemo.gif" width = "800">
</p> 

## 3. Point-Of-Contact Frame Collection 

Next, it is very important to collect the timestamps where the player is hitting the shuttle because these are the decision points where a decision will be made by the model. We don't want the model to take every frame in consideration, and rather only a sequence of about 20 frames before impact to predict the shot direction. 

We tried two approaches here: 

1. **Sound Event Detection:** Hardly picked up 50 timestamps in a 7 minute gameplay video, some of which were even part of replay frames [Refer to branch "ChaosAdmStudent-patch-2"] 

2. **Convolutional Neural Networks:** This is an on-going approach where the idea is to manually annotate some timeframes from the match with labels signifying a "hit" or "non" hit frame. This is done by converting the audio snippets of the manually annotated timestamps into spectrograms which are then passed into the model for classification purpose.
