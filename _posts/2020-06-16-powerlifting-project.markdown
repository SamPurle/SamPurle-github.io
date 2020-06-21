---
layout: post
title:  "Powerlifting Data Analysis"
date:   2020-06-16 21:47:55 +0100
categories: project upload
---

### Overview
#### Motivation

With the spare time I have available as a result of the Covid-19 induced lockdown, I have been self-studying Python, data analysis, and Machine Learning. I have developed a number of Machine Learning models, primarily with pre-cleaned data and with a defined goal in mind such as the competitions found on [Kaggle](https://www.kaggle.com/). While building these models was useful to understand some core ML concepts and become familiar with Python syntax, I wanted a project with which I could set my own goals and identify a variety of uses for the underlying data. I also wanted to be genuinely interested in the subject matter, and decided upon Powerlifting as a perfect basis for my first self-directed project. The sport is rapidly growing (the dataset upon which I based this project contains 250,000 individual competitive performances), is exceptionally easy to quantify numerically, and quickly became my primary hobby after leaving University.

#### Project Scope

For this project I wanted to make use of the [OpenPowerlifting](https://www.openpowerlifting.org/) dataset, for the purposes of Exploratory Data Analysis,
rudimentary visualisations, and facilitate comparison between an individual's competitive performance and the wider Powerlifting community. Functionality that I have currently implemented includes:

 - **Data Cleaning:** Processing the raw .csv file into a more usable format
 - **Growth Modelling:** An illustration of how the number of unique competitors has changed over recent years
 - **General Trends:** Plotting trajectories of Total (a metric for absolute strength), Wilks score (a metric for relative strength) and Bodyweight over the course of individuals' competitive careers
 - **Lift Analysis:** Allowing an individual to identify where their individual lifts stand relative to the wider community, both in terms of percentile and variation from the respective means
 - **Competition Classification:** Basic use of a Machine Learning Clustering model to classify competitions into Regional, National, and International meets
 - **Performance Prediction:** Allowing a user to determine their likely placing at an upcoming competition based on historical data, or retroactively assess the standard of competition at a past meet
 - **Lift Dependency:** An investigation as to how athletes' dependence on the different lifts that constitute their Total varies with Bodyweight and Training Age
 - **Lift Correlations:** Visualisation of the inter-dependence of the different lifts

Things that I wish to introduce or improve upon includes:

 - **Growth Projections:** Develop a mathematical model to predict future growth of the sport under normal circumstances
 - **Refining the Machine Learning model:** Produce more accurate classifications with a more sophisticated model
 - **Allow visitor input:** Allow users to predict placing and analyse the competitiveness of their own lifts
 - **Formatting:** Make things more concise, legible, and visually appealing
 - **PED Detection:** Investigate how accurately a Machine Learning model could determine the likelihood that a competitor tests positive for PED use, based on their performance

 Each heading contains a hyperlink to the underlying Python code, viewable on Github.

### [Data Cleaning](https://github.com/SamPurle/PowerProject/blob/master/Cleaning.py)

The OpenPowerlifting dataset is community-maintained, with a variety of different managers that are responsible for uploading competition data from many different federations as they occur across 65 different countries. It appears the site utilises a relational database at the back-end, as individual competitors are able to link a variety of different Social Media accounts to their competitive profiles, which in turn are tied to Meets in which they have competed. This infrastructure is not available to site visitors however, and the raw data is available as a simple '.csv' file.

The presence of many different federations within the raw data has the consequence that competitors do not have a Unique ID tied to them, and can only be identified by name. As such, detection of individual competitors is entirely dependent on the name they elected to use while entering any given competition. As an example of this I appear as two distinct competitors within the dataset: as Samuel Purle on 2019-04-06, and then as Sam Purle on 2020-01-26. This will introduce a degree of noise into the raw data, the removal of which I do not believe to be possible without the use of extensive and complex string operations. Even subsequent to this, the issue would not be entirely alleviated due to the presence of competitors from many different countries and cultures within the data - where naming conventions differ substantially from those in the West.

### [Growth of Powerlifting](https://github.com/SamPurle/PowerProject/blob/master/GrowthModelling.py)

Powerlifting as a sport has experienced rapid growth over recent years. There are likely a multitude of reasons for this. One of which has been the growing popularity of "Raw" lifting as opposed to the original "Equipped" lifting, which has resulted in the sport becoming more accessible to the gym-going population. Additionally, health and fitness has become a prominent subject on a variety of social media platforms (most notably Instagram and YouTube), which has resulted in greater awareness of the sport.

{:refdef: style="text-align: center;"}
<img src="{{site.url}}/{{site.baseurl}}/assets/SportGrowth.png">
{: refdef}

It will be interesting to see the impact which the Covid-19 pandemic has upon the influx of new competitors to the sport.

### [General Trends](https://github.com/SamPurle/PowerProject/blob/master/Plotting.py)

After data cleaning and calculating the Elapsed Time since competitors' first Meets, it is possible to plot the general trajectories followed by individuals over the course of their competitive careers. I originally planned to identify an inflexion point where the positive impact of Training Age begins to be surpassed by the negative impact of biological aging, although unfortunately there is insufficient information relating to biological age within the dataset. Only data on Age Class is present, which is not specific enough to be useful for this purpose (the 24-39 year old "Senior" class is far too wide to be able to identify any meaningful trend).

The three metrics I wish to investigate are:

- Total - a metric for Absolute Strength
- Wilks - a metric for Relative Strength
- Bodyweight

#### Total

A lifter's Total is generally their primary competitive goal - to lift the highest combined total between their Squat, Bench Press and Deadlift.

{:refdef: style="text-align: center;"}
<img src="{{site.url}}/{{site.baseurl}}/assets/TotalPlot.png">
{: refdef}


- Both sexes see a significant increase in Total over the first couple of years of competing. This plateaus in Men after ~2 years of competing, but continues to increase marginally in Women until ~6 years into competing.
- Both sexes see a significant drop in absolute strength after 8 years or more of competing, presumably as the effects of biological aging outpace the improvements made with increased training age.



#### Wilks

The Wilks score was developed as a method of comparing the relative strengths of lifters across multiple weight classes and genders. It is effectively a "power to weight ratio" used to score performance within competitive Powerlifting. Although lean muscle mass is an enormous component in determining a lifter's ability to generate force, other factors also influence this:

 - Muscle fibre diameter
 - Actin : Myosin ratio within muscle fibres
 - Motor Neuron Recruitment, which is heavily influenced by adrenaline
 - Angle of interplay between tendons and attached bones (with can vary significantly as muscles grow throughout an athlete's career)

{:refdef: style="text-align: center;"}
<img src="{{site.url}}/{{site.baseurl}}/assets/WilksPlot.png">
{: refdef}


- On average Men have an initial Wilks score roughly 15% higher than Women.

- Over the course of their competitive careers, on average Women see a 33% increase in Wilks score (peaking at around 380 ~8 years into competing), relative to 18% in Men (peaking at 390 ~3 years into competing).

The fact the Women have a lower initial Wilks could be due to unfairly calculated coefficients, but looking at other plots it makes more sense that Women start competing earlier on in their lifting careers than Men.

#### Bodyweight

It is relatively commonplace for lifters to undertake the decision to move towards heavier weight classes as they progress in their lifting careers. This is in part a by-product of the natural propensity to gain muscle through resistance training, but also due to phenomenon that lifters will generally become more competitive as they gain body mass. An analysis of the biomechanics involved shows that - all other things being equal - taller lifters have to generate more force with a smaller mass of muscle to lift an equivalent weight as a shorter, stockier competitor. Consequently, in order to maximise their competitive potential a lifter should aim to be the shortest, heaviest lifter in their weight class.

{:refdef: style="text-align: center;"}
<img src="{{site.url}}/{{site.baseurl}}/assets/BodyweightPlot.png">
{: refdef}

- Male lifters tend to put on 5-10kg of weight in their first 2-4 years of competing. Average body-weight in Men continues to increase after their Total plateaus, which explains Men's Wilks scores peaking much earlier than Women's.

- Bodyweight of Female lifters doesn't change significantly over the course of their competitive careers, aside from a slight decrease towards the end.

Although a social scientist would be better equipped to explain this, this could either be due to Women being more disciplined on average in terms of maintaining a low body fat percentage, or being less willing to put on weight to chase after a perceived competitive advantage.

### [Lift Analysis](https://github.com/SamPurle/PowerProject/blob/master/LiftAnalysis.py)
#### Comparison with the wider community

The relative strengths of a individual's lifts have substantial implications on way they will approach a competitive situation, as well as providing insight on areas on which to focus training and potentially review technique. Biomechanics and a lifter's bodily proportions play a significant role in determining which lifts an individual will naturally favour, and it is somewhat rare for an individual to be equally strong in all three lifts relative to the general Powerlifting population.

Although similar tools currently exist online, these are of limited use for individuals interested in lifting competitively. A novice/intermediate Powerlifter will often be classified as having advanced/elite level strength by such tools.

{:refdef: style="text-align: center;"}
<img src="{{site.url}}/{{site.baseurl}}/assets/LiftPlot.png">
{: refdef}

This overview can be further broken down by showing the distributions of each individual lift:

{:refdef: style="text-align: center;"}
<img src="{{site.url}}/{{site.baseurl}}/assets/SquatDist.png">
{: refdef}

{:refdef: style="text-align: center;"}
<img src="{{site.url}}/{{site.baseurl}}/assets/BenchDist.png">
{: refdef}

{:refdef: style="text-align: center;"}
<img src="{{site.url}}/{{site.baseurl}}/assets/DeadliftDist.png">
{: refdef}

{:refdef: style="text-align: center;"}
<img src="{{site.url}}/{{site.baseurl}}/assets/TotalDist.png">
{: refdef}


#### Takeaways

The above plots help quantify what I already suspected: my Squat is by far my biggest competitive advantage, which I would benefit from leveraging by placing other competitors on the back foot after the first event within a competition.

Despite substantial effort on my part, my Bench Press continues to be a competitive weak-point. I believe this is primarily due to past injury and naturally long arms relative to my height - which causes a loaded barbell to create a greater turning moment at my shoulder. My natural leaning towards lower-body strength limits the amount of muscle I can carry around my torso while staying in the lighter weight classes. As such, I expect to see an improvement in the competitiveness of my Bench Press as I move up in Weight Class and the effect of this limitation is mitigated.

My Deadlift is marginally stronger than other competitive lifters in my weight class. I believe the lower body strength that provides the foundation for my Squat to be the primary reason for this. Moving forwards, I plan to capitalise on this strength by continuing to develop my "Sumo" Deadlift technique - which differs substantially from my current "Conventional" technique and is generally considered to increase involvement of the quadriceps, which are also a large component of the Squat movement.

### [Clustering](https://github.com/SamPurle/PowerProject/blob/master/ClusteringModel.py)
#### A Machine Learning model to classify Competitions

There is a high level of variation in the calibre of lifters seen at Competitions, between federations, countries, and even between weight classes - such as the infamously "stacked" 83kg IPF class where a large number of Men fall due to the natural size of their frames.

In order to predict placing at an upcoming competition, it would be useful to be able to approximate the level of difficulty that is expected. Clearly a National level Meet with strict Qualifying Totals will see a higher standard of competitions than a Regional University Meet, where the majority of lifters are only just beginning their lifting careers and are likely to fall in the Junior and Sub-Junior age classes.

{:refdef: style="text-align: center;"}
<img src="{{site.url}}/{{site.baseurl}}/assets/ClusterPlot.png">
{: refdef}

Above is a representation of how the model works, and is not indicative of its actual parameters and classifications. The model currently in place uses more than 2 parameters to generate predictions, and so cannot easily be visualised in a 2D plot. I originally considered approaching this problem by developing a Random Forest Classifier and providing a small list of labelled Competitions that could be extrapolated to the dataset as whole. However, providing a small training set would produce large inaccuracies, whereas generating a large training set would be very labour-intensive.

### [Performance Predictor](https://github.com/SamPurle/PowerProject/blob/master/PerformancePredictor.py)
#### A tool to predict competitive performance

Even for lifters that enter competitions with their own self-determined goals other than placing, it is useful to be able to approximate where they are likely to fall within the Meet's rankings. Information on where a lifter's total falls within the general population's distribution can be combined with a description of the level of the meet and labels from the clustering algorithm above. This can be used to calculate the percentile on which a competitor falls among historical competitive performances from lifters in their weight class competing at their level. A probability distribution can then be generated based on the number of confirmed competitors attending an upcoming meet.

To generate this distribution it was necessary to make use of the mathematical "choose" function in order to correctly calculate probability. There is only one way in which a competitor can place 1st - by beating every other competitor in their class. However, there are many different way in which a competitor could place 2nd - any one of the other competitors in their class could place higher than them.

{:refdef: style="text-align: center;"}
<img src="{{site.url}}/{{site.baseurl}}/assets/PlacingPlot.png">
{: refdef}

The plot above shows my predicted placing amongst a competition consisting of twelve 74kg lifters at a Regional level, based on my pre-lockdown lifts. Based on past experience this appears to be a relatively accurate prediction.

### [Lift Dependency](https://github.com/SamPurle/PowerProject/blob/master/LiftDependency.py)
#### Weight Class

{:refdef: style="text-align: center;"}
<img src="{{site.url}}/{{site.baseurl}}/assets/M_Dependency.png">
{: refdef}

{:refdef: style="text-align: center;"}
<img src="{{site.url}}/{{site.baseurl}}/assets/F_Dependency.png">
{: refdef}

It is notable that the reliance upon the Deadlift decreases with increasing bodyweight for both Men and Women. This may be due to the fact that heavier athletes are likely to be taller than their lighter counterparts, which will place their spine and hips at a less mechanically efficient position to provide the necessary acceleration off the floor during the lift. Furthermore, a wider torso can prohibit effective use of the Sumo Deadlift, and may cause athletes to adopt a wider-than-ideal grip in order to prevent friction between their arms and their larger torsos.

In Men, the decrease in dependency upon the Deadlift is made up for with both the Squat and the Bench Press. In Women however, this decrease is only accounted for with an increase in the Squat. It is theorised that a larger torso and stomach may produce a "bounce" out of the bottom of the Squat in heavier athletes, which would go some way to explaining the observed phenomenon.

Both Men and Women have a limit on the mass of muscle they are able to carry upon their frames, without the use of performance enhancing drugs. This is generally governed by the individual's Myostatin concentrations within the body. It is possible that this prevents Women from carrying excessive amounts of muscle around their upper bodies, as this may have proved to be an evolutionary disadvantage throughout the majority of mankind's history. This is purely speculative to explain the observed trends - I plan to carry out further research to better understand this.

#### Years competing

{:refdef: style="text-align: center;"}
<img src="{{site.url}}/{{site.baseurl}}/assets/M_AgeDependency.png">
{: refdef}

{:refdef: style="text-align: center;"}
<img src="{{site.url}}/{{site.baseurl}}/assets/F_AgeDependency.png">
{: refdef}

For both Men and Women there are slight but sustained variations in their dependence on the different lifts: a decrease in reliance upon the Deadlift and an increase in reliance upon the Bench Press. I would argue that this is likely due to the fact that the Conventional Deadlift is probably the least technical lift within Powerlifting, whereas the Bench Press is generally considered to be the most technical. All lifts are likely to increase over time as an athlete gains strength. However the Bench Press is likely to benefit disproportionately as an athlete also develops in skill, whereas this effect is likely to be less prominent in the Deadlift.

### [Lift Correlations](https://github.com/SamPurle/PowerProject/blob/master/InverseCorrelations.py)

As mentioned earlier, the amount of weight an individual is able to lift is dependent on a wide variety of factors. Many of these are related to training and nutrition, but bodily proportions and leverages also have a substantial impact on which lifts a competitor will naturally excel at. Leverages that provide an advantage in one lift may prove to be a disadvantage in others.

Demonstrating these relationships was not an easy task. Simply plotting the correlation between one lift and another produced a false positive correlation: as stronger athletes will generally perform well across all three lifts. Conversely, plotting lifts as a percentage of Total against each other produced a false negative correlation: as the combined percentage must always sum to 100%.

I decided to instead plot the percentiles on which athletes lie for each lift against each other. If no inverse correlation exists between the lifts then a 1:1 relationship should be observed: an athlete whose Squat lies on the 50th percentile should have a Bench Press that also lies on the 50th percentile, and so forth. Deviations from this expected trend show that the factors that make an athlete strong in one lift simultaneously detract from their performance in another.

{:refdef: style="text-align: center;"}
<img src="{{site.url}}/{{site.baseurl}}/assets/Bench-Deadlift.png">
{: refdef}

The observed trend between the Bench Press and Deadlift should be no surprise to competitors in Powerlifting. The primary reason for this is likely due to the conflicting impact of arm length. Long arms allow a competitor to set-up in a more upright and mechanically strong position during the Deadlift, but simultaneously create a greater turning force at the shoulder during the Bench Press which the athlete must overcome.

{:refdef: style="text-align: center;"}
<img src="{{site.url}}/{{site.baseurl}}/assets/Squat-Deadlift.png">
{: refdef}

This relationship is almost a direct 1:1 correlation between the two lower-body centric lifts. The Squat and the Deadlift utilise many of the same muscle groups, and so strength in one is highly likely to carry over to the other.

{:refdef: style="text-align: center;"}
<img src="{{site.url}}/{{site.baseurl}}/assets/Squat-Bench.png">
{: refdef}

There appears to be a slight counter-correlation between the Squat and the Bench Press. From personal experience, I believe it is possible that athletes that have relatively strong lower bodies may choose to focus their training upon the Squat and Deadlift leading to their Bench Press lagging behind their other lifts.  

### Final Thoughts

#### Positives

I'm very happy that the level of difficulty of this project was appropriate for my skill level. I did not have to acquire much new knowledge for the tasks that I wanted to accomplish - instead I found myself applying existing knowledge in new ways. There are some exceptions to this: I was not aware of the "groupby" function within Pandas, and had never built a K-Means clustering model prior to this.

#### Limitations

In truth, I do not believe Python libraries to be the ideal visualisation tool for this project. Other technologies would be capable of presenting the data more legibly, and potentially adding a degree of interactivity. This being said, they are a very powerful tool which can be used to visualise basic trends, and can be used easily during data processing.

Originally I had planned to use the data to develop a complex time-series Machine Learning model: capable of predicting the performance trajectory of a competitor over their career. However, there was insufficient data for this purpose, as on average each competitor had only completed 2-3 registered competitions. This explains my inclusion of the clustering model: despite its limited utility I wanted to implement some form of Machine Learning into my project which could impact the visualisations and insights generated.

#### Changes

If I were to repeat this project I would think more carefully about the structure of my code-base. Part-way through I found that I had to re-structure my code to make things modular to allow modification of individual functions, which I consider to be a beginner's error. I would also be more careful to plan out the questions to be answered from the outset, so that work could be done chronologically/in a logical order. However, I found that I came up with many new questions as I became more familiar with the dataset.
