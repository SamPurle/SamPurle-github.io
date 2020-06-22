---
layout: post
title:  "Titanic - Random Forest Classification to predict survival"
date:   2020-06-19 12:53:00 +0100
categories: project upload
---

### overview

This project aims to make use of a Random Forest Classifier model to predict whether passengers survived the Titanic disaster, based on a variety of factors. Although somewhat morbid, this provides a good introduction to Machine Learning model construction, and key concepts such as Cleaning and Encoding.

### Data Structure

The raw data is available in two separate '.csv' files: for Train and Test data. The training data contains the "ground truth": whether the passenger survived the disaster or not, represented as a binary value. The other columns are:

- Passenger Class: The class of ticket held by the passenger, which can be used as a proxy for Socio-Economic Status
- Name: The passenger's Name
- Sex: The passenger's biological Sex
- Age: The Age of the passenger
- Siblings and Spouses ("SibSp"): The number of Siblings and Spouses of the passenger that were also on-board
- Parents and Children ("Parch"): The number of Parents and Children of the passenger that were also on-board
- Ticket: The passenger's ticket number
- Fare: The fare paid by the passenger to obtain their Ticket
- Cabin: The passenger's Cabin Number
- Embarked: The port of Embarkation

### Cleaning

The first stage during data analysis is to "clean" the data, and ensure that it exists in a usable format. For this purpose I find it useful to separate the features used for prediction from the Ground Truth, and concatenate the two DataFrames. This allows cleaning operations to be performed on the two datasets simultaneously, and prevents issues arriving during encoding (when a categorical value appears in one dataset but not the other). It is important to maintain the ability to separate the two datasets after Cleaning and Feature Engineering to avoid cross-contamination. In this case it is possible to simply record the "PassengerId" index for each original DataFrame.

Next, it is necessary to identify which columns contain sufficient information to be of use during model construction.

{{site.data.Nulls}}
