# Final Report: Home Purchase Assistant
## Shibo Zang (sz428), Yangwen Wan (yw762)

### Problem Overview
When someone is looking for a house to buy, he or she would be really interested in the following questions: If the current price of the house is fair and acceptable? Should I consider buying a house right now or wait for a while? Our goal is to provide suggestions to the users that if the housing prices listed by an agent or on a website is reasonable, and would the price go higher or lower in the future.

According to statistics, there were 5,250,000 existing homes and 510,000 newly constructed homes were sold in 2015, which means there were around 11 transactions completed on home trading per minute. In addition, large real estate marketplace like Zillow has more than 10,000,000 listings nationwide. We can see that the real estate market is very active and is critical to people's daily life. However, there is a lack of a powerful assistant tool that is able to help users to decide when to buy a house and which houses are worthy of consideration. By analyzing the data of past transactions and current local listings, we can provide answers to the questions listed above.

### Dataset Description
Our data sets consists of annual property sales data from 2003 to 2009 in the 5 boroughs of New York City. Except the sale price which is the value we seek to predict, we have 19 features for each of the properties/transactions. The number of transactions in a certain borough and a certain year vary greatly. As an example, the following table demonstrates the summary statistics of the data of all boroughs in 2009. Also, it is noteworthy that for a large portion of the transactions, the sale price and the number of squared feet of the property are 0. We consider these records as incomplete and exclude them in calculating the summary statistics as well as in further charts  

<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/Summary%20Statistics%20for%202009%20Transactions.png">

Generally the price distribution in all 5 boroughs are skewed to the left with a long tail on the right except the fact that there are dramatic numbers of prices that are close to zero even though we already exclude the zero prices  

<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/Properties%20in%20Manhattan.png">
<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/Properties%20in%20Brooklyn.png">

Furthermore, the average price in all boroughs underwent a decline since 2008 due to the impact of the financial crisis which means constructing the autoregressive (AR) model to predict price trend becomes even trickier.  

<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/ar.png">

### Techniques & Methods

#### Data Cleaning & Preprocessing
The first thing we noticed about the dataset was a large part of missing data. As you can see from the above dataset description, we have around 33.8% sales records that are incomplete (incomplete records are those who has value 0 in at least one field).  

In addition, we found that some of the entries are questionable even in those where there's no missing data. For example, some entries with really low or high property sale price is clearly wrong and cannot be used. To define a reasonable threshold, we did some research online. From https://www.trulia.com/real_estate/New_York-New_York/market-trends/, it is reasonable to say that the real estate property unit price should be at least \$10 per square feet. Thus, we only consider the properties whose price is above \$10 per square feet. There's several other cases where the data is wrong, such as entries with gross square feet less than 10, or the residential unit and commercial unit doesn't add up to the corresponding total unit.

<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/data_distribution.png">

We thought about using other methods that are taught in class to deal with missing data such as filling in missing entries with row or column mean, or predicting the missing entries using other rows or columns. The reasons we believed that just dropping missing entries work the best in our case is that: 1). We have ~ 35% missing entries and most missing entries are blank entries. If we just filled them with mean or a certain predicted value, the entries we filled in may largely affect our model. 2). We have ~ 75,000 records in total for a year. After data preprocessing & cleaning, the clean entries left are ~ 25,000, which is adequate for our training process.

#### Feature Selection & Transformation

<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/ro.png">

<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/bo.png">

<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/go.png">

#### Models & Algorithms
##### Technique 1 Split data into training, validation, and testing set.
Compared to our initial idea in the midterm report to train individual model for each borough, currently we trained our model using all the data in New York City, and using the "BOROUGH" as a new feature in the model. The advantage enables us to have enough data to train, validate, and test our model. In the current setting, we split our data to training (70%), validation(20%), and testing (10%) set.

##### Technique 2 Least Absolute (l1) loss without regularizer
We applied l1 loss function without regularizer on our training set and compare our prediction to the actual housing price on the validation set.  
<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/ratios.png">  
From the above histogram we observed that most of our prediction can be considered as "good estimates", which means they fall in the range within $\pm$ 25% of the actual value.

##### Techinque 3 Quadratic Loss with zero regularizer (Least Squares Regression), l1 regularizer (Lasso Regression), and l2 regularizer (Ridge Regression)
**Least Square Regression**  
<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/lsr.png">  
**Lasso Regression**  
<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/lasso.png">  
**Ridge Regression**  
<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/ridge.png">  

The above histograms show the prediction results we got from three different models. We observed that most prediction results fall in the "good estimates" range. Lasso regression performs slightly worse than least square regression and ridge regression.

##### Technique 4 Huber Loss with zero regularizer, l1 regularizer, and l2 regularizer
**Zero regularizer**  
<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/huber_zero.png">  
**L1 regularizer**  
<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/huber_one.png">  
**L2 regularizer**  
<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/huber_two.png">  

As we can see from the above histograms, huber regression performed almost the same given three different regularizer, meaning huber loss can help to reduce the impact from outliers.

##### Technique 5 Apply proximal gradient method on the above models
To get a better estimation of the stepsize we should use in the gradient descent method, we applied proximal gradient linear searching provided by Professor Udell to achieve faster convergence. 

### Results Evaluation
We defined an "acceptable range" ($\pm$ 25%) to evaluate the models. That is to say, if the predicted price is within $\pm$ 25% of the actual selling price, we considered the predicted price is correct. We believed 25% is a reasonable range for us since large web-based real estate companies like Zillow and Redfin use 20% as the threshold to evaluate their predictions.
  
| Loss Function | Regularizer | Training Error | Validation Error | Accuracy |
| ------| ------ | ------ | ------ | ------ |
| L1 | Zero | 214304 | 213033 | 62.9% |
| Quadratic | Zero | 251257 | 255271 | 53.1% |
| Quadratic | L1 | 288793 | 292811 | 40.2% |
| Quadratic | Quadratic | 271446 | 269484 | 51.2% |
| Huber | Zero | 215756 | 207882 | 58.0% |
| Huber | L1 | 215757 | 207884 | 58.0% |
| Huber | Quadratic | 215758 | 207884 | 58.1% |

The experiment shows that L1 regression achieved the highest accuracy rate. And we also observed that both training error and validation error met our expectation given that we are predicting housing price.  

By applying l1 regression on the test set, the results are as follow:

| Loss Function | Regularizer | Testing Error | Accuracy |
| ------| ------ | ------ | ------ | ------ |
| L1 | Zero | 199168 | 61.6% |

### Conclusion & Future Work
In this project, we tried to predict the housing price in New York City given the past sales data. We did data cleaning, data preprocessing, feature selection and transformation, and we tried several regresssion algorithms and choose the one with the best performance on the validation set.  

In all, we think the main difficulty of our project is the messy data. Besides missing / wrong entries, most of the fields in the dataset are categorical features. In the future we can try to explore more methods on dealing with categorical features and try to reduce the dependency among features.