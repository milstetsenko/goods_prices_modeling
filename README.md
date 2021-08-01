The following project using Stan in Python to model the fluctuations of the prices among the 10 most common goods across 7 countries in the world. The project aims to give an opportunity to predict the prices in different countries for these same products given the coefficient of the country - how more or less expensive the prices are for a given country.


## Model Justification

I create three models that will give three posterior parameters
: the base price for each of the product (similar to the average price of the product without the affect of the location and the store type); the multiplier for the country, and the multiplier for the store type: budget, luxury, mid-range. Multiplication of the base price to the country and the store type parameters will give the predicted price for the good. 

To create these three parameters, I build three models that will output the three parameters. Then I combine them into another model to predict the actual price. 

### The Base Price

Positive real number. 

1. All the prices were converted to USD to make the model more precise and easier to work with. The prior for this model will be Exponential distribution because its support is positive values, most of the products we are evaluating should be similar in price with some more expensive items (like chicken and meat). The lambda parameter is derived from gamma distribution with the parameters (1, 0.2) with $E(X) = 0.2$ so that the value correspond to dollars (see figure below). The Exponential distribution with the parameter 0.2 will have the majority of the prices below \$10 - this is my prior knowledge of prices for the products of interest.






𝐿𝑎𝑚𝑏𝑑𝑎 ∼ 𝐺𝑎𝑚𝑚𝑎(1, 0.2)

𝐵𝑎𝑠𝑒𝑃𝑟𝑖𝑐𝑒 ∼ 𝐸𝑥𝑝(𝑙𝑎𝑚𝑏𝑑𝑎)




### Country and Store Type multipliers

Positive real numbers. 

For both of these multipliers, it should be centered around 1 and the values should be positive. Based on this information, the prior will be the log-normal distribution. The parameters of the distribution will be 0 and 0.5 (0 accounts for the centering, 0.5 accounts for the variance, which we do not want to be too wide). 

𝑆𝑡𝑜𝑟𝑒𝑇𝑦𝑝𝑒 ∼ 𝐿𝑜𝑔𝑁(1,𝑒0) 

𝐶𝑜𝑢𝑛𝑡𝑟𝑦 ∼ 𝐿𝑜𝑔𝑁(1,𝑒0)

### Likelihood Function Outputting the three multipliers


 Likelihood function is Normal. Since we are trying to predict the base price, the assumption here is that the prices are normally distributed. It will take the parameters $x$ and $\sigma$. The x parameter will include the multiplication of the base price (the data used is the prices for each of the produce), store type (each datum is classified into one of the three categories - budget, mid-rand, luxury), and the country multipliers(each datum is classified with the category of the country). Sigma will be derived from another prior.


𝑃𝑟𝑖𝑐𝑒 ∼ 𝑁((𝑆𝑡𝑜𝑟𝑒𝑇𝑦𝑝𝑒 ⋅ 𝐶𝑜𝑢𝑛𝑡𝑟𝑦 ⋅ 𝐵𝑎𝑠𝑒𝑃𝑟𝑖𝑐𝑒),𝑠𝑖𝑔𝑚𝑎)


### Sigma Prior Justification

I choose the gamma hyperprior with parameters $(1,1)$ so that the variance is moderate since little information about uncertainty exists. 

𝑆𝑖𝑔𝑚𝑎 ∼ 𝐺𝑎𝑚𝑚𝑎(1, 1)

## Data Processing

To feed the model in Stan language, I need to pre-format the data with all necessary known pieces.
The data needed is: the prices, the country index for each price, the store type index for each price, the product index for each price. I also need to find the length of these lists to iterate successfully in the model in Stan.

To identify and format the data, I proceed with the following steps:

• drop the unnecessary rows in the csv file

• convert the prices into the prices for 1 kg or 1 count by dividing the current price by the good’s weight or count

• convert the currency of the products to USD by multiplying by the conversion rate

• formalize the locations of the countries to 6 different categories: USA, Sweden, Ukraine, Kenya, Brazil, and Bangladesh

• formalize the store types to 3 different categories: budget, mid-rane, luxury

• formalize the product names to 10 different categories: Apples, Bananas, Tomatoes, Potatoes, Flou, Rice, Milk, Butter, Eggs, Chicken.

• restructure the data to include columns: prices, products, countries, and store type.

• drop the Nan values

I create the dictionary of these values so then they will be fed into the model.

## Model Inference

The output of the model is the parameters sigma and lambda for the priors, the prediction values for each product’s base price, each country’s multiplier, and each store type multiplier, as shown in the figure below:

<img width="814" alt="Screen Shot 2021-08-01 at 6 59 31 PM" src="https://user-images.githubusercontent.com/65287937/127777515-76a864ca-5e44-466f-9a2f-06c38cdb23af.png">

### Model Precision Evaluation

Other parameters are outputted in the results section that justify the reliability of the model but they are omitted in the report for simplicity of understanding. From the first glance, the results correctly predict the prices, based on the data. Since the predicted price is averaged across different observations, the price might not exactly match the price at store X for the same components of store type, country, and product.

The reason for that is that the data varies greatly for the same range of prices, product, and country. For example, let’s take a look at apples in USA in mid-range stores because this is one of the biggest samples. Apples in Texas cost $6-$8 per 1 pound in mid-range store and $3 for 3 pounds in San Francisco. Converted to kilograms, this will be $12-$16 per 1 kilo and $2.20 for 1 kilo in San Francisco.

Predicted price in dollars for 1 kilogram:

2.66 ⋅ 1.78 ⋅ 1.56 = $7.38

For the reasons above, the predicted price for the apple in San Francisco will be higher than the actual price and lower than the price in Texas.



## Visualization of the Model’s Outcomes
### Base prices

As shown on the figure below, the base prices distributions for each of the product represent the data. The prices for butter, chicken are the highest and also with the highest variance. Theproducts with the lowest prices are eggs, milk, flour, and bananas.

Because multiple locations are included in the model, multiple products are seasonal or grow in only some parts of the world, the prices can vary.

<img width="814" alt="Screen Shot 2021-08-01 at 6 59 11 PM" src="https://user-images.githubusercontent.com/65287937/127777497-016a1aa3-9165-4292-bfa3-fdfd5ca44f15.png">

Figure 4: Samples for base prices of different products from the posterior distribution

### Country Multiplier

The countries multipliers look correct. The more expensive prices are in the USA and Sweden - these countries are more developed and the price of living there is higher. The rest of the
countries are cheaper, so the produce is cheaper there, as well.

Ukraine, Brazil, and Bangladesh’s prices have less variance, because the data was more consis- tent (or also because of the smaller number of samples from these countries). Sweden only had 2 observations from budget and luxury store, therefore, there is much uncertainty in results. As mentioned in the previous subsection, the USA had many various prices for the same products in different parts of the country. Its variance is high because it is hard to formulate a certain value that will apply to all the stores.
<img width="814" alt="Screen Shot 2021-08-01 at 6 58 15 PM" src="https://user-images.githubusercontent.com/65287937/127777461-a4483cde-5f74-47ae-a820-0244abba0b33.png">
<img width="814" alt="Screen Shot 2021-08-01 at 6 58 10 PM" src="https://user-images.githubusercontent.com/65287937/127777458-4dd22f3e-dd45-4d00-90ec-d68f35bf6ad7.png">

### Store Type Multiplier

Budget is clearly smaller value than the Mid-range and Luxury. It is mostly concentrated right below 1.
Mid-range and Luxury are harder to distinguish between, probably because of the imperfection of the model and the real difference between the prices of the stores in the data collected.

## Model Conclusion

In general, the model predicts well, considering the data and its purpose - average prices for the 3 criteria: product, country, range of store. More improvements can be done for the model as data increase and model iteration, splitting apart the different states of the country (like states for the US which have different prices). Than the model will be able to more precisely predict the prices for that specific location.
