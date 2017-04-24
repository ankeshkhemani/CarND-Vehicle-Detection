# CarND-Vehicle-Detection
My submission for the Udacity Self-Driving Car Nanodegree program Project 5 - Vehicle Detection and Tracking

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.


[image1]: ./output_images/images1.png

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.
Code available in vehicle_detection_project.ipynb under heading "Histogram of Oriented Gradients (HOG)"

I started by reading in all the `vehicle` and `non-vehicle` images. 
I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.


#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters. orientations: [8,9,11], pixels_per_cell:[2,4,8], cells_per_block:[2,4,8]
I visualised the hog features of a number of cars vs a non-cars. It helped me to have an intuition for which parameters, hog features of a car are distinguishable from that of a non-car. 
I also tweaked the hog parameters in favour of a better classifier test accuracy when using a linear SVM.
I finally settled for orientations:11, pixels_per_cell:8, cells_per_block:2 because it gave me the best test accuracy.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).
Code available in vehicle_detection_project.ipynb under heading "Train a Classifier"

I trained a Linear SVM classifier `LinearSVC` from `sklearn.svm`C using just the hog features.
I tried tuning the penalty parameter `C` of LinearSVC() which penalises wrong classification by a factor of C while training. It has a default value of 1 and I tried for 10,100,1000 and had seen an accuracy improvement for some settings. However, when I finalised the hog parameters as above, the default value of C=1 gave me the maximum accuracy. 
The final accuracy is 98.03%

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?
Code available in vehicle_detection_project.ipynb under heading "Sliding Window Search"

I implemented overlapping sliding windows of scale 1.0,1.5,2.0,3.0 in the area where we are likely to find cars. I also tried with scale of 0.5 but it was resulting in a very high number of false positives so I dropped the 0.5 scale windows.

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Here are some examples of test images to demonstrate how my pipeline is performing.

![alt text][image1]

Here are the details of my classifier:
I extracted ALL hog features from the YUV colour channel of the images.
I chose hog parameters orientations:11, pixels_per_cell:8, cells_per_block:2
I used a linear SVM classifier with default parameters.
Instead of svc.predict, I used svc.decision_function > 0.75 to get rid of several false positives in the images.
I used a high number of sliding windows and then used a heat map and heat threshold to get rid of other false positives.

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video_out.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.
Code available in vehicle_detection_project.ipynb under heading `Video Implementation`

At first I tried detecting cars with fewer number of windows per scale and found all the true positive positions. However, I was also getting several false positive detections. The difference between the two was that true positive positions got detected in multiple windows. 
For leveraging this behaviour as the key differentiator, I used heat maps by giving a heat point on the image for each correct classification. Then I set a threshold of 2 meaning any point with less than or equal to two correct classification was not considered as correct classification. For this to work, I needed more number of correct detections for true positives so I increased the number of overlapping sliding windows for every scale. For example, instead of sliding windows by 32 pixels, now we would slide them by 16 pixels and so on. 
I labeled the threholded heat maps using `label` function from `scipy.ndimage.measurements` to identify individual blobs in the heatmap.
Then, assuming each blob as a car, I constructed bounding rectangle boxes covering each blob to detect cars positions in my image.
Each step with visualisation can be seen in project html or ipynb notebook.



---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I faced a lot of trouble due to false positives, some of which were not going away even with a svc.decision_function > 0.75. 
So, I used a rather large number of sliding windows and this resulted in slower processing of each frame and I submit that my pipeline is not optimal for real-time processing. 
For the limitation of fast enough processing power, we may have to use some other technique for removing false positive rather than an overkill of sliding windows. I hope to visit this project again to try out colour histograms as additional features to handle false positives.

References: Visualisation tricks and ideas are attributed to the Udacity community.
