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

### Data Cleaning & Preprocessing
The first thing we noticed about the dataset was a large part of missing data. As you can see from the above dataset description, we have around 33.8% sales records that are incomplete (incomplete records are those who has value 0 in at least one field).  

In addition, we found that some of the entries are questionable even in those where there's no missing data. For example, some entries with really low or high property sale price is clearly wrong and cannot be used. To define a reasonable threshold, we did some research online. From https://www.trulia.com/real_estate/New_York-New_York/market-trends/, it is reasonable to say that the real estate property unit price should be at least \$10 per square feet. Thus, we only consider the properties whose price is above \$10 per square feet. There's several other cases where the data is wrong, such as entries with gross square feet less than 10, or the residential unit and commercial unit doesn't add up to the corresponding total unit.

<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/data_distribution.png">

### Feature Selection & Transformation

<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/ro.png">

<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/bo.png">

<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/go.png">

### Results Evaluation

<img src="https://github.com/AaronZang/ORIE4741-Home-Purchase-Assistance/blob/master/image/ratios.png">
