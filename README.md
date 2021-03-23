# Is this image fake?

Using machine learning to detect digitally altered images with Error Level Analysis.

## Error Level Analysis: <a id="ela"></a>

One powerful open sourced algorithm to help us is called Error Level Analysis.

It is a forensic method to identify portions of an image that has different levels of compression.  It will allow us to see areas of a photo that has been altered or changed.  We can use this technique to determine if a picture has been digitally modified.  The added fake contents (layers) on top of an image is different from that of the original image and most importantly, ELA can detect this.  If an image has not been modified, the altered grid should be at a higher error potential in respect to the remaining part of the image.

ELA works by re-saving the image at 90% - 95% compression and compares the difference between the original and the compressed.  Modified areas are easily seen in the ELA representation.

![Original](images/ela1.png "Original")(Image in question)
![ELA](images/ela2.png "ELA")(ELA representation)

Using the ELA image, we now can have a *common factor* among the fake images in hopes that our model can learn these signals.  Since reading ELA images requires a trained eye as the image can produce a wide range of variations, and painfully the process is not automated, we can leverage machine learning models to assist us.

<sup>Note: This technique is not perfect and we will go over the caveats in the 'What's next?' section.</sup>


## Data Flow: <a id="data"></a>

To start, we need data, more specifically, we need fake and real images.  

Fake images are described as images that have been digitally altered in any way.  This includes images being touched up or going through Photoshop.  Real images are described as images that are not altered.

The distributions of these sources:
![sources](images/sources.png "Sources")

- 2009 Fake images from Imgur.
- 946 Real images from iPhone.
- 550 Real images from DSLR.
- 512 Real images from Imgur.


Here is our data flow on how the model was trained and tested on:


![training](images/training.jpg "Training")

---

![testing](images/testing.jpg "Testing")

## Model Selection: <a id="selection"></a>

Now that we have gathered our data and a general flow on how to train our model, we can now use the same pipeline to test on multiple models.

Since this is a classification (predicting fake or not fake) problem, these models were taken into consideration:

* Logistic Regression
* Random Forest
* Gradient Boosting
* Convolutional Neural Network

Here are the results displayed under a ROC Curve.  A ROC Curve is a way to visual the performance of the binary classifier model.  The area under the curve (AUC) measures the performance of a binary classifier averaged across all possible decision thresholds.

![ROC](images/ROC_Curve.png "ROC")

<sup>Generally greater area under the curve, the better.</sup>

As you can see, **Convolutional Neural Network** gives us the best accuracy (94%) from our test set and has the highest AUC value.  Both the precision and recall are also high.  

In our case, **precision** ``` (TP)/(TP+FP) ``` tells us the proportion of images that we classified as fake are *actually* fake.  

**Recall** ``` (TP)/(TP+FN) ``` tells us what proportion of images that *actually* were fake were predicted by model as being fake.

![CNN F1](images/CNN_F1.png "CNN_F1")


We will continue by using Convolutional Neural Network as our model selection.

<sup>Please reference the EDA.ipynb file for a more detailed approach.</sup>

## Convolutional Neural Network: <a id="cnn"></a>

With the common factor among the fake images from ELA, we can now do some preprocessing on the ELA image and feed it into the convolutional neural network (CNN).  

CNN is primarily used for image classification because it has the ability to learn basic things liked edges, dots, bright spots, and dark spots.  When a computer views an image, it sees an array of values depending on the size of the image.  The pixel values itself is between 0 to 225 which describes the pixel intensity.  We will feed in our CNN model with an array of numbers and the goal is for the model to learn the pattern and output the probability of the image as fake.

Our network consists of 1 input layer, 1 hidden layer, and 1 output layer with Dropout in each layer to reduce overfitting by preventing a layer from seeing twice the exact same pattern.

![summary](images/cnn_summary.png "cnn summary")

## Results: <a id="results"></a>
### 94% accuracy with Convolutional Neural Network.

A closer dive into our model,

![CNN CF](images/cnn_confusion_matrix.png "CNN CF")

According to our confusion matrix.  We have:

- High True Positives
- High True Negatives
- Low False Positives
- Low False Negatives

This is indicative a good model.  Because of the limitations of Error Level Analysis (for example successive resave operations will reach a minimum lower level), we expect to have a slightly higher level false negative, where model will think the image as real, but in reality, a fake.

![CNN Loss](images/binary_loss.png "CNN Loss")
![CNN Accuracy](images/binary_acc.png "CNN Accuracy")

Measuring loss function is very important as it is used to measure the inconsistency between predicted values.  It implies how well or poorly the model behaves after each iteration of optimization.  Ideally, we should expect the reduction of loss after several iterations.  

As the loss value converges around 20 epochs, the test accuracy is greater than training accuracy.  This means that the model has generalized very well with 94% accuracy.


## References: <a id='ref'></a>

* [https://ijcsits.org/papers/vol7no22017/4vol7no2.pdf](https://https://ijcsits.org/papers/vol7no22017/4vol7no2.pdf)
* [https://fotoforensics.com/tutorial-ela.php](https://fotoforensics.com/tutorial-ela.php)
* [http://cs231n.github.io/convolutional-networks/](http://cs231n.github.io/convolutional-networks/)


## Steps:
1) Get all real images
2) Get all the fake images.  If from imgur, use "gen-imgur-csv.py".
3) Download the images from CSV. "download_images_from_csv.py"
4) Generate ela images by "ela-image.py"
5) Resize ela images by "resize-images.py"
