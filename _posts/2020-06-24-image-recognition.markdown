---
layout: post
title:  "Image Recognition"
date:   2020-06-24 10:05:00 +0100
categories: project upload
---

### Overview

It has become apparent that Neural Networks are exceptionally well-suited for Image Recognition tasks, which is of great importance for Facial Recognition, Driverless Cars, and general Robotics.
Neural Networks are very capable of identifying specific features within an image, and adjusting internal weights and biases according to the relative importance of these features in identifying the image correctly.

The MNIST hand-written digit dataset is an ideal starting point for learning some of the concepts behind Neural Networks and Image Recognition. The training dataset is comprised of 42,000 labelled hand-written digits, stored as 28x28 greyscale images.

### Preparation

#### Data Structure

The training and test data exist as .csv files: with each row representing an individual image and the 785 columns representing the pixel values - with 1 column acting as the image label. It is necessary to isolate the target label from the training data prior to preparation. It is also necessary to convert the target variables to a categorical value, such that each outcome can exist as an individual neuron in the final prediction layer of the network.

{% highlight python %}

# Load data

dfTrain = pd.read_csv('{}/train.csv'.format(DATA_FILEPATH))
yTrain = dfTrain['label']
dfTrain.drop(columns = 'label', inplace = True)
TrainIndex = dfTrain.index

dfTest = pd.read_csv('{}/test.csv'.format(DATA_FILEPATH))
OriginalTestIndex = dfTest.index
dfTest.index += (dfTrain.index.max() + 1)
TestIndex = dfTest.index

# Convert labels to categorical

NumClasses = len(yTrain.unique())
y = keras.utils.to_categorical(yTrain, num_classes = NumClasses)

{% endhighlight %}

#### Reshaping

Subsequently, it is useful to reshape the raw data into a format appropriate for input to the network. In this case, the raw data can be reshaped into an "array of arrays", with each sub-array corresponding to one 28x28 greyscale image. As a result, only 1 channel is necessary, as opposed to the 3 channels occupied by RGB images.

{% highlight python %}

"""
DataPrep: A function to prepare the data into a more usable format
"""

def DataPrep(df):

    NumImages = df.shape[0]
    Array = df.values

    Array = Array.reshape(NumImages, NUM_COLS, NUM_ROWS, 1)
    Array = Array / 255

    return Array

{% endhighlight %}

It should be noted that the greyscale pixel values are divided by 255 (the maximum possible value) for normalisation purposes: such that each pixel value exists on a scale between 0 and 1. This is not strictly necessary for this task, although this is a useful concept in some other tasks to prevent inputs from certain sensors overpowering those from other sensors which produce lower absolute values.

### Modelling

I chose to use Keras' sequential model for this task. It is relatively simple in comparison to other Neural Network structures, and is described as being a "Linear stack of layers" by the library's documentation. Layers can be added individually to such a model:

{% highlight python %}

"""
BuildModel: A function to build and compile the neural network
"""

def BuildModel():

    Model = Sequential()

    Model.add(Conv2D(20, kernel_size = (3,3), activation = 'relu',
                     input_shape = (NUM_COLS, NUM_ROWS, 1)))
    Model.add(Conv2D(20, kernel_size = (3,3), activation = 'relu'))
    Model.add(Flatten())
    Model.add(Dense(28, activation = 'relu'))
    Model.add(Dense(NumClasses, activation = 'softmax'))

    Model.compile(optimizer = 'adam', loss = 'categorical_crossentropy',
                  metrics = ['accuracy'])

    return Model

{% endhighlight %}

#### Convolution Layers

A convolution layer applies a distortion or convolution to a small area of the image, which produces a tensor output. These are very useful for identifying small features which make up larger shapes and objects within the image: such as lines and curves.

- **Filter:** The number of different feature detection mechanisms applied to the image. In example above, both convolution layers use 20 filters.
- **Kernel Size:** The size of the convolution grid in pixels. A 3x3 convolution as used above will only be able to identify very basic features such as edges.
- **Activation:** The activation mechanism for neurons within the layer. Rectified Linear Unit (relu) activation signifies that the neuron will not activate until a certain threshold is reached, after which point its output will scale linearly with the input of the neuron.
- **Input Shape:** The input shape, equal to the output shape of the previous layer. This must be specified for the first layer of the network, but can be inferred subsequent to this.

#### Flatten Layers

These are used to flatten feature maps generated by convolution layers into a single column that can be passed to fully-connected layers in the network.

#### Dense Layers

Dense layers are "fully connected", such that each neuron is connected to every neuron from the previous layer. During creation of a dense layer, the shape of the output must be specified as well as the activation function to be used.

These can be thought of as condensing the information from previous layers to be passed to the prediction layer. The relative importance of features identified by convolution layers (after flattening) will be determined during adjustment of the model's weights and biases during fitting.

The final prediction layer exists as a dense layer: with the shape of the output being equal to the number of different image labels. In this layer the "SoftMax" function is used: which can be thought of as the probability that the network has determined the output neuron is the correct label for the provided input data.

#### Compiling

Prior to training, the model must be compiled. This uses TensorFlow's backend, and automatically determines an efficient way to compute and optimise model parameters (in addition to whether this should be performed on the CPU or GPU). Some hyper-parameters must be supplied in order for the model to compile:

- **Optimizer:** The gradient-descent algorithm to be used. This will have a strong influence on the learning rate of the model.
- **Loss:** The loss function to be calculated. The loss is equivalent to the absolute difference in desired output and generated output in the final layer of the model. In this example loss is minimised when the correct digit is assigned a probability (SoftMax activation) of 1 by the network, and every other node is assigned a probability of 0.
- **Metrics:** The metrics to monitor during model training.

#### Fitting

After model construction and compiling is complete, the model can be fitted:

{% highlight python %}

"""
Fit: A function to fit the model to the data
"""

def Fit(df, Model):

    Model.fit(TrainImages, y, batch_size = 128, epochs = 20, validation_split = 0.2)

    return

{% endhighlight %}

There are some parameters that must be specified during model fitting:

- **Batch Size:** The number of samples to model before the gradient descent optimizer is updated based on the model's loss.
- **Epochs:** The number of times the training data is iterated over.
- **Validation Split:** The proportion of training data used for validation purposes. This is useful to prevent overfitting, and can be used to gain a more accurate indication of where the specified metrics are likely to lie during model testing.

### Outcome

Once fitting is complete, predictions can be made and outputs can be passed to a '.csv' file ready for submission.

{% highlight python %}

"""
Predict: A function to make predictions
"""

def Predict(xTest):

    yPred = Model.predict(xTest)
    yPred = np.argmax(yPred, axis = 1)

    return yPred

{% endhighlight %}

The output of the prediction layer - denoted as 'yPred' - is an array whose shape is dependent on the number of possible labels. As such, it is necessary to determine the 'argmax' of this array: the value which the model calculates has the highest probability of being the correct label.

Once these predictions have been generated, they can be passed to a '.csv'.

{% highlight python %}

"""
Output: A function to output predictions for submission
"""

def Output(yPred, SUBMIT_FILEPATH, FileName, OriginalTestIndex):

    OutDf = pd.DataFrame({'Label' : yPred})
    OutDf.index = OriginalTestIndex + 1
    OutDf.index.rename('ImageId', inplace = True)

    OutDf.to_csv('{}/{}.csv'.format(SUBMIT_FILEPATH, FileName))

    return

{% endhighlight %}

These predictions achieved an accuracy of 98.5% on unseen test data. A sample from the Test dataset can be seen below.

{:refdef: style="text-align: center;"}
<img src="{{site.url}}/{{site.baseurl}}/assets/ImageRec/Index42Demo.png">
{: refdef}

### Adapting the model

An MNIST Fashion dataset is also available, which contains 60,000 labelled low-resolution images of clothing items from the retailer Zalando's catalogue. This is paired with 10,000 labelled test images. Because these images are in the same format as Digit dataset, it is trivial to fit the model to this new data.

{:refdef: style="text-align: center;"}
<img src="{{site.url}}/{{site.baseurl}}/assets/ImageRec/Fashion50Demo.png">
{: refdef}

The key different between these images and hand-written digits is that Arabic Numerals are specifically designed to be easily distinguishable. The images of clothing, however, are sufficiently low-resolution that they are difficult to distinguish even for humans. Nonetheless, without any adaptation the existing model achieved an accuracy of 90.6% on unseen test data.

I experimented with optimising the existing model to see if this score could be improved:

- Increasing the kernel size and filter count of the convolution layers to capture greater complexity of features within the images
- Increasing the quantity of convolution layers to allow the model to capture arrangements of different features more easily
- Increasing the dimensionality of the output of the Dense layer, to increase the quantity of information used to generate predictions at the expense of training time

Despite adjusting these parameters quite substantially, I was unable to produce a model that achieved more than 91.6% test accuracy (even when training accuracy was in excess of 99.5%). As far as I can tell, this indicates that the level of information at the input layer is insufficient to reliably generate a highly accurate model.

{:refdef: style="text-align: center;"}
<img src="{{site.url}}/{{site.baseurl}}/assets/ImageRec/Fashion3Demo.png">
{: refdef}

As an example of the difficulties faced when working with the Fashion dataset the image above was incorrectly classified as a Shirt, when its label indicates that it is in fact a Pullover. I think many humans would label this as a Coat or a Shirt, and so it is easy to see how the inaccuracies in the model's predictions arise.
