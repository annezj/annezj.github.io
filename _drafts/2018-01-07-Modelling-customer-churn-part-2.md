---
title: Modelling customer churn - Part 2
published: true
tags:
 - R
 - churn
 - survival analyis
 - classification
---
The [first of this series of posts](https://annezj.github.io//Modelling-customer-churn-part-1/) introduced the concept of customer churn as time-to-event process, and carried out some preliminary data exploration, and survival analysis for the [Telco customer dataset from IBM](https://www.ibm.com/communities/analytics/watson-analytics-blog/predictive-insights-in-the-telco-customer-churn-data-set/). 

In this second post I'm going to move on to fitting some classification models to the dataset to see how well we can predict which customers are at risk of churning based on the information we have anout them. 

Assuming you're continuing from last time, you should have a cleaned data frame with 20 covariates in addition to the class variable "Churn":

```
head(df.tel[,1:11], n=3)
customerID gender SeniorCitizen Partner Dependents tenure PhoneService MultipleLines InternetService OnlineSecurity OnlineBackup
1 7590-VHVEG Female             0     Yes         No      1           No    No service             DSL             No          Yes
2 5575-GNVDE   Male             0      No         No     34          Yes            No             DSL            Yes           No
3 3668-QPYBK   Male             0      No         No      2          Yes            No             DSL            Yes          Yes

head(df.tel[,12:21], n=3)
DeviceProtection TechSupport StreamingTV StreamingMovies Contract PaperlessBilling     PaymentMethod MonthlyCharges TotalCharges Churn
1               No          No          No              No    Month              Yes Electronic\ncheck          29.85            1    No
2              Yes          No          No              No One year               No     Mailed\ncheck          56.95           34    No
3               No          No          No              No    Month              Yes     Mailed\ncheck          53.85            2   Yes

```
Last time, we saw that the first covariate is an ID number which we can discard. We also saw that "TotalCharges" was just a proxy for tenure, so that too can be discarded.

```
df.tel=df.tel[,c(2:19, 21)]
```
We also saw that contract had the largest effect on churn rate, and that the curves were discontinuous over the contract expiry
time. Therefore, it may be helpful to include contract expiry as an additional predictor. (Note that when fitting a predictive model you should ideally split your data and retain an independent test set before looking at the data and doing feature engineering).

```
df.tel$contract_expired=F
df.tel$contract_expired[which(df.tel$Contract=="Month" & df.tel$tenure>=1)]=T
df.tel$contract_expired[which(df.tel$Contract=="One year" & df.tel$tenure>=12)]=T
df.tel$contract_expired[which(df.tel$Contract=="Two year" & df.tel$tenure>=24)]=T
```
We should also bear in mind that the data are not balanced: 

```
table(df.tel$Churn)/nrow(df.tel)

       No       Yes 
0.7346301 0.2653699 
```
Only 27% of customers churned, so we may need to introduce some sampling at the training stage to avoid biased results. 

Next we load the caret library and split our data into training and test sets:

```
library(caret)
seed = 7
set.seed(seed)
train_index=createDataPartition(df.tel$Churn, p=0.8, list=F)
train_data=df.tel[train_index,c(1:18,20,19)]
test_data=df.tel[-train_index,c(2:18,20,19)]
```

Caret offers a huge range of models to choose from, many with sophisticated regularisation features. Here, I'll compare
three models: 1) Naive Bayes (simple model we started to explore via plots last time), 2) GLMnet, a linear method: logistic regression with lasso or elastic net regularisation, and 3) Random forest, a non-linear method with regularisation via tree depth.

Note that we mainly have categorical predictors, which Caret will automatically convert to dummy variables (a single binary variable for each category value) during fitting.

I prefer to use the ROC area as the performance metric so I set this up ready for fitting, also specifying repeated cross validation
with 10 folds and repeated 3 times in order to fit the reglularization parameters.

```
control = trainControl(method="repeatedcv", number=10, repeats=3,
                       summaryFunction=twoClassSummary, classProbs=TRUE)

metric = "ROC"
preProcess=c("center", "scale")

```
Fit the three methods to the training set. Note the random forest model is slow due to the cross-validation, so if you have problems
reduce the number of repeats or use a smaller training set.

```
# GLMNET - logistic regression with regularisation
set.seed(seed)
fit.glmnet <- train(Churn~., data=train_data, method="glmnet", metric=metric, preProc=preProc, trControl=control)
# Naive Bayes
set.seed(seed)
fit.nb <- train(Churn~., data=train_data, method="nb", metric=metric, preProc=preProc, trControl=control)
# Random Forest
set.seed(seed)
fit.rf <- train(Churn~., data=train_data, method="rf", metric=metric, preProc=preProc, trControl=control)

```
Compare performance on the test set using ROC area under the curve. To get full predictive power from the classifier it's important to get the probability from the predictions, and not just use the raw results which will predict a binary response based on probabilities above or below 0.5.

```
library(pROC)
compare=data.frame()
rocs=list()
i=1
for (name in c("glmnet", "nb", "rf")) 
{
  fit = get(paste0("fit.",name))
  predictions=predict(fit, test_data, type="prob")
  roc.obj=roc(test_data$Churn=="Yes", predictions$Yes)
  aucval=auc(roc.obj)[1]
  compare=rbind(compare, data.frame(model=name, auc=aucval))
  rocs[[i]]=roc.obj
  i=i+1
}
compare
ggroc(rocs)+scale_color_discrete("model")


model       auc
1 glmnet 0.8356457
2     nb 0.8095439
3     rf 0.8142382

```

So the regularised logistic regression model comes out top, and performs better than the simplest naive bayes method and the slow random forest model.






Next, we compare different methods to account for the class imbalance mentioned earlier. (Read ore about class imbalance [here](https://svds.com/learning-imbalanced-classes/)). Again, Caret has a number of options. The simplest two are undersampling and oversampling, both of which have disadvantages. Undersampling randomly undersamples the majority class (in this case those who did not churn), therefore discarding some data. Oversampling resamples the minority class (in this case those who churned), therefore assigning more weight to the minority class values. More sophisticated methods are also available, one of the most well known of which is [SMOTE](https://www.cs.cmu.edu/afs/cs/project/jair/pub/volume16/chawla02a-html/chawla2002.html) (Synthetic Minority Oversampling TEchnique), which combines majority class undersampling with minority class oversampling, and generates new minority samples by interpolating existing ones. We'll include it here, along with the two simpler methods.


