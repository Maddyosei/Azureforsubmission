# Optimizing an ML Pipeline in Azure

## Overview
This project is part of the Udacity Azure ML Nanodegree.
In this project, we build and optimize an Azure ML pipeline using the Python SDK and a provided Scikit-learn model.
This model is then compared to an Azure AutoML run.

## Summary
**In 1-2 sentences, explain the problem statement: e.g "This dataset contains data about... we seek to predict..."**
This dataset contains data about individuals for the marketing sector of a bank. The datasets contains personal data of individuals who have had contact with the bank's marketing campaign. We seek to predict whether the client will subscribe to a bank term deposit.

**In 1-2 sentences, explain the solution: e.g. "The best performing model was a ..."**
The best performing model was the HyperDrive model. This model was derived from a Scikit-learn pipeline and had an accuracy of 0.91760. In contrast, the AutoML model where the VotingEnsemble algorithm was used had an accuracy of 0.91618.

## Scikit-learn Pipeline
**Explain the pipeline architecture, including data, hyperparameter tuning, and classification algorithm.**
The specified the parameter sampler used is as below:

ps = RandomParameterSampling(
    {
        '--C' : choice(0.001, 0.01, 0.1, 1, 10, 20, 50, 100, 200, 500, 1000),
        '--max_iter' : choice(50, 100, 200, 300)
    }
)

I chose discrete values for both parameters with choice, C and max_iter. 

C is the Regularization 
max_iter is the maximum number of iterations.

**What are the benefits of the parameter sampler you chose?**
RandomParameterSampling was chosen because it is fast and supports the early termination of low-performance runs. GridParameterSampling is expensive so if there is enough budget it could be used to exhaustively search over the search space. BayesianParameterSampling could also be used to explore the hyperparameter space.

**What are the benefits of the early stopping policy you chose?**
An early stopping policy is used to automatically terminate badly performing runs and therefore, improving computational efficiency. I chose the BanditPolicy and specified as below:

policy = BanditPolicy(evaluation_interval=2, slack_factor=0.1)

Parameters
Evaluation_interval: Although, this parameter is optional, it represents the frequency for applying the policy. Every time the training script logs the primary metric counts as one interval.

Slack_factor: This parameter details the amount of slack allowed with respect to the best performing training run and specifies it as a ratio.

A run will be terminated if it doesn't fall within the slack factor or slack amount of the evaluation metric with respect to the best performing run. The best performing runs will execute until they finish.

## AutoML
**In 1-2 sentences, describe the model and hyperparameters generated by AutoML.**
The following configuration for the AutoML run was used:

automl_config = AutoMLConfig(
    compute_target = compute_target,
    experiment_timeout_minutes=30,
    task='classification',
    primary_metric='accuracy',
    training_data=ds,
    label_column_name='y',
    n_cross_validations=3)

Parameter
experiment_timeout_minutes
This is an exit criterion and is used to for the experiment should continue to run in minutes. I used a minimum time of 30 minutes in order to avoid experiment time out failures.

task
This parameter defines the experiment type which in this case is classification.

primary_metric
The primary metric being used is accuracy.

n_cross_validations
This parameter sets how many cross validations to perform. I chose three folds as one cross-validation could result in overfit.

The best model selected by AutoML was a voting ensemble which provided an accuracy of ~91.8%. The model selected enabled penalty to be placed on the number of non-zero model coefficients using regularisation. A soft voting method was used in which the class probabilities of all models were averaged and the highest probablility was selected in order to make a prediction. 

## Pipeline comparison
**Compare the two models and their performance. What are the differences in accuracy? In architecture? If there was a difference, why do you think there was one?**
HyperDrive Model	
Accuracy	0.91760
AutoML Model	
Accuracy	0.916176

The difference in accuracy between the two models is very minimal and although the HyperDrive model performed better in terms of accuracy, the AutoML model is actually better because of its AUC_weighted metric and is a betterfit for the data available. If we were given more time to run the AutoML, the resulting model would certainly be much more better and the best thing is that AutoML would make all the necessary calculations. This is the difference with the Scikit-learn Logistic Regression pipeline, in which we have to make adjustments to achieve the final model.


## Future work
**What are some areas of improvement for future experiments? Why might these improvements help the model?**

Highly imbalanced data
Class imbalance is a common issue in machine learning. Imbalanced data will impact the model's accuracy because the majority class will overcome the results of the minority class. Accuracy is a difficult metric to use since it can be misleading with imbalanced data.

The different ways to deal with imbalanced data include using:

- A different algorithm
- A different metric; for example, AUC_weighted which is more fit for imbalanced data
- Random Under-Sampling of majority class
- Random Over-Sampling of minority class
- The imbalanced-learn package

I would improve n_cross_validations as another factor. As cross-validation is the process of taking many subsets of the full training data and training a model on each subset, the higher the number of cross validations is, the higher the accuracy achieved is. However, a high number also raises computation time and therefore costs.

## Proof of cluster clean up
**If you did not delete your compute cluster in the code, please complete this section. Otherwise, delete this section.**
