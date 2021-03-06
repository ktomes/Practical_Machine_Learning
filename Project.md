Practical Machine Learning - Course Project
========================================================

Background

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset). 

Goal:

The goal of this project is to produce a model that predicts the accuracy / manner in which six test subjects did a weight lifting excercise. All subjects were wearing accelerometer sensors on their Arm, Belt, Forearm and on the Dumbell. This data was collected and made available ffrom the HAR website at http://groupware.les.inf.puc-rio.br/har. 

## Load Raw Data
The first task was to load both the training and test set.

```{r eval=FALSE}
PMLtrain <- read.csv("pml-training.csv",na.strings=c("NA",""))
PMLtest  <-read.csv("pml-testing.csv",na.strings=c("NA",""))
```

## Cross Validation
The provided test set was divided into 2 sets: 70% for training, 30% for validation. The data set was partioned by the `classe` variable which is designated as the outcome variable. 

```{r eval=FALSE}
trainInx <- createDataPartition(y = PMLtrain$classe, p=0.7,list=FALSE)  
training <- PMLtrain[trainInx,]
validate <- PMLtrain[-trainInx,]
dim(training);dim(validate)
```
## Data Clean and Feature Selection

After examining the data sets it is determined that columns containing NA's should be removed using this code:
```{r eval=FALSE}
NAs <- apply(training,2,function(x) {sum(is.na(x))})  
training <- training[,which(NAs == 0)] 
validate <-validate[,which(NAs == 0)]
dim(training);dim(validate)
```

Discard the following features as predictors that do not contain sensor data: timestamp, X, user_name, new_window, num_window. This will produce a final tidy traing test set that we can model against.
```{r eval=FALSE}
removeFeatures <- grep("timestamp|X|user_name|new_window|num_window",names(training)) 
training <- training[,-removeFeatures]
validate <- validate[,-removeFeatures]
```
## Modeling
Two modeling approaches were tried - Random Forests (RF), and Gradiant Boosting Machine (GBM).
Applying the confusionMatrix function showed that The Random Forests function returned a higher 
degree of accuracy (.96 vs .94) respectively.
```{r eval=FALSE}
> accuracy.rf$overall[1]   > accuracy.gbm$overall[1]
Accuracy                     Accuracy
0.9634664                    0.9413764 
```              
Create a Model using Random Forest from the Caret Package. Test the accuracy of the model against our validation training set created earlier. We used a set.seed(825) to aid in reproducing our results.
```{r eval=FALSE}
set.seed(825)
modFit <- train(training$classe ~.,data = training,method="rf",trControl = trainControl(method='cv'))
accuracy.rf <- confusionMatrix(validate$classe, predict(modFit,validate))
accuracy.rf$overall[1]
```
## Prediction

Predict the outcomes based on the created Random Forest model.
The provided test set must now match the tidy training set so we repeat the process of removing NAs and non-sensor values. 
```{r eval=FALSE}
NAs <- apply(PMLtest,2,function(x) {sum(is.na(x))})  
TestSet <- PMLtest[,which(NAs == 0)]
removeFeatures <- grep("timestamp|X|user_name|new_window|num_window",names(TestSet)) 
TestSet <- TestSet[,-removeFeatures] 
```
Make prediction and pass the outcomes to function pml_write_files (Courtesy of Coursera)
```{r eval=FALSE}
prediction<-(predict(modFit, TestSet))
class(prediction)
prediction
```
The Coursera provide function to write predicions (A,B,C,D or E) to 
individual text files
```{r eval=FALSE}
pml_write_files = function(x){
        n = length(x)
        for(i in 1:n){
                filename = paste0("problem_id_",i,".txt")
                write.table(x[i],file=filename,quote=FALSE,row.names=FALSE,col.names=FALSE)
        }
```
