# Midterm Report: Home Purchase Assistant
## Shibo Zang (sz428), Yangwen Wan (yw762)

### Dataset Description
Our data sets consists of annual property sales data from 2003 to 2009 in the 5 boroughs of New York City. Except the sale price which is the value we seek to predict, we have 19 features for each of the properties/transactions. The number of transactions in a certain borough and a certain year vary greatly. As an example, the following table demonstrates the summary statistics of the data of all boroughs in 2009. Also, it is noteworthy that for a large portion of the transactions, the sale price and the number of squared feet of the property are 0. We consider these records as incomplete and exclude them in calculating the summary statistics as well as in further charts  

<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/Summary%20Statistics%20for%202009%20Transactions.png">

Generally the price distribution in all 5 boroughs are skewed to the left with a long tail on the right except the fact that there are dramatic numbers of prices that are close to zero even though we already exclude the zero prices  

<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/Properties%20in%20Manhattan.png">
<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/Properties%20in%20Brooklyn.png">

Furthermore, the average price in all boroughs underwent a decline since 2008 due to the impact of the financial crisis which means constructing the autoregressive (AR) model to predict price trend becomes even trickier.  

<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/ar.png">

### Feature Selection & Engineering
We would like to select those features that have close relationship with the price of real estate properties and hopefully can be independent with each other. There are several factors that we felt would have an impact on the price: location / neighborhood, building type (apartment / house / condo), year built, building class (one-bedroom / two-bedroom / studio), etc.. Basing on the dataset we have on hand, we selected or generated the following features for our model:

- Tax Class at Present: Every property in the city is assigned to one of four tax classes based on the use of the property.
Total Units: Sum of residential units (number of residential units at the listed property) and commercial units (number of commercial units at the listed property).
- Land Square Feet: The land area of the property listed in square feet.
- Gross Square Feet: The total area of all the floors of a building. The reason we would like to keep both Land Square Feet and Gross Square Feet as features is that we figured they are not totally positive correlated - Gross Square Feet contains some information that Land Square Feet doesn’t possess, like number of floors, other land area and space within any building or structure on the property.
- Year Built: Year the structure on the property was built.
- Neighborhood Average Price: Each real estate property unit is marked uniquely by a Borough-Block-Lot identifier. This gave us a natural advantage to group close real properties together by “Block” number, whereas using street address, which is hard to analyze and organize, and zipcode is too broad to define a neighborhood. Therefore, the neighborhood average price would be a strong indicator signifying the price of the property you are looking at.  

As we discussed above, you might find that some features are numerical, and some features are categorical like ‘Tax Class At Present’. Since most regression models or ML algorithms cannot fit categorical variables, we are going to use dummy coding to convert a categorical feature into continuous variable. For example, there are four tax classes, we can have four features ‘Tax Class A’, ‘Tax Class B’, ‘Tax Class C’, and ‘Tax Class D’. Presence of a tax class is represented by 1 and absence is represented by 0.  

The other thing we did when we cleaned and prepared the data is to standardize the range of those features. As you can see the range of the year when the property was built is from 1900 to 2009, whereas the neighborhood average price ranges from 10,000 to 5,000,000. We normalize features by calculating their standard scores $x’ = \frac{x-\bar{x}}{\sigma}$, where $\bar{x}$ is mean and $\sigma$ is the standard deviation.  

Since we have more than 5,000 records per borough each year, we are expecting to have the underfitting issue in our model. From feature engineering’s perspective, the way to avoid underfitting is to add more features, either from external resources or apply transformation on existing features. There are some other financial factors that may affect price of real estate properties such as people’s average annual income, GDP, inflation rate, etc. We are trying to gather these information and append them as new features to our model. On the other hand, we can generate new features using existing features by computing the production or exponentiation of feature values.

### Preliminary Analyses
<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/Housing%20Price%20-%20Block.png">  

As we can see from the above picture, real estate properties located between block 500 to block 1500 generally have higher prices than other properties located in other properties.  

<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/Housing%20Price%20-%20Building%20Class.png">   

The impact of building class on real estate properties.  

<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/Housing%20Price%20-%20Total%20Units.png">  

The total units in the properties is negatively correlated with housing prices.

### Preliminary Regressions
As described in the previous section, we have included the following columns in our input matrix for regression models: average price in the neighborhood, year the property was built, 4 “one-hot-encoding” columns to represent the categorical tax class, land squared feet and gross squared feet along the offset. For example, the first row in the matrix looks like the following: [ 1.37516e6  1920.0  0.0  1.0  0.0  0.0   2401.0   6920.0  1.0 ]

We chose the 2009 Manhattan sales data for a quick look and split the data set into training set and test set that are equally large. Then we train L2 linear regression, ridge regression (λ=1) and Lasso regression (λ=1) models on the training set and checked out their performance on the test sets.

<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/Summary%20Statistics%20for%202009%20Transactions.png">  

### Model Evaluation
We plan to build individual model for each year and for each borough individually. In the future work, we would like to split our dataset into training (70%), validation (20%), test(10%) on each year and each borough to further tune our parameters. Thus, we can use cross validation or k-fold validation to evaluate our model.

### Future Work
We plan to incorporate many more relevant features in our model and try out different feature mapping functions φ(x).

First, building class at time of sale and building class at present are two highly correlated features that are much more specific than the feature of tax class. Each building belongs to one of more than 20 classes. Also, the feature called total units can break into the number of residential and commercial units separately. 

More importantly, there are 6 significant geographical features remain unused which mean a great deal in justifying the transaction price. They are borough, neighborhood, block, lot, address (number plus street name) and zip code. Apparently, these feature depend on each other as a specific address must lie within a fixed zip code area. We ignore address because it is too hard to encode them in our matrix. Although lot is the most specific geographical feature next to address, we expect using block along will suffice. Unlike lot, every single block in the New York City has a unique code in this data set so we can use it directly. And we know the information conveyed in this feature already incorporates information conveyed in broader features including neighborhood, zip code and borough. 

Finally, we could consider including the time of sale as house prices may experience seasonal fluctuations. Given the general trend of property price is probably determined by larger changing economic environment, we may collect more data reflecting the overall financial or economic condition of the city or the entire country if time permits such that our model can predict changing prices over years.
