# Project 02: Customer Churn Analysis
> This study analyzes the customer data to identify why customers are leaving and the potential indicators that are causing them to leave. The data is derived from a telecommunications company through Kaggle and will be using three different predictive modeling: (1) binary logistic regression (2) decision trees and (3) random forest.

Table of Contents
---
1. [General Information](#general-information)
2. [Summary](#summary)
3. [Tech Stack](#tech-stack)
4. [Data Preprocessing/Cleaning](#data-preprocessingcleaning)
5. [Data Visualization](#data-visualization)
6. [Data Analysis](#data-analysis)
7. [Modeling](#modeling)
8. [Solution](#solution)
9. [Key Takeaways](#key-takeaways)
10. [References](#references)

<a name="https://github.com/sangtvo/Seattle-PD-Funding-Eligibility#general-information"/>
<a name="https://github.com/sangtvo/Seattle-PD-Funding-Eligibility#summary"/>
<a name="https://github.com/sangtvo/Seattle-PD-Funding-Eligibility#tech-stack"/>
<a name="https://github.com/sangtvo/Seattle-PD-Funding-Eligibility#data-preprocessingcleaning"/>
<a name="https://github.com/sangtvo/Seattle-PD-Funding-Eligibility#data-visualization"/>
<a name="https://github.com/sangtvo/Seattle-PD-Funding-Eligibility#data-analysis"/>
<a name="https://github.com/sangtvo/Seattle-PD-Funding-Eligibility#modeling"/>
<a name="https://github.com/sangtvo/Seattle-PD-Funding-Eligibility#solution"/>
<a name="https://github.com/sangtvo/Seattle-PD-Funding-Eligibility#key-takeaways"/>
<a name="https://github.com/sangtvo/Seattle-PD-Funding-Eligibility#references"/>

General Information
---
The project is part of a graduate course (_Data Mining and Analytics II_) at Western Governor's University. The raw data set contains 7,043 observations and 21 features. To expand the project even further (originally binary logistic regression), decision trees and random forest analysis are added.


Summary
---
The best fitted linear regression model is y = 1.491x + 21.914 which means that for every additional incident reported (x), you can expect officers on site to increase by an average of 1.491. The overall mean of the data is 1.889 which is below the 2.5 threshold and is not qualified for additional funding. In order to increase their eligibility for funding, the Seattle PD must focus on zones with average officers at scene that is less than or equal to 2 and the largest reported incident zones. There are 12 zones that need assistance and can lead to funding eligibility.

Tech Stack
---
* R Studio
    * plyr
    * tidyverse
    * ggplot2
    * ggcorrplot
    * randomForest
    * caret
    * cowplot
    * RColorBrewer
    * pROC
    * rpart
    * rpart.plot

Data Preprocessing/Cleaning
---
#### Irrelevant:
Removed customerID variable since it is not necessary for the purpose of this analysis.
```r
cdf$customerID <- NULL
```

#### Recoding: 
Recode some of the categorical variables for simplicity.
```r
cdf$SeniorCitizen <- as.factor(mapvalues(cdf$SeniorCitizen, from=c("0","1"), to=c("No", "Yes")))
cdf$MultipleLines <- as.factor(mapvalues(cdf$MultipleLines, from=c("No phone service"), to=c("No")))

for (i in 9:14){
  cdf[,i] <- as.factor(mapvalues(cdf[,i], from=c("No internet service"), to=c("No")))
}
```

Recode the dependent variable as a factor in the clean data frame instead of characters.
```r
cdf[, 'Churn'] <- as.factor(cdf[, 'Churn'])
```

#### Missing Data:
Checking for missing data.
```r
sapply(df, function(x) sum(is.na(x)))
```
```
      customerID           gender    SeniorCitizen          Partner       Dependents           tenure     PhoneService    MultipleLines  InternetService 
               0                0                0                0                0                0                0                0                0 
  OnlineSecurity     OnlineBackup DeviceProtection      TechSupport      StreamingTV  StreamingMovies         Contract PaperlessBilling    PaymentMethod 
               0                0                0                0                0                0                0                0                0 
  MonthlyCharges     TotalCharges            Churn 
               0               11                0 
```

Calculating the percentage of missing values on TotalCharges variable.
```r
sum(is.na(df$TotalCharges))/nrow(df)
```
```
[1] 0.001561834
```

Since the data has 0.16% missing data in the TotalCharges variable, a new data frame is created to remove the missing values.
```r
cdf <- na.omit(df)
```

### The final clean data frame: 
```r
summary(cdf)
```
```
    gender          SeniorCitizen   Partner           Dependents            tenure      PhoneService       MultipleLines InternetService   
 Length:7032        No :5890      Length:7032        Length:7032        Min.   : 1.00   Length:7032        No :4065      Length:7032       
 Class :character   Yes:1142      Class :character   Class :character   1st Qu.: 9.00   Class :character   Yes:2967      Class :character  
 Mode  :character                 Mode  :character   Mode  :character   Median :29.00   Mode  :character                 Mode  :character  
                                                                        Mean   :32.42                                                      
                                                                        3rd Qu.:55.00                                                      
                                                                        Max.   :72.00                                                      
 OnlineSecurity OnlineBackup DeviceProtection TechSupport StreamingTV StreamingMovies   Contract         PaperlessBilling   PaymentMethod     
 No :5017       No :4607     No :4614         No :4992    No :4329    No :4301        Length:7032        Length:7032        Length:7032       
 Yes:2015       Yes:2425     Yes:2418         Yes:2040    Yes:2703    Yes:2731        Class :character   Class :character   Class :character  
                                                                                      Mode  :character   Mode  :character   Mode  :character  
                                                                                                                                              
                                                                                                                                              
                                                                                                                                              
 MonthlyCharges    TotalCharges    Churn     
 Min.   : 18.25   Min.   :  18.8   No :5163  
 1st Qu.: 35.59   1st Qu.: 401.4   Yes:1869  
 Median : 70.35   Median :1397.5             
 Mean   : 64.80   Mean   :2283.3             
 3rd Qu.: 89.86   3rd Qu.:3794.7             
 Max.   :118.75   Max.   :8684.8             
```

For the full notebook, please check out Customer Churn Analysis.Rmd

Exploratory Data Analysis
---

<table>
  <tr><th colspan=2>Univariate Analysis</th></tr>
  <tr><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/bar_1.png?raw=true"> </td><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/bar_2.png?raw=true"> </td></tr>
  <tr><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/bar_3.png?raw=true"> </td><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/bar_4.png?raw=true"> </td></tr>
</table>

* ***Gender*** and ***Partner*** are almost evenly split 50/50 and that 83.76% of the data are not ***SeniorCitizens***. 
* 90.33% of the data have single ***PhoneService*** lines, but in terms of extra services, there are roughly 70% of the customers that do not have ***OnlineSecurity*** and ***TechSupport***. 
* While ***InternetService*** is a pretty common add-on with phone lines, **fiber optic** is a favorable internet service which accounts for 44.03% of the customers. 
* 55.11% of the customers are on a **month-to-month** ***Contract*** and 33.63% of the customers pay their bill with an **electronic check**.
* The ***Churn*** rate of the data is 26.58%.


<table>
  <tr><th colspan=2>Bivariate Analysis</th></tr>
  <tr><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/bivar_1.png?raw=true"> </td><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/bivar_2.png?raw=true"> </td></tr>
  <tr><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/bivar_3.png?raw=true"> </td><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/bivar_4.png?raw=true"> </td></tr>
  <tr><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/bivar_5.png?raw=true"> </td>
</table>

* ***Gender*** percentages are almost similar and therefore, may not have influence on churn. 
* ***OnlineSecurity*** and ***TechSupport*** have very similar percentages and perhaps they are somewhat correlated whether a customer is churning or not.
* ***OnlineBackup*** and ***DeviceProtection*** are also similar indicating that they might be correlated with one another.
* Customers with ***PhoneService*** are more likely to churn those who don't have service.
* **Month-to-month** ***Contract*** customers are much higher than longer contracts, which makes sense because there is no obligation to stay longer if it was a month-to-month basis and can leave at any time. 
* Customers with ***InternetService*** are more likely to churn than those that don't have internet.
  * **Fiber optic** churn rate is 3x higher than **DSL** and 9x without internet service.
* Churn percentage is higher for customers who utilizes ***PaperlessBilling*** option.
* Customers who use **electronic checks** are almost 5x as high to churn compared to other ***PaymentMethod*** options.

<table>
  <tr><th colspan=2>Distribution of Continuous Independent Variables</th></tr>
  <tr><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/distribution_MonthlyCharges.png?raw=true"> </td><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/distribution_TotalCharges.png?raw=true"> </td></tr>
  <tr><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/distribution_tenure.png?raw=true"> </td><td>
</table>

* The number of customers who have ***MonthlyCharges*** of $25 or less are extremely high--it seems that $0-25 is a very common monthly charge.  
  * The distributions starting around $30 per month is similar between customers who churned or not.
* ***TotalCharges*** has a positive/right skew meaning that the right side of the distribution is longer or flatter. 
  * Mean and median is greater than the mode.
  * The frequency count at $2,500 total charges starts to decrease slowly in count and not as rapid. 
* ***Tenure*** distributions are different between customers who churned or not. 
  * For customers who churned, the distribution is positive/right skew which may indicate that the first 10 weeks or so, customers are more likely to cancel service.
  * For customers who did not churn, the highest peak is 70 weeks (almost 6 years) which indicates that there is a large amount of customers who kept their service for at least 5 years. 

![Correlation](https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/correlation.png?raw=true)

* ***TotalCharges*** and ***tenure*** have a positive correlation which means that the correlation coefficient is greater than 0, but not a perfect correlation of 1.0 as colored in the correlation matrix above.
  * Correlation coefficients are indicators of the strength of the linear relationship between two variables. 
  * While there is a positive correlation, it does not mean that one causes the other. 

<table>
  <tr><th colspan=2>Boxplots of Continuous Independent Variables</th></tr>
  <tr><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/bp_MonthlyCharges.png?raw=true"> </td><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/bp_TotalCharges.png?raw=true"> </td></tr>
  <tr><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/bp_tenure.png?raw=true"> </td><td>
</table>

* There are no obvious outliers in the continuous variables where there are values beyond the whiskers of the boxplots. 

Logistic Regression
---
The data frame is split 70% training and 30% testing data.
```r
set.seed(123)
train_test_data <- createDataPartition(cdf$Churn,p=0.7,list=FALSE)
train_d <- cdf[train_test_data,]
test_d <- cdf[-train_test_data,]
```

Run the base logistic regression model with all variables.
```r
lr_model <- glm(Churn ~., data=train_d, family=binomial(link='logit'))
summary(lr_model)
```
```
Call:
glm(formula = Churn ~ ., family = binomial(link = "logit"), data = train_d)

Deviance Residuals: 
    Min       1Q   Median       3Q      Max  
-1.8637  -0.6953  -0.2890   0.7553   3.2200  

Coefficients:
                                       Estimate Std. Error z value Pr(>|z|)    
(Intercept)                           1.0746695  0.9571525   1.123 0.261532    
genderMale                           -0.0482269  0.0771304  -0.625 0.531797    
SeniorCitizenYes                      0.1200467  0.1014491   1.183 0.236683    
PartnerYes                           -0.0910761  0.0918164  -0.992 0.321228    
DependentsYes                        -0.0843260  0.1068161  -0.789 0.429849    
tenure                               -0.0621135  0.0074487  -8.339  < 2e-16 ***
PhoneServiceYes                       0.0233358  0.7635434   0.031 0.975618    
MultipleLinesYes                      0.3794100  0.2078464   1.825 0.067935 .  
InternetServiceFiber optic            1.5613737  0.9372764   1.666 0.095741 .  
InternetServiceNo                    -1.5047493  0.9480893  -1.587 0.112481    
OnlineSecurityYes                    -0.2149292  0.2098780  -1.024 0.305804    
OnlineBackupYes                       0.0008494  0.2053088   0.004 0.996699    
DeviceProtectionYes                   0.1177891  0.2084585   0.565 0.572041    
TechSupportYes                       -0.1280168  0.2128673  -0.601 0.547579    
StreamingTVYes                        0.5187839  0.3819512   1.358 0.174386    
StreamingMoviesYes                    0.5783808  0.3840289   1.506 0.132045    
ContractOne year                     -0.8451411  0.1304035  -6.481 9.11e-11 ***
ContractTwo year                     -1.4644194  0.2039949  -7.179 7.04e-13 ***
PaperlessBillingYes                   0.3020753  0.0884590   3.415 0.000638 ***
PaymentMethodCredit card (automatic) -0.0093796  0.1357499  -0.069 0.944914    
PaymentMethodElectronic check         0.2897469  0.1124649   2.576 0.009985 ** 
PaymentMethodMailed check            -0.1002695  0.1367827  -0.733 0.463524    
MonthlyCharges                       -0.0336622  0.0372618  -0.903 0.366314    
TotalCharges                          0.0003653  0.0000844   4.328 1.50e-05 ***
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 5702.8  on 4923  degrees of freedom
Residual deviance: 4117.1  on 4900  degrees of freedom
AIC: 4165.1

Number of Fisher Scoring iterations: 6
```
* Variables that are significant are: tenure, contract, payment method electronic check, and total charges.

Akaike Information Criterion (AIC) evaluates how well a model fits the data without over-fitting. The AIC score rewards models that have a high goodness-of-fit score, but penalizes when models become more complex. AIC score is useful when comparing AIC scores of competing models and the lower the AIC score, the better the fit (more parsimonious model). 

Compute the confusion matrix table.
```r
lr_prob1 <- predict(lr_model, test_d, type="response")
lr_pred1 <- ifelse(lr_prob1 > 0.5,"Yes","No")

table(Predicted=lr_pred1, Actual=test_d$Churn)
```
```
         Actual
Predicted   No  Yes
      No  1385  229
      Yes  163  331
```
```
         Actual
Predicted   No  Yes
      No    TN  FN
      Yes   FP  TP
```
```
Specificity = TN/(TN + FP)
Sensitivity = TP / (FN + TP)
Negative Predicted Value (-PV) = TN/(TN+FN)
Positive Predicted Value (+PV or precision) = TP/(FP+TP)
```
* Specificity = 1385/(1385+163) = 0.8947
* Sensitivity = 331/(229+331) = 0.5910
* -PV = 1385/(1385+229) = 0.8581
* +PV = 331/(163+331) = 0.6700

Validation measures are specificity and sensitivity. Sensitivity is the true positive rate where the proportion of test in question is correctly classified. Specificity is the true negative rate where the proportion of negative test in question is correctly classified. Sensitivity and specificity are inversely proportional, meaning that as sensitivity increases, the specificity decreases and vice versa. 

Positive predicted value, or precision, tells how often a positive test represents a true positive. Negative predicted value tells how often a negeative test represents a true negative. 

Calculate the accuracy.
```r
lr_prob2 <- predict(lr_model, train_d, type="response")
lr_pred2 <- ifelse(lr_prob2 > 0.5,"Yes","No")
lr_tab1 <- table(Predicted = lr_pred2, Actual = train_d$Churn)
lr_tab2 <- table(Predicted = lr_pred1, Actual = test_d$Churn)
lr_accuracy <- sum(diag(lr_tab2))/sum(lr_tab2)
lr_accuracy
```
```
[1] 0.8140417
```

Calculcate the Receiver Operating Character (ROC) Curve.
```r
lr_roc <- roc(train_d$Churn, lr_prob2)
plot(lr_roc, col='Blue', main="ROC Curve")
auc(lr_roc)
```
```
Area under the curve: 0.8446
```
![BaseLR](https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/base_lr_roc.png?raw=true)

ROC is a probability curve that shows the performance of a classification model at all classification thresholds. The x-axis is the false positive rate and the y-axis is the true positive rate. The AUC represents the measure of separability that shows how much the model can distinguish between classes. The higher the AUC, the better at predicting 0s and 1s. AUC ranges in value from 0 to 1 and a model  with predictions are 100% wrong has an AUC of 0 and predictions that are 100% correct has an AUC of 1.

The accuracy of the base regression model is 81.40% with AUC of 84.46%. The model has a 84.46% chance that it will distinguish between a positive and negative class. 

However, we can improve the base model by using the stepAIC for variable selection. This method will add or remove variables and comes up with a final set of variables. This method simplifies the model and gives the most parsimonious model. 

```r
lrstep_model <- MASS::stepAIC(lr_model, trace=0)
summary(lrstep_model)
```
```
Call:
glm(formula = Churn ~ SeniorCitizen + Partner + tenure + MultipleLines + 
    InternetService + OnlineSecurity + StreamingTV + StreamingMovies + 
    Contract + PaperlessBilling + PaymentMethod + MonthlyCharges + 
    TotalCharges, family = binomial(link = "logit"), data = train_d)

Deviance Residuals: 
    Min       1Q   Median       3Q      Max  
-1.8885  -0.6953  -0.2900   0.7595   3.2334  

Coefficients:
                                       Estimate Std. Error z value Pr(>|z|)    
(Intercept)                           1.027e+00  3.249e-01   3.161 0.001571 ** 
SeniorCitizenYes                      1.436e-01  9.914e-02   1.449 0.147360    
PartnerYes                           -1.189e-01  8.345e-02  -1.425 0.154237    
tenure                               -6.222e-02  7.440e-03  -8.364  < 2e-16 ***
MultipleLinesYes                      3.800e-01  1.022e-01   3.719 0.000200 ***
InternetServiceFiber optic            1.571e+00  2.119e-01   7.411 1.25e-13 ***
InternetServiceNo                    -1.480e+00  2.095e-01  -7.064 1.62e-12 ***
OnlineSecurityYes                    -2.223e-01  1.059e-01  -2.098 0.035877 *  
StreamingTVYes                        5.151e-01  1.145e-01   4.500 6.81e-06 ***
StreamingMoviesYes                    5.792e-01  1.142e-01   5.071 3.97e-07 ***
ContractOne year                     -8.506e-01  1.294e-01  -6.572 4.95e-11 ***
ContractTwo year                     -1.495e+00  2.011e-01  -7.435 1.05e-13 ***
PaperlessBillingYes                   3.041e-01  8.822e-02   3.447 0.000567 ***
PaymentMethodCredit card (automatic) -8.973e-03  1.355e-01  -0.066 0.947215    
PaymentMethodElectronic check         2.931e-01  1.123e-01   2.611 0.009029 ** 
PaymentMethodMailed check            -1.090e-01  1.365e-01  -0.798 0.424728    
MonthlyCharges                       -3.322e-02  6.473e-03  -5.132 2.87e-07 ***
TotalCharges                          3.694e-04  8.407e-05   4.394 1.11e-05 ***
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 5702.8  on 4923  degrees of freedom
Residual deviance: 4121.1  on 4906  degrees of freedom
AIC: 4157.1

Number of Fisher Scoring iterations: 6
```

Checking variance inflation factor (VIF) for **multicollinearity**. Multicollinearity is when an independent variable is highly correlated with one or more of the other independent variables. This is an issue because it undermines the significance of the independent variable. 
```r
rms::vif(lrstep_model)
```
```
                    SeniorCitizenYes                           PartnerYes                               tenure                     MultipleLinesYes 
                            1.101765                             1.126056                            16.066630                             1.740315 
          InternetServiceFiber optic                    InternetServiceNo                    OnlineSecurityYes                       StreamingTVYes 
                            7.288274                             2.719026                             1.282502                             2.163958 
                  StreamingMoviesYes                     ContractOne year                     ContractTwo year                  PaperlessBillingYes 
                            2.151208                             1.289170                             1.317572                             1.141450 
PaymentMethodCredit card (automatic)        PaymentMethodElectronic check            PaymentMethodMailed check                       MonthlyCharges 
                            1.623259                             2.127947                             2.067715                            20.886103 
                        TotalCharges 
                           20.728399 
```

Remove VIF > 10 and re-run the step-wise logistic regression model. In addition, I manually remove the least significant variables with the final output (see R markdown for full code):
```
Call:
glm(formula = Churn ~ tenure + InternetService + OnlineSecurity + 
    StreamingTV + StreamingMovies + Contract + PaperlessBilling, 
    family = binomial(link = "logit"), data = train_d)

Deviance Residuals: 
    Min       1Q   Median       3Q      Max  
-1.7869  -0.6783  -0.3040   0.7767   3.1201  

Coefficients:
                           Estimate Std. Error z value Pr(>|z|)    
(Intercept)                -0.52568    0.09620  -5.464 4.65e-08 ***
tenure                     -0.03262    0.00244 -13.367  < 2e-16 ***
InternetServiceFiber optic  0.92930    0.09092  10.221  < 2e-16 ***
InternetServiceNo          -0.83840    0.14600  -5.742 9.34e-09 ***
OnlineSecurityYes          -0.40620    0.09849  -4.125 3.71e-05 ***
StreamingTVYes              0.27410    0.09214   2.975  0.00293 ** 
StreamingMoviesYes          0.37260    0.09227   4.038 5.39e-05 ***
ContractOne year           -0.97795    0.12628  -7.744 9.63e-15 ***
ContractTwo year           -1.64444    0.19597  -8.391  < 2e-16 ***
PaperlessBillingYes         0.35244    0.08681   4.060 4.91e-05 ***
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 5702.8  on 4923  degrees of freedom
Residual deviance: 4187.3  on 4914  degrees of freedom
AIC: 4207.3

Number of Fisher Scoring iterations: 6
```
* We can see that the most significant variables are tenure, internet service, online security, streaming tv and movies, contract, and paperless billing. 

Confusion matrix for the step-wise logistic regression model:
```
        Actual
Predicted   No  Yes
      No  1386  256
      Yes  162  304
```
* Specificity = 1386/(1386+162) = 0.8953
* Sensitivity = 304/(256+304) = 0.5428
* -PV = 1386/(1386+256) = 0.8440
* +PV = 304/(162+304) = 0.6423

The accuracy of the model is 80.17% and AUC of 83.77%. 

**Comparison Between Base Logistic Regression & Step-Wise Logistic Regression**
Logistic Regression (Base) ROC Curve             |  Step-wise Logistic Regression ROC Curve
:-------------------------:|:-------------------------:
![BaseLR](https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/base_lr_roc.png?raw=true)  |  ![StepLR](https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/step_roc.png?raw=true)
AUC = 84.46% | AUC = 83.77%
Accuracy = 81.40% | Accuracy = 80.17%

Between the two models, the best parsimonious model would be the step-wise model despite a lower accuracy than the base model. This model is preferrable since it only includes the least variables that are highly significant with a lower AIC score. 

Decision Tree
---
![Dtree](https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/dtree.png?raw=true)

![DtreeROC](https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/dtree_roc.png?raw=true)


Random Forest
---
![Top10](https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/Top10_Variables.png?raw=true)

![rfROC](https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/rf_roc.png?raw=true)


A linear regression model is created by using Microsoft Excel Data Analysis Toolpak. The model provides a population regression parameter where the slope is β<sub>1</sub> = 1.491 and a y-intercept of β<sub>0</sub> = 21.914. The linear regression model displays an upward trend and indicates that the number of incidents in a specific district are closely related to the numbers of officers responding to the scene. However, there are two outliers that is skewing the regression line, (1,1) and (125,165) which may show that it is not necessarily the best fit.

By calculating R<sup>2</sup> for the model with and without outliers, we can see how much variability of the response data around its mean. Ideally, the higher the R<sup>2</sup>, closer to 1, the better the model fits the data. However, there are some limitations with R<sup>2</sup> and sometimes does not indicate whether a regression model is adequate. Therefore, we must also consider sum of absolute error (SAE) calculation as it shows how far the regression line is from the actual data points. The lower the SAE, the better the fit.

Linear Regression             |  Residual Plot
:-------------------------:|:-------------------------:
![LR](https://github.com/sangtvo/Seattle-PD-Funding-Eligibility/blob/main/images/LR_outlier.gif?raw=true)  |  ![Residual](https://github.com/sangtvo/Seattle-PD-Funding-Eligibility/blob/main/images/Residual_outlier.gif?raw=true)

![ANOVA](https://github.com/sangtvo/Seattle-PD-Funding-Eligibility/blob/main/images/ANOVA.GIF?raw=true)

x             |  E(y) |  y |  E(y) - y = e
:-------------------------:|:-------------------------:|:-------------------------:|:-------------------------:
83	| 145.667	| 158	| -12.333
125	| 208.289| 	165	| 43.289
37	| 77.081| 	86| 	-8.919
64| 	117.338| 	131| 	-13.662
60	| 111.374| 	121| 	-9.626
31| 	68.135| 	72	| -3.865
52	| 99.446| 	96	| 3.446
60| 	111.374	| 124| 	-12.626
44| 	87.518| 	82| 	5.518
1	| 23.405	| 1	| 22.405
41	| 83.045| 	77| 	6.045
62	| 114.356| 	120| 	-5.644
38	| 78.572	| 72| 	6.572
44	| 87.518	| 76| 	11.518
91	| 157.595	| 176| 	-18.405
53	| 100.937| 	117| 	-16.063
35	| 74.099| 	68| 	6.099
39	| 80.063| 	76| 	4.063
86	| 150.14	| 158	| -7.86
| | | SAE= | 0.048

The linear regression model with outliers demonstrate a R<sup>2</sup> = 0.8795 and SAE = 0.048 while the linear regression model without outliers have a R<sup>2</sup> = 0.9591 and SAE = 68.007 (not shown). Even though the linear regression model without outliers has a higher R<sup>2</sup>, the SAE is not minimized.

The residual plot above shows that the two outliers seem to be pulling data points away from the trend line (y=0). However, when removing the outliers in the residual plot (not shown) becomes much more randomly dispersed and the “mean of zero” assumption almost holds true, meaning the mean is approximately zero all the way across the plot. Despite this and the limited observations, the best fit for linear regression model is with outliers. 

When looking at the ANOVA table, the model demonstrates that the F value is less than 0.05 and therefore statistically significant.

<table>
  <tr><th colspan=2>Descriptive Statistics</th></tr>
  <tr><td>Mean</td><td>1.889952153</td></tr>
  <tr><td>Standard Error</td><td>0.036811154</td></tr>
  <tr><td>Median</td><td>2</td></tr>
  <tr><td>Mode</td><td>1</td></tr>
  <tr><td>Standard Deviation</td><td>1.189974282</td></tr>
  <tr><td>Sample Variance</td><td>1.416038791</td></tr>
  <tr><td>Kurtosis</td><td>-0.93853082</td></tr>
  <tr><td>Skewness</td><td>0.151756917</td></tr>
  <tr><td>Range</td><td>4</td></tr>
  <tr><td>Minimum</td><td>0</td></tr>
  <tr><td>Maximum</td><td>4</td></tr>
  <tr><td>Sum</td><td>1975</td></tr>
  <tr><td>Count</td><td>1,045</td></tr>
</table>

The current threshold to receive additional funding for the Seattle Police Department is 2.5, however, they do not meet the expected threshold. When calculating the mean of the clean data, the mean is 1.889 which is below the 2.5 threshold (1.904 on graph due to one zone not removed yet). This means that the department averages 1.889 officers at the scene per incident and is ineligible for additional funding. However, there is a limitation of this study. Since the study is only for a span of 3 days, it is best to collect data for the whole month or quarter as a better estimate. 

Solution
---
Since the Seattle PD is not eligible for funding due to a mean of 1.889, the department should focus on zones with less average officer per site and zones with the largest reported incidents. By targeting zones with average officers that have a mean of 2 or less and reach a mean score of 2+, it is possible it will bring the overall mean to 2.5. If it is not reached still, then targeting the highest reported incidents zone can also bring it up by mandating 3 officers per incident. If the police department is understaffed, that would mean that officers are working over-time and arriving at multiple scenes to keep up with the incoming calls. This inefficiency will continue to drive down (or similar amount) the mean of officers at scene and the department will never reach that 2.5 threshold to receive any funding in future years. If the police department hired a human resources analyst, this person can detect how many officers are needed and analyze their work schedules to be more effective in order to meet demand.

Key Takeaways
---
* The number of incidents on Sunday (March 27, 2016) is 2x as high compared to Friday and Saturday that same weekend due to Easter Sunday.
* The top 3 incidents are disturbances, traffic related calls, and suspicious circumstances which are 3x more common than other incidents.
* At least 2 officers arrive onsite and show up more often in the outskirts of downtown Seattle. 
* The W zone has the lowest incidents reported due to Burley and Bethey districts having less population and more deserted, but the highest mean of officers onsite (2.324).
* The H zone which is downtown Seattle has the highest reported incidents, but the lowest mean of officers onsite of 1.32. 

References
---
Balk, G. (2019, April 4). Is Seattle 'dying'? Crime rates tell a different story. Retrieved June 12, 2020, from http://www.seattletimes.com/seattle-news/data/is-seattle-dying-not-if-you-look-crime-rates-from-the-80s-and-90s/
