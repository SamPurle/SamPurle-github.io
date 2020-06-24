---
layout: post
title:  "Image Recognition"
date:   2020-06-24 10:05:00 +0100
categories: project upload
---

### Overview

It has become apparent that Neural Networks are exceptionally well-suited for Image Recognition tasks, which is of great importance for Facial Recognition, Driverless Cars, and general Robotics.
Neural Networks are very capable of identifying specific features within an image, and adjusting internal weights and biases according to the relative importance of these features in identifying thee image correctly.

The MNIST hand-written digit dataset is an ideal starting point for learning some of thee concepts behind Neural Networks and Image Recognition. The training dataset is comprised of 42,000 labelled hand-written digits, stored as 28x28 greyscale images.

### Preparation

#### Data Structure

The training and test data exist as .csv files: with each row representing an individual image and the 785 columns representing the pixel values - with 1 column acting as the image label. It is necessary to isolate the target label from the training data prior to preparation. It is also necessary to convert tthe target variables to a cattegorical value, such that each outcome can exist as an individual neuron in the final prediction layer of the network.

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

Subsequently, it is useful to reshape the raw data into a format appropriate for inputting to the network. In this case, the raw data can be reshaped into an "array of arrays", with each sub-array corresponding to one 28x28 greyscale image. As a result, only 1 channel is necessary, as opposed to the 3 channels occupied by RGB images.

{% highlight python %}

def DataPrep(df):

    NumImages = df.shape[0]
    Array = df.values

    Array = Array.reshape(NumImages, NUM_COLS, NUM_ROWS, 1)
    Array = Array / 255

    return Array

{% endhighlight %}


### Modelling
