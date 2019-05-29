
# Practical Machine Learning Assignment

### Ehsan Kourkchi

#### May 28, 2019

# Problem Defenition

Data on human activity during workout sessions are acquired using multiple devices attached to human body and dumbbells. The goal is to build a model to predict the quality of performance when using dumbbells. 

More infomation on the provided data is available [Here](http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har).

Wearable devices provide inforamtion on the movements and accelaration of waist, wrist, forearm and dumbbell. The quality of executation is then alphabetically labelled as following 

Class A. exactly according to the specification
Class B. throwing the elbows to the front
Class C. lifting the dumbbell only halfway
Class D. lowering the dumbbell only halfway
Class E. throwing the hips to the front

## Loaing data

We use R packages ggplot2 for plotting and caret for implementing our model.


```R
library(ggplot2)
library(caret)


Training <- read.csv(file="pml-training.csv", header=TRUE, sep=",")
Testing  <- read.csv(file="pml-testing.csv" , header=TRUE, sep=",")

dim(Training)
dim(Testing)
```


<ol class=list-inline>
	<li>19622</li>
	<li>160</li>
</ol>




<ol class=list-inline>
	<li>20</li>
	<li>160</li>
</ol>



## Slicing the Training Data

In order to be able to peroform some out-of-training evaluation, we only use 70% of our training data for training our model and leave out the rest for cross validation.

We perform the best model on the Testing data at the end. 


```R
inTrain <- createDataPartition(y=Training$classe, p=0.7, list=FALSE)
training <- Training[inTrain,]
crossvalidation <- Training[-inTrain,]
```

## Checking out what we've got

A quick investigation using Open Office excell shows that there are many columns that are either empy or filled with "NA".
These columns are also missing in the Test data we have laoded. Therefore there is no point to use these columsn for training our model. 
Therefore we refine our data sets to only contain the relevant healthy data points. 


```R
summary(training$classe)

print("Dimension of Training Set: ") ; dim(training)
print("Dimension of Cross Validation Set: ") ; dim(crossvalidation)

all_features = names(training)
for (i in seq_along(all_features)) {
    
    cat(sprintf("Quiz #%d \t \"%s\"\n", i, all_features[i]))
    
} 

str(training)
```


<dl class=dl-horizontal>
	<dt>A</dt>
		<dd>3906</dd>
	<dt>B</dt>
		<dd>2658</dd>
	<dt>C</dt>
		<dd>2396</dd>
	<dt>D</dt>
		<dd>2252</dd>
	<dt>E</dt>
		<dd>2525</dd>
</dl>



    [1] "Dimension of Training Set: "



<ol class=list-inline>
	<li>13737</li>
	<li>160</li>
</ol>



    [1] "Dimension of Cross Validation Set: "



<ol class=list-inline>
	<li>5885</li>
	<li>160</li>
</ol>



    Quiz #1 	 "X"
    Quiz #2 	 "user_name"
    Quiz #3 	 "raw_timestamp_part_1"
    Quiz #4 	 "raw_timestamp_part_2"
    Quiz #5 	 "cvtd_timestamp"
    Quiz #6 	 "new_window"
    Quiz #7 	 "num_window"
    Quiz #8 	 "roll_belt"
    Quiz #9 	 "pitch_belt"
    Quiz #10 	 "yaw_belt"
    Quiz #11 	 "total_accel_belt"
    Quiz #12 	 "kurtosis_roll_belt"
    Quiz #13 	 "kurtosis_picth_belt"
    Quiz #14 	 "kurtosis_yaw_belt"
    Quiz #15 	 "skewness_roll_belt"
    Quiz #16 	 "skewness_roll_belt.1"
    Quiz #17 	 "skewness_yaw_belt"
    Quiz #18 	 "max_roll_belt"
    Quiz #19 	 "max_picth_belt"
    Quiz #20 	 "max_yaw_belt"
    Quiz #21 	 "min_roll_belt"
    Quiz #22 	 "min_pitch_belt"
    Quiz #23 	 "min_yaw_belt"
    Quiz #24 	 "amplitude_roll_belt"
    Quiz #25 	 "amplitude_pitch_belt"
    Quiz #26 	 "amplitude_yaw_belt"
    Quiz #27 	 "var_total_accel_belt"
    Quiz #28 	 "avg_roll_belt"
    Quiz #29 	 "stddev_roll_belt"
    Quiz #30 	 "var_roll_belt"
    Quiz #31 	 "avg_pitch_belt"
    Quiz #32 	 "stddev_pitch_belt"
    Quiz #33 	 "var_pitch_belt"
    Quiz #34 	 "avg_yaw_belt"
    Quiz #35 	 "stddev_yaw_belt"
    Quiz #36 	 "var_yaw_belt"
    Quiz #37 	 "gyros_belt_x"
    Quiz #38 	 "gyros_belt_y"
    Quiz #39 	 "gyros_belt_z"
    Quiz #40 	 "accel_belt_x"
    Quiz #41 	 "accel_belt_y"
    Quiz #42 	 "accel_belt_z"
    Quiz #43 	 "magnet_belt_x"
    Quiz #44 	 "magnet_belt_y"
    Quiz #45 	 "magnet_belt_z"
    Quiz #46 	 "roll_arm"
    Quiz #47 	 "pitch_arm"
    Quiz #48 	 "yaw_arm"
    Quiz #49 	 "total_accel_arm"
    Quiz #50 	 "var_accel_arm"
    Quiz #51 	 "avg_roll_arm"
    Quiz #52 	 "stddev_roll_arm"
    Quiz #53 	 "var_roll_arm"
    Quiz #54 	 "avg_pitch_arm"
    Quiz #55 	 "stddev_pitch_arm"
    Quiz #56 	 "var_pitch_arm"
    Quiz #57 	 "avg_yaw_arm"
    Quiz #58 	 "stddev_yaw_arm"
    Quiz #59 	 "var_yaw_arm"
    Quiz #60 	 "gyros_arm_x"
    Quiz #61 	 "gyros_arm_y"
    Quiz #62 	 "gyros_arm_z"
    Quiz #63 	 "accel_arm_x"
    Quiz #64 	 "accel_arm_y"
    Quiz #65 	 "accel_arm_z"
    Quiz #66 	 "magnet_arm_x"
    Quiz #67 	 "magnet_arm_y"
    Quiz #68 	 "magnet_arm_z"
    Quiz #69 	 "kurtosis_roll_arm"
    Quiz #70 	 "kurtosis_picth_arm"
    Quiz #71 	 "kurtosis_yaw_arm"
    Quiz #72 	 "skewness_roll_arm"
    Quiz #73 	 "skewness_pitch_arm"
    Quiz #74 	 "skewness_yaw_arm"
    Quiz #75 	 "max_roll_arm"
    Quiz #76 	 "max_picth_arm"
    Quiz #77 	 "max_yaw_arm"
    Quiz #78 	 "min_roll_arm"
    Quiz #79 	 "min_pitch_arm"
    Quiz #80 	 "min_yaw_arm"
    Quiz #81 	 "amplitude_roll_arm"
    Quiz #82 	 "amplitude_pitch_arm"
    Quiz #83 	 "amplitude_yaw_arm"
    Quiz #84 	 "roll_dumbbell"
    Quiz #85 	 "pitch_dumbbell"
    Quiz #86 	 "yaw_dumbbell"
    Quiz #87 	 "kurtosis_roll_dumbbell"
    Quiz #88 	 "kurtosis_picth_dumbbell"
    Quiz #89 	 "kurtosis_yaw_dumbbell"
    Quiz #90 	 "skewness_roll_dumbbell"
    Quiz #91 	 "skewness_pitch_dumbbell"
    Quiz #92 	 "skewness_yaw_dumbbell"
    Quiz #93 	 "max_roll_dumbbell"
    Quiz #94 	 "max_picth_dumbbell"
    Quiz #95 	 "max_yaw_dumbbell"
    Quiz #96 	 "min_roll_dumbbell"
    Quiz #97 	 "min_pitch_dumbbell"
    Quiz #98 	 "min_yaw_dumbbell"
    Quiz #99 	 "amplitude_roll_dumbbell"
    Quiz #100 	 "amplitude_pitch_dumbbell"
    Quiz #101 	 "amplitude_yaw_dumbbell"
    Quiz #102 	 "total_accel_dumbbell"
    Quiz #103 	 "var_accel_dumbbell"
    Quiz #104 	 "avg_roll_dumbbell"
    Quiz #105 	 "stddev_roll_dumbbell"
    Quiz #106 	 "var_roll_dumbbell"
    Quiz #107 	 "avg_pitch_dumbbell"
    Quiz #108 	 "stddev_pitch_dumbbell"
    Quiz #109 	 "var_pitch_dumbbell"
    Quiz #110 	 "avg_yaw_dumbbell"
    Quiz #111 	 "stddev_yaw_dumbbell"
    Quiz #112 	 "var_yaw_dumbbell"
    Quiz #113 	 "gyros_dumbbell_x"
    Quiz #114 	 "gyros_dumbbell_y"
    Quiz #115 	 "gyros_dumbbell_z"
    Quiz #116 	 "accel_dumbbell_x"
    Quiz #117 	 "accel_dumbbell_y"
    Quiz #118 	 "accel_dumbbell_z"
    Quiz #119 	 "magnet_dumbbell_x"
    Quiz #120 	 "magnet_dumbbell_y"
    Quiz #121 	 "magnet_dumbbell_z"
    Quiz #122 	 "roll_forearm"
    Quiz #123 	 "pitch_forearm"
    Quiz #124 	 "yaw_forearm"
    Quiz #125 	 "kurtosis_roll_forearm"
    Quiz #126 	 "kurtosis_picth_forearm"
    Quiz #127 	 "kurtosis_yaw_forearm"
    Quiz #128 	 "skewness_roll_forearm"
    Quiz #129 	 "skewness_pitch_forearm"
    Quiz #130 	 "skewness_yaw_forearm"
    Quiz #131 	 "max_roll_forearm"
    Quiz #132 	 "max_picth_forearm"
    Quiz #133 	 "max_yaw_forearm"
    Quiz #134 	 "min_roll_forearm"
    Quiz #135 	 "min_pitch_forearm"
    Quiz #136 	 "min_yaw_forearm"
    Quiz #137 	 "amplitude_roll_forearm"
    Quiz #138 	 "amplitude_pitch_forearm"
    Quiz #139 	 "amplitude_yaw_forearm"
    Quiz #140 	 "total_accel_forearm"
    Quiz #141 	 "var_accel_forearm"
    Quiz #142 	 "avg_roll_forearm"
    Quiz #143 	 "stddev_roll_forearm"
    Quiz #144 	 "var_roll_forearm"
    Quiz #145 	 "avg_pitch_forearm"
    Quiz #146 	 "stddev_pitch_forearm"
    Quiz #147 	 "var_pitch_forearm"
    Quiz #148 	 "avg_yaw_forearm"
    Quiz #149 	 "stddev_yaw_forearm"
    Quiz #150 	 "var_yaw_forearm"
    Quiz #151 	 "gyros_forearm_x"
    Quiz #152 	 "gyros_forearm_y"
    Quiz #153 	 "gyros_forearm_z"
    Quiz #154 	 "accel_forearm_x"
    Quiz #155 	 "accel_forearm_y"
    Quiz #156 	 "accel_forearm_z"
    Quiz #157 	 "magnet_forearm_x"
    Quiz #158 	 "magnet_forearm_y"
    Quiz #159 	 "magnet_forearm_z"
    Quiz #160 	 "classe"
    'data.frame':	13737 obs. of  160 variables:
     $ X                       : int  1 2 3 4 6 7 8 11 12 13 ...
     $ user_name               : Factor w/ 6 levels "adelmo","carlitos",..: 2 2 2 2 2 2 2 2 2 2 ...
     $ raw_timestamp_part_1    : int  1323084231 1323084231 1323084231 1323084232 1323084232 1323084232 1323084232 1323084232 1323084232 1323084232 ...
     $ raw_timestamp_part_2    : int  788290 808298 820366 120339 304277 368296 440390 500302 528316 560359 ...
     $ cvtd_timestamp          : Factor w/ 20 levels "02/12/2011 13:32",..: 9 9 9 9 9 9 9 9 9 9 ...
     $ new_window              : Factor w/ 2 levels "no","yes": 1 1 1 1 1 1 1 1 1 1 ...
     $ num_window              : int  11 11 11 12 12 12 12 12 12 12 ...
     $ roll_belt               : num  1.41 1.41 1.42 1.48 1.45 1.42 1.42 1.45 1.43 1.42 ...
     $ pitch_belt              : num  8.07 8.07 8.07 8.05 8.06 8.09 8.13 8.18 8.18 8.2 ...
     $ yaw_belt                : num  -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 ...
     $ total_accel_belt        : int  3 3 3 3 3 3 3 3 3 3 ...
     $ kurtosis_roll_belt      : Factor w/ 397 levels "","-0.016850",..: 1 1 1 1 1 1 1 1 1 1 ...
     $ kurtosis_picth_belt     : Factor w/ 317 levels "","-0.021887",..: 1 1 1 1 1 1 1 1 1 1 ...
     $ kurtosis_yaw_belt       : Factor w/ 2 levels "","#DIV/0!": 1 1 1 1 1 1 1 1 1 1 ...
     $ skewness_roll_belt      : Factor w/ 395 levels "","-0.003095",..: 1 1 1 1 1 1 1 1 1 1 ...
     $ skewness_roll_belt.1    : Factor w/ 338 levels "","-0.005928",..: 1 1 1 1 1 1 1 1 1 1 ...
     $ skewness_yaw_belt       : Factor w/ 2 levels "","#DIV/0!": 1 1 1 1 1 1 1 1 1 1 ...
     $ max_roll_belt           : num  NA NA NA NA NA NA NA NA NA NA ...
     $ max_picth_belt          : int  NA NA NA NA NA NA NA NA NA NA ...
     $ max_yaw_belt            : Factor w/ 68 levels "","-0.1","-0.2",..: 1 1 1 1 1 1 1 1 1 1 ...
     $ min_roll_belt           : num  NA NA NA NA NA NA NA NA NA NA ...
     $ min_pitch_belt          : int  NA NA NA NA NA NA NA NA NA NA ...
     $ min_yaw_belt            : Factor w/ 68 levels "","-0.1","-0.2",..: 1 1 1 1 1 1 1 1 1 1 ...
     $ amplitude_roll_belt     : num  NA NA NA NA NA NA NA NA NA NA ...
     $ amplitude_pitch_belt    : int  NA NA NA NA NA NA NA NA NA NA ...
     $ amplitude_yaw_belt      : Factor w/ 4 levels "","#DIV/0!","0.00",..: 1 1 1 1 1 1 1 1 1 1 ...
     $ var_total_accel_belt    : num  NA NA NA NA NA NA NA NA NA NA ...
     $ avg_roll_belt           : num  NA NA NA NA NA NA NA NA NA NA ...
     $ stddev_roll_belt        : num  NA NA NA NA NA NA NA NA NA NA ...
     $ var_roll_belt           : num  NA NA NA NA NA NA NA NA NA NA ...
     $ avg_pitch_belt          : num  NA NA NA NA NA NA NA NA NA NA ...
     $ stddev_pitch_belt       : num  NA NA NA NA NA NA NA NA NA NA ...
     $ var_pitch_belt          : num  NA NA NA NA NA NA NA NA NA NA ...
     $ avg_yaw_belt            : num  NA NA NA NA NA NA NA NA NA NA ...
     $ stddev_yaw_belt         : num  NA NA NA NA NA NA NA NA NA NA ...
     $ var_yaw_belt            : num  NA NA NA NA NA NA NA NA NA NA ...
     $ gyros_belt_x            : num  0 0.02 0 0.02 0.02 0.02 0.02 0.03 0.02 0.02 ...
     $ gyros_belt_y            : num  0 0 0 0 0 0 0 0 0 0 ...
     $ gyros_belt_z            : num  -0.02 -0.02 -0.02 -0.03 -0.02 -0.02 -0.02 -0.02 -0.02 0 ...
     $ accel_belt_x            : int  -21 -22 -20 -22 -21 -22 -22 -21 -22 -22 ...
     $ accel_belt_y            : int  4 4 5 3 4 3 4 2 2 4 ...
     $ accel_belt_z            : int  22 22 23 21 21 21 21 23 23 21 ...
     $ magnet_belt_x           : int  -3 -7 -2 -6 0 -4 -2 -5 -2 -3 ...
     $ magnet_belt_y           : int  599 608 600 604 603 599 603 596 602 606 ...
     $ magnet_belt_z           : int  -313 -311 -305 -310 -312 -311 -313 -317 -319 -309 ...
     $ roll_arm                : num  -128 -128 -128 -128 -128 -128 -128 -128 -128 -128 ...
     $ pitch_arm               : num  22.5 22.5 22.5 22.1 22 21.9 21.8 21.5 21.5 21.4 ...
     $ yaw_arm                 : num  -161 -161 -161 -161 -161 -161 -161 -161 -161 -161 ...
     $ total_accel_arm         : int  34 34 34 34 34 34 34 34 34 34 ...
     $ var_accel_arm           : num  NA NA NA NA NA NA NA NA NA NA ...
     $ avg_roll_arm            : num  NA NA NA NA NA NA NA NA NA NA ...
     $ stddev_roll_arm         : num  NA NA NA NA NA NA NA NA NA NA ...
     $ var_roll_arm            : num  NA NA NA NA NA NA NA NA NA NA ...
     $ avg_pitch_arm           : num  NA NA NA NA NA NA NA NA NA NA ...
     $ stddev_pitch_arm        : num  NA NA NA NA NA NA NA NA NA NA ...
     $ var_pitch_arm           : num  NA NA NA NA NA NA NA NA NA NA ...
     $ avg_yaw_arm             : num  NA NA NA NA NA NA NA NA NA NA ...
     $ stddev_yaw_arm          : num  NA NA NA NA NA NA NA NA NA NA ...
     $ var_yaw_arm             : num  NA NA NA NA NA NA NA NA NA NA ...
     $ gyros_arm_x             : num  0 0.02 0.02 0.02 0.02 0 0.02 0.02 0.02 0.02 ...
     $ gyros_arm_y             : num  0 -0.02 -0.02 -0.03 -0.03 -0.03 -0.02 -0.03 -0.03 -0.02 ...
     $ gyros_arm_z             : num  -0.02 -0.02 -0.02 0.02 0 0 0 0 0 -0.02 ...
     $ accel_arm_x             : int  -288 -290 -289 -289 -289 -289 -289 -290 -288 -287 ...
     $ accel_arm_y             : int  109 110 110 111 111 111 111 110 111 111 ...
     $ accel_arm_z             : int  -123 -125 -126 -123 -122 -125 -124 -123 -123 -124 ...
     $ magnet_arm_x            : int  -368 -369 -368 -372 -369 -373 -372 -366 -363 -372 ...
     $ magnet_arm_y            : int  337 337 344 344 342 336 338 339 343 338 ...
     $ magnet_arm_z            : int  516 513 513 512 513 509 510 509 520 509 ...
     $ kurtosis_roll_arm       : Factor w/ 330 levels "","-0.02438",..: 1 1 1 1 1 1 1 1 1 1 ...
     $ kurtosis_picth_arm      : Factor w/ 328 levels "","-0.00484",..: 1 1 1 1 1 1 1 1 1 1 ...
     $ kurtosis_yaw_arm        : Factor w/ 395 levels "","-0.01548",..: 1 1 1 1 1 1 1 1 1 1 ...
     $ skewness_roll_arm       : Factor w/ 331 levels "","-0.00051",..: 1 1 1 1 1 1 1 1 1 1 ...
     $ skewness_pitch_arm      : Factor w/ 328 levels "","-0.00184",..: 1 1 1 1 1 1 1 1 1 1 ...
     $ skewness_yaw_arm        : Factor w/ 395 levels "","-0.00311",..: 1 1 1 1 1 1 1 1 1 1 ...
     $ max_roll_arm            : num  NA NA NA NA NA NA NA NA NA NA ...
     $ max_picth_arm           : num  NA NA NA NA NA NA NA NA NA NA ...
     $ max_yaw_arm             : int  NA NA NA NA NA NA NA NA NA NA ...
     $ min_roll_arm            : num  NA NA NA NA NA NA NA NA NA NA ...
     $ min_pitch_arm           : num  NA NA NA NA NA NA NA NA NA NA ...
     $ min_yaw_arm             : int  NA NA NA NA NA NA NA NA NA NA ...
     $ amplitude_roll_arm      : num  NA NA NA NA NA NA NA NA NA NA ...
     $ amplitude_pitch_arm     : num  NA NA NA NA NA NA NA NA NA NA ...
     $ amplitude_yaw_arm       : int  NA NA NA NA NA NA NA NA NA NA ...
     $ roll_dumbbell           : num  13.1 13.1 12.9 13.4 13.4 ...
     $ pitch_dumbbell          : num  -70.5 -70.6 -70.3 -70.4 -70.8 ...
     $ yaw_dumbbell            : num  -84.9 -84.7 -85.1 -84.9 -84.5 ...
     $ kurtosis_roll_dumbbell  : Factor w/ 398 levels "","-0.0035","-0.0073",..: 1 1 1 1 1 1 1 1 1 1 ...
     $ kurtosis_picth_dumbbell : Factor w/ 401 levels "","-0.0163","-0.0233",..: 1 1 1 1 1 1 1 1 1 1 ...
     $ kurtosis_yaw_dumbbell   : Factor w/ 2 levels "","#DIV/0!": 1 1 1 1 1 1 1 1 1 1 ...
     $ skewness_roll_dumbbell  : Factor w/ 401 levels "","-0.0082","-0.0096",..: 1 1 1 1 1 1 1 1 1 1 ...
     $ skewness_pitch_dumbbell : Factor w/ 402 levels "","-0.0053","-0.0084",..: 1 1 1 1 1 1 1 1 1 1 ...
     $ skewness_yaw_dumbbell   : Factor w/ 2 levels "","#DIV/0!": 1 1 1 1 1 1 1 1 1 1 ...
     $ max_roll_dumbbell       : num  NA NA NA NA NA NA NA NA NA NA ...
     $ max_picth_dumbbell      : num  NA NA NA NA NA NA NA NA NA NA ...
     $ max_yaw_dumbbell        : Factor w/ 73 levels "","-0.1","-0.2",..: 1 1 1 1 1 1 1 1 1 1 ...
     $ min_roll_dumbbell       : num  NA NA NA NA NA NA NA NA NA NA ...
     $ min_pitch_dumbbell      : num  NA NA NA NA NA NA NA NA NA NA ...
     $ min_yaw_dumbbell        : Factor w/ 73 levels "","-0.1","-0.2",..: 1 1 1 1 1 1 1 1 1 1 ...
     $ amplitude_roll_dumbbell : num  NA NA NA NA NA NA NA NA NA NA ...
      [list output truncated]



```R
### This feature is not useful, because in most cases it is either blank or equals #DIV/0!
summary(training$kurtosis_yaw_belt)
```


<dl class=dl-horizontal>
	<dt>1</dt>
		<dd>13460</dd>
	<dt>#DIV/0!</dt>
		<dd>277</dd>
</dl>




```R
#  These are the columns (fetures) with useful data. We only use these columns for our work here.

#                 "roll_belt","pitch_belt","yaw_belt","total_accel_belt",
#                 "gyros_belt_x","gyros_belt_y","gyros_belt_z",
#                 "accel_belt_x","accel_belt_y","accel_belt_z",
#                 "magnet_belt_x","magnet_belt_y","magnet_belt_z",
#                 "roll_arm","pitch_arm","yaw_arm","total_accel_arm",
#                 "gyros_arm_x","gyros_arm_y","gyros_arm_z",
#                 "accel_arm_x","accel_arm_y","accel_arm_z",
#                 "magnet_arm_x","magnet_arm_y","magnet_arm_z",
#                 "roll_dumbbell","pitch_dumbbell","yaw_dumbbell","total_accel_dumbbell",
#                 "gyros_dumbbell_x","gyros_dumbbell_y","gyros_dumbbell_z",
#                 "accel_dumbbell_x","accel_dumbbell_y","accel_dumbbell_z",
#                 "magnet_dumbbell_x","magnet_dumbbell_x","magnet_dumbbell_z",
#                 "roll_forearm","pitch_forearm","yaw_forearm","total_accel_forearm",
#                 "gyros_forearm_x","gyros_forearm_y","gyros_forearm_z",
#                 "accel_forearm_x","accel_forearm_y","accel_forearm_z",
#                 "magnet_forearm_x","magnet_forearm_y","magnet_forearm_z"
```

## Make some quick observations

Below, I have plotted "roll_belt" versus "pitch_belt" parameters, and colored points by their class. As seen in some cases only these two parameters are enought to make accurate predictions. For example, if there exists a point with pitch_belt>15 and roll_belt<25, it 100% belongs to class "E" with no confusion. 


```R
qplot(roll_belt, pitch_belt, color=classe, data=training)
```


![png](output_10_0.png)


## Making more observations 

Making a pair plot for only 4 parameters, we discover that our data points are somehow clusterred in different regions. Although different classes are not completely differntiated in the following plot, it suggests that there might be a chance that by adding more features we would be able to make accurate differentiations. 


```R
featurePlot(x=training[,c("roll_belt","pitch_belt","yaw_belt", "total_accel_belt")],
            y = training$classe,
            plot="pairs")
```


![png](output_12_0.png)


## Using Random Forest for prediction

In the following cell, we perform a random forest analysis to get our model. To begin with, we only use those parameters that are related to "belt" and ignore the otehrs to see if we are able to make good predictions.

To get better results, we do some standarization preprocessing. We also control our training process by setting trControl parameter. This allows the training algrotihm to set away 30% of the given training data for cross validation and tunes the internal parameters to reach the best accuracy after 5 iterations. 


```R
set.seed(125)

RFmodel <- train(classe ~ roll_belt+pitch_belt+yaw_belt+total_accel_belt+
                gyros_belt_x+gyros_belt_y+gyros_belt_z+
                accel_belt_x+accel_belt_y+accel_belt_z+
                magnet_belt_x+magnet_belt_y+magnet_belt_z, 
                data = training, 
                method = "rf", 
                metric = "Accuracy", 
                preProcess = c("center", "scale"), 
                trControl = trainControl(method = "cv", 
                                       number = 5,           
                                       p = 0.7, 
                                       allowParallel = TRUE),                                       
                prox = TRUE)
```


    Confusion Matrix and Statistics
    
              Reference
    Prediction    A    B    C    D    E
             A 1571   24   39   28   12
             B   34 1049   41   13    2
             C   51   47  876   50    2
             D   56    8   20  876    4
             E   17    3    5    5 1052
    
    Overall Statistics
                                              
                   Accuracy : 0.9217          
                     95% CI : (0.9145, 0.9284)
        No Information Rate : 0.2938          
        P-Value [Acc > NIR] : < 2.2e-16       
                                              
                      Kappa : 0.9008          
                                              
     Mcnemar's Test P-Value : 0.001006        
    
    Statistics by Class:
    
                         Class: A Class: B Class: C Class: D Class: E
    Sensitivity            0.9086   0.9275   0.8930   0.9012   0.9813
    Specificity            0.9752   0.9811   0.9694   0.9821   0.9938
    Pos Pred Value         0.9385   0.9210   0.8538   0.9087   0.9723
    Neg Pred Value         0.9625   0.9827   0.9784   0.9805   0.9958
    Prevalence             0.2938   0.1922   0.1667   0.1652   0.1822
    Detection Rate         0.2669   0.1782   0.1489   0.1489   0.1788
    Detection Prevalence   0.2845   0.1935   0.1743   0.1638   0.1839
    Balanced Accuracy      0.9419   0.9543   0.9312   0.9417   0.9876


## Checking the trained RF model for the trainign set

Here we use the resulting random forest to make predictions on the training set. As seen the accuracy is 100%, which means our model totally follows the shape of our data. We don't expect to get the same accuracy for the cross validation set.


```R
rf.result.training <- predict(RFmodel, training)
confusionMatrix(training$classe, rf.result.training)
```


    Confusion Matrix and Statistics
    
              Reference
    Prediction    A    B    C    D    E
             A 3906    0    0    0    0
             B    0 2658    0    0    0
             C    0    0 2396    0    0
             D    0    0    0 2252    0
             E    0    0    0    0 2525
    
    Overall Statistics
                                         
                   Accuracy : 1          
                     95% CI : (0.9997, 1)
        No Information Rate : 0.2843     
        P-Value [Acc > NIR] : < 2.2e-16  
                                         
                      Kappa : 1          
                                         
     Mcnemar's Test P-Value : NA         
    
    Statistics by Class:
    
                         Class: A Class: B Class: C Class: D Class: E
    Sensitivity            1.0000   1.0000   1.0000   1.0000   1.0000
    Specificity            1.0000   1.0000   1.0000   1.0000   1.0000
    Pos Pred Value         1.0000   1.0000   1.0000   1.0000   1.0000
    Neg Pred Value         1.0000   1.0000   1.0000   1.0000   1.0000
    Prevalence             0.2843   0.1935   0.1744   0.1639   0.1838
    Detection Rate         0.2843   0.1935   0.1744   0.1639   0.1838
    Detection Prevalence   0.2843   0.1935   0.1744   0.1639   0.1838
    Balanced Accuracy      1.0000   1.0000   1.0000   1.0000   1.0000


## Evaluating the trained RF model for the Cross Validation Set

As expected the performance of our model is worse (~98%) on the cross validation set. Also, 98% is still a good accuracy. This means that only using the data from the belt sensors would be enough to get good predictions. 


```R
rf.result <- predict(RFmodel, crossvalidation)
confusionMatrix(crossvalidation$classe, rf.result)
```


    Confusion Matrix and Statistics
    
              Reference
    Prediction    A    B    C    D    E
             A 1647    8   11    5    3
             B    6 1120   10    3    0
             C   14   11  985   15    1
             D   17    1    3  942    1
             E    8    1    0    0 1073
    
    Overall Statistics
                                             
                   Accuracy : 0.9799         
                     95% CI : (0.976, 0.9834)
        No Information Rate : 0.2875         
        P-Value [Acc > NIR] : <2e-16         
                                             
                      Kappa : 0.9746         
                                             
     Mcnemar's Test P-Value : 0.0178         
    
    Statistics by Class:
    
                         Class: A Class: B Class: C Class: D Class: E
    Sensitivity            0.9734   0.9816   0.9762   0.9762   0.9954
    Specificity            0.9936   0.9960   0.9916   0.9955   0.9981
    Pos Pred Value         0.9839   0.9833   0.9600   0.9772   0.9917
    Neg Pred Value         0.9893   0.9956   0.9951   0.9953   0.9990
    Prevalence             0.2875   0.1939   0.1715   0.1640   0.1832
    Detection Rate         0.2799   0.1903   0.1674   0.1601   0.1823
    Detection Prevalence   0.2845   0.1935   0.1743   0.1638   0.1839
    Balanced Accuracy      0.9835   0.9888   0.9839   0.9858   0.9967


## Make predictions using the Test set

The followig cell takes the test set and uses the same trained RF model to predict the class of each row. 


```R
rf.result.Testing <- predict(RFmodel, Testing)
for (i in Testing$problem_id) {
cat(sprintf("Quiz #%d \t \"%s\"\n", Testing$problem_id[i], rf.result.Testing[i]))
}
```

    Quiz #1 	 "B"
    Quiz #2 	 "A"
    Quiz #3 	 "B"
    Quiz #4 	 "A"
    Quiz #5 	 "A"
    Quiz #6 	 "E"
    Quiz #7 	 "D"
    Quiz #8 	 "B"
    Quiz #9 	 "A"
    Quiz #10 	 "A"
    Quiz #11 	 "B"
    Quiz #12 	 "C"
    Quiz #13 	 "B"
    Quiz #14 	 "A"
    Quiz #15 	 "E"
    Quiz #16 	 "E"
    Quiz #17 	 "A"
    Quiz #18 	 "B"
    Quiz #19 	 "B"
    Quiz #20 	 "B"


## Training a more complicated Random Forest using all data features

To even get better results, we train a more complex random forest model that uses all data columns. This part take more computation resources.


```R
set.seed(125)

RFmodel.all <- train(classe ~ new_window+num_window+ 
                roll_belt+pitch_belt+yaw_belt+total_accel_belt+
                gyros_belt_x+gyros_belt_y+gyros_belt_z+
                accel_belt_x+accel_belt_y+accel_belt_z+
                magnet_belt_x+magnet_belt_y+magnet_belt_z+
                roll_arm+pitch_arm+yaw_arm+total_accel_arm+
                gyros_arm_x+gyros_arm_y+gyros_arm_z+
                accel_arm_x+accel_arm_y+accel_arm_z+
                magnet_arm_x+magnet_arm_y+magnet_arm_z+
                roll_dumbbell+pitch_dumbbell+yaw_dumbbell+total_accel_dumbbell+
                gyros_dumbbell_x+gyros_dumbbell_y+gyros_dumbbell_z+
                accel_dumbbell_x+accel_dumbbell_y+accel_dumbbell_z+
                magnet_dumbbell_x+magnet_dumbbell_x+magnet_dumbbell_z+
                roll_forearm+pitch_forearm+yaw_forearm+total_accel_forearm+
                gyros_forearm_x+gyros_forearm_y+gyros_forearm_z+
                accel_forearm_x+accel_forearm_y+accel_forearm_z+
                magnet_forearm_x+magnet_forearm_y+magnet_forearm_z, 
                data = training, 
                method = "rf", 
                metric = "Accuracy", 
                preProcess = c("center", "scale"), 
                trControl = trainControl(method = "cv", 
                                       number = 5,           
                                       p = 0.7, 
                                       allowParallel = TRUE),                                       
                prox = TRUE)
```


    Confusion Matrix and Statistics
    
              Reference
    Prediction    A    B    C    D    E
             A 1672    2    0    0    0
             B    0 1139    0    0    0
             C    0    1 1025    0    0
             D    0    0    1  963    0
             E    0    0    0    3 1079
    
    Overall Statistics
                                              
                   Accuracy : 0.9988          
                     95% CI : (0.9976, 0.9995)
        No Information Rate : 0.2841          
        P-Value [Acc > NIR] : < 2.2e-16       
                                              
                      Kappa : 0.9985          
                                              
     Mcnemar's Test P-Value : NA              
    
    Statistics by Class:
    
                         Class: A Class: B Class: C Class: D Class: E
    Sensitivity            1.0000   0.9974   0.9990   0.9969   1.0000
    Specificity            0.9995   1.0000   0.9998   0.9998   0.9994
    Pos Pred Value         0.9988   1.0000   0.9990   0.9990   0.9972
    Neg Pred Value         1.0000   0.9994   0.9998   0.9994   1.0000
    Prevalence             0.2841   0.1941   0.1743   0.1641   0.1833
    Detection Rate         0.2841   0.1935   0.1742   0.1636   0.1833
    Detection Prevalence   0.2845   0.1935   0.1743   0.1638   0.1839
    Balanced Accuracy      0.9998   0.9987   0.9994   0.9983   0.9997



```R
plot(RFmodel.all$finalModel,main="Model error of Random forest model by number of trees")
```


    Error in plot(RFmodel.all$finalModel, main = "Model error of Random forest model by number of trees"): object 'RFmodel.all' not found
    Traceback:


    1. plot(RFmodel.all$finalModel, main = "Model error of Random forest model by number of trees")


## Evaluating the perforamnce of the RF-all on the training set

Like the other model, the accuracy is 100%. Now the question is whether we get better results on the cross validation set.


```R
rf.result.training <- predict(RFmodel.all, training)
confusionMatrix(training$classe, rf.result.training)
```


    Confusion Matrix and Statistics
    
              Reference
    Prediction    A    B    C    D    E
             A 3906    0    0    0    0
             B    0 2658    0    0    0
             C    0    0 2396    0    0
             D    0    0    0 2252    0
             E    0    0    0    0 2525
    
    Overall Statistics
                                         
                   Accuracy : 1          
                     95% CI : (0.9997, 1)
        No Information Rate : 0.2843     
        P-Value [Acc > NIR] : < 2.2e-16  
                                         
                      Kappa : 1          
                                         
     Mcnemar's Test P-Value : NA         
    
    Statistics by Class:
    
                         Class: A Class: B Class: C Class: D Class: E
    Sensitivity            1.0000   1.0000   1.0000   1.0000   1.0000
    Specificity            1.0000   1.0000   1.0000   1.0000   1.0000
    Pos Pred Value         1.0000   1.0000   1.0000   1.0000   1.0000
    Neg Pred Value         1.0000   1.0000   1.0000   1.0000   1.0000
    Prevalence             0.2843   0.1935   0.1744   0.1639   0.1838
    Detection Rate         0.2843   0.1935   0.1744   0.1639   0.1838
    Detection Prevalence   0.2843   0.1935   0.1744   0.1639   0.1838
    Balanced Accuracy      1.0000   1.0000   1.0000   1.0000   1.0000


## Evaluating RFmodel.all on the cross-validation set

This time, the accouracy is ~99.9% which is of course better than 98% we got using the our first Random Forest model. Our conclusion is that adding more features definitely improves our prediction accuracy. 

This means that for improving accuracy by 2%, we needed to use all ifnroamtion from all sensors. Sometimes adding more sensors is expensive and is not worth the few percent improvment in predictions.


```R
rf.result.all <- predict(RFmodel.all, crossvalidation)
confusionMatrix(crossvalidation$classe, rf.result.all)
```


    Confusion Matrix and Statistics
    
              Reference
    Prediction    A    B    C    D    E
             A 1672    2    0    0    0
             B    0 1139    0    0    0
             C    0    1 1025    0    0
             D    0    0    1  963    0
             E    0    0    0    3 1079
    
    Overall Statistics
                                              
                   Accuracy : 0.9988          
                     95% CI : (0.9976, 0.9995)
        No Information Rate : 0.2841          
        P-Value [Acc > NIR] : < 2.2e-16       
                                              
                      Kappa : 0.9985          
                                              
     Mcnemar's Test P-Value : NA              
    
    Statistics by Class:
    
                         Class: A Class: B Class: C Class: D Class: E
    Sensitivity            1.0000   0.9974   0.9990   0.9969   1.0000
    Specificity            0.9995   1.0000   0.9998   0.9998   0.9994
    Pos Pred Value         0.9988   1.0000   0.9990   0.9990   0.9972
    Neg Pred Value         1.0000   0.9994   0.9998   0.9994   1.0000
    Prevalence             0.2841   0.1941   0.1743   0.1641   0.1833
    Detection Rate         0.2841   0.1935   0.1742   0.1636   0.1833
    Detection Prevalence   0.2845   0.1935   0.1743   0.1638   0.1839
    Balanced Accuracy      0.9998   0.9987   0.9994   0.9983   0.9997


## Using RFmodel.all to make predictions on the Test set


```R
rf.result.Testing <- predict(modFit.all, Testing)
for (i in Testing$problem_id) {
cat(sprintf("Quiz #%d \t \"%s\"\n", Testing$problem_id[i], rf.result.Testing[i]))
}
```

    Quiz #1 	 "B"
    Quiz #2 	 "A"
    Quiz #3 	 "B"
    Quiz #4 	 "A"
    Quiz #5 	 "A"
    Quiz #6 	 "E"
    Quiz #7 	 "D"
    Quiz #8 	 "B"
    Quiz #9 	 "A"
    Quiz #10 	 "A"
    Quiz #11 	 "B"
    Quiz #12 	 "C"
    Quiz #13 	 "B"
    Quiz #14 	 "A"
    Quiz #15 	 "E"
    Quiz #16 	 "E"
    Quiz #17 	 "A"
    Quiz #18 	 "B"
    Quiz #19 	 "B"
    Quiz #20 	 "B"



```R
crossvalidationPC <- predict(preProc,crossvalidation[,-54])
rf.result.all <- predict(modFit.all, crossvalidationPC)
confusionMatrix(crossvalidation$classe, rf.result.all)
```


```R
## This part needs some modifications

set.seed(125)

modFit.all <- train(training$classe ~ ., 
                method = "rf", 
                metric = "Accuracy", 
                preProcess = c("center", "scale"), 
                trControl = trainControl(method = "cv", 
                                       number = 5,           
                                       p = 0.7, 
                                       allowParallel = TRUE),   
                data = trainingPC, 
                prox = TRUE)
```

# Further Investigations 

## The possible use of PCA in this analysis

## Please stop here if you are grading this asssignment. This is just for my further curiosity.
   
So far, we have used 54 useful features to make a perfect prediction model.
Below, I am trying to make a compression and see how many PCA I can generate to hold 95% of the scatter in the data.


```R
features = c("classe", 
                "new_window","num_window", 
                "roll_belt","pitch_belt","yaw_belt","total_accel_belt",
                "gyros_belt_x","gyros_belt_y","gyros_belt_z",
                "accel_belt_x","accel_belt_y","accel_belt_z",
                "magnet_belt_x","magnet_belt_y","magnet_belt_z",
                "roll_arm","pitch_arm","yaw_arm","total_accel_arm",
                "gyros_arm_x","gyros_arm_y","gyros_arm_z",
                "accel_arm_x","accel_arm_y","accel_arm_z",
                "magnet_arm_x","magnet_arm_y","magnet_arm_z",
                "roll_dumbbell","pitch_dumbbell","yaw_dumbbell","total_accel_dumbbell",
                "gyros_dumbbell_x","gyros_dumbbell_y","gyros_dumbbell_z",
                "accel_dumbbell_x","accel_dumbbell_y","accel_dumbbell_z",
                "magnet_dumbbell_x","magnet_dumbbell_x","magnet_dumbbell_z",
                "roll_forearm","pitch_forearm","yaw_forearm","total_accel_forearm",
                "gyros_forearm_x","gyros_forearm_y","gyros_forearm_z",
                "accel_forearm_x","accel_forearm_y","accel_forearm_z",
                "magnet_forearm_x","magnet_forearm_y","magnet_forearm_z")


training <- training[names(training) %in% features]
crossvalidation <- crossvalidation[names(crossvalidation) %in% features]
Testing <- Testing[names(Testing) %in% features]

head(training)
```


<table>
<caption>A data.frame: 6 × 54</caption>
<thead>
	<tr><th></th><th scope=col>new_window</th><th scope=col>num_window</th><th scope=col>roll_belt</th><th scope=col>pitch_belt</th><th scope=col>yaw_belt</th><th scope=col>total_accel_belt</th><th scope=col>gyros_belt_x</th><th scope=col>gyros_belt_y</th><th scope=col>gyros_belt_z</th><th scope=col>accel_belt_x</th><th scope=col>⋯</th><th scope=col>gyros_forearm_x</th><th scope=col>gyros_forearm_y</th><th scope=col>gyros_forearm_z</th><th scope=col>accel_forearm_x</th><th scope=col>accel_forearm_y</th><th scope=col>accel_forearm_z</th><th scope=col>magnet_forearm_x</th><th scope=col>magnet_forearm_y</th><th scope=col>magnet_forearm_z</th><th scope=col>classe</th></tr>
	<tr><th></th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>⋯</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;fct&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>1</th><td>no</td><td>11</td><td>1.41</td><td>8.07</td><td>-94.4</td><td>3</td><td>0.00</td><td>0.00</td><td>-0.02</td><td>-21</td><td>⋯</td><td>0.03</td><td> 0.00</td><td>-0.02</td><td>192</td><td>203</td><td>-215</td><td>-17</td><td>654</td><td>476</td><td>A</td></tr>
	<tr><th scope=row>3</th><td>no</td><td>11</td><td>1.42</td><td>8.07</td><td>-94.4</td><td>3</td><td>0.00</td><td>0.00</td><td>-0.02</td><td>-20</td><td>⋯</td><td>0.03</td><td>-0.02</td><td> 0.00</td><td>196</td><td>204</td><td>-213</td><td>-18</td><td>658</td><td>469</td><td>A</td></tr>
	<tr><th scope=row>5</th><td>no</td><td>12</td><td>1.48</td><td>8.07</td><td>-94.4</td><td>3</td><td>0.02</td><td>0.02</td><td>-0.02</td><td>-21</td><td>⋯</td><td>0.02</td><td> 0.00</td><td>-0.02</td><td>189</td><td>206</td><td>-214</td><td>-17</td><td>655</td><td>473</td><td>A</td></tr>
	<tr><th scope=row>7</th><td>no</td><td>12</td><td>1.42</td><td>8.09</td><td>-94.4</td><td>3</td><td>0.02</td><td>0.00</td><td>-0.02</td><td>-22</td><td>⋯</td><td>0.02</td><td> 0.00</td><td>-0.02</td><td>195</td><td>205</td><td>-215</td><td>-18</td><td>659</td><td>470</td><td>A</td></tr>
	<tr><th scope=row>8</th><td>no</td><td>12</td><td>1.42</td><td>8.13</td><td>-94.4</td><td>3</td><td>0.02</td><td>0.00</td><td>-0.02</td><td>-22</td><td>⋯</td><td>0.02</td><td>-0.02</td><td> 0.00</td><td>193</td><td>205</td><td>-213</td><td> -9</td><td>660</td><td>474</td><td>A</td></tr>
	<tr><th scope=row>9</th><td>no</td><td>12</td><td>1.43</td><td>8.16</td><td>-94.4</td><td>3</td><td>0.02</td><td>0.00</td><td>-0.02</td><td>-20</td><td>⋯</td><td>0.03</td><td> 0.00</td><td>-0.02</td><td>193</td><td>204</td><td>-214</td><td>-16</td><td>653</td><td>476</td><td>A</td></tr>
</tbody>
</table>



The following plot illustrates how data poitns are clustered when different set of PCAs are used.
Using more PCA components would improve the classification task.


```R
## I only require 95% of the scatter
## and I end up with 25 PCAs. This means that training a Random Forest with these PCA features
## could be possibly faster and produce the same accuracy

preProc <- preProcess(training[,-54], method=c('center', 'scale', 'pca'), thresh=0.95)

trainingPC <- predict(preProc,training[,-54])
head(trainingPC)
qplot(trainingPC[,2], trainingPC[,3], color=training$classe)
```


<table>
<caption>A data.frame: 6 × 26</caption>
<thead>
	<tr><th></th><th scope=col>new_window</th><th scope=col>PC1</th><th scope=col>PC2</th><th scope=col>PC3</th><th scope=col>PC4</th><th scope=col>PC5</th><th scope=col>PC6</th><th scope=col>PC7</th><th scope=col>PC8</th><th scope=col>PC9</th><th scope=col>⋯</th><th scope=col>PC16</th><th scope=col>PC17</th><th scope=col>PC18</th><th scope=col>PC19</th><th scope=col>PC20</th><th scope=col>PC21</th><th scope=col>PC22</th><th scope=col>PC23</th><th scope=col>PC24</th><th scope=col>PC25</th></tr>
	<tr><th></th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>⋯</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>1</th><td>no</td><td>2.972763</td><td>3.686171</td><td>2.089551</td><td>1.540422</td><td>-1.512766</td><td>1.792066</td><td> 0.01717150</td><td>-2.777324</td><td>0.2172460</td><td>⋯</td><td>-0.6437110</td><td>0.05970558</td><td>0.5677851</td><td>0.3438208</td><td>1.538746</td><td>-0.3820322</td><td>0.07106292</td><td>-0.3766739</td><td>-0.3737574</td><td>0.1949081</td></tr>
	<tr><th scope=row>3</th><td>no</td><td>2.968619</td><td>3.711323</td><td>2.092228</td><td>1.541832</td><td>-1.519221</td><td>1.795919</td><td>-0.01006257</td><td>-2.755328</td><td>0.2517540</td><td>⋯</td><td>-0.6629756</td><td>0.09638254</td><td>0.5759817</td><td>0.3332674</td><td>1.541581</td><td>-0.3577407</td><td>0.06750221</td><td>-0.3704316</td><td>-0.3509060</td><td>0.1900447</td></tr>
	<tr><th scope=row>5</th><td>no</td><td>2.944507</td><td>3.756857</td><td>2.052044</td><td>1.517674</td><td>-1.545017</td><td>1.885336</td><td>-0.05838001</td><td>-2.721783</td><td>0.2323673</td><td>⋯</td><td>-0.6123764</td><td>0.13444709</td><td>0.5911024</td><td>0.3136318</td><td>1.537784</td><td>-0.3912644</td><td>0.02233997</td><td>-0.3842049</td><td>-0.3486524</td><td>0.2191613</td></tr>
	<tr><th scope=row>7</th><td>no</td><td>2.978695</td><td>3.685720</td><td>2.068711</td><td>1.531830</td><td>-1.520426</td><td>1.809691</td><td>-0.03159538</td><td>-2.767830</td><td>0.2414495</td><td>⋯</td><td>-0.6395848</td><td>0.07284003</td><td>0.5387824</td><td>0.3412634</td><td>1.516248</td><td>-0.3886653</td><td>0.04780037</td><td>-0.3882825</td><td>-0.3607784</td><td>0.2016864</td></tr>
	<tr><th scope=row>8</th><td>no</td><td>2.966336</td><td>3.711241</td><td>2.083815</td><td>1.544567</td><td>-1.547465</td><td>1.831619</td><td>-0.02977678</td><td>-2.774799</td><td>0.2402008</td><td>⋯</td><td>-0.6483164</td><td>0.09504551</td><td>0.5489882</td><td>0.3359293</td><td>1.519738</td><td>-0.3712759</td><td>0.03583650</td><td>-0.3683829</td><td>-0.3626895</td><td>0.2304104</td></tr>
	<tr><th scope=row>9</th><td>no</td><td>2.982772</td><td>3.734518</td><td>2.068318</td><td>1.524030</td><td>-1.504500</td><td>1.799560</td><td>-0.00608277</td><td>-2.735300</td><td>0.2499761</td><td>⋯</td><td>-0.6614894</td><td>0.08114417</td><td>0.5539319</td><td>0.3394942</td><td>1.535773</td><td>-0.3469845</td><td>0.06319026</td><td>-0.3839697</td><td>-0.3230635</td><td>0.1930678</td></tr>
</tbody>
</table>




![png](output_35_1.png)

