---
title: "Practical machine learning course project"
author: "Harnos Andrea"
date: "10/24/2020"
output: 
  html_document:
    keep_md: yes
---



## Introduction

The goal of this project to develop a classification model for ditinguisning correct and incorrect workout technique based on data measured by three-axis gyro sensors on the belt, forearm, arm, and dumbell.

The measurements were made on six male participants who had to perform one set of 10 repetitions of the Unilateral Dumbbell Biceps Curl in five different fashions:

- Class A: exactly according to the specification
- Class B: throwing the elbows to the front
- Class C: lifting the dumbbell only halfway 
- Class D: lowering the dumbbell only halfway
- Class F: throwing the hips to the front

Class A means the correctly executed  exercise, while the other 4 classes are incorrect. 


## Getting data and loading the caret package



Explore the data.


```r
glimpse(train)
```


```r
train <- read.csv("pml-training.csv", na.strings = c("",NA), stringsAsFactors = T)
test <- read.csv("pml-testing.csv", na.strings = c("",NA), stringsAsFactors = T)
```

The first seven variables are not measurements (name of the participant and timestamp etc.), and should be excluded from the dataset. Also variables with too many missing values sholud be excluded. Columns with more than 20% missing values will be left out from the analysis.

Delete variables with more than 20% missing values


```r
limit_of_NAs <- nrow(train) * 0.2
remaining_cols <- which(colSums(!is.na(train)) > limit_of_NAs)
train<-train[,remaining_cols]
test<-test[,remaining_cols]
```

The first 7 variables are also left out.


```r
train <- train[,8:length(train)]
test <- test[,8:length(test)]
```

Create a validation set


```r
set.seed(15)
invalid <- createDataPartition(y=train$classe,p=0.5, list=FALSE) 
valid.data <- train[invalid,]
train.data <- train[-invalid,]
```

As referenced paper I built a random forest model to predict `classe` on all of the other variables. 


```r
set.seed(12)
if (!file.exists("RFmod.RData"))
{modelRf <- train(classe ~ ., data=train.data, method="rf", ntree=250)
}else
{
  load("RFmod.RData")
}
  
#save(modelRf, file="RFmod.RData")
```

### Performance of the model on the validation data set


```r
pred_rf <- predict(modelRf, valid.data)
confusionMatrix(valid.data$classe, pred_rf)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 2779    4    7    0    0
##          B   31 1861    7    0    0
##          C    0   39 1667    5    0
##          D    5    0   28 1573    2
##          E    0    0    4    4 1796
## 
## Overall Statistics
##                                           
##                Accuracy : 0.9861          
##                  95% CI : (0.9836, 0.9884)
##     No Information Rate : 0.2869          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.9825          
##                                           
##  Mcnemar's Test P-Value : NA              
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9872   0.9774   0.9731   0.9943   0.9989
## Specificity            0.9984   0.9952   0.9946   0.9957   0.9990
## Pos Pred Value         0.9961   0.9800   0.9743   0.9782   0.9956
## Neg Pred Value         0.9949   0.9946   0.9943   0.9989   0.9998
## Prevalence             0.2869   0.1940   0.1746   0.1612   0.1832
## Detection Rate         0.2832   0.1897   0.1699   0.1603   0.1830
## Detection Prevalence   0.2843   0.1935   0.1744   0.1639   0.1839
## Balanced Accuracy      0.9928   0.9863   0.9839   0.9950   0.9989
```
The accuracy of the model is quite high. No need to try an other model. The expected out-of-sample error is 1-accuracy in the cross-validation data: 1-0.9861 = 0.0139.


Prediction without printing out the result


```r
pred.test <- predict(modelRf,test)
```

