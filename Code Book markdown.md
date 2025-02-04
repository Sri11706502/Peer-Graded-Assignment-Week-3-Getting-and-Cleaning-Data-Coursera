# Peer Graded Assignment Week 3 
# Getting and Cleaning Data | Coursera

## Description
This file describes the code for creation of tidy data set from the raw data

## Prerequisites

### Data source

This code requires pre downloaded data source
If not downloaded, download from :
https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip

#### Variables used

- features          consists of data from 'features.txt'
- activities        consists of data from 'activity_lables.txt'
- subject_test      consists of data from 'subject_test.txt'
- subject_train     consists of data from 'subject_train.txt'
- x_test            consists of data from 'x_test.txt'
- y_test            consists of data from 'y_test.txt'
- x_train           consists of data from 'x_train.txt'
- y_train           consists of data from 'y_train.txt'
- X                 consists of data from rbind of x_train and x_test
- Y                 consists of data from rbind of y_train and y_test
- subject           consists of data from rbind of subject_train and subject_test
- Merged_Data       consists of data from cbind of X, Y and subject
- TidyData          consists of data from cleaning of Merged_Data








#### Download syntax :

URL <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"

download.file(URL, "Coursera_DS3_Final.zip", method="curl")

Unzip the file once downloaded

To check if folder exists

if (!file.exists("UCI HAR Dataset")) { 
  unzip(filename) 
}

This code requires pre installation of dplyr library
If not installed then it can be installed using install.packages("dplyr")

## Code Area

## Preparing the data

```{r}
# Reading the data into variables
# Reading features
features <- read.table("UCI HAR Dataset/features.txt",
                       col.names = c("n","functions"))
# Reading activities
activities <- read.table("UCI HAR Dataset/activity_labels.txt",
                         col.names = c("code","activity"))
# Reading test files : subject_test, x_test, y_test
subject_test <- read.table("UCI HAR Dataset/test/subject_test.txt",
                           col.names = "subject")
x_test <- read.table("UCI HAR Dataset/test/X_test.txt",
                     col.names = features$functions)
y_test <- read.table("UCI HAR Dataset/test/y_test.txt",
                     col.names = "code")
# Reading train files : subject_train, x_train, y_train
subject_train <- read.table("UCI HAR Dataset/train/subject_train.txt",
                            col.names = "subject")
x_train <- read.table("UCI HAR Dataset/train/X_train.txt",
                      col.names = features$functions)
y_train <- read.table("UCI HAR Dataset/train/y_train.txt",
                      col.names = "code")
```

## Step 1 : Merging datasets : train and test using rbind

```{r, warning = FALSE, message = FALSE}
# Importing the dplyr library
library(dplyr)
#Merging datasets : train and test using rbind
X <- rbind(x_train, x_test)
Y <- rbind(y_train, y_test)
subject <- rbind(subject_train, subject_test)
# Merging X, Y and subject to create final dataset
Merged_data <- cbind(subject,X,Y)
```


## Step 2 : Extracting only measurements on mean and standard deviation for each measurement

```{r}
# Extracting mean and standard deviation of each measurement
TidyData <- select(Merged_data, subject, code,
                   contains("mean"),
                   contains("std"))
```


## Step 3 : Uses descriptive activity names to name the activities in the data set

```{r}
TidyData$code <- activities[TidyData$code, 2]
```


## Step 4 : Renaming the names of columns to meaningful ones

```{r}
# Renaming the names of columns to meaningful ones
names(TidyData)[1] <- "Subject"
names(TidyData)[2] <- "Activity"
names(TidyData) <- gsub("Acc","_Accelerometer",names(TidyData))
names(TidyData) <- gsub("Gyro","_Gyroscope",names(TidyData))
names(TidyData) <- gsub("BodyBody","Body",names(TidyData))
names(TidyData) <- gsub("Mag","_Magnitude",names(TidyData))
names(TidyData) <- gsub("Jerk", "_Jerk", names(TidyData))
names(TidyData) <- gsub("^t","Time",names(TidyData))
names(TidyData) <- gsub("^f","Frequency",names(TidyData))
names(TidyData) <- gsub("tBody","Time_Body",names(TidyData))
names(TidyData) <- gsub("meanFreq", "Mean_Frequency", names(TidyData), ignore.case = TRUE)
names(TidyData) <- gsub("mean", "Mean", names(TidyData), ignore.case = TRUE)
names(TidyData) <- gsub(".std", ".STD", names(TidyData), ignore.case = TRUE)
names(TidyData) <- gsub("angle", "Angle", names(TidyData))
names(TidyData) <- gsub("gravity", "Gravity", names(TidyData))
names(TidyData) <- gsub("\\.\\.\\.", "_", names(TidyData))
names(TidyData) <- gsub("\\.\\.", "", names(TidyData))
names(TidyData) <- gsub("GravityMean", "Gravity_Mean", names(TidyData))
#Viewing changed names
names(TidyData[1:15])
```

## Step 5 : From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject

```{r, warning = FALSE}
Final_Dataset <- group_by(TidyData,Subject,Activity)
Final_Dataset <- summarise_all(Final_Dataset,funs(mean))
```

## Writing data to a file

```{r}
# Writing to a file
write.table(Final_Dataset, "Final_Dataset.txt", row.names = FALSE)
```
