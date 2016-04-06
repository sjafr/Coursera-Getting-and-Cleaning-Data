#The analysis file
The analysis file follows a top down flow of process to create tidyData file.
The major processes are clearly marked and also provide message to show progresss when it is run. Intuitive/descriptive activty names are used to identify processes.

##The required library

library(plyr)

##Set up the working directory

setwd ("C:/Users/Salim")

##Creates set up for checking and then downloading and extracting the Data file if it does not exist.

DataFile <- "./getdata_projectfiles_UCI HAR Dataset.zip"

dirDataFile <- "./UCI HAR Dataset"

###Set download
message("Download data file if required")
DataURL <- 'https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip'

###performing checks to verify if download data exists

if (file.exists(DataFile) == FALSE) { download.file(DataURL, destfile = DataFile) }

###Unzip data file

if (file.exists(dirDataFile) == FALSE) { unzip(DataFile) }

***************
##1 Merges the training and the test sets to create one data set.
message("loading train/test data")
subject_train = read.table("./UCI HAR Dataset/train/subject_train.txt", header = FALSE)

subject_test = read.table("./UCI HAR Dataset/test/subject_test.txt", header = FALSE)

y_train = read.table("./UCI HAR Dataset/train/y_train.txt", header = FALSE)

y_test = read.table("./UCI HAR Dataset/test/y_test.txt", header = FALSE)

X_train = read.table("./UCI HAR Dataset/train/X_train.txt", header = FALSE)

X_test = read.table("./UCI HAR Dataset/test/X_test.txt", header = FALSE)

message("Merging train/test data")
x_bind<-rbind(X_train, X_test)
y_bind<-rbind(y_train, y_test)
y_sub<-rbind(subject_train, subject_test)

***************
##2) Extracts only the measurements on the mean and standard deviation for each measurement.
message("Reading feature labels and remove un-needed data")
features = read.table("./UCI HAR Dataset/features.txt")

message("Selecting Mean/Standard deviation data")

### filter for required values Std and the mean
mean_Std_vals <- grep("mean\\(|std\\(", features[,2])

***************
##3 Use descriptive names from  features
x_bind <- x_bind[, mean_Std_vals]

### Re order the names
names(x_bind) <- features[mean_Std_vals, 2]

### Get and use descriptive activity labels

activity_labls <- read.table("./UCI HAR Dataset/activity_labels.txt")

### Update values with correct activity names
y_bind[, 1] <- activity_labls[y_bind[, 1], 2]

### Add descriptive column name
names(y_bind) <- "Activity"

***************
##4.Appropriately labels the data set with descriptive variable names.

message("Adding descriptive variable names")
names(y_sub ) <- "Subject"

###Combine data table by columns and create table
tidyData_raw<- cbind(x_bind,y_sub, y_bind )
write.table(tidyData_raw, "tidyData_raw.txt", row.name=FALSE)

***************
##5. From the data set in step 4, creating a second, independent tidy data set with the average of each variable for each activity and each subject.

message("Creating a tidy data set with the average of each variable for each activity and each subject.")
### making provisions for the last two row Subject, Activity as non numeric
tidyData <- ddply(tidyData_raw, .(Subject, Activity), function(x) colMeans(x[, 1:66]))

write.table(tidyData, "tidyData.txt", row.name=FALSE)
