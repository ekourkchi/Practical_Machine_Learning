
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


![png](output_9_0.png)


## Making more observations 

Making a pair plot for only 4 parameters, we discover that our data points are somehow clusterred in different regions. Although different classes are not completely differntiated in the following plot, it suggests that there might be a chance that by adding more features we would be able to make accurate differentiations. 


```R
featurePlot(x=training[,c("roll_belt","pitch_belt","yaw_belt", "total_accel_belt")],
            y = training$classe,
            plot="pairs")
```


![png](output_11_0.png)


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

