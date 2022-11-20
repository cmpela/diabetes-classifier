## Undiagnosed Diabetes Mellitus Prediction on general health and behavior indicators using supervised learning techniques

### Contributors: 

[Conrado Pelá - CP3017133](https://github.com/cmpela/)<br>
[Thales Pomari - CP3013456](https://github.com/thalespomari)<br>
[Tuanny Leite - CP3016285](https://github.com/tuleite)

Work developed and presented for "Big Data" and "Machine Learning and Pattern recognition" courses at Data Science Specialization of IFSP/Campinas

---


### Highlights - Start reading here:
* Diabetes is a metabolic disease that results in high concentrations of sugar in the blood which impairs the functioning of organs and tissues.
* Currently there are 3 methods of diagnosing DM, all of which rely on laboratory tests
* It is estimated that 1 in 5 Americans has diabetes but has not been diagnosed
* We propose an approach to notify possible carriers of the disease who have not been diagnosed using supervised machine learning techniques
* We were able to use data available in the health system to run our model and hope to reduce underreporting
---

### Diabetes Mellitus

Diabetes Mellitus (DM) is a complex metabolic disease that can present different manifestations, although it is mostly characterized by elevated levels of blood glucose. Diabetes’ primary cause is related to Insulin deficiency (in Type I, autoimmune response that destroys insulin-producing cells) or Insulin resistance (in Type II, when insulin receptors lose their sensibility to it).

Insulin is an hormone that transport glucose molecules inside cells, where they are transformed in energy. In both DM Types cases, there is a malfunctioning of this molecular transportation mechanism [1]. As a result the body 'starves', while blood glucose accumulate and reacts with organs and tissues causing problems as kidney failure, blindness, heart-attack, limb amputation and strokes[1][5].

--- 

### How DM is diagnosed? 

There are three tests used as standard for DM diagnosis [9] [10]: 1. Fasting Blood sugar test, that consists in measuring blood glucose after 8-12h fasting 2. A1C test, that consist in measuring the percentage of glycosylated hemoglobin in blood, 3. Glucose tolerance test, that measure glycemia over time after ingestion of a controlled amount of glucose.

Depending on the combination of test results, it is possible to characterize the patient being in Normal, Prediabetes and Diabetes.

![DM results](https://i.imgur.com/wLOuCJ1.png, "DM Thresholds")

---

## Motivation

DM prevalence worldwide is underestimated due to healthcare access limitations. According to Center for Disease Control (CDC), in US 1 in 5 people with DM is undiagnosed, and 8 in 10 people with prediabetes are unaware of their condition [12]. In Brazil 15.102 participants were evaluated between 2008 and 2010 at the Brazilian Longitudinal Study of Adult Health (ELSA-Brasil), where 19.7% had DM but half (50.4%) were previously undiagnosed [8].

Brazilians state that scheduling tests and appointments are the biggest challenges they face when using national healthcare system (40% and 38%. respectively) [14][15]. Additionally, only 15% of the system users are engaged in preventive initiatives involving physical activities.[15]

The combination of deficient health monitoring and absence of preventive DM habits in population leads to delay in diagnostic and worsen of health condition in untreated patients [15]. As a reflex, factors as related deaths, early retirement and absenteeism end up being very relevant to the DM expenditure estimation. Consequently, these indirect costs are considered to play a major contribution - around 70% - in low/medium-income countries. Including direct and indirect costs, the 2030 projection for DM related costs in Brazil surpass US 5𝑏𝑖𝑙𝑙𝑖𝑜𝑛,𝑚𝑜𝑟𝑒𝑡ℎ𝑎𝑛𝑑𝑜𝑢𝑏𝑙𝑒𝑜𝑓𝑡ℎ𝑒2016𝑏𝑖𝑙𝑙(133.4 2108 [16]. A more recent work from 2018 considering hospitalization risk for DM patients in Brazil found that the average cost of an adult hospitalization due to diabetes was US$845, 19% higher than hospitalization without DM [17].

A ML classification model could benefit the public healthcare system driving to more effective targeting in actions for diagnosis expansion and preventive initiatives. An opportunity lies in the healthcare system informatization efforts, as seen in Citizen's Electronic Health Record (Prontuário Eletrônico do Cidadão - PEC) initiative. PEC consists in a software aggregating clinical and administrative patient information, and therefore could feed a DM detection model. [18]

A supervised machine learning model to predict DM risk based on general health status and behavior of patients can greatly contribute with identifying the most at-risk individuals in our population.

---

## Solution Planning

* Classification Approach
* Main evaluation metric
  * Recall
* Data Source:
  * [2015 Behavioral Risk Factor Surveillance System Survei in Diabetes](https://www.kaggle.com/datasets/alexteboul/diabetes-health-indicators-dataset?select=diabetes_binary_health_indicators_BRFSS2015.csv)
* Developing Stack:
  - Jupyter with python3 kernel
  - AWS Cloud (s3 + Sagemaker)

### Project Deliverables: 
* An experimantation study to define the better techniques to handle our problem
* A model trained in production using all discoveries from the previous stage

---

## Model Deploy Architecture

Overall our implementation consists in the use of 2 main AWS tools: 
* S3 buckets
* SageMaker

As shown in the diagram below, he raw database was saved in a S3 bucket in the **/data/raw/** directory to be consumed via a jupyter notebook created in SageMaker.

![ Diagram of Architecture used to train and deply the winner model .](https://i.imgur.com/KPjyayc.jpeg)

Our solution is based on the deploy of a pipeline wich consists in some simple stages:

* Tranformation and normalization of raw data
* Split data into train and test
* Creating a linear learner solution
* Save the train results into S3 bucket
* Creating and endpoint to retrieve training dat and run the predction tasks
* Deletetion endpoint (enviroment clean up task)


### Amazon S3

Was created a single *bucket* to this deploy pipeline, with some subfolders, as showed in the image below:

* data
  * Raw - Raw input data
  * Train - Contains dataset after tranformer pipeline
  * Test - Unseen data

Due to the sensitivity of the stored data, it was decided to create the *bucket* with automatic encryption of the stored data, as shown in the following figure.

### Amazon SageMaker

Was created a notebook for the *pipeline* run based on the instance default specifications:

* Machine: *ml.m4.xlarge*
  * 4 vCPUs
  * 16GiB RAM

Was created a notebook to run the pipeline. Also was deployed an endpoint to train the model with new and unseen data.

More details avaiable in the "Architecture folder" in this repository

## Some Results

#### Model Validation in Test Set (Unseen data) Outputs

![ Final results](https://i.imgur.com/DMxzwyK.png)


All models apllied showed good results. This metrics, derived from test set, are not far from the ones spotted in training set. The models are doing a decent job in generalizating the classification. 
<br/>

Our elected model is ***SVM with RBF kernel, with hyperparameters C = 0.01*** as already seen in training validation, with best Recall (0.805489), ROC_AUC, Accuracy and F1-Score metrics.

---

#### For future work, we plan:
- Establish thresholds for the discovery of pre-diabetic patients based on probabilities. 
- Develop the least viable model based on the most relevant features, making data acquisition feasible and less expensive.
- Develop data ingestion with DATASUS for automatically input new local data. 
- Develop a unsupervised machine learning model to reduce annotated data dependency.
