Example of analysing data from a clinical trial using synthetic data.

## The Data

**clinical_study.csv**

|COLUMNS|DESCRIPTION|
|-------|-----------|
|subject_id|Patient ID|
|sex|Whether the patient is male or female|
|age|The age of the patient|
|weight|The weight of the patient|
|height|The height of the patient|
|trt_grp|Whether the patient is receiving the new drug or the standard of care (control)|
|response|Whether the patient responsed|

|subject_id|age|sex|weight|height|trt_grp|RESPONSE|
|----------|---|---|------|------|-------|--------|
|SUBJ_001|46.0|Female|84.66|1.59|DRUG|N|
|SUBJ_002|47.0|Female|71.21|1.64|DRUG|N|
|SUBJ_003|48.0|Female|69.85|1.73|CONTROL|N|
|SUBJ_004|59.0|Female|62.94|1.50|DRUG|Y|
|SUBJ_005|59.0|Female|113.91|1.63|CONTROL|N|
|SUBJ_006|63.0|Male|79.33|1.77|CONTROL|Y|
|SUBJ_007|77.0|Male|96.12|1.77|CONTROL|N|
|SUBJ_008|57.0|Male|93.50|1.63|DRUG|N|
|SUBJ_009|72.0|Male|85.57|1.68|DRUG|N|

**protein_levels.csv**

|COLUMNS|DESCRIPTION|
|-------|-----------|
|participant_id|Patient_ID|
|protein_concentration|Concentration of a blood protein (ug/L) that might be a potential predictive biomarker of response|


|participant_id|protein_concentration|
|--------------|---------------------|
|SUBJ_001|148.0|
|SUBJ_002|85.0|
|SUBJ_003|183.0|
|SUBJ_004|89.0|
|SUBJ_005|137.0|
|SUBJ_006|116.0|
|SUBJ_007|78.0|
|SUBJ_008|115.0|
|SUBJ_009|197.0|

We want to adress questions such as:
* Do patients that take the drug respond more to treatment compared to those in the control group?
* Can we use data science to predict whether a patient will respond better to the new treatment?

## Examining Data

`clinical_study.info()`

|#|Column|Non-Null Count|Dtype|
|-|------|--------------|-----|
|0|subject_id|772 non-null|object|
|1|age|772 non-null|float64|
|2|sex|772 non-null|object|
|3|weight|761 non-null|float64|
|4|height|772 non-null|float64|
|5|trt_grp|772 non-null|object|
|6|RESPONSE|772 non-null|object|

`clinical_study.describe()`

|   |age|weight|height|
|---|---|------|------|
|count|772.00|761.00|772.00|
|mean|61.58|91.11|1.677|
|std|7.87|22.49|0.10|
|min|7.20|22.31|1.19|
|25%|57.00|75.55|1.60|
|50%|62.00|88.87|1.67|
|75%|67.00|104.65|1.76|
|max|79.00|182.50|1.94|

`protein_levels.info()`

|#|Column|Non-Null Count|Dtype|
|-|------|--------------|-----|
|0|participant_id|768 non-null|object|
|1|protein_concentration|763 non-null|float64|

`protein_levels.describe()`

|   |protein_concentration|
|---|---------------------|
|count|763.00|
|mean|121.69|
|std|30.54|
|min|44.00|
|25%|99.00|
|50%|117.00|
|75%|141.00|
|max|199.00|

## Replacing missing values

`clinical_study.isnull().sum()`

|column|# of missing values|
|------|-------------------|
|subject_id|0|
|age|0|
|sex|0|
|weight|11|
|height|0|
|trt_grp|0|
|RESPONSE|0|

`protein_levels.isnull().sum()`

|column|# of missing values|
|------|-------------------|
|participant_id|0|
|protein_concentration|5|

The two columnms with missing values are *weight* and *protein_concentration* There are many ways to deal with missing values. For example depending on the data and task, we could drop the rows with missing values or replace the missing values with the mean, median, mode, or even a constant value . We can use a box plot or density plot to help decide what method to follow

![image](https://github.com/Daniel-ET/clinical_trial/assets/96924468/07944ea7-eb7b-42e6-8e8c-3443a2f731c9)
![image](https://github.com/Daniel-ET/clinical_trial/assets/96924468/cd4aa035-cdb9-4819-b0d3-19e7b5b1ff48)

As we can see, the data for age and protein is slighly skewed. Therefore it would be better to use the median to replace the missing values

```clinical_study = clinical_study.fillna(clinical_study.median())```

```protein_levels = protein_levels.fillna(protein_levels.median())```

## Create new variables and merge dataframes

#### a. BMI

BMI is calculated by dividing weight by the square of the height

``clinical_study['bmi'] = clinical_study['weight']/clinical_study['height']**2``


|subject_id|age|sex|weight|height|trt_grp|RESPONSE|bmi|
|----------|---|---|------|------|-------|--------|---|
|SUBJ_001|46.0|Female|84.66|1.59|DRUG|N|33.487599|
|SUBJ_002|47.0|Female|71.21|1.64|DRUG|N|26.476056|
|SUBJ_003|48.0|Female|69.85|1.73|CONTROL|N|23.338568|
|SUBJ_004|59.0|Female|62.94|1.50|DRUG|Y|27.973333|

#### b. merge dataframes

we must first rename the patient id column in one of the dataframes since they are named differently. We will be renaming the 'participant id' column in the protein_levels dataframe

`protein_levels = protein_levels.rename(columns={'participant_id': 'subject_id'})`

We can then merge the two dataframes on the subject id

`df = clinical_study.merge(protein_levels[['subject_id', 'protein_concentration']], on = 'subject_id')`

|subject_id|age|sex|weight|height|trt_grp|RESPONSE|bmi|protein_concentration|
|----------|---|---|------|------|-------|--------|---|---------------------|
|SUBJ_001|46.0|Female|84.66|1.59|DRUG|N|33.487599|148.0|
|SUBJ_002|47.0|Female|71.21|1.64|DRUG|N|26.476056|85.0|
|SUBJ_003|48.0|Female|69.85|1.73|CONTROL|N|23.338568|183.0|
|SUBJ_004|59.0|Female|62.94|1.50|DRUG|Y|27.973333|89.0|

## Aggregating data

#### comparing averages in the two treatment groups

![a](https://github.com/Daniel-ET/clinical_trial/assets/96924468/3afff38f-fa95-4dcd-ae54-cd03e90f18af)

#### comparing responders and non-responders

![b](https://github.com/Daniel-ET/clinical_trial/assets/96924468/6c9401c4-d62b-4b0b-9063-e1a8f3739706)

#### compare responders and non-responders in the two treatment groups

![c ](https://github.com/Daniel-ET/clinical_trial/assets/96924468/daeb7baa-99ec-463d-be56-0b32c9a611d2)

## Visualizing the data

#### i. Boxplot of age(y-axis) by response (x-axis)

![image](https://github.com/Daniel-ET/clinical_trial/assets/96924468/e6951263-54da-448f-8a21-b1d7f8a42555)

##### seperated by treatment group

![image](https://github.com/Daniel-ET/clinical_trial/assets/96924468/3d8b83e1-5fbe-4d1d-85e6-39b1f36e16ea)

#### ii. Boxplot of weight/BMI(y-axis) by response (x-axis)

![image](https://github.com/Daniel-ET/clinical_trial/assets/96924468/5d307e32-3d10-4f75-ba4c-8d0151e9baf6)

##### seperated by treatment group

![image](https://github.com/Daniel-ET/clinical_trial/assets/96924468/a23a4b89-1eec-461f-9878-f9ad7e3ba007)

#### iii. Boxplot of protein_concentration(y-axis) by response(x-axis)

![image](https://github.com/Daniel-ET/clinical_trial/assets/96924468/41937305-74dd-437e-92d5-d35b1d4e72d7)

##### Seperated by treatment group

![image](https://github.com/Daniel-ET/clinical_trial/assets/96924468/d1c0d875-849f-4001-846e-875946083cf8)

## Modelling (Logistic Regression)

We cam use data science to predict whether a patient will response to treatment based on their age, sex, weight, height, trt_grp, bmi, and protein concentration. This is a binary classification task so we will be using a logistic regression although other algorithms such as decision trees, random forests, or support vector machines might be appropriate.

First we prepare the data by splitting it into features(X) and the target variable(y):

`X = df.drop(['RESPONSE', 'subject_id'], axis=1)`

`y = df['RESPONSE']`

we then encode categorical variables. In this case we will use one-hot encoding

`categorical_columns = ['sex', 'trt_grp']`

`encoder = OneHotEncoder(sparse=False, drop='first')`

`encoded_cols = pd.DataFrame(encoder.fit_transform(X[categorical_columns]))`

`encoded_cols.columns = encoder.get_feature_names_out(categorical_columns)`

we then drop the original categorical columns from X

`X = X.drop(categorical_columns, axis=1)`

and then concatenate the encoded categorical columns with X.

`X_encoded = pd.concat([X, encoded_cols], axis=1)`

We now split the data into training and tests,

`X_train, X_test, y_train, y_test = train_test_split(X_encoded, y, test_size=0.2, random_state=42)`

scale the features,

`scaler = StandardScaler()`

`X_train = scaler.fit_transform(X_train)`

`X_test = scaler.transform(X_test)`

Create and train the model

`model = LogisticRegression()`

`model.fit(X_train, y_train)`

Make predictions on the test set

`y_pred = model.predict(X_test)`

finally we evaluate the performance of the model

` accuracy = accuracy_score(y_test, y_pred)`

This model achieved an accuracy of 82%




     
































