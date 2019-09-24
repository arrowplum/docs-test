### Customer Churn Analysis, Predictive Modelling and Risk Analysis 

A Churn Model utilizes Predictive Modelling and feeds into the Risk Model.
 
Churn refers to an existing customer deciding to end the business relationship. Customer typically refers to a client paying for services.  However, the analysis can also be extended to retention of human resources in a workplace, retention of membership in groups, social or professional. It addresses the issue of why, and how often, a member of a group decides to voluntarily part ways.

Traditionally churn is measured based on simple aggregations over time. In this case, it identifies outliers or periods of higher rates of churn from normal rates but does not offer any insight into the possible causes. It does not explore relationships between members of the separating group and offers no insight on possible causes and ameliorative actions that may be taken. Finally, without a deeper full breadth analysis, it is hard to predict churn.

#### Predictive Modelling

Predicting churn requires understanding churn influencing attributes specific to the business and events in the business cycle, looking at past patterns and patterns across similar businesses that underwent like events.

In traditional churn analysis, customers can be broadly divided into active, inactive but have potential to be re-engaged, customers that bring insignificant revenue or inactive and lost.  Churn analysis can be refined and targeted towards one or more of the above sub-groups. However with real time predictive capabilities, a new paradigm of customer “about to churn” is evolving who has the highest potential to be re-engaged. Aerospike is an ideal choice for targeting this churn prevention implementation.

Purchases by customers can be assigned attributes related to their frequency, aggregate value and timestamp.

#### Churn Predictive Models are not static

Churn models predict probability of churn given influencing factors or key factors affecting churn. If action is taken to address the factors that influence churn, the model in turn becomes obsolete and must be rebuilt with new churn data and influencing factors.  

Logistic Regression is a commonly used for modelling churn if data set is large. It computes maximum likelihood coefficients and given a set of conditions, it can be used to predict whether a particular customer will churn or not. It achieves smaller asymptotic error than Naive-Bayes classifier with increasing dataset size although Naive-Bayes converges faster (ie with a smaller dataset) than Logistic Regression. Unlike Logistic Regression, Naive-Bayes assumes factors are independent of each other, computes conditional probabilities from the data set and is not prone to overfitting. In either model, Aerospike can be used to store and update the model coefficients along with all the attribute data used to build the model.

#### Risk Analysis

Risk for an individual depends on the uncertainty in the event and exposure to the event. Even if the event carries a lot uncertainty, if an individual is not exposed to the event there is no risk to the individual. For example, a driver driving without a seat belt is exposed to the finite uncertainty of getting into an automobile accident resulting in serious injury and is assuming risk. Whereas a person watching this driver from afar carries no risk from this driver not wearing the seatbelt even though this event carries a finite uncertainty of serious injury.

In our case, we are interested in analysing business risks. Based on results from churn prediction models, it is possible to calculate risk on not realising expected revenue. Since risk relates to uncertainty, a customer that can be predicted to churn or not churn with a very high level of confidence presents a lower risk.

Risk models can also be employed independent of churn analysis in the financial services industry, for example in modelling risk in the insurance underwriting industry to determine how much money should be set aside to honor claims versus how much should be invested to make a return on investment.  However, these are not low latency problems.

Churn and risk modelling can be employed in low latency applications to prevent churn by promptly re-engaging a customer predicted to churn in real time either through a spot offer, via instant chat etc. Providing spot discounts to customers who are very unlikely to churn is a direct loss to the bottom line. Responsive web applications delivering dynamic content on any device to prevent churn when it is most likely, need low latency key value store like Aerospike to compute churn probability and take rules based action in real time. 



<center>
{{#figure "" "Aerospike in Churn Prediction" size="large" }}
<img src="/docs/connectors/assets/images/AerospikeInChurnPrediction.png" >
{{/figure}}
</center>

