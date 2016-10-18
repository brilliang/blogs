---
layout: researcher
title: SOURCE CODE READING, SPARK, LogisticRegression
tags: source code reading,spark,logisticregression
---

# SPARK, Logistic Regression
(version 2.0.0)

With spark, it would be very easy to build up the data pipeline of data preparing and modeling.

If you want to use spark to train a Logistic Regression model, just build a dataframe with two columns named 'label' and 'features'. the 'label' column is Double type, and the 'feature' column is Vector type with double value elements.

It is recommended to go through the official document of [Spark Machine Learning Library (MLlib) Guide](http://spark.apache.org/docs/latest/ml-guide.html).

Now I am going to record something I get from reading the source code of org.apache.spark.ml.classification.LogisticRegression

## parameters

Unlike [liblinear](http://www.csie.ntu.edu.tw/~cjlin/liblinear/), which applies 'trust region method' and involves more parameter setting, Spark Logistic Regression uses the L-BFGS as the optimisation method and needs simple and straightforward parameter setting.

###regularization

```scala
        val regParamL1 = $(elasticNetParam) * $(regParam)
        val regParamL2 = (1.0 - $(elasticNetParam)) * $(regParam)
```

Inside Spark, the regularization parameters are set using the code above. Therefore you have to set the next two parameters to control the regularization.

```scala
setRegParam
setElasticNetParam
```

### iterate
```scala
setMaxIter
setTol
```
If the data is convergence, then smaller convergence tolerance needs bigger number of iterations.

###intercept

```scala
setFitIntercept
```
if you set it false, it means the intercept will be set as zero:

```scala
		if ($(fitIntercept)) {
          (Vectors.dense(rawCoefficients.dropRight(1)).compressed, rawCoefficients.last,
            arrayBuilder.result())
        } else {
          (Vectors.dense(rawCoefficients).compressed, 0.0, arrayBuilder.result())
        }
```
The code above returns a tunple of (coefficients, intercept, history info).




## train

LogisticRegression extends from Predictor, which extends from Estimator.  The most important interface function of Estimator is fit(). Estimator overrides and implements it as follow:

```scala
  override def fit(dataset: Dataset[_]): M = {
    // This handles a few items such as schema validation.
    // Developers only need to implement train().
    transformSchema(dataset.schema, logging = true)
    copyValues(train(dataset).setParent(this))
  }
```


So, I am going to explain train() of LogisticRegression step by step:

* select 3 columns of 'lable', 'instance weight' and 'features' to construct a RDD[Instance] and persist it only when origin dataset's storageLevel is NONE

```scala
val handlePersistence = dataset.rdd.getStorageLevel == StorageLevel.NONE
...
if (handlePersistence) instances.persist(StorageLevel.MEMORY_AND_DISK)
```

Hence, the speed of training will be **faster** if not to persist the original dataset. 

* use two summarizers to get statistics of samples's label and features. and then check whether it is a binary classification problem and if there is anything wrong about the labels, e.g. all samples have the only one label. It is very important to check logs here when you get some strange train result.

```scala
val (summarizer, labelSummarizer) = {
...
}
```


* get regularization parameters
* construct LogisticCostFun, BreezeLBFGS or BreezeOWLQN (if regParamL1 != 0) as optimizer, initial coefficients and intercept

> For binary logistic regression, when we initialize the coefficients as zeros, it will converge faster if we initialize the intercept such that it follows the distribution of the labels.

* optimise for the solution:

```scala
val states = optimizer.iterations(new CachedDiffFunction(costFun),
          initialCoefficientsWithIntercept.asBreeze.toDenseVector)
```

* finally, check the *state* of each optimise iteration. when iteration ends, log warning if it is not actually converged. extract (coefficients, intercept, history info).
* constrcut LogisticRegressionModel using the optimise iteration result. 
* construct BinaryLogisticRegressionTrainingSummary by applying the new model on training data; this summary contains ROC, AUC, precision, and recall rate, etc.

Next article I am going to talk about the optimiser used in Spark Logistic Regression.