---
layout: post
title: Automatically Assessing Image Quality for AptDeco.com
comments: true
---

Training an AdaBoost model to automatically quantify the quality of furniture images for AptDeco.com

<!--more-->

{% include math.html %}

Over the past three weeks, I've been consulting on a data science project for AptDeco.com.  AptDeco is a peer-to-peer online marketplace for buying and selling used furniture.  The website simplifies the resale process by handling all of the logistics for its users.  The AptDeco team fills in any missing details about the furniture, creates high quality listings on their website, and even delivers the furniture when it's purchased.

The first step of creating a listing on AptDeco.com is to submit a picture of the furniture.  The editors at AptDeco will review the pictures, and decide if they are of high enough quality to be edited and displayed on the front page of the listing.  Unfortunately, many of the submitted images are low quality, and can't be displayed in the online store.  This means that AptDeco's team spends a large amount of time sifting through low quality pictures and requesting improvements from their users.  In addition to the time sink of this process, some users never submit better images, leading to a lost sale for AptDeco and its users.

With this in mind, I created DecoRater &mdash; an algorithm for automatically assessing the quality of furniture images on AptDeco.com.  DecoRater provides immediate feedback to AptDeco users, increasing the odds that they will submit high quality images for their listings.  DecoRater can also be used behind the scenes, to reduce the number of low-quality photos that the AptDeco team has to sift through, and to flag listings which are in high need of editor intervention.  

In this post, I'll explain how I created DecoRater from ground up.  The discussion will include:

* [Choosing an Appropriate Model](#ChoosingAModel)
* [Extracting Different Color Spaces From an Image](#ColorSpaces)
* [Extracting Image Features](#ImageFeatures)
* [Training an Image Quality Model](#initSetup)
* [Assessing Model Performance](#customize)
* [Implementing the Model](#ideas)

# <a name="ChoosingAModel"></a> Choosing an appropriate model

Before we describe how to assess the quality of each image, we need to define what the "quality" of an image means.  AptDeco's database has a number of metrics which we could use to define "quality."  Most of these metrics, such as a listing's click through rate and conversion rate, are influenced by outside factors, such as the price of a listing, or the age of the furniture.  Even if we were to account for these factors, the images that are displayed in the store are already edited.  This means that any model which uses these metrics could only suggest improvements to the already-edited images, rather than to the raw images that users upload directly.

A more useful measure of image quality can be defined by AptDeco's choice to edit and display an image, or to reject it.  AptDeco's editors don't record the outcome of this decision in their database, but they were willing to help me hand-label a subset of 2000 images for this purpose.  In this light, the goal of assessing image quality becomes a binary classification problem in which we try to predict the probability that an unedited picture will be edited and displayed on the front page of a listing.  

To accomplish this goal, we'll need to train a machine learning model to emulate a human's subjective opinion of each image's quality.  The go-to model for image processing is a convolutional neural network (CNN), which requires minimal preprocessing of an image, and can easily account for the spatial correlation of individual pixels.  Unfortunately, we lack the large amount of labeled data that is required to train a CNN.  

Instead, we'll need to implement a more simple classification model.  After considering multiple algorithms, I settled on an AdaBoost random forest classifier due to its higher test performance relative to other models.  The Adaboost model focuses on hard to classify samples, which helps boost performance on borderline images.  Most importantly, the model can output the probability that an image belongs to the "high quality" class, rather than simply classifying the images on a binary scale.  This is a nice feature to have, since we can use the continuous probability to rank the quality of images within the "high quality" and "low quality" groups. (Note: This is why support vector machines were not considered, since the output of these models should not be interpreted as a continuous probability.)

# <a name="ColorSpaces"></a> Extracting Different Color Spaces From an Image

The input features for our model will need to describe certain characteristics of each image.  We'll discuss the details of these features in the next section, but suffice it to say that many of them will describe how colors change throughout the picture.  We can represent these colors in a number of different [color spaces](https://en.wikipedia.org/wiki/List_of_color_spaces_and_their_uses), which each have their own advantages.  In this section, I'll discuss the four different color spaces that we'll be working with.

1) **The RGB Color Space**

<center>
<img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/DecoRater/ColorSpaces/RGB.png" width="300"/>
</center>

The [RGB (Red,Green,Blue) color space](https://en.wikipedia.org/wiki/RGB_color_model) describes each color as a fraction of red, green, and blue light.  The intensity of each component determines the final color, with zero intensity from each channel resulting in black, and full intensity from each channel resulting in white.  This color space is useful for measuring the brightness of images, as well as the gradients of the red, green, and blue color channels across the image.

2) **The HSV Color Space**

<center>
<img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/DecoRater/ColorSpaces/HSV.png" width="400"/>
</center>

The [HSV (Hue,Saturation,Value) color space](https://en.wikipedia.org/wiki/HSL_and_HSV) is a cylindrical representation of the RGB color model that attempts to be more perceptually relevant than the standard RGB space.  Within the cylinder, the angle around the vertical axis represents to the "Hue" of the color, which denotes the wavelength of light which is most dominant.  Since hue is measured as an angle around a cylinder, we'll need to use circular statistics when quantifying it.  The distance from the center represents the "saturation" of the color, which measures the colorfulness or purity of a color relative to how bright it is.  Mixing a completely saturated color with white lowers the saturation, causing the color to appear white-washed a lower saturation values.  Finally, value measures the brightness of the color, with a low value corresponding to a low brightness.  The HSV color space is useful for measuring how white-washed or dark an image appears, and for measuring how complimentary certain colors appear to the human eye.

3) **The LAB Color Space**

<center>
<img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/DecoRater/ColorSpaces/LAB.png" Width="300">
</center>

The [LAB (Lightness, a, b) color space](https://en.wikipedia.org/wiki/Lab_color_space) represents colors using two color-opponent dimensions (a and b) and the overall lightness of the image.  It is particularly useful for measuring how colorful an image appears, since a wider area in the a-b plane corresponds to a wider range of colors in the image.

4) **The Grayscale Channel**

<center>
<img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/DecoRater/ColorSpaces/Grayscale.png" width="400"/> 
</center>

The grayscale channel of an image encodes color intensity with varying shades of gray.  The grayscale channel varies from black, at zero starsintensity, to white, at full intensity.  This channel is useful for calculating non-color related features, such as the blurriness of an image, or the location of focal points.

# <a name="ImageFeatures"></a> Extracting Image Features

Now that we have a number of color spaces to work with, we're ready to extract features from our images.  We'll start with the low hanging fruit, before moving on to more complex features.

1) **Image size - 2 features**

All of the pictures that we use begin with a 1500x1500 pixel format.  Many of these pixels are filled with white borders that would not be displayed in the online listings.  After we removing these white borders, the images take on a wide range of shapes and sizes.  We can measure the total number of pixels in the cropped images, as well as the aspect ratio (height-to-width ratio) to produce two features that tell us about the shape of a picture.


2) **Average Color Intensity -- 6 features**

Although a user won't have much control over the color of their pictures, the overall color of the image may nonetheless impact its aesthetic quality. We can easily measure the average intensity of each color by working in the individual channels of the RGB or HSV color spaces. For each channel, we calculate the average intensity over the image and use the result to create six more features.

<center>
<div class="container">
<img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/DecoRater/AverageColorIntensity/Organized.png" Width="600"> 

<div style="width:600px;">
<i> Top: An image with low intensity in the blue color channel
<br>
Bottom: An image with high intensity in the blue color channel
</i>
</div>

</center>


3) **Variation of Color Intensity - 6 features**


We can also measure the variation of intensity within each color channel.  A low standard deviation of the intensity in channel may indicate that the image is tinted that color, while a high standard deviation may indicate bright lights or shadows in the image.  Calculating the standard deviation for both the RGB and HSV color spaces provides six additional image features.

<center>
<div>
<img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/DecoRater/ColorVariation/ColorVariation_B0p55_original.png" Width="300"> <img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/DecoRater/ColorVariation/ColorVariation_B0p55.png" Width="270">
</div>

<div>
<img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/DecoRater/ColorVariation/ColorVariation_B0p74_original.png" Width="200"> <img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/DecoRater/ColorVariation/ColorVariation_B0p74.png" Width="200">
</div>

<div style="width:600px;">
<i> Top: An image with low variation in the blue color channel, indicating a blue tint of the image
<br>
Bottom: An image with high variation the blue color channel, indicating the presence of a bright light source and shadows.
</i>
</div>

</center>



4) **Color Gradients - 12 features**

In calculus, the "gradient" of a function defines how quickly that function changes with respect to a particular variable.  A function which changes quickly with respect to a variable will have a high gradient, while a function which changes slowly with respect to a variable will have a low gradient.  We can use this concept to measure how quickly each color changes throughout our image.  The gradient is measured using a [Sobel Filter](http://docs.opencv.org/3.0-beta/doc/py_tutorials/py_imgproc/py_gradients/py_gradients.html), which removes noise from neighboring pixels before calculating the difference of intensity between them.  We can calculate the gradient of the RGB and HSV filters, in both the horizontal and vertical directions, providing 12 additional image features.  These features are particularly useful for detecting shadows or glares in the picture, which will present as high gradients due to the quickly varying shades of light.  

It is worth noting that the "color gradient" and "variation of color intensity" are similar, but are measuring two different quantities.  Specifically, the gradient features take correlation of nearby pixels into account, while the standard deviation is applied to the image as a whole.

<center>
<img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/DecoRater/AverageColorGradient/Organized.png" Width="600">
</div>

<div style="width:600px;">
<i> Top: An image with low horizontal gradients in the blue color channel
<br>
Bottom: An image with high horizontal gradient in the blue color channel, indicating the presence of shadows.
</i>
</div>

</center>


5) **Colorfulness of an image - 1 feature**

The LAB color space is particularly useful when measuring how "colorful" an image is.  The A and B channels represent opposing colors, so a the total area of an image in the AB plane corresponds to the colorfulness of an image.  [Hasler and Suesstrunk](https://www.researchgate.net/publication/243135534_Measuring_Colourfulness_in_Natural_Images) present a formula for quantifying the perceived colorfulness of an image based on their 2003 experimental study.  The formula is given by:

$$\mathsf{Colorfulness} = \sigma_a + \sigma_b + 0.39 \sqrt{\mu_a^2 + \mu_b^2}$$

where the first two terms are essentially measuring the spread of the image in along the A and B axis, and the third term is a slight modification that relates to the average color of the picture.

<center>

<div>
<img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/DecoRater/Colorfulness/Colorfulness_72.png" Width=165"> <img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/DecoRater/Colorfulness/Colorfulness_132.png" Width="275">
</div>

<div>
<img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/DecoRater/Colorfulness/Comparison.png" Width="400"> 
</div>

<div style="width:600px;">
<i> Top: An image with low colorfulness (left) and an image with high colorfulness (right)
<br>
Bottom: The two images projected on the A-B plane.  Note that the colorful image (green) occupies more area than the colorless image (red).
</i>
</div>

</center>

6) **Complimentary Colors - 1 feature**



Complimentary colors are located directly across from each other on the color hue circle, as shown in the figure above.  We can use complex analysis to measure the overall amount of complementary colors in an image.  Specifically, the equation

$$e^{i \theta} = cos\theta + i sin\theta$$

defines a circle of radius one in the "complex plane," where the real coordinate of the number is given by $$cos\theta$$, and the imaginary part is given by $$sin\theta$$. By imagining the circle in the complex plane as the color hue circle, we can use the real and imagine components coordinates of each color to measure how complimentary each one is.

If you are unfamiliar with complex analysis, this will be a hard concept to grasp.  It may be helpful to think of the problem in term of the real component alone &mdash; that is, in terms of $$cos\theta$$.  If we have one color that is located at an angle of zero on the hue circle ($$\theta = 0$$), and a complimentary color located on the other side of the circle, at $$\theta = 180$$, then the cosine of these angles will equal $$1$$ and $$-1$$, respectively.  With this scheme, the sum of two complimentary colors would equal zero.  It's more intuitive to assign a high value to complimentary colors, which we can achieve by multiplying the hue angle by a factor of two.  In our previous example, the colorfulness values would become $$cos(2 \times 0) =1$$, and $$cos(2 \times 180) = 1$$, allowing the sum of complimentary colors to add together.  This transformation also allows uncomplimentary colors to cancel out. For instance, a color at $$theta=0$$ and a color at $$theta=90$$ on the hue circle would have values of $$cos(2*0)=1$$ and $$cos(2*90)=-1$$ respectively.  

Using the method described above, we can measure the total amount of complimentary colors in an image by calculating $$e^(2*\theta)$$ for each pixel of an image, where $\theta$ represent the Hue angle for that pixel.  Next, we add the absolute value of the resulting numbers together, and divide by the total number of pixels in the image:

Complimentary Color Index $$= \sum \frac{\abs(e^(2*H*i)}{ N_{pixels} }$$

The result is a number between zero in one, with one representing an image made of perfectly complimentary images (such as red and green), and zero representing an image made of perfectly uncomplimentary images (such as red and blue).

<center>
<div>
<img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/DecoRater/ColorSpaces/ComplementaryColors.png" Width="300"> 
</div>

<div style="width:600px;">
<b>Figure 1 </b>: <i> An illustration of complementary colors on the Hue wheel.
</i>
</div>

</center>

7) **Darkness of an image - 3 features**

The easiest way to measure the "brightness" of an image is to make a histogram of the intensity values within the R,G, and B channels.  Since values that are closer to zero represent black, and values that are closer to 255 represent white, we can take the average value of the intensity histogram as a measure of how dark the image is.


A more nuanced way to measure the "brightness" of an image is to use the average [relative luminance](https://en.wikipedia.org/wiki/Relative_luminance) of the image, which emphasizes the physiological aspects of the human eye.  Relative luminance is defined by

$$L = 0.2126R + 0.7152G + 0.0722B$$

We can also use the average [perceived luminance](https://www.w3.org/TR/AERT#color-contrast) of the image, which accounts for psychological aspects of human perception, and is given by:

$$L = 0.299R + 0.587G + 0.114B$$

Note that since these three image features originate from the relative intensities of the RGB channels, they will be highly correlated.


<center>

<div>
<img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/DecoRater/Darkness/Darkness_36p2.png" Width=165"> <img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/DecoRater/Darkness/Darkness_150.4.png" Width="150">
</div>

<div>
<img src="https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/DecoRater/Darkness/Comparison.png" Width="400"> 
</div>

<div style="width:600px;">
<i> Top: An image with low luminosity (left) and an image with high luminosity (right)
<br>
Bottom: A histogram of the pixel intensity for the two images above. Note that the bright image (green) has a right-leaning histogram, while the dark image (red) has a left-leaning histogram.</i>
</div>

</center>


8) **Blurriness of an image - 5 features**

We have taken two separate approaches to measure how blurry an image is.  The first approach applies a [Laplacian Filter](http://docs.opencv.org/2.4/doc/tutorials/imgproc/imgtrans/laplace_operator/laplace_operator.html) to the image, which calculates the second derivative of single channel from the image.  If you aren't familiar with calculus, this means the laplacian filter calculates how quickly color changes accelerate in the image.  Typically, edges that are present in the image will have a high value in the laplacian filter.  If the variance of the laplacian filter is high, then the image has a wide range of edge-like and non-edge-like responses.  On the other hand, if the variance of the laplacian is low, then there are very few edges present in the picture &mdash; likely indicating a blurry image.  Since the laplacian filter is applied to a single channel, we can measure the blurriness of the HSV and grayscale channels separately, producing 4 additional features.


A more complicated way to measure the blurriness of an image is to compute the [Fast Fourier Transform ](https://en.wikipedia.org/wiki/Fast_Fourier_transform) of the image.  Without going into the details, the Fourier Transform decomposes the image into high frequency and low frequency variations.  High frequency variations indicate rapid changes in the image, which correspond to edge-like features in the image.  Therefore, the average frequency found in the Fourier Transform provides a measure of how blurry an image is, with higher frequencies corresponding to sharper images.

9) **Focal points of an image - 3 features**

We can identify focal points of an image by constructing a [saliency map](https://en.wikipedia.org/wiki/Salience_(neuroscience)) of the image.  The concept of saliency originates from neuroscience, and measures the extent to which it stands out relative to its surroundings.  The method we use to create a saliency map is described by Itti, Koch, and Niebur in their 1998 paper [A model of Saliency-based Visual Attention for Rapid Scene Analysis](http://www.lira.dist.unige.it/teaching/SINA_08-09/papers/itti98model.pdf). This method takes a bottom-up approach, in which the image is decomposed into 12 color maps, 6 intensity maps, and 24 orientation maps. The extent to which an object stands out within each map is calculated, and a normalized saliency map representing each of the three categories (color, intensity, and orientation) are produced.  The final saliency map is created by averaging over the three normalized maps. Itti, Hoch, and Niebur's visualization of this algorithm is included in Figure XX, and an example saliency map is included in Figure XX.

Once we have identified areas of high saliency in our image, we can calculate the average Hue, Saturation, and Value at these locations to produce three additional image features.

10) **Rule of Thirds - 5 features**

The rule of thirds is a common guideline which applies to the composition of photographs. The rule states that an image should be divided into nine parts, with two horizontal lines and two vertical lines equally dividing the image into three rows and three columns.  Proponents of the rule of thirds believe that aligning the subject of a photo with the intersection of these lines produces a more interesting composition than simply centering the subject.  

In our case, AptDeco's editors strongly prefer front-facing pictures of furniture, since they are easier to edit and display on the front page.  Nonetheless, we can use the rule of thirds to measure just how centered a piece of furniture is in the image.  We begin by drawing the lines of thirds on an image.  After locating the intersection of these lines, we create a mask to select pixels which are close to the points of intersection, as shown in Figure XX.

Once we have identified the rule of thirds intersections, we can calculate the average HSV and saliency at these locations, producing 4 additional image features.  We can also calculate the distance between the highest saliency point and any of the intersections to determine how close the focal point is to one of the rule of thirds.

11) **Image Symmetry - 16 Features**

Measuring the symmetry of an image is accomplished by decomposing the image into symmetric ($I_s$) and antisymmetric ($I_a$) components.  The symmetric component of an image is produced by mirroring the image across the axis of symmetry, and adding it to the original image.  For instance, if we are measuring the horizontal symmetry of an image, we mirror the image across the vertical axis to produce the symmetric component.  Similarly, the antisymmetric component of an image is produced by mirroring the image across the axis of symmetry, and subtracting it from the original image.  The total symmetry of an image is calculated by taking the ratio of the total intensity of the symmetric component to the total intensity of both components:

Symmetry = $ \frac{\sum I_s^2 }{ \sum I_s^2 + \sum I_a^2 }$

We can apply the symmetry calculation to the full HSV and saliency channels of an image, as well as the HSV and saliency channels near the rule of thirds alone.  This produces an additional 16 image features for our model. (4 channels, measured in the full image or near rule of thirds, and measured in the horizontal and vertical directions)

12) **Number of Objects in an Image - 2 Features**

Our last two features assess how "busy" an image appears.  To accomplish this, we automatically threshold the image using [Otsu's Method](https://en.wikipedia.org/wiki/Otsu's_method) to produce a black and white picture.  Next, we draw contours around connected black pixels to identify individual objects within the picture.  This process works extremely well for background subtracted images (as shown in Figure XX), but can unintentionally divide single objects into multiple contours when the thresholding process is not straightforward (as shown in Figure XX).  To resolve this issue, we also keep track of the standard deviation of the centers of each contour.  A low standard deviation indicates that many of the contours were found in the same location, and that the individual contours most likely originate from a single object.


