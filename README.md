# runAnalysis Readme
## Course Project for Getting and Cleaning Data subject
This readme file describes how the script 'runAnalysis.R' works.  
The data source comes from the Samsung dataset found at this [link](http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones).  

### Requirements  
A quick background introduction on the requirements for this script are:  
1. Merges the training and the test sets to create one data set  
2. Extracts only the measurements on the mean and standard deviation for each measurement.  
3. Uses descriptive activity names to name the activities in the data set  
4. Appropriately labels the data set with descriptive variable names.  
5. From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.

### install the load the following packages.
install.packages("data.table")
library(data.table)

### 1. Merge
### First, import the data files into the data sets variable in R. Inertia signals data are ignored since they are not required in the analysis.

features <- readLines("features.txt")  
Actlabels <- readLines("activity_labels.txt")  
x.test <- read.table("test/X_test.txt")  
y.test <- fread("test/y_test.txt")  
sub.test <- fread("test/subject_test.txt")  
x.train <- read.table("train/X_train.txt")  
y.train <- fread("train/y_train.txt")  
sub.train <- fread("train/subject_train.txt")

### THe activities are labelled before the data sets are combined with the labels., let's work on our activities' labels.  
### The activities are referenced by their index numbers in 'y.test' and 'y.train'. These are converted to characters for easier data manipulation/comparison.

y.test <- as.data.table(lapply(y.test, as.character))  
y.train <- as.data.table(lapply(y.train, as.character))  

### Column label are renamed to make it more human readable. They are cleaned by removing the space, index digit and underscore.

Actlabels <- substr(Actlabels, 3, nchar(ColLabel))  
Actlabels <- gsub("*_", " ", Actlabels)  

### Using a for loop,  copy the appropriate labels to the rows with corresponding index number in 'y.test' and 'y.train'.

for(i in 1:length(Actlabels)) { y.test[V1==i] <- Actlabels[i] }  
for(i in 1:length(Actlabels)) { y.train[V1==i] <- Actlabels[i] }  

### Add new columns to the test and train data sets with subjects' index (sub.test and sub.train) and Actlabels (y.test and y.train). Finally, append the rows of data in 'train' to 'test' and form a new data set 'merged'.

test <- cbind(sub.test, y.test, x.test)  
train <- cbind(sub.train, y.train, x.train)  
mergedData <- rbind(test, train)  


### 2. For each measurement, extract their mean and standard deviation by searching for keywords "mean" and "std" 
### the match is case sensitive so Angle functions which also contains 'Mean' and 'Std' must be clean up first.
### A 'MatchData' vector is created with the two columns 'Subjects' and 'Activities'

MatchData <- c("Subjects", "Activities", "mean", "std")  

### grep function used here to search for matching words in the column names of the 'merged' data set.  

columns <- grep(paste(MatchData, collapse="|"), colnames(mergedData), ignore.case=F)

### Copy the matched column from the 'merged' data set and duplicate to a new data set 'filtered'.

filtered <- mergedData[, columns, with=FALSE]


### 3. Name the activities  
### as per step1 prep work 

### 4. Label variable names  
### To make the dataset more easy to understand, the column names are renamed and clean up.  
### * Removed all brackets ()  
### * Variable preceding with "t" is replaced with with "time-", to describe this is a time reading  
### * Variable preceding with "f" is replaced with "freq-" to describe this is a frequency reading  
### * All blank spaces are removed.  
### column names 'Subjects' and 'Activities' have to be added back to the front of the vector in order.

features <- sub("()", "", fixed=T, features)  
features <- sub("t", "time-", fixed=T, features)  
features <- sub("f", "freq-", fixed=T, features)  
features <- sub(".* ", "", fixed=F, features)  
features <- append(features, c("Subjects", "Activities"), 0)  
suppressWarnings(colnames(mergedData) <- features)  


### 5. Average of each activity and subject
### To get the average by subjects and activities, aggregate and mean function is applied to the above tidy dataset. 
### 'Subjects' and 'Activities' columns are no longer relevant and the correct results are in the columns 'Group.1' and 'Group.2'.
### Irrelevant columns are removed or and renamed accordingly.

average <- suppressWarnings(aggregate(filtered, by=list(filtered$Subjects,filtered$Activities), FUN=mean))  
average$Subjects <- average$Activities <- NULL  
setnames(average, old=c("Group.1", "Group.2"), new=c("Subjects", "Activities"))  
