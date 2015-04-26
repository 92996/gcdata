---
title: "Readme.md"
author: '92996'
date: "26 Apr 2015"
output: html_document
---

This is a repository with the assignment for "Getting and Cleaning Data".
It includes the script `run_analysis.R`. To use it the files:
```
X_test.txt
X_train.txt
Y_test.txt
Y_train.txt
activity_labels.txt
features.txt
subject_test.txt
subject_train.txt
```
should be in the same directory (symbolic links also work)

The code is commented and I've tried to make it easy to follow. Surely it can be compacted and optimized. In short, it reads the datasets, merges them (rbind),
adds column labels, selects those that are "mean" or "std". Adds subject and
activity (cbind). Aggregates and calculates mean. Finally fixes variable
names a bit, but keeps XYZ uppercase at the end. The meaning of the variables can be found in the file README.txt and features_info.

````{r}
#requirements
library(plyr)
#read tables (helps files or links are in the working directory)
#in working directory X_*.txt, Y_*.ttx, features.txt
#activity_labels.txt, subject*.txt
test <- read.table("X_test.txt", header = FALSE)
train <- read.table("X_train.txt", header = FALSE)
#join (test (2947 rows) first, then train (7352))
# 2947 rows are test
# 7352 rows aew train
#10299 rows total
all <- rbind(test, train)
#add column labels
fea <- read.table("features.txt", header = FALSE, stringsAsFactors = FALSE)
fea <- fea$V2
colnames(all) <- fea
#clean
rm(list = c("test", "train"))
#find columns with "mean" and "std"
colsel_a <- grep ("mean",fea)
colsel_b <- grep("std",fea)
colsel <- c(colsel_a, colsel_b)
#sort indexes to help readability future table
colsels <- sort(colsel)
#make a new table with only these columns
all_mean_std <- all[, colsels]
#clean
rm(list = c("colsel","colsel_a","colsel_b"))
#get activities and label them
act_test <- read.table("Y_test.txt", header = FALSE, stringsAsFactors = FALSE)
act_train <- read.table("Y_train.txt", header = FALSE, stringsAsFactors = FALSE)
act_lab <- read.table("activity_labels.txt", header = FALSE, stringsAsFactors = FALSE)
act_test_words <- sapply(act_test, function(x) { as.character(act_lab$V2[match(x, act_lab$V1)])})
act_train_words <- sapply(act_train, function(x) { as.character(act_lab$V2[match(x, act_lab$V1)])})
#check all is there regardin number of rows
dim(act_test_words)
dim(act_train_words)
#combine and add as a column to all_mean_std
activity <- c(act_test_words[,1], act_train_words[,1])
all_mean_std <- cbind(activity, all_mean_std)
#make a data frame that includes subjects
#get subjects and combine
subject_test <- read.table("subject_test.txt", header = FALSE, stringsAsFactors = FALSE)
subject_train <- read.table("subject_train.txt", header = FALSE, stringsAsFactors = FALSE)
subject <- c(subject_test[,1], subject_train[,1])
#convert to factor, perhaps will help id
subject <- factor(subject)
all_mean_std_sub <- cbind(subject, all_mean_std)
#clean a bit
rm(list = c("subject_test", "subject_train"))
#averages according to subject and activity. Warnings can be ignored.
#They are caused by the two factor columns
all_mean_std_sub_agg_mean <- aggregate(all_mean_std_sub, by = list(subject, activity), FUN = "mean")
#fix columns(delete and rename)
drop <- c("subject","activity")
all_mean_std_sub_agg_mean <- all_mean_std_sub_agg_mean[, !(names(all_mean_std_sub_agg_mean) %in% drop)]
names(all_mean_std_sub_agg_mean)[names(all_mean_std_sub_agg_mean) == "Group.1"] <- "Subject"
names(all_mean_std_sub_agg_mean)[names(all_mean_std_sub_agg_mean) == "Group.2"] <- "Activity"
#work on all names
colnam <- colnames(all_mean_std_sub_agg_mean)
colnam <- sub("\\)","", colnam,)
colnam <- sub("\\(","", colnam,)
colnam <- gsub("-","",colnam,)
colnam <- tolower(colnam)
#keep uppercase XYZ
colnam <- sub("x$","X", colnam,)
colnam <- sub("y$","Y", colnam,); #fix activity
colnam <- sub("z$","Z", colnam,)
colnam[2] <- "activity"
colnam <- sub("bodybody","body",colnam,)
names(all_mean_std_sub_agg_mean) <- colnam
write.table(all_mean_std_sub_agg_mean,"assignment.txt",row.names=FALSE)
rm(list = c("colnam"))
#done
````



*Note: the output file ```assignment.txt``` is included for reference.*