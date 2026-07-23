- ML is like the new Software (_software 2.0_)

- Main Things
	- There need to be a Capacity to *Learn*
	- There need to be *Complex Patterns* to Learn that are too expensive to manually figure out.
	- *Data* is the key
		- Zero - Shot learning is possible , still the model needs to be trained on some other data.
	- A problem should be frameable as a *Predictive Problem*
	- The Data that we are going to use for predicting should be of the same distribution as the data that we used in training.
	- ML Models are great when the patterns are repetitive
	- We Should use ML systems when the cost of wrong predictions are cheap or average cost of right predictions out weight the one or two majorly affecting wrong predictions
	- Continual Learning -> models should be retrained on change of data to accommodate the change
	
- When Not To Use ML
	- When its unethical
	- When Simpler Solutions can do the work
	- When its not cost-effective

- Consumer ML World Vs Enterprise ML World
	- enterprises are stricter in requirements about the accuracy rather than latency
		- an consumer might not notice 0.5% increase in accuracy
		- but a resource allocation system's efficiency improvement by 0.1% might save a company millions of dollars

- Areas Where Enterprise Use ML
	- Fraud Detection
	- Price Optimization 
	- Demand Forecasting 
	- Customer Acquisition Optimization 
	- Customer Churn Prediction 
	- Automated Support Ticket Classification 
	- Brand Monitoring 
	- Sentiment Analysis
	- Healthcare Diagnosis and Medical Detection

## Understanding ML Systems

**PRODUCTION ML IS SO MUCH DIFFERENT FROM RESEARCH ML**
 
|                      | Research                       | Production                  |
| -------------------- | ------------------------------ | --------------------------- |
| Requirments          | SOTA                           | different requirments       |
| Computation Priority | Fast Training, High throughput | Fast Inference, low latency |
| Data                 | static                         | constant shifiting          |
| Fairness             | Not a focus                    | must be considered          |
| Interpretability     | Not a focus                    | must be considered          |
### Requirments

### Computation Priority

- Latency = time taken to process one request; lower latency means faster responses.  Throughput = number of requests processed per second; higher throughput means more work done.  
- Real-time ML systems prioritize **low latency** (fast user response).  
- Batch or large-scale systems prioritize **high throughput**, often using batching which can increase latency.
- Latency is not an individual number but a distribution
### Data

- In production, data, if available, is a lot more messy. It’s noisy, possibly unstructured, constantly shifting. It’s likely biased, and you likely don’t knowhow it’s biased. Labels, if there are any, might be sparse, imbalanced, or incorrect. 

### Fairness



