![Stock_healthcare_image](shutterstock_400002673-1.jpg)
# How imperative is it for hospitals to focus on CMS metrics?

*Healthcare within the United States is extremely complex. The United States has attributes of all healthcare models(Medicare akin to Canada's model and Veteran's Affair like Germany's model) around the world present; however, the majority of citizens, [around 66%, have private insurance, while 35.7% have a form of public healthcare insurance](https://www.census.gov/library/publications/2022/demo/p60-278.html). Citizens with private insurance utilize private hospitals/providers for their healthcare needs. Their insurance provider pays/reimburses the private hospitals/providers for their clients. How much they pay the provider for their services is negotiated for between the insurance company and provider. These negotiated rates have many factors that go into determining reimbursement and includes various economic factors (supply vs demand, economies of scales, cost of services, etc) alongside internal metrics that are unknown. Most of these prices are closed book or held private by insurance companies within the USA, at least until recently. Congress passed [amendments that required health insurance companies to publish their reimbursement data for procedures](https://www.uhc.com/united-for-reform/health-reform-provisions/transparency-in-coverage-rule). This can be used by hospitals to determine how much other hospitals may be charged for a similar procedure. In addition, since the Affordable Care Act, the whole healthcare industry has been heading towards performance based pricing, or using performance metrics of providers to determine their reimbursement. Currently the government has been doing this for various providers and certain private insurance companies have been public about using similar metrics. My goal is determine which metrics hospitals should focus on in order to increase their reimbursement from private insurance companies.*

## 1. Data

For this project, I used the United Healthcare Insurance Dataset, which has JSON files for all the companies covered by United Health Care and all the purchases for the given year for these clients. This is an extremely large dataset so I was only able to use a part of it (some of these files are terabytes in size). Secondly, I needed files to link providers to hospitals alongside hospital metrics. Fortunately the CMS has publicly available metrics and provider affiliations to hospitals. The folder here contains many metrics; I will only use some of these.

> * [United Healthcare Insurance Dataset](https://transparency-in-coverage.uhc.com/?_gl=1*5it7ok*_ga*NjMzOTkzMDA0LjE2NzI3OTc4MjA.*_ga_HZQWR2GYM4*MTY3Mjc5NzgyMC4xLjAuMTY3Mjc5NzgyMC4wLjAuMA)

> * [CMS Metric and Provider Data](https://github.com/tennisvs/Springboard/blob/33da40f392e705b0158f1e08198eb5c78c1e615c/Capstone/Hospital_Metrics)

## 2. Method

There are several steps to approach this problem of how to determine which features would increase negotiated rates and which one is best for hospitals to focus on. I kept this open ended due to the data available to me:

1. **Match Providers and Provider Groups to Hospitals:** The JSON files from the United Healthcare Insurance Dataset are extremely large and organized in a transaction fashion. This requires reading each JSON file and determining which provider (NPI) is associated with which hospital. This is part of the parsing process. CMS has NPIs that match to hospitals, which we will use. We made some assumptions that provider groups/hospitals negotiate together, as this would increase their negotiating power for reimbursement rates.

2. **Determine Negotiated Rates for Procedures and Hospitals:** Part of the parsing process, I need to compile a dataset from the United Healthcare Insurance Dataset of negotiated prices for each procedure for different hospitals. This will be the main dataset I work with. There are various ways procedures are coded, so I will need a way to decipher these codes. Fortunately, there are some publicly available [websites](https://www.aapc.com/codes/cpt-codes-range/) that can be used to determine, which codes match certain procedures.

3. **Determine Procedures and Metrics to build a Model:** Once I have a workable dataset, I will go through the Data Cleaning and EDA, ultimately using EDA to determine which procedure suitable for this analysis. This includes, but is not limited to, how many negotiated rates are available for each procedure, do the metric descriptions match the procedure that they may be used for, how expensive are these procedures, etc. This takes some background knowledge in the healthcare system.

| Code Type | Code Value | Description | Associated CMS Metric | Metric Details
| --- | --- | --- | --- | --- |
| CPT | 36556 | Under Insertion of Central Venous Access Device | HAI_1 | Central Line Associated Bloodstream Infection
| CPT | 51701 | Under Introduction Procedures on the Bladder | HAI_2 | Catheter Associated Urinary Tract Infections
| CPT | 51702 | Under Introduction Procedures on the Bladder | HAI_2 | Catheter Associated Urinary Tract Infections
| HCPCS | A4314 | Insertion tray with drainage bag with indwelling catheter, Foley type, 2-way latex with coating (Teflon, silicone, silicone elastomer or hydrophilic, etc.) | HAI_2 | Catheter Associated Urinary Tract Infections
| HCPCS | A4315 | Insertion tray with drainage bag with indwelling catheter, Foley type, 2-way, all silicone | HAI_2 | Catheter Associated Urinary Tract Infections
| HCPCS | G9312 | Surgical site infection | HAI_3 | Surgical Site Infection - Colon Surgery
| CPT | 58150 | Under Hysterectomy Procedures | HAI_4 | Surgical Site Infection - Abdominal Hysterectomy
| CPT | 15920 | Under Pressure Ulcers (Decubitus Ulcers) Procedures | PSI-3 | Pressure Ulcer Rate
| CPT | 35800 | Under Repair, Excision, Exploration, Revision Procedures on Arteries and Veins | PSI-9 | Postoperative hemorrhage or hematoma rate
| HCPCS | J1650 | Injection, enoxaparin sodium, 10 mg | PSI-12 | Perioperative pulmonary embolism or deep vein thrombosis rate
| HCPCS | C9604 | Percutaneous transluminal revascularization of or through coronary artery bypass graft (internal mammary, free arterial, venous), any combination of drug-eluting intracoronary stent, atherectomy and angioplasty, including distal protection when performed | MORT_30_CABG | Death rate for CABG surgery patients

4. **Build the Model:** Finally I will build a supervised learning model. I will explore gradient trees, to K means clusters and evaluate performance on metrics discussed below. Ultimately we will use this model to select a hospital and determine what metrics I can manipulate to help increase the negotiated reimbursement. Under this circumstance, I will assume the hospital has limited resources financially.

## 3. Data Cleaning 

In a collaborative-filtering system there are only three columns that matter to apply the machine learning algorithms: the user, the item, and the explicit rating (see the example matrix above). I also had to clean & normalize all the reference information (location, difficulty grade, etc.) to the route so that my user could get a useful and informative recommendation.

* **Problem 1:** This dataset is all user-entered information. There are a couple drop down options, but for the most part the user is able to completely make-up, or list something incorrectly. **Solution:** after normalizing & cleaning all the columns, I created a three-tier groupby system that I could then take the mode of each entry and fill in the column with that mode. For example: a route listed 12 times had the country Greece associated with it 11 times, but one person incorrectly listed it located in the USA. By grouping together three other indicator columns and then computing the mode of the country, I was able to catch and change some of the user-entered errors and increase the accuracy of my dataset.

* **Problem 2:** Being this is an international rock climbing website, the names of the rock climbing routes were differing based on if the user enters accent marks or not. **Solution:** normalize all names to the ascii standards. 

* **Problem 3:** Spelling issues with the route name. For example: if there was a route named "red rocks canyon" it could be spelled "red rock", "red rocks", "red canyon" etc. **Solution:** at first, I was hopeful and tried two different phonetic spelling algorithms (soundex & double metahpone). However, both of these proved to be too aggressive in their grouping and sometimes would group together up to 20 different individual routes as the same item! My final solution was to create an accurate filter for route names. The logic being that if up to x number of users all entered that *exact same* route name, the chances were good that it was an actual route spelled correctly. I played around with 4 different filters and kept these until I could test their prediction accuracy in the ML portion. I found the greatest prediction accuracy came from the dataset that filtered out any routes listed less than 6 times.

## 4. EDA

[EDA Report](https://colab.research.google.com/drive/14AKVsyXy7yJSxBjmEBFyz7kEX7e9ioM_)

* The star-rating distributions all checked out to be normal. It is very common with explicit ratings to see a diminished number of low ratings.

![](./6_README_files/star_counts.png)

## 5. Algorithms & Machine Learning

[ML Notebook](https://colab.research.google.com/drive/1kAlvwwJnGcdCAJD8oFokT3gtJF2UnyZP)

I chose to work with the Python [surprise library scikit](http://surpriselib.com/) for training my recommendation system. I tested all four different filtered datasets on the 11 different algorithms provided, and every time the Single Value Decomposition++ (SVD++) algorithm performed the best. It should be noted that this algorithm, although the most accurate is also the most computationally expensive, and that should be taken into account if this were to go into production.

![](./6_README_files/algo.png)

>***NOTE:** I choose RMSE as the accuracy metric over mean absolute error(MAE) because the errors are squared before they are averaged which gives the RMSE a higher weight to large errors. Thus, the RMSE is useful when large errors are undesirable. The smaller the RMSE, the more accurate the prediction because the RMSE takes the square root of the residual errors of the line of best fit.*

**WINNER: SVD++ Algorithm**

This algorithm is an improved version of the SVD algorithm that Simon Funk popularized in the million dollar Netflix competition that also takes into account implicit ratings (*yj*). Using stochastic gradient descent (SGD), parameters are learned using the regularized squared error objective.

![](./6_README_files/forumla.png)

## 6. Which Dataset to choose?

[More details about this process...](https://colab.research.google.com/drive/1kAlvwwJnGcdCAJD8oFokT3gtJF2UnyZP)

After choosing the SVD++ algorithm, I tested the accuracy of all four different filtered datasets. The dataset which filtered out any route names occurring less than 6 times performed the most accurate predictions. Thus, it was chosen to be the dataset I trained on.

>* All of the dataframes displayed discrepancies with the 1 star ratings(This is to be expected due to the inherent skewed positive ratings). Also, the one star ratings are not imperative to this project's goal. It is more important that the 1 star ratings are different enough to be filtered out of the top ten routes recommended to users. 
>* Notice the 3-star rating has a fat bulge at the top of the "violin" which indicates its predicting 3-star ratings for some of the true 3-star routes. This was not as prominent in the other dataframes
>* The 1-star rating also has a fatter tail than the other datasets displayed


![](./6_README_files/accuracy.png)

## 7. Coldstart Threshold
[More details about this process...](https://colab.research.google.com/drive/1kAlvwwJnGcdCAJD8oFokT3gtJF2UnyZP)

**Coldstart Threshold**: There is a problem when only using collaborative based filtering: *what to recommend to new users with very little or no prior data?* Remember, we already set our cold start threshold for the routes by choosing the dataset that filtered out any route occurring less than 6 times. Now, let investigate where to put the threshold for users.

![](./6_README_files/20user_thresh.png)

*It is my hypothesis that the initial filtering of the routes is what affected the RMSE of the users* 

>* Increasing the user threshold to 5 would increase the RMSE by .005 & would lose approximately 40% of the data.
>* Increasing the user threshold to 13 would increase the RMSE by .0075 & would lose approximately 60% of the data
>* If there were a larger increase in the RMSE (>= .01) I would trade my users' data for this improvement. However, these improvements are too minuscule to give up 40%-60% of my data to train on. Instead, I voted to keep some of these outliers to help the model train, and will focus on fine tuning my parameters using gridsearch to improve the RMSE


## 7. Predictions

The potential scenarios I explored:
1. Increasing either PSI 15 by a certain percentage or increase HAI 2 SIR by a certain percentage
2. Increase Demand by certain percentage by acquiring another hospital, either through discharge ratio/hospital days
3. Increase Number of Interns/Residents
4. Increase both PSI 15 and HAI 2 by a certain percentage

I accounted for each of these scenarios to determine the new price to be charged:

**For scenario 1:** The model supported this outcome, particularly PSI 15 metric. However the price change would be minimal.
**For scenario 2:** The model did not support this outcome and it would be extremely costly.
**For scenario 3:** The model did not support this outcome, as it would make little change to price of the procedure.
**For scenario 4:** The model supported this outcome, especially if increasing all PSI metrics by +30% or more.

*Scenario 4* seems the most appealing to me. Increasing all PSI metrics go hand in hand and hospitals would be required to implement similar measures to increase all metrics. In addition, we did not account for decreases in HAI values, which would probably decrease as well with any hospital implementation of these metrics. However, our hospital seems to be doing well in their metrics.
[Scenario 4](https://github.com/tennisvs/Springboard/blob/9b03c6b77c0d53012ef0ff02cd39e5930b574ebe/Capstone/psi_hai.png)

## 8. Future Improvements

* There were certain limitations to the work done. First, looking at the data, I was not able to parse all the values from the JSON files just due to processing power. We were limited in areas where these prices came from, mostly from the state of Florida. In addition, we limited the procedures to look at. The datasets were matched on different time periods given the limitations of this analysis. Using a cloud based server would serve me well for future improvements.

* Additionally, our model did not have great accuracy, at around 79%. I believe a deeper model might be able to predict the prices at a better rate. Our predictions on how prices affect revenue limits our analysis on what decision is best. But this does show proof of concept.

* Creating an interactive program with different insurance companies would be more informative to different hospitals.

## 9. Credits

