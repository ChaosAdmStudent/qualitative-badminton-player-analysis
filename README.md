# Qualitative Badminton Player Analysis

> **Status:** Completed

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

It is crucial to detect the player positions on the court as it's one of the most important parameters for predicting shot direction (if we plan to implement it in the future) in most cases. For this purpose, we tried using three methods for object detection: 
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

While making our particle filter script, we noticed the script completely crashing whenever a close-up replay frame was encountered. We're also only interested in the over-the-court camera angle rather than the close-ups. For this purpose, we developed a CNN model to classify a frame into a "play" or "non" play frame. The frame IDs for a non-play frame are stored in a textfile which is referenced by the algorithm in section 2.3. Figures below show "non-play" and "play" frames.

<p align ="center">
  <img src="https://github.com/ChaosAdmStudent/badminton-shot-prediction/assets/59221653/65ceb0b4-d37a-4bc0-8097-8457026f19d7" width = "300">
  <img src="https://github.com/ChaosAdmStudent/badminton-shot-prediction/assets/59221653/83fd11e9-afee-49e2-8211-1b651a9f6fc7" width = "300">
</p> 

### 2.3 Particle Filter 

We won't be talking about how this algorithm works, but rather the configuration unique to our implementation used for particle filter: 

- Two sets of particles, one for each player 
- Particles for bottom player spread out in the trapezium-like shape (as seen in the image in the previous section), while particles for upper player spread out in a rectangular region extending slightly outside the court (because the actual body of the player appears outside the court coordinates) 

- For each frame in the video, draw pure black lines on the vertical court boundaries to avoid particles from converging on the court boundaries in case the player jersey color is similar to that of the court boundaries. 

- For each frame, check if that frame id has been identified as a replay frame (done in section 2.2). If yes, re-initialize particles and pause object tracking till a "play" frame is encountered again. 

<p align ="center">
  <img src="https://github.com/ChaosAdmStudent/badminton-shot-prediction/blob/main/demos/ParticleFilterDemo.gif" width = "800">
</p> 

## 3. Player Stroke Detection 

Another important component of this project is to predict the kind of stroke/shot played by the players. Here, we have considered 3 primary types of shots for demonstration purposes, namely, 'net-drop return', 'smash' and 'defense'. In order to predict these shots, we have manually collected the training data by extracting the frames where these shots occur and labeling them. After this, the training data is passed through a multi-class classifier and a stroke detection is made using an Artificial Neural Network model like CNN. The **accuracy** of this model is currently around **81%**, which can be improved upon by expanding the dataset manually.

Convolutional Neural Networks architecture of the proposed system is given below:

<p align = "center">
  <img src = "https://github.com/ChaosAdmStudent/badminton-shot-prediction/assets/59221653/d86df4bb-b3c1-4ee7-b58b-147eb3f7af71">
</p>

The types of strokes predicted (net-drop return, smash and defense) using the proposed CNN model are as follows:

<p align ="center">
  <img src="https://github.com/ChaosAdmStudent/badminton-shot-prediction/assets/59221653/a18bddd4-56fd-4712-97f1-35efbb7e0439">
  <img src="https://github.com/ChaosAdmStudent/badminton-shot-prediction/assets/59221653/cc72d972-4299-4652-8ba5-9424f6c9b2e3">
  <img src="https://github.com/ChaosAdmStudent/badminton-shot-prediction/assets/59221653/d6352f6a-07c8-4ef2-acc8-4b1d3f89fb07">
</p> 

## 4. Player Pose Estimation using OpenPose library 

Here, 19 individual body parts along with their paired connections are defined. Then a predefined tensorflow model is used to train on the input image and generate a graph of lines and points to indicate each joint connection. The image below demonstrates the same.

<p align = "center">
  <img src = "https://github.com/ChaosAdmStudent/badminton-shot-prediction/assets/59221653/4d0464af-7257-44d1-b941-7dc6311ebcd3">
</p>
