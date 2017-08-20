# Vehicle-Detecton
Comparison of vehicle detection using color/HOG features and LeNet-5

## **1. Introduction**

This project explores the problem of vehicle detection using a Linear Support Vector Machine (SVM) implemented using Scikit-Learn and a Neural Network approach implenting a LeNet-5 architecture using Keras over Tensorflow. The approaches are compared for speed, accuracy and simplicity. This project was done as part of Udacity's Self Driving Car Nanodegree program.

This repo consists of the following project files:
* [SVM Pipeline](./Vehicle-Detection-SVM.ipynb)
* [Neural Net Pipeline](./Vehicle-Detection-NN.ipynb)
* [Trained Models](./models)

## **2. Feature Extraction**

### 2.1 Project Data

The training data available for the project consisted of a combination of the GTI vehicle image database, the KITTI vision benchmark suite as well as samples extracted from the Udacity provided project test videos. The images were labeled as 'cars' or 'not cars', hence this was a binary classification problem. The car images consisted of vehicle rears whereas the not-car images consisted of background images typically encountered on roadways. The characteristics of the data set were as follows:

* Number of car images: 8792
* Number of not-car images: 8968
* Total # of images: 17760
* Image size: 64x64, RGB

The image below shows an example of one of the training car and not-car images:
<img src="./output_images/example_image.png">

### 2.2 SVM Training Features

The SVM training features consisted of the raveled spatially binned image pixel values, the binned histogram color values and the image HOG (Histogram of Oriented Gradients) features concatenated into a single column vector.  

The 64x64 RGB image was first converted into the ```LUV``` colorspace where the chromacity of an image is defined by the U and V pixel values and the luminance by the L pixel values. This helps isolate the effects of a varying brightness in the training and test data into a single channel. The image was the resized to 32x32 pixels and raveled using the ```bin_spatial()``` function to give a feature vector length of:

``` Spatial Features: 32 x 32 x 3 = 3072```

Next, the ```color_hist()``` function was used to compute the color histograms for each channel in the input image and bin them into 32 discrete color bins ranging from pixel values of 0 - 255. The histograms for each channel were concatenated to give a feature vector length of:

``` Color Histogram Features: 32 x 3 = 96 ```

Finally, the HOG features for the image were calculated using the ```get_hog_features()``` function. The following parameters were used to extract the hog features

```
orient = 9  # HOG orientations
pix_per_cell = 8 # HOG pixels per cell
cell_per_block = 2 # HOG cells per block
hog_channel = "ALL" # Can be 0, 1, 2, or "ALL"
```
The values of the parameters above were selected after experimenting with different combinations to achieve the optimum results. Given that the input image size was 64x64, a ```pix_per_cell``` value of 8 allowed for sufficient granularity in the resulting gradients. Similarly, there wasn't a substantial performance gain achieved by increasing the granularity of the HOG orientation greater than 9 (or bins of 40 degrees). This is attributed to the fixed shape of vehicles where the gradient lines of the defining features are expected to be almost horizontal (top and bottom of vehicle rear & rear lights) or almost vertical (sides of vehicle). The image below shows a visualization of the hog features for each of the image channels in LUV space.

<img src="./output_images/HOG_vis.png"

The resulting feature vector size is:

```HOG Features: channels x orient x cell_per_block grid size x cell_per_block x cell_per_block = 3 x 9 x (7 x 7) x 2 x 2 = 5292```

This results in a total feature vector length of:

```Feature vector length = Spatial + Color + HOG = 3072 + 96 + 5292 = 8460```

### 2.3 Neural Network Training Features


* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./examples/car_not_car.png
[image2]: ./examples/HOG_example.jpg
[image3]: ./examples/sliding_windows.jpg
[image4]: ./examples/sliding_window.jpg
[image5]: ./examples/bboxes_and_heat.png
[image6]: ./examples/labels_map.png
[image7]: ./examples/output_bboxes.png
[video1]: ./project_video.mp4


###Histogram of Oriented Gradients (HOG)

####1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the first code cell of the IPython notebook (or in lines # through # of the file called `some_file.py`).  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![alt text][image2]

####2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and...

####3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using...

###Sliding Window Search

####1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I decided to search random window positions at random scales all over the image and came up with this (ok just kidding I didn't actually ;):

![alt text][image3]

####2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image4]
---

### Video Implementation

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video.mp4)


####2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

Here's an example result showing the heatmap from a series of frames of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

### Here are six frames and their corresponding heatmaps:

![alt text][image5]

### Here is the output of `scipy.ndimage.measurements.label()` on the integrated heatmap from all six frames:
![alt text][image6]

### Here the resulting bounding boxes are drawn onto the last frame in the series:
![alt text][image7]



---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

