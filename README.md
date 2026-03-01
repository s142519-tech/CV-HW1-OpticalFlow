# CV-HW1-OpticalFlow
Computer Vision Homework 1

# Homework 1: Optical Flow and Video Dynamics 

## Option A: Camera Stabilization Mini-System

### Overview
This repository contains a complete pipeline for video stabilization using Optical Flow. The primary goal is to estimate global camera motion from shaky footage, smooth the camera's trajectory, and apply motion compensation (warping) to generate a stabilized video.

### Pipeline Details & Methodology
1. **Feature Tracking (Optical Flow):** Used the Lucas-Kanade (LK) sparse optical flow method (`cv2.calcOpticalFlowPyrLK`) to track key features (corners, distinct architectural lines, and sharp edges) between consecutive frames.
2. **Global Motion Estimation:** Calculated the rigid affine transformation (translation `dx, dy` and rotation `da`) using `cv2.estimateAffinePartial2D` to map the exact movement of the camera.
3. **Trajectory Smoothing:** Applied a low-pass filter (Moving Average Window) to the absolute, accumulated camera trajectory to eliminate high-frequency jitters while preserving intentional camera panning.
4. **Motion Compensation (Warping):** Applied the inverse of the smoothed transformation to each frame using `cv2.warpAffine` to stabilize the scene.
5. **Border Artifact Correction:** Used dynamic cropping (zooming) to eliminate the empty black borders introduced by the warping process.

### Key Parameters Used
* `maxCorners = 200`: Found to be a sufficient number for tracking stable background features without slowing down computation.
* `SMOOTHING_RADIUS = 30`: This window size provided a good balance between removing heavy jitter and avoiding over-smoothing that requires excessive cropping.
* `ZOOM_FACTOR = 1.05`: Applied to scale the final frames up by 5% to reduce the black compensation borders at the edges without excessively cropping the original field of view.

###  Error Analysis & Observations
During the development and testing of this pipeline, several key observations were made regarding the limitations of optical flow for stabilization:

* **Border Artifacts (The Black Edges):** Warping frames to compensate for severe motion naturally leaves empty spaces at the edges of the canvas. This was mitigated by applying a slight zoom, but extreme shakes still require larger crops, reducing the overall field of view.
* **Motion Blur Vulnerability:** Fast, sudden camera jerks cause significant motion blur. This destroys the distinct features (corners) that LK relies on, leading to temporary tracking loss or erratic motion vectors.
* **Texture Dependency:** The LK method performs excellently on scenes with rich textures (like buildings, cityscapes, or distinct objects). However, if the camera pans across a uniform surface (like a clear sky, blank wall, or dark shadows), `cv2.goodFeaturesToTrack` fails to find enough reliable points, causing the affine estimation to fail.
* **Foreground Interference:** If large moving objects (like vehicles or people) dominate the frame, the algorithm might track their movement instead of the background. This causes the stabilizer to incorrectly "lock on" to the moving object, resulting in the background violently warping around it.

### Repository Structure
* `src/notebook.ipynb`: The main code containing the feature tracking, smoothing, and warping pipeline.
* `shaky_video.mp4`: The original input footage.
* `stabilized_video.mp4`: The final stabilized output.
* `trajectory_analysis.png`: Visual plots comparing the X, Y, and Angle trajectories before and after the moving average filter.
