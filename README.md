### Introduction ###
This is a Readme-Document, which aims to explain the work flow of processing run_analysis.R to accomplish the programming assignment of the *Getting and Cleaning Data* class.

### run_analysis.R ###
#### Part 1: Merge the training and the test sets to create one data set. ####
After setting the working directory, the datasets are loaded as tables into R:
```{r}
test_x<-read.table('test/X_test.txt')
test_y<-read.table('test/y_test.txt')
test_subject<-read.table('test/subject_test.txt')
```
Each table (*test_x*, *subject* and *test_y*) displays the same number of rows. Therefore they can easily be merged together. *test_y* contains hardcoded data for each measured activity and is added to the main dataset *test_x* as a new column named *label*. Table *test_subject* contains numbers corresponding to each proband and is added to the main dataset as a new column named *subject*:
```{r}
test_x<-read.table('test/X_test.txt')
test_y<-read.table('test/y_test.txt')
test_subject<-read.table('test/subject_test.txt')
test_x$label<-test_y$V1
test_x$subject<-test_subject$V1
```
The same procedure is repeated for the files located in the *train* subdirectory of the UCI HAR dataset:
```{r]}
train_x<-read.table('train/X_train.txt')
train_y<-read.table('train/y_train.txt')
train_subject<-read.table('train/subject_train.txt')
train_x$label<-train_y$V1
train_x$subject<-train_subject$V1
```
The resulting two independent datasets *test_x* and *train_x* display equal numbers of columns and column names. These are merged together to a new dataset called *mergedDataset*:
```{r}
mergedDataset<-rbind(train_x,test_x)
```
#### Part 2: Extract only the measurements on the mean and standard deviation for each measurement. ####
The file *activity_labels.txt* can be used to decode the *label* column in *mergedDataset*.
The file *features.txt* contains names for all measured features and displays two columns less than *mergedDataset*. The two missing columns correspond to the two newly created columns *labels* and *subject*. Both files are loaded into R.
```{r}
activity_labels<-read.table("activity_labels.txt")
features<-read.table("features.txt")
```
As only measurements on the mean and standard deviation should be used in the assignment, *features* is filtered by column names containing the corresponding abbreviations:
```{r]}
substring_mean<-"mean()"
substring_std<-"std()"
relevant_features<-features[grepl(substring_mean,features$V2,fixed=TRUE)|grepl(substring_std,features$V2,fixed=TRUE),]
filter<-relevant_features$V1
```
Before *mergedDataset* can be subsetted by the created filter, the column names *labels* and *subject* (the two newly created columns in *mergedDataset*) must be added to the filter to keep them in the resulting dataset.
```{r}
label_subject<-c(ncol(mergedDataset)-1,ncol(mergedDataset)) #labels and subject columns
updated_filter<-c(filter,label_subject)
```
Now *mergedDataset* is subsetted to the new *filteredDataset*:
```{r}
filteredDataset<-mergedDataset[updated_filter]
```
#### Part 3 & 4: Use descriptive activity names to name the activities in the data set. Appropriately label the data set with descriptive variable names ####
All columns are labeled and the *filteredDataset* is merged with the table *activity_labels* to decode the *label* column. The results are stored in a newly added column named *activity*. The *label* column is now obsolete and thus will be dropped:
```{r}
collabels<-relevant_features$V2
collabels<-as.character(collabels)
colnames(filteredDataset)[1:length(collabels)]<-collabels
mergeActivityLabel<-merge(filteredDataset,activity_labels,by.x="label",by.y="V1")
names(mergeActivityLabel)[length(mergeActivityLabel)]<-"activity"
mergeActivityLabel$label<-NULL
```
#### Part 5: From the data set in step 4, create a second, independent tidy data set with the average of each variable for each activity and each subject ####
The dataset is renamed to *finalDataset*:
```{r}
finalDataset<-mergeActivityLabel
```
Subsequently, the *dplyr* package comes into play. The dataset is grouped by the *subject* and the *activity* column. The result is stored in a new dataset named *grouped_data*. Finally, the *summarise* function is applied on *grouped_data* to obtain the means of the variables.
```{r}
grouped_data<-group_by(finalDataset,subject,activity)
result<-summarise_each(grouped_data,funs(mean))
```

### The Codebook ###
The Codebook for the *result* dataset can be found in this GitHub-repo. It contains updated text material of the original codebook-file *features-info.txt*, which has been part of the original *UCI HAR Dataset*.
