---
title: "CookBook"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## R Markdown

This is an R Markdown document as a cook book of the course project assignment that describes the analysis result data, and its variables.
The assignment generates three data sets:
1. the merged data set that is saved as "merged_dataset.txt". This data set combine the training data and test data with 30 experiment subjects and 6 types of activities.
2. the data set of measurements on means and standard deviation, the data set is saved as "measurements_on_mean_sd.txt". 
3. a new second data set with the average of each variable for each activity and each subject, the data set is saved as "subject_activity_summary.txt"

## variables in "merged_dataset.txt"
variables in "merged_dataset.txt":

V1~V561:  measurements for the experiment features that are described in "features.txt"
experiment_subject:   the subjects that participated in the experiment
activity_desc:  description of the experiment activity
experiment_group:   describes whether the data is from training experiment group or from test experiment group


## variables in "measurements_on_mean_sd.txt"
variables in "measurements_on_mean_sd.txt":

V~:   variables start with "V" follow by number are measurements on mean or standard deviation, detail description is saved in "features_On_Means_Standard_Deviation.txt"
experiment_subject:   the subjects that participated in the experiment
activity_desc:    description of the experiment activity
experiment_group:   describes whether the data is from training experiment group or from test experiment group


## variables in "subject_activity_summary.txt"
variables in "subject_activity_summary.txt":

experiment_subject:   the subjects that participated in the experiment
activity_desc:  description of the experiment activity
V~:   variables start with "V" follow by number are the average of each measurement on mean or standard deviation, variable detail description is saved in "features_On_Means_Standard_Deviation.txt"

## Performed work and transformation
This project needs to merge the training set and test set into one data set; and then extract the measurements on means and standard deiation; the experiment activity need to convert to a descriptive pattern. The work I performed in following steps:
1. load the training data, activities, activity labels, training subjects, and feature tables
2. give proper names to variables for above sets, but leave the variables names as it is for training measurements since it is described in features cookbook.
3. using inner_join to transform activities into activity description
4. combine the columns from training data, activity, and subject
5. doing the same work for test data sets
6. combine the row from training data and test data into merged data set "merged_dataset.txt"
7. use grep() function to get the features on mean and standard deviation and save the result to "features_On_Means_Standard_Deviation.txt" as a cookbook for measurements on mean and standard deviation
8. sub-setting the merged dataset at step 6 with measurements on means and standard deviation and save it to "measurements_on_mean_sd.txt"
9. using group_by(), summarize() and across() function the create a new data set with average of each variable for each activity and each subject, and save the result to "subject_activity_summary.txt" 

## script as
```{r}
library(dplyr)


measurements <- read.table("./UCI HAR Dataset/features.txt")
train_data <- read.table("./UCI HAR Dataset/train/X_train.txt")
train_activities <- read.table("./UCI HAR Dataset/train/y_train.txt")
train_subjects <- read.table( "./UCI HAR Dataset/train/subject_train.txt")
colnames(train_subjects)<- c("experiment_subject")

activity_labels <- read.table("./UCI HAR Dataset/activity_labels.txt")


colnames(activity_labels) <- c("activity","activity_desc")
colnames(train_activities) <- c("activity")
train_activities <- inner_join(train_activities,activity_labels,by=c("activity" = "activity")) %>% select(activity_desc)



train_data <- cbind(train_data,train_subjects,train_activities)

train_data <- mutate(train_data,experiment_group="training")



test_data <- read.table("./UCI HAR Dataset/test/X_test.txt")
test_activities <- read.table("./UCI HAR Dataset/test/y_test.txt")
test_subjects <- read.table( "./UCI HAR Dataset/test/subject_test.txt")
colnames(test_subjects)<- c("experiment_subject")



colnames(test_activities) <- c("activity")
test_activities <- inner_join(test_activities,activity_labels,by=c("activity" = "activity")) %>% select(activity_desc)


test_data <- cbind(test_data,test_subjects,test_activities)



test_data <- mutate(test_data,experiment_group="test")


experiment_result <- rbind(train_data,test_data)
write.table(experiment_result,"./merged_dataset.txt",row.names = FALSE)


colnames(measurements) <-c("variable_seq","feature")


mean_std_features <- measurements[grep("mean|std",measurements$feature),]


meanFreq <- mean_std_features[grep("meanFreq",mean_std_features$feature),1]
mean_std_features <- filter(mean_std_features,!variable_seq %in% meanFreq)
write.table(mean_std_features,"./features_On_Means_Standard_Deviation.txt",row.names = FALSE)


mmsd_vars <- paste0("V",mean_std_features$variable_seq)
subset_vars<- c("experiment_subject","experiment_group","activity_desc",mmsd_variables)


mmsd <- select(experiment_result,all_of(subset_vars))
write.table(mmsd,"./measurements_on_mean_sd.txt",row.names = FALSE)


subject_activity_summary <- group_by(mmsd,experiment_subject,activity_desc) %>% 
                            summarize(across(mmsd_variables,mean))

write.table(subject_activity_summary,"./subject_activity_summary.txt",row.names = FALSE)


rm(train_data)
rm(test_data)
rm(train_activities)
rm(train_subjects)
rm(test_activities)
rm(test_subjects)
```