---
layout: post
title:  "Titanic - Random Forest Classification to predict survival"
date:   2020-06-19 12:53:00 +0100
categories: project upload
---

### Overview

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

Next, it is necessary to identify which columns contain sufficient information to be of use during model construction. The Null values in each column can be counted and divided by the length of the DataFrame, which will show the percentage of values in each column containing Nulls. For this model, any column containing 25% or more Null values was dropped from the dataset.

{{site.data.Nulls}}

The only column which satisfies this condition is the "Cabin" column, which contains 77% Nulls. This is far too high a percentage for Imputing to be appropriate or effective, and so this data cannot be included in the model. This leaves 3 columns remaining for which Imputing is necessary:

- Age
- Embarked
- Fare

### Extraction

In some cases, it may be necessary to extract general information from more specific data within the dataset. In this example, I decided it was appropriate to extract a passenger's title from their Name. The exact Name of a passenger is unlikely to be of any use, whereas their Title can be used as a proxy for Socio-Economic Status, Age, Marital Status and Profession.

This was a relatively simple task which necessitated the use of string operations. This was made easy due to the fact that the naming conventions were entirely uniform throughout the dataset, and would have been much more complex had the Passenger Names been recorded differently (potentially involving string indexing from a Dictionary of common titles across different countries and cultures).

{% highlight python %}

"""
Title: A function to extract title from Passenger Name
"""

def Title(df, FANCY_THRESH):

    # Extract title from name

    df['Title'] = df['Name'].str.split(', ', expand = True)[1].str.split('.', expand = True)[0]
    df.drop(columns = 'Name', inplace = True)

    # Group rare titles into one category

    TitleCounts = df.groupby(['Sex', 'Title']).Title.count()

    FancyTitles = TitleCounts.loc[TitleCounts.values < 10]

    MaleFancy = FancyTitles.loc['male']
    MaleIndexReplace = df.join(MaleFancy, on = 'Title', how = 'inner', rsuffix = 'F').index
    df.loc[MaleIndexReplace, 'Title'] = 'FancyMan'

    FemaleFancy = FancyTitles.loc['female']
    FemaleIndexReplace = df.join(FemaleFancy, on = 'Title', how = 'inner', rsuffix = 'F').index
    df.loc[FemaleIndexReplace, 'Title'] = 'FancyLady'

    return df

{% endhighlight %}

There were 4 common titles within the Dataset: "Mr" and "Master" for males and "Mrs" and "Miss" for females. There were many uncommon titles within the Dataset, such as "Jonkheer" and "The Countess" - all of which implied high Socio-Economic Status and none of which pertained to 10 passengers or more. As such, these titles were grouped together as "FancyMan" for males and "FancyLady" for females.

### Visualisation

### Imputing

Prior to encoding and modelling the data, it is necessary to specify how null values are to be handled. Although this can be done using the SimpleImputer object from the sklearn library, I found that better results were achieved through pandas' fillna function to calculate continuous values. The Embarkation column was imputted using the SimpleImputer object with the "most_frequent" strategy.

{% highlight python %}

"""
CatImpute: A function to impute nulls
"""

def CatImpute (df, NULL_THRESH):

    NullPer = df.isnull().sum() / len(df)
    NullPer.sort_values(ascending = False, inplace = True)        
    NullCols = NullPer.loc[NullPer > NULL_THRESH].index      
    df.drop(columns = NullCols, inplace = True)

    df['Age'].fillna(df.groupby(['Sex','Pclass'])['Age'].transform('mean'), inplace = True)
    df['Fare'].fillna(df.groupby(['Sex','Pclass'])['Fare'].transform('mean'), inplace = True)
    df['Embarked'] = SimpleImputer(strategy = 'most_frequent').fit_transform(df[['Embarked']])

    return df

{% endhighlight %}

### Encoding

After null values within the dataset have been handled, it is necessary to encode the categorical features such that they can be processed by Machine Learning algorithms.

{% highlight python %}

"""
Encode: A function to encode categorical variables
"""

def Encode(df, CAT_NUM):

    CatCols = list(df.columns.drop(df._get_numeric_data().columns))
    CatCols.extend(CAT_NUM)

    Enc = OneHotEncoder(sparse = False)

    for x in CatCols:
        EncCol = Enc.fit_transform(df[[x]])
        EncNames = Enc.get_feature_names([x])
        EncDf = pd.DataFrame(EncCol)
        EncDf.columns = EncNames
        EncDf.index = df.index

        df = df.join(EncDf)    
        df.drop(columns = x, inplace = True)

    return df

{% endhighlight %}
