When it comes to image classification, the human eye has the incredible ability to process an image in a couple of milliseconds, and to determine what it is about (label). It is so amazing that it can do it whether it is a drawing or a picture.

![image](https://user-images.githubusercontent.com/46167070/80891065-ecf0c400-8cc1-11ea-93cc-ab4b14a180b9.png)


## The Canny edge detection algorithm is composed of 5 steps:
Noise reduction;
Gradient calculation;
Non-maximum suppression;
Double threshold;
Edge Tracking by Hysteresis.
After applying these steps, you will be able to get the following result:

### Noise Reduction
Since the mathematics involved behind the scene are mainly based on derivatives (cf. Step 2: Gradient calculation), edge detection results are highly sensitive to image noise.
One way to get rid of the noise on the image, is by applying Gaussian blur to smooth it. To do so, image convolution technique is applied with a Gaussian Kernel (3x3, 5x5, 7x7 etc…). The kernel size depends on the expected blurring effect. Basically, the smallest the kernel, the less visible is the blur. In our example, we will use a 5 by 5 Gaussian kernel.
The equation for a Gaussian filter kernel of size (2k+1)×(2k+1) is given by:

![image](https://user-images.githubusercontent.com/46167070/80891106-288b8e00-8cc2-11ea-9deb-088ec56a822b.png)


### Gaussian filter kernel equation
Python code to generate the Gaussian 5x5 kernel:
Gaussian Kernel function
After applying the Gaussian blur, we get the following result:\
Gradient Calculation
The Gradient calculation step detects the edge intensity and direction by calculating the gradient of the image using edge detection operators.
Edges correspond to a change of pixels’ intensity. To detect it, the easiest way is to apply filters that highlight this intensity change in both directions: horizontal (x) and vertical (y)
When the image is smoothed, the derivatives Ix and Iy w.r.t. x and y are calculated. It can be implemented by convolving I with Sobel kernels Kx and Ky, respectively:


Sobel filters for both direction (horizontal and vertical)
Then, the magnitude G and the slope θ of the gradient are calculated as follow:

### Gradient intensity and Edge direction
Below is how the Sobel filters are applied to the image, and how to get both intensity and edge direction matrices

Blurred image (left) — Gradient intensity (right)
The result is almost the expected one, but we can see that some of the edges are thick and others are thin. Non-Max Suppression step will help us mitigate the thick ones.
Moreover, the gradient intensity level is between 0 and 255 which is not uniform. The edges on the final result should have the same intensity (i-e. white pixel = 255).
Non-Maximum Suppression
Ideally, the final image should have thin edges. Thus, we must perform non-maximum suppression to thin out the edges.
The principle is simple: the algorithm goes through all the points on the gradient intensity matrix and finds the pixels with the maximum value in the edge directions.
Let’s take an easy example:

The upper left corner red box present on the above image, represents an intensity pixel of the Gradient Intensity matrix being processed. The corresponding edge direction is represented by the orange arrow with an angle of -pi radians (+/-180 degrees).


Focus on the upper left corner red box pixel
The edge direction is the orange dotted line (horizontal from left to right). The purpose of the algorithm is to check if the pixels on the same direction are more or less intense than the ones being processed. In the example above, the pixel (i, j) is being processed, and the pixels on the same direction are highlighted in blue (i, j-1) and (i, j+1). If one those two pixels are more intense than the one being processed, then only the more intense one is kept. Pixel (i, j-1) seems to be more intense, because it is white (value of 255). Hence, the intensity value of the current pixel (i, j) is set to 0. If there are no pixels in the edge direction having more intense values, then the value of the current pixel is kept.
Let’s now focus on another example:


In this case the direction is the orange dotted diagonal line. Therefore, the most intense pixel in this direction is the pixel (i-1, j+1).
Let’s sum this up. Each pixel has 2 main criteria (edge direction in radians, and pixel intensity (between 0–255)). Based on these inputs the non-max-suppression steps are:
Create a matrix initialized to 0 of the same size of the original gradient intensity matrix;
Identify the edge direction based on the angle value from the angle matrix;
Check if the pixel in the same direction has a higher intensity than the pixel that is currently processed;
Return the image processed with the non-max suppression algorithm.
The result is the same image with thinner edges. We can however still notice some variation regarding the edges’ intensity: some pixels seem to be brighter than others, and we will try to cover this shortcoming with the two final steps.


### Double threshold
The double threshold step aims at identifying 3 kinds of pixels: strong, weak, and non-relevant:
Strong pixels are pixels that have an intensity so high that we are sure they contribute to the final edge.
Weak pixels are pixels that have an intensity value that is not enough to be considered as strong ones, but yet not small enough to be considered as non-relevant for the edge detection.
Other pixels are considered as non-relevant for the edge.
Now you can see what the double thresholds holds for:
High threshold is used to identify the strong pixels (intensity higher than the high threshold)
Low threshold is used to identify the non-relevant pixels (intensity lower than the low threshold)
All pixels having intensity between both thresholds are flagged as weak and the Hysteresis mechanism (next step) will help us identify the ones that could be considered as strong and the ones that are considered as non-relevant.
The result of this step is an image with only 2 pixel intensity values (strong and weak):
Non-Max Suppression image (left) — Threshold result (right): weak pixels in gray and strong ones in white.
 
 
### Edge Tracking by Hysteresis
Based on the threshold results, the hysteresis consists of transforming weak pixels into strong ones, if and only if at least one of the pixels around the one being processed is a strong one, as described below:



