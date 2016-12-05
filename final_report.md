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
The first thing we noticed about the dataset was a large part of missing data. As you can see from the above dataset description, we have around 33.8% sales records that are incomplete (incomplete records are those who has value 0 in sale price or gross squared feet of the property or total units columns (see below for definitions).  

In addition, we found that some of the entries are questionable even in those where there's no missing data. For example, some entries with really low or high property sale price is clearly wrong and cannot be used. To define a reasonable threshold, we did some research online. From https://www.trulia.com/real_estate/New_York-New_York/market-trends/, it is reasonable to say that the real estate property unit price should be at least \$10 per square feet. Thus, we only consider the properties whose price is above \$10 per square feet. There's several other cases where the data is wrong, such as entries with gross square feet less than 10, or the residential unit and commercial unit doesn't add up to the corresponding total unit.

<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/data_distribution.png">

We thought about using other methods that are taught in class to deal with missing data such as filling in missing entries with row or column mean, or predicting the missing entries using other rows or columns. The reasons we believed that just dropping missing entries work the best in our case is that: 1). We have ~ 35% missing entries and most missing entries are blank entries. If we just filled them with mean or a certain predicted value, the entries we filled in may largely affect our model. 2). We have ~ 75,000 records in total for a year. After data preprocessing & cleaning, the clean entries left are ~ 30,000, which is adequate for our training process.

#### Feature Selection & Transformation

The dataset consists of many relevant attributes of the property sales. However many of the features are highly correlated or redundant in nature and we choose to drop out some of them. More importantly, we have performed some tranformations on these features to better represent them when constructing our input space matrix. The followings are the original features in the dataset we choose to use with their explanations and transformations we performed:

- Borough: This is one of the 5 boroughs in NYC (Manhattanm, Queens, Brooklyn, Bronx and Staten Island) in which the property is located. Since this is a categorical feature, we use add 5 columns and use dummy coding to represent its value.
- Tax Class at Time of Sale: Every property in the city is assigned to one of four tax classes based on the use of the property. In the dataset we actually have more similar features such as "Tax Class at Present" and other more trival classification of the tax class but we believe "tax class at the time of sale" is more relevant and simple. And again we use dummy coding to represent this.
- Building Class Category: This implies the expected type of usage of the properties. There are many categories are substantially different from residential homes in nature and their attributes and prices do not follow the same pattern such as "factories", "hospitals" and "vacant land". After a close examniation of these types, we choose to consider only 8 of them: "01  ONE FAMILY HOMES", "02  TWO FAMILY HOMES", , "03  THREE FAMILY HOMES", "04  TAX CLASS 1 CONDOS", "07  RENTALS - WALKUP APARTMENTS", "12  CONDOS - WALKUP APARTMENTS", "13  CONDOS - ELEVATOR APARTMENTS", "15  CONDOS - 2-10 UNIT RESIDENTIAL". We direcly exlude other categories from tarining or test set. That left us with about 26000 entries for 2009. Again we precompute each category's average price per sqft and use that to replace the norminal values.  
- Neighborhood Average Price: Each property must lies within one of 184 neigborhoods (e.g. Soho, Greenwhich Village) in NYC. While this value is categorical, we did not want to add 184 columns to  use dummy coding for simplicity. We expect properties in the same neigborhood are good subsitites for each other and their price per sqft should be highly correlated. So we precomputed each neigborhood's averge price per sqft and use that to replace the norminal neigborhood name.
- Total Units: Sum of residential units (number of residential units at the listed property) and commercial units (number of commercial units at the listed property). We also have columns for the number of residential and commerical units in the cleaned data set but for many records these 2 values are not reported but the total units are valid. So we decided to use the total units only.
- Gross Squared Feet: The total area of all the floors of a building. While we still have land sqft in the dataset, they are mostly not reported in Manhattan or belong to sales of special kinds of properties (like "vacant land") we don't want to predict
- Year Built: Year the structure on the property was built.

Besides these original features we have considered the sale price might also be linear with the products or powers of them. Especially after we have precomputed the averge per sqft price of all neigborhoods and all building categories, it is intuitive that we can multiply these averages by property sizes (gross sqft). we have found that that these products values have demonsatrated apparent linear patterns with the sale price and decided to use them in the input space.

<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/ro.png">

<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/bo.png">

We take this idea a litter further and precompte the average per sqft price for every possible pair of neighborhood and building category. We end up with a even stronger linear pattern between the product and actual sale price.

<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/go.png">

#### Models & Algorithms
**Technique 1: Split data into training, validation, and testing set.**  
  
Compared to our initial idea in the midterm report to train individual model for each borough, currently we trained our model using all the data in New York City, and using the "BOROUGH" as a new feature in the model. The advantage enables us to have enough data to train, validate, and test our model. In the current setting, we split our data to training (70%), validation(20%), and testing (10%) set.

**Technique 2: Least Absolute (l1) loss without regularizer** 
  
We applied l1 loss function without regularizer on our training set and compare our prediction to the actual housing price on the validation set.  
<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/ratios.png">  
From the above histogram we observed that most of our prediction can be considered as "good estimates", which means they fall in the range within $\pm$ 25% of the actual value.

**Techinque 3: Quadratic Loss with zero regularizer (Least Squares Regression), l1 regularizer (Lasso Regression), and l2 regularizer (Ridge Regression)**  
  
**Least Square Regression**  
<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/lsr.png">  
**Lasso Regression**  
<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/lasso.png">  
**Ridge Regression**  
<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/ridge.png">  

The above histograms show the prediction results we got from three different models. We observed that most prediction results fall in the "good estimates" range. Lasso regression performs slightly worse than least square regression and ridge regression.

**Technique 4: Huber Loss with zero regularizer, l1 regularizer, and l2 regularizer**  
  
**Zero regularizer**  
<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/huber_zero.png">  
**L1 regularizer**  
<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/huber_one.png">  
**L2 regularizer**  
<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/huber_two.png">  

As we can see from the above histograms, huber regression performed almost the same given three different regularizer, meaning huber loss can help to reduce the impact from outliers.

**Technique 5: Apply proximal gradient method on the above models**  
  
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
