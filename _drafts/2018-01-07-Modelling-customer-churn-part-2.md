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
We should also check for class imbalance, since it is likely that the numbers churned and not churned are unequal: 

```
table(df.tel$Churn)/nrow(df.tel)

       No       Yes 
0.7346301 0.2653699 
```
We see 27% of customers churned, so there is some class imbalance, but it is not extreme.(You can read more about techniques to correct class imbalance [here](https://svds.com/learning-imbalanced-classes/)). For imbalances at a rate of around 0.1 or lower we would need to consider techniques to address the imbalance when fitting models. For now, we'll use the data as it is.

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

I prefer to use the area under the [ROC curve](https://en.wikipedia.org/wiki/Receiver_operating_characteristic) as the performance metric so I set this up ready for fitting, also specifying repeated cross validation
with 10 folds and repeated 3 times in order to fit the reglularization parameters.

```
control = trainControl(method="repeatedcv", number=10, repeats=3,
                       summaryFunction=twoClassSummary, classProbs=TRUE)

metric = "ROC"
preProcess=c("center", "scale")

```
Fit the three methods to the training set. Note the random forest model is slow due to the cross-validation, so if you have problems reduce the number of repeats or use a smaller training set.

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
Compare performance on the test set using ROC area under the curve. To get full predictive power from the classifier it's important to get the probability from the predictions, and not just use the raw results which will predict a binary response based on probabilities above or below 0.5. The pROC library also gives an estimate of the 95% confidence interval
on the ROC using bootrap resampling.


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
  compare=rbind(compare, data.frame(model=name, auc=auc(roc.obj)[1], 
                                                auc.lci95=ci.auc(roc.obj)[1],
                                                auc.uci95=ci.auc(roc.obj)[3]))
  rocs[[i]]=roc.obj
  i=i+1
}
print(compare, digits=3)
model   auc auc.lci95 auc.uci95
1 glmnet 0.836     0.813     0.858
2     nb 0.810     0.784     0.835
3     rf 0.814     0.789     0.840


```
Using the pROC library, we can also plot the ROC curve for the three models, which shows 1-specificity, or the false positive rate (x), plotting against the sensitivity or hit rate (y). The black line shows
the performance expected if the forecast consisted of the same probability for all customers, which would give a ROC area of 0.5. A perfect forecast would have a false positive rate of zero (specificity of 1) and a sensitivity of 1: all the data points would be clustered at (1,1) on the plot, giving a ROC area of 1. Therefore the greater the area between the black line and the ROC curve for each model, the more skilful the prediction from that particular model.

```
ggroc(rocs)+scale_color_discrete("model", labels=modelnames)+theme_bw()+
  geom_segment(data=data.frame(x=1, y=0, xend=0, yend=1), 
               aes(x=x, y=y, xend=xend, yend=yend), color='black')
```

({{site.baseurl}}/assets/images/posts/churn-variables-categorical.png)

So the regularised logistic regression model comes out top, and performs better than the simplest (unregularised) naive bayes method but also better than the random forest model. We also see from the overlap of the confidence intervals and the ROC plot that there is not so much difference between the models, but glmnet does look a little better. The ROC area tells us the probability that, two randomly selected customers, one from each category, the probability that the classification model will assign the greater probability of churn to the one that did in fact churn. So, for our best model this ability to distingiush churned customers is around 0.84, and this is significantly better than a non-informative forecast (AUC of 0.5) at 95% confidence.

Let's examine the best model fit a little more.

```
coeff=predict(fit.glmnet$finalModel, type = "coefficients", 
                            s=fit.glmnet$bestTune$lambda)
coeff=data.frame(name=coeff@Dimnames[1], value=coeff@x)
colnames(coeff)=c('name','value')
coeff[order(abs(coeff$value), decreasing = T),]
varImp(fit.glmnet)
```


To improve the model fit further, we could go back 


