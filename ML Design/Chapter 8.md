Data Distributions

Covariate Shift 

The input distributions changes , but the conditional probability of an output given an input remains the same.

for us the input features are covariate and the output is our variable of direct interest 

To make this concrete, consider the task of detecting breast cancer. You know that the risk of breast cancer is higher for women over the age of 40, so you have a variable “age” as your input. You might have more women over the age of 40 in your training data than in your inference data, so the input distributions differ for your training and inference data.