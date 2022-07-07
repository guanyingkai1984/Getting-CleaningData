## The script performs the data preparation and then followed by steps required as described in the course project’s definition.

### 0 Prepare the Data : Using the Prepare.R Script

```
## Download the ZIP and UnZIP
filename <- "Files.zip"

# Checking if ZIP already exists, if not download
if (!file.exists(filename)){
  fileURL <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
  download.file(fileURL, filename, method="curl")
}  

# Checking if folder exists, if not unzip
if (!file.exists("UCI HAR Dataset")) { 
  unzip(filename) 
}

## Assigning all data frames
features <- read.table("UCI HAR Dataset/features.txt", col.names = c("n","functions"))
activities <- read.table("UCI HAR Dataset/activity_labels.txt", col.names = c("code", "activity"))
subject_test <- read.table("UCI HAR Dataset/test/subject_test.txt", col.names = "subject")
x_test <- read.table("UCI HAR Dataset/test/X_test.txt", col.names = features$functions)
y_test <- read.table("UCI HAR Dataset/test/y_test.txt", col.names = "code")
subject_train <- read.table("UCI HAR Dataset/train/subject_train.txt", col.names = "subject")
x_train <- read.table("UCI HAR Dataset/train/X_train.txt", col.names = features$functions)
y_train <- read.table("UCI HAR Dataset/train/y_train.txt", col.names = "code")
```

### The following using the run_analysis.R Script

### 1 Merges the training and the test sets to create one data set.
```
# Row Bind Train & Test
X <- rbind(x_train, x_test)
Y <- rbind(y_train, y_test)
Subject <- rbind(subject_train, subject_test)

# Row Bind Subject, Y, X (all)
Merged_Data <- cbind(Subject, Y, X)
```

### 2 Extracts only the measurements on the mean and standard deviation for each measurement.
```
# Only select Subject, Code, and Column Name containing "mean" & "std"

TidyData <- Merged_Data %>% 
  select(subject, code, contains("mean"), contains("std"))
```

### 3 Uses descriptive activity names to name the activities in the data set

```
# Entire numbers in code column of the TidyData replaced with corresponding activity taken from second column of the activities variable

TidyData$code <- activities[TidyData$code, 2]
```

### 4 Appropriately labels the data set with descriptive variable names. 

```
# code column in TidyData renamed into activities
names(TidyData)[2] = "activity"
# All Acc in column’s name replaced by Accelerometer
names(TidyData)<-gsub("Acc", "Accelerometer", names(TidyData))
# All Gyro in column’s name replaced by Gyroscope
names(TidyData)<-gsub("Gyro", "Gyroscope", names(TidyData))
# All BodyBody in column’s name replaced by Body
names(TidyData)<-gsub("BodyBody", "Body", names(TidyData))
# All Mag in column’s name replaced by Magnitude
names(TidyData)<-gsub("Mag", "Magnitude", names(TidyData))
# All start with character f in column’s name replaced by Frequency
names(TidyData)<-gsub("^f", "Frequency", names(TidyData))
# All start with character t in column’s name replaced by Time
names(TidyData)<-gsub("^t", "Time", names(TidyData))
# All tBody in column’s name replaced by TimeBody
names(TidyData)<-gsub("tBody", "TimeBody", names(TidyData))
# All -mean() in column’s name replaced by Mean
names(TidyData)<-gsub("-mean()", "Mean", names(TidyData), ignore.case = TRUE)
# All -std() in column’s name replaced by STD
names(TidyData)<-gsub("-std()", "STD", names(TidyData), ignore.case = TRUE)
# All -freq() in column’s name replaced by Frequency
names(TidyData)<-gsub("-freq()", "Frequency", names(TidyData), ignore.case = TRUE)
# All angle in column’s name replaced by Angle
names(TidyData)<-gsub("angle", "Angle", names(TidyData))
# All gravity in column’s name replaced by Gravity
names(TidyData)<-gsub("gravity", "Gravity", names(TidyData))
```

### 5 From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.
```
FinalData <- TidyData %>%
  group_by(subject, activity) %>%
  summarise_all(funs(mean))
```

### Save the FinalData
```
write.table(FinalData, "FinalData.txt", row.name=FALSE)
```
