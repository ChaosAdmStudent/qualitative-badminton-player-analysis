# Badminton Shot Prediction
In this group project carried out with @Anapop7, the aim is to take a professional badminton match video as an input and predict the most probable space on the court where the shot will be hit by the player on the near side of the court. This is done by considering the positions of the players relative to each other and their body pose at the point-of-contact with the shuttle. Efficient use of Computer Vision ensures that this model is immune to subtle camera angle changes that are prominent across different tournaments with slight change of parameters in the setup stage. (More on this in the process explanation further!)

# Why this project? 
The inspiration for the project stems from my deep interest in badminton. Unfortunately, not a lot of technological support can be seen in badminton to the extent seen in sports like tennis and cricket. With this project, we have two major goals: 

  1. Helping aspiring badminton players to use the model as a guide for their own matches to get a most-likely shot placement that would have been done by a professional badminton player at the point-of-contact with the shuttle in a particular position on the court. 
  
  2. Enhancing user viewing experience. Greater inclusion of AI tools in sports makes it more appealing for the audience watching which subsequently helps the sport to grow. 

# Process Walkthrough 

## Court Line Detection 

The task here is to detect the boundaries of the court to get the court corner coordinates. This is required so that all the processing and prediction going forward is with respect to relative positions of the players on the court, and not the frame. This is required because the differences in camera angles across different tournaments will change the players' position even if they are standing at the exact same position on the court, which is bound to give inaccurate results. 

1. **Image Binarization**: The video frame is binarized using Binary Threshold to highlight the white lines of the court and remove as much of the surroundings as possible. 
2. **Drawing Hough Lines**: Next, Hough Lines are drawn over all the edges. The vertical and horizontal ones are filtered and we craft a function to extract the lines we need using length of the lines as the constraint. This is the parameter that has to be tweaked for a new tournament match in order to extract the border lines for new tournament matches. 
3. **Finding Intersection of Boundary Lines**: Using determinants, all the intersection points of the filtered lines are found out. 
4. **Grouping points together**: Lastly, K-Means clustering is done to form 4 clusters for the 4 corners of the court. This is done to address the issue of multiple overlapping boundary lines which passed through the aforementioned Hough Lines filter. 
