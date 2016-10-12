---
layout: researcher
title: SOURCE CODE READING, SPARK, LogisticRegression
tags: source code reading,spark,logisticregression
---

# SPARK, LogisticRegression
(version 2.0.0)

With spark, it would be very easy to build up the data pipeline of data preparing and modeling.

If you want to use spark to train a Logistic Regression model, just build a dataframe with two columns named 'label' and 'features'. the 'label' column is Double type, and the 'feature' column is Vector type with double value elements.

It is recommended to go through the official document of [Spark Machine Learning Library (MLlib) Guide](http://spark.apache.org/docs/latest/ml-guide.html).

Now I am going to record something I get from reading the source code of org.apache.spark.ml.classification.LogisticRegression

## parameters

the parameters of LogisticRegression are very straightforward, because the logistic regression model itself is very straightforward.

---

```scala
setRegParam
```
This function sets the regularization parameter shared by L1 and L2. you can use

```scala
setElasticNetParam
```
to how to use the regularization parameter. eventually, the regularization parameter is set like this:

```scala
        val regParamL1 = $(elasticNetParam) * $(regParam)
        val regParamL2 = (1.0 - $(elasticNetParam)) * $(regParam)
```

___

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
in the code above, the 2nd value of is the intercept.

___


```scala
setMaxIter
setTol
```
If the data is convergence, then smaller convergence tolerance needs bigger number of iterations.

___

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
So next I am going to focus on train() of LogisticRegression.

* select 3 columns of 'lable', 'instance weight' and 'features' to construct a RDD[Instance] and persist it only when origin dataset's storageLevel is NONE

```scala
val handlePersistence = dataset.rdd.getStorageLevel == StorageLevel.NONE
...
if (handlePersistence) instances.persist(StorageLevel.MEMORY_AND_DISK)
```

Hence, the speed of training will be faster if not to persist the original dataset.

* use two summarizers to get statistics of samples's label and features. and then check whether it is a binary classification problem and if there is anything wrong about the labels, e.g. all samples have the only one label.

NOTE: it is very necessary to check those warning log.

* get regularization parameters
* construct LogisticCostFun, BreezeLBFGS or BreezeOWLQN as optimizer, initial coefficients and intercept

> For binary logistic regression, when we initialize the coefficients as zeros, it will converge faster if we initialize the intercept such that it follows the distribution of the labels.

* optimise for the solution:

```scala
val states = optimizer.iterations(new CachedDiffFunction(costFun),
          initialCoefficientsWithIntercept.asBreeze.toDenseVector)
```

