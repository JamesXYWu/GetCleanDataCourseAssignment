# GetCleanDataCourseAssignment
Course assignment for getting and cleaning data

```{r}
library(dplyr)
```

# load training data sets
```{r}
measurements <- read.table("./UCI HAR Dataset/features.txt")
train_data <- read.table("./UCI HAR Dataset/train/X_train.txt")
train_activities <- read.table("./UCI HAR Dataset/train/y_train.txt")
train_subjects <- read.table( "./UCI HAR Dataset/train/subject_train.txt")
colnames(train_subjects)<- c("experiment_subject")

activity_labels <- read.table("./UCI HAR Dataset/activity_labels.txt")
```
# convert training activities to a descriptive name
```{r}
colnames(activity_labels) <- c("activity","activity_desc")
colnames(train_activities) <- c("activity")
train_activities <- inner_join(train_activities,activity_labels,by=c("activity" = "activity")) %>% select(activity_desc)
```
#combine training data with training activities and training subjects training data, training activities, and training subjects have the same observations of 7352, need combine these three data frames together by columns
```{r}
train_data <- cbind(train_data,train_subjects,train_activities)
```
# add an experiment group column (name it as experiment_group) and mark this data set's observation as "training" to tell apart this part of data from testing data when we are going to merge the training data set and the test data set
```{r}
train_data <- mutate(train_data,experiment_group="training")
```

# Load test data sets
```{r}
test_data <- read.table("./UCI HAR Dataset/test/X_test.txt")
test_activities <- read.table("./UCI HAR Dataset/test/y_test.txt")
test_subjects <- read.table( "./UCI HAR Dataset/test/subject_test.txt")
colnames(test_subjects)<- c("experiment_subject")
```
# convert testing activity to a descriptive name
```{r}
colnames(test_activities) <- c("activity")
test_activities <- inner_join(test_activities,activity_labels,by=c("activity" = "activity")) %>% select(activity_desc)
```
#combine testing data with test activity and test subject test data, test activity, and test subject have the same observations of 7352, combine these three data frames together by columns
```{r}
test_data <- cbind(test_data,test_subjects,test_activities)
```
# add an experiment group column (named it as exp_group) and mark test_data data set observation as "test" to tell apart this part of data from training data when we merge the training data set and the test data set
```{r}
test_data <- mutate(test_data,experiment_group="test")
```
# Both training data and test data have same set of feature measurements and activity, we combine the observations (rows) from these two data set into one data set and name it as experiment_result,and write the merged data set to "./merged_dataset.txt"
```{r}
experiment_result <- rbind(train_data,test_data)
write.table(experiment_result,"./merged_dataset.txt",row.names = FALSE)
```
#	Extracts only the measurements on the mean and standard deviation for each measurement and save it into a data frame
```{r}
colnames(measurements) <-c("variable_seq","feature")
```
# filter out the features with mean() and std()
```{r}
mean_std_features <- measurements[grep("mean|std",measurements$feature),]
```
# "meanFreq()" is contained in the subset that needs to be removed
```{r}
meanFreq <- mean_std_features[grep("meanFreq",mean_std_features$feature),1]
mean_std_features <- filter(mean_std_features,!variable_seq %in% meanFreq)
write.table(mean_std_features,"./features_On_Means_Standard_Deviation.txt",row.names = FALSE)
```
# sub-setting the measurements on means and standard devidation
# 1. prepare the variable names 
```{r}
mmsd_vars <- paste0("V",mean_std_features$variable_seq)
subset_vars<- c("experiment_subject","experiment_group","activity_desc",mmsd_variables)
```
# 2. sub-setting the measurement result, and save the data set
```{r}
mmsd <- select(experiment_result,all_of(subset_vars))
write.table(mmsd,"./measurements_on_mean_sd.txt",row.names = FALSE)
```
# Creating a new second data set with the average of each variable for each activity and each subject, and save the data set
```{r}
subject_activity_summary <- group_by(mmsd,experiment_subject,activity_desc) %>% 
                            summarize(across(mmsd_variables,mean))

write.table(subject_activity_summary,"./subject_activity_summary.txt",row.names = FALSE)
```
The result contains four output files:
"merged_dataset.txt": contains the merged data from training sets and test sets
"measurements_on_mean_sd.txt": contains the measurements only on mean and standard deviation
"features_On_Means_Standard_Deviation.txt": cook book for measurements on mean and standard deviation
"subject_activity_summary.txt": contains data with the average of each variable for each activity and each subject