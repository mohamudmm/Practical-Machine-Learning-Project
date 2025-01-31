---
title: "Practical Machine Learning Project"
author: "Mohamud M Mohamud"
date: "Friday, August 22, 2014"
output: html_document
---
#Introduction

This report is for the Coursera Practical Machine Learning Project.The goal of this project is to predict how a subject is performing a task from a set  of measurements.
With the advances in wearable devices that record data about the activities a person engages in,it becomes possible to use these devices to monitor the elderly,the disabled and the sick.

In this project the data is from  http://groupware.les.inf.puc-rio.br/har. we want to train our data on the training data and apply the resulting machine learning method on the testing data.

##Data processing

```{r}
library(caret)
library(plyr)

```

First we read in the data contained in the training and testing folders.
```{r}
##read the training and testing files into two separate files.
pml_train <- read.csv("pml-training.csv",na.strings=c("#DIV/0!","NA"))
pml_test <- read.csv("pml-testing.csv",na.strings=c("#DIV/0!","NA"))

```
Next we impute the missing values with the mean of the feature.

```{r cache=TRUE}

for (i in which(sapply(pml_train, is.numeric))) {
    pml_train[is.na(pml_train[, i]), i] <- mean(pml_train[, i],  na.rm = TRUE)
}
for (i in which(sapply(pml_test, is.numeric))) {
    pml_test[is.na(pml_test[, i]), i] <- mean(pml_test[, i],  na.rm = TRUE)
}

```

##Exploratory Data Analysis

Representative plots of the features vs the activity are shown below.

```{r}
featurePlot(x=pml_train[,14:18],y=pml_train$classe)


```

##Data Transformation and feature selection

In order to get a better handle of the analysis we would like to reduce the features and select those most relevant.

First all the features with zero or near zero variance were identified and removed.

Secondly those features denoting times of exercise,participant's name,or serial numbers given to participants were removed.

```{r cache=TRUE}
#remove the features with zero and near zero variances
nzv <- nearZeroVar(pml_train)
filtered_pml_train <- pml_train[,-nzv]
nzv1 <- nearZeroVar(pml_test)
filtered_pml_test <- pml_test[,-nzv]
#remove the features indicating participant names,times
#of exercise.
filtered_pml_train <- filtered_pml_train[,-c(1,2,3,4,5,6,7)]
filtered_pml_test <- filtered_pml_test[,-c(1,2,3,4,5,6,7)]

```

These operations provided a clean dataset on which the training of the data was conducted.


##Training the data

The Caret library was used for the training of the data.The training data was split into a 70:30 training and testing sets.The testing set was used to cross validate
the prediction model.The primary goal of this project is to build a model that predicts the class of the activity as one of five classes A-E.
Since this is a classification problem I chose a tree based method the Generalized Boosting Model(gbm).


```{r cache=TRUE,results='hide'}
set.seed(1217)
Intrain <- createDataPartition(filtered_pml_train$classe,p=0.7,list=FALSE)
filtTrain <- filtered_pml_train[Intrain,]
filtTest <- filtered_pml_train[-Intrain,]
fitControl <- trainControl(method="cv",number=5)
fit1 <- train(classe~.,data=filtTrain,method="gbm",trControl = fitControl)
predictions <- predict(fit1,filtTest)
```
After the test method was developed it was applied to the cross-validation set to determine the out of sample error. 
The model developed in this project has an accuray of 
0.962 with 95% CI(0.958,0.966) and p-value of 0 and thus is statistically significant.
The predicted accuracy was borne by the result of applying the model to the 20 test cases where it correctly predicted all of them.
```{r cache = TRUE}
confusionMatrix(predictions,filtTest$classe)
testpredictions <- predict(fit1,filtered_pml_test)
testpredictions
```


