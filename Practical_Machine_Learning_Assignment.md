
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

    Registered S3 methods overwritten by 'ggplot2':
      method         from 
      [.quosures     rlang
      c.quosures     rlang
      print.quosures rlang
    Loading required package: lattice



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
     $ X                       : int  1 2 3 4 5 6 7 8 9 11 ...
     $ user_name               : Factor w/ 6 levels "adelmo","carlitos",..: 2 2 2 2 2 2 2 2 2 2 ...
     $ raw_timestamp_part_1    : int  1323084231 1323084231 1323084231 1323084232 1323084232 1323084232 1323084232 1323084232 1323084232 1323084232 ...
     $ raw_timestamp_part_2    : int  788290 808298 820366 120339 196328 304277 368296 440390 484323 500302 ...
     $ cvtd_timestamp          : Factor w/ 20 levels "02/12/2011 13:32",..: 9 9 9 9 9 9 9 9 9 9 ...
     $ new_window              : Factor w/ 2 levels "no","yes": 1 1 1 1 1 1 1 1 1 1 ...
     $ num_window              : int  11 11 11 12 12 12 12 12 12 12 ...
     $ roll_belt               : num  1.41 1.41 1.42 1.48 1.48 1.45 1.42 1.42 1.43 1.45 ...
     $ pitch_belt              : num  8.07 8.07 8.07 8.05 8.07 8.06 8.09 8.13 8.16 8.18 ...
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
     $ gyros_belt_x            : num  0 0.02 0 0.02 0.02 0.02 0.02 0.02 0.02 0.03 ...
     $ gyros_belt_y            : num  0 0 0 0 0.02 0 0 0 0 0 ...
     $ gyros_belt_z            : num  -0.02 -0.02 -0.02 -0.03 -0.02 -0.02 -0.02 -0.02 -0.02 -0.02 ...
     $ accel_belt_x            : int  -21 -22 -20 -22 -21 -21 -22 -22 -20 -21 ...
     $ accel_belt_y            : int  4 4 5 3 2 4 3 4 2 2 ...
     $ accel_belt_z            : int  22 22 23 21 24 21 21 21 24 23 ...
     $ magnet_belt_x           : int  -3 -7 -2 -6 -6 0 -4 -2 1 -5 ...
     $ magnet_belt_y           : int  599 608 600 604 600 603 599 603 602 596 ...
     $ magnet_belt_z           : int  -313 -311 -305 -310 -302 -312 -311 -313 -312 -317 ...
     $ roll_arm                : num  -128 -128 -128 -128 -128 -128 -128 -128 -128 -128 ...
     $ pitch_arm               : num  22.5 22.5 22.5 22.1 22.1 22 21.9 21.8 21.7 21.5 ...
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
     $ gyros_arm_x             : num  0 0.02 0.02 0.02 0 0.02 0 0.02 0.02 0.02 ...
     $ gyros_arm_y             : num  0 -0.02 -0.02 -0.03 -0.03 -0.03 -0.03 -0.02 -0.03 -0.03 ...
     $ gyros_arm_z             : num  -0.02 -0.02 -0.02 0.02 0 0 0 0 -0.02 0 ...
     $ accel_arm_x             : int  -288 -290 -289 -289 -289 -289 -289 -289 -288 -290 ...
     $ accel_arm_y             : int  109 110 110 111 111 111 111 111 109 110 ...
     $ accel_arm_z             : int  -123 -125 -126 -123 -123 -122 -125 -124 -122 -123 ...
     $ magnet_arm_x            : int  -368 -369 -368 -372 -374 -369 -373 -372 -369 -366 ...
     $ magnet_arm_y            : int  337 337 344 344 337 342 336 338 341 339 ...
     $ magnet_arm_z            : int  516 513 513 512 506 513 509 510 518 509 ...
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
     $ pitch_dumbbell          : num  -70.5 -70.6 -70.3 -70.4 -70.4 ...
     $ yaw_dumbbell            : num  -84.9 -84.7 -85.1 -84.9 -84.9 ...
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
		<dd>13461</dd>
	<dt>#DIV/0!</dt>
		<dd>276</dd>
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
             A 1545   27   53   42    7
             B   34 1042   43   17    3
             C   44   53  887   41    1
             D   50    5   32  872    5
             E   16    2    6    4 1054
    
    Overall Statistics
                                              
                   Accuracy : 0.9176          
                     95% CI : (0.9103, 0.9245)
        No Information Rate : 0.287           
        P-Value [Acc > NIR] : < 2e-16         
                                              
                      Kappa : 0.8957          
                                              
     Mcnemar's Test P-Value : 0.04805         
    
    Statistics by Class:
    
                         Class: A Class: B Class: C Class: D Class: E
    Sensitivity            0.9147   0.9229   0.8688   0.8934   0.9850
    Specificity            0.9693   0.9796   0.9714   0.9813   0.9942
    Pos Pred Value         0.9229   0.9148   0.8645   0.9046   0.9741
    Neg Pred Value         0.9658   0.9817   0.9724   0.9789   0.9967
    Prevalence             0.2870   0.1918   0.1735   0.1658   0.1818
    Detection Rate         0.2625   0.1771   0.1507   0.1482   0.1791
    Detection Prevalence   0.2845   0.1935   0.1743   0.1638   0.1839
    Balanced Accuracy      0.9420   0.9513   0.9201   0.9374   0.9896


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
             A 1674    0    0    0    0
             B    0 1139    0    0    0
             C    0    1 1025    0    0
             D    0    0    2  961    1
             E    0    0    0    2 1080
    
    Overall Statistics
                                              
                   Accuracy : 0.999           
                     95% CI : (0.9978, 0.9996)
        No Information Rate : 0.2845          
        P-Value [Acc > NIR] : < 2.2e-16       
                                              
                      Kappa : 0.9987          
                                              
     Mcnemar's Test P-Value : NA              
    
    Statistics by Class:
    
                         Class: A Class: B Class: C Class: D Class: E
    Sensitivity            1.0000   0.9991   0.9981   0.9979   0.9991
    Specificity            1.0000   1.0000   0.9998   0.9994   0.9996
    Pos Pred Value         1.0000   1.0000   0.9990   0.9969   0.9982
    Neg Pred Value         1.0000   0.9998   0.9996   0.9996   0.9998
    Prevalence             0.2845   0.1937   0.1745   0.1636   0.1837
    Detection Rate         0.2845   0.1935   0.1742   0.1633   0.1835
    Detection Prevalence   0.2845   0.1935   0.1743   0.1638   0.1839
    Balanced Accuracy      1.0000   0.9996   0.9989   0.9987   0.9993


## Using RFmodel.all to make predictions on the Test set


```R
rf.result.Testing <- predict(RFmodel.all, Testing)
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

