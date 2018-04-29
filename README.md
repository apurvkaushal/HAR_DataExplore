# ProgrammingAssignment4-CleanData

# Uploading necessary libraries for data preprocessing
library(dplyr)
library(tidyr)
library(data.table)

# Reading files from local storage
x_train<-read.delim("train/X_train.txt",header=FALSE)
y_train<-read.delim("train/y_train.txt",header=FALSE)
x_test<-read.delim("test/X_test.txt",header=FALSE)
y_test<-read.delim("test/y_test.txt",header=FALSE)

# x_test file contains data in inapprorpriate dimensions, the below code breaks down the single column into the necessary columns of different features.
x_test$V1<-as.character(x_test$V1)
x_test$V1<-gsub("  "," ",x_test$V1)
x_test$V1<-sub(".*? ","",x_test$V1)

# Inputting features.txt file to get feature names and insert into the dataframes
features<-read.delim("features.txt",header=FALSE)
features$V1<-as.character(features$V1)
features<-gsub(".* ","",features$V1)
ftr<-as.matrix(features)

# Splitting the  untidy columns into the required fields
x_test<-separate(x_test,V1,ftr,sep=" ")

# Coherent naming of train and test datasets
names(x_train)<-names(x_test)

# Joining test and train data for feature vectors
finaldf<-rbind(x_train,x_test)

# Joining test and train data for target vector
y<-rbind(y_train,y_test)

# Extracting only mean and standard deviation valued columns
newdf<-finaldf[grep("mean|std",names(finaldf),ignore.case=TRUE)]

# Converting character type data into numeric data for computations.
colnames<-names(newdf)
newdf[colnames]<-sapply(newdf[colnames],as.numeric)

# Appending Activity column to the dataset
newdf$Activity<-y$V1

# Changing numeric coding of activities to descriptive coding
activities<-read.delim("activity_labels.txt",header=FALSE)
activities$V1<-gsub(".* ","",activities$V1)
newdf<-mutate(newdf,Activity=activities[newdf$Activity,])

# Inputting subject_train & subject_test text files that contain the descriptive indicators for the subjects that were a part of the experiment
subject_train<-read.delim("train/subject_train.txt",header=FALSE)
subject_test<-read.delim("test/subject_test.txt",header=FALSE)

# Merging test and train data for subject indicators.
subject<-rbind(subject_train,subject_test)

# Adding subject column to the data set
newdf$Subject<-subject$V1

# Creating dataset that has averaged values for all variables, grouped by Activity & Subject
meandf<-aggregate(select(newdf,-Activity,-Subject),list(newdf$Activity,newdf$Subject),mean)

# Renaming columns for Subject & Activity
colnames(meandf)[colnames(meandf)=="Group.1"]<-"Activity"
colnames(meandf)[colnames(meandf)=="Group.2"]<-"Subject No."

# Printing the final data set
print(meandf)
