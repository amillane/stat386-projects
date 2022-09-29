---
layout: post
title:  "A Nature Walk Into Machine Learning"
date:   2022-09-22
author: Drew Millane 
description: A simple tutorial on how to use random forests in R programming.
image: https://www.timeforkids.com/wp-content/uploads/2019/09/final-cover-forest.jpg
---

# **The Power of Machine Learning and Random Forests**

<p align="center">
<img src = "https://images.unsplash.com/photo-1580927752452-89d86da3fa0a?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=2070&q=80" width = "500" height='250'>
</p>
	
If you are wanting to know how to implement random forests in R, then you have come to the right place!  Machine learning is a powerful tool that helps us accurately predict certain outcomes by creating a model with the information we give it. In other words, it ‘learns’ with what is given and predicts with data that hasn’t been seen yet. Isn’t that amazing?! In this article, you will get the chance to wield this power by making a random forest model using a classic Kaggle dataset called **Titanic**, predicting which passengers lived and died based on their background and information. To grab the datasets and if you want to learn more about Kaggle click this [link](https://www.kaggle.com/c/titanic)

# **Steps To Making a Successful Random Forest**

### **Load in Libraries and Datasets**

The first step we must take is installing and loading the libraries we will use for this random forest example. We won’t go into depth about what each of the libraries do, but if you are curious, you can follow this [link](https://www.rdocumentation.org) to the R documentation website to learn more. 

```

library(tidyverse)
library(viridis)
library(vroom)
library(ranger)
library(missForest)

```

Next, we will load the two datasets of the Titanic passengers. The difference between the two is that the training dataset will help us make our model, and our test dataset will be plugged into our model. A good way of telling the difference is that the test dataset does not have the response variable **Survived**. The random forest model will produce those for the test data. 

```
# Loading The Data --------------------------------------------------------
train <- vroom("~/Desktop/RA /RA/titanic/train.csv") %>% 
  mutate(Survived = as.factor(Survived))
test <- vroom("~/Desktop/RA /RA/titanic/test.csv")

```
### **Combining The Datasets**

Before we can train a random forest model, we need to do some data cleaning to both datasets. We must make sure that there is no missing data or our model cannot be created. Using the function **plot_missing()**  we can produce a graph that shows us which columns have missing data. To make our lives easier we will combine the two datasets and clean them at the same time to save lines of code. 

```
train1 <- select(train,-c(Survived))

combine <- rbind(train1,test) 

plot_missing(combine)

```


### **Removing Columns and Factoring Columns**
	
For the sake of simplicity we are going to delete some of the columns. However, they have useful information that we can use to train our model. That information can be collected using “feature importance”. This article will not cover this topic, but you can learn more in this article [here](https://towardsdatascience.com/understanding-feature-importance-and-how-to-implement-it-in-python-ff0287b20285). 

```
combine <- combine %>%
  mutate(Pclass = as.factor(Pclass),
         Sex = as.factor(Sex),
         Embarked = as.factor(Embarked)) %>% 
  select(-c(Ticket, Name, Cabin))
  
```
Another thing we will do is make some of our discrete variables into factors so that they can be distinguished separately rather than as numbers. We will also convert binary categorical data to be 1’s and 0’s, which will better enhance the model we train. 

```


#Male and female
combine$Male[combine$Sex == 'male'] <- 1
combine$Male[combine$Sex != 'male'] <- 0
combine$Male <- as.factor(combine$Male)

combine$Female[combine$Sex == 'female'] <- 1
combine$Female[combine$Sex != 'female'] <- 0
combine$Female <- as.factor(combine$Female)

#Embarked
combine$Emb_C[combine$Embarked == "C"] <- 1
combine$Emb_C[combine$Embarked != "C"] <- 0
combine$Emb_C <- as.factor(combine$Emb_C)

combine$Emb_S[combine$Embarked == "S"] <- 1
combine$Emb_S[combine$Embarked != "S"] <- 0
combine$Emb_S <- as.factor(combine$Emb_S)

combine$Emb_Q[combine$Embarked == "Q"] <- 1
combine$Emb_Q[combine$Embarked != "Q"] <- 0
combine$Emb_Q <- as.factor(combine$Emb_Q)
```
### **Imputing Dataset and Uncombining Datasets**

The last thing we are going to use is an imputing function to fill in the missing ages for some of the passengers and give missing fare values the mean of the column. Now we can see by using the same plotting function that there is no more missing data! Now we separate the test and train datasets into their own objects. 

```
#Get rid of old columns 
combine <- combine %>% 
  select(-c(Sex,Embarked)) %>%
  as.data.frame()

#imputation of fare column 
combine$Fare[is.na(combine$Fare)] <- mean(combine$Fare)


#imputation
imputed <- missForest(combine,ntree = 100)

combine <- imputed$ximp

plot_missing(combine)

#seperating the data out again 

new_test <- combine[892:nrow(combine),]

new_train <- combine[1:891,] %>% mutate(Survived = train$Survived)

```

### **Training a Random Forest**
This portion of code below is conducting cross-validation, or trying to discover the best parameters to use in our random forest model. *mtry* is describing how many variables to select from when splitting our tree, *n_trees* describes how many trees to make, and *min_n* describes the minimum amount of observations in the last leaves of a tree. 


```
##############
## Train a Random forest
#############

## Set Possible Tuning Parameters
mtry.grid <- 1:2
n_trees <- 100
min_n <- c(1:10)
tune.grid <- expand.grid(mtry=mtry.grid, min_n=min_n)

## Split dataset into K-pieces for K-fold cross validation
K <- 5
folds <- caret::createFolds(new_train$Survived, k=K, list=FALSE)

## Run Cross-validation
misclass <- matrix(0, nrow=nrow(tune.grid), ncol=K)
for(p in 1:nrow(tune.grid)){
  for(cv in 1:K){
    
    ## Fit the RF
    rf <- ranger(formula = Survived~., 
                 data=new_train[folds!=cv,],
                 num.trees=n_trees, 
                 mtry=tune.grid$mtry[p],
                 min.node.size=tune.grid$min_n[p])
    
    ## Predict the test set
    preds <- predict(rf, data=new_train[folds==cv,])
    
    ## Calculate missclass
    misclass[p,cv] <- mean(preds$predictions != new_train$Survived[folds==cv])
  }
}
misclass <- rowMeans(misclass)
ggplot() + geom_raster(aes(x=tune.grid$mtry, y=tune.grid$min_n, fill= misclass)) +
  scale_fill_viridis()
```

### **Using Best Parameters and Predicting**
Now that we have discovered our best parameters, we are going to add those to our random forest model and then predict using our test dataset!

```
## Retrain the RF using the best setting
mtry <- tune.grid$mtry[which.min(misclass)]
min_n <- tune.grid$min_n[which.min(misclass)]

rf <- ranger(formula = Survived~., 
             data=new_train,
             num.trees=n_trees, 
             mtry=mtry,
             min.node.size=min_n)
             
##Using test data set 
ship_preds <- predict(rf, data = new_test)

## Put into a dataframe and create a file 
predict_data <- data.frame(PassengerId = new_test$PassengerId, Survived = ship_preds)
predict_data

write_csv(predict_data,"predict_titanic.csv")

```
# **The Tree Line Doesn't Stop Here!**

<p align="center">
<img src = "https://wallpapercave.com/wp/wp9554121.jpg" width = "700" height='250'>
</p>

You have learned how to impute data, tune parameters, and predict using a trained random forest model in R! However, this is **ONLY** a basic tutorial on how to harness the power of random forests. There is so much more that can be done to this particular dataset to have a more accurate model. I encourage you, if you are really interested in enhancing your data science skills, to learn more about the power of random forests!
