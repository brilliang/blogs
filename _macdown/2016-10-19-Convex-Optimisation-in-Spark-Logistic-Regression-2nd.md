# Optimizer in Spark Logistic Regression (Souce Code Analysis)

The current version of spark we are talking about is 2.0.0


[breeze](https://github.com/scalanlp/breeze) provides the convex optimisation methods for Spark. in ``` org.apache.spark.ml.classification.LogisticRegression``` we can find breeze optimizer is used like following:

* construct the optimizer depending on regularization paremeter

```scala
val optimizer = if ($(elasticNetParam) == 0.0 || $(regParam) == 0.0) {
          new BreezeLBFGS[BDV[Double]]($(maxIter), 10, $(tol))
        } else {
          ...
          new BreezeOWLQN[Int, BDV[Double]]($(maxIter), 10, regParamL1Fun, $(tol))
        }
```

* set objective function and initial coefficients, then start iteration

```scala
val states = optimizer.iterations(new CachedDiffFunction(costFun),
          initialCoefficientsWithIntercept.asBreeze.toDenseVector)
```

* iterate:

```scala
while (states.hasNext) {
          state = states.next()
          ...
        }
```

* finally, when iteration stops, check the states and extract *coefficients* and *intercept* to construct the 

```scala 
val model = copyValues(new LogisticRegressionModel(uid, coefficients, intercept)) 
```

# Objective Function
if we model the LR like this: $p(y=1|x, w) = \frac{1}{1 + e^{-w^Tx}}$, so that $p(y=0|x, w) = 1 - \frac{1}{1 + e^{-w^Tx}}$

likelihood function is $L(w) = \prod [p(y=1|x, w)^y*p(y=0|x, w)^{1-y}]$.

If we note $\sigma(x) = \frac{1}{1+e^{-x}}$, and make use of the property of $\sigma(x)'=\sigma(x)(1-\sigma(x))$, it would be easy to derive:

$\frac{\partial ln(L(w))}{\partial w} = \sum[(y-\sigma(x))x]$

The calculation of the objective funciont is implemented inside ```LogisticCostFun```. 

A ```treeAggregate``` function is invoked against all samples in the training data, which is called ```instances``` in the code, to get the objective function value and its gradient. It takes the advantage of spark and makes a <span style="color:red;font-size:2em;">distributed</span> calculation.

I believe the ```standardization``` makes the code much more complex to read and maintain. I think it is about feature processing and should not be involved here inside optimization process.


# Optimizer

In this paragraph I am going to talk about the detail for ``` BreezeLBFGS ``` and ``` BreezeOWLQN ```. Mathematical theory are discussed in the last post.

Chain of class extending is this:

```scala
OWLQN extends LBFGS extends FirstOrderMinimizer extends Minimizer
```

## trait Minimizer

```scala
trait Minimizer[T,-F] {
  def minimize(f: F, initial: T): T
}
```

## class FirstOrderMinimizer
It defines a type ```History``` to record last iteration's info: $x_{k-1}, g_{k-1}, H_{k-1}$, etc. A case class ```State``` to record current itertation's info: $x_k, g_k, H_k$, etc.

Inside each *iteration*, the logic is exactly what we learnt in last post:

* get the direction and step length by last iteration's state

```scala
        val dir = chooseDescentDirection(state, adjustedFun)
        val stepSize = determineStepSize(state, adjustedFun, dir)
        logger.info(f"Step Size: $stepSize%.4g")
        val x = takeStep(state,dir,stepSize)
```

* take step and get values at current point: $x_k, g_k, H_k$

```scala
        val x = takeStep(state,dir,stepSize)
        val (value,grad) = calculateObjective(adjustedFun, x, state.history)
        val (adjValue,adjGrad) = adjust(x,grad,value)
        val oneOffImprovement = (state.adjustedValue - adjValue)/(state.adjustedValue.abs max adjValue.abs max 1E-6 * state.initialAdjVal.abs)
        logger.info(f"Val and Grad Norm: $adjValue%.6g (rel: $oneOffImprovement%.3g) ${norm(adjGrad)}%.6g")
        val history = updateHistory(x,grad,value, adjustedFun, state)
        val newAverage = updateFValWindow(state, adjValue)
```

by default, ```adjust()``` function changes nothing.

* package current values into case class ```State```, which will be used in next iteration

```scala
        var s = State(x,value,grad,adjValue,adjGrad,state.iter + 1, state.initialAdjVal, history, newAverage, 0)
        val improvementFailure = (state.fVals.length >= minImprovementWindow && state.fVals.nonEmpty && state.fVals.last > state.fVals.head * (1-improvementTol))
        if(improvementFailure)
          s = s.copy(fVals = IndexedSeq.empty, numImprovementFailures = state.numImprovementFailures + 1)
        s
```

## class LBFGS
It specified the type ```History``` inside ```FirstOrderMinimizer```:

```scala
case class ApproximateInverseHessian[T]
```

The 2 arrays of $s_k$ and $y_k$ are stored & updated like this:

```scala
 def updated(step: T, gradDelta: T) = {
      val memStep = (step +: this.memStep) take m
      val memGradDelta = (gradDelta +: this.memGradDelta) take m

      new ApproximateInverseHessian(m, memStep,memGradDelta)
}
```

the key step of L-BFGS algorithm, $p_{k-1} = -H_{k-1}^{-1}g_{k-1}$, is implemented inside function ``` def *(grad: T) ```:

* M3 is used to scale the L-BFGS method:

```scala
     val diag = if(historyLength > 0) {
       val prevStep = memStep.head
       val prevGradStep = memGradDelta.head
       val sy = prevStep dot prevGradStep
       val yy = prevGradStep dot prevGradStep
       if(sy < 0 || sy.isNaN) throw new NaNHistory
       sy/yy
     } else {
       1.0
     }
```

* the main part of L-BFGS method, which is almost the same as algorithm flowchart in last post

```scala
     val dir = space.copy(grad)
     val as = new Array[Double](m)
     val rho = new Array[Double](m)

     for(i <- 0 until historyLength) {
       rho(i) = (memStep(i) dot memGradDelta(i))
       as(i) = (memStep(i) dot dir)/rho(i)
       if(as(i).isNaN) {
         throw new NaNHistory
       }
       axpy(-as(i), memGradDelta(i), dir)
     }

     dir *= diag

     for(i <- (historyLength - 1) to 0 by (-1)) {
       val beta = (memGradDelta(i) dot dir)/rho(i)
       axpy(as(i) - beta, memStep(i), dir)
     }

     dir *= -1.0
     dir
```

It is important for us to pay attention here, that all these calculation is done inside the driver machine, <span style="color:red;font-size:3em;">NOT</span> distributed. 

Actually, it is already inside the breeze package, which is invoked by spark. No distribute infrastructure breeze can use here.

```LineSearch``` is used to determine the step length.

## class OWLQN
the key part is to add in the regularization stuff to the gradient, which is implemented in function ``` override protected def adjust ```

```scala 
case 0.0 => {
  val delta_+ = v + l1regValue
  val delta_- = v - l1regValue
  if (delta_- > 0) delta_- else if (delta_+ < 0) delta_+ else 0.0
}
case _ => v + math.signum(xv) * l1regValue
```

```v``` is the un-regularized part's gradient, ```l1regValue``` is the paremeter of L1 reg. If standardization is applied, it is possible that l1regValue not always equals to 1.0

Using the adjusted gradient to calculate the Descent Direction, there is still something should be correct:

```scala
val descentDir = super.chooseDescentDirection(state.copy(grad = state.adjustedGradient), fn)

// The original paper requires that the descent direction be corrected to be
// in the same directional (within the same hypercube) as the adjusted gradient for proof.
// Although this doesn't seem to affect the outcome that much in most of cases, there are some cases
// where the algorithm won't converge (confirmed with the author, Galen Andrew).
val correctedDir = space.zipMapValues.map(descentDir, state.adjustedGradient, { case (d, g) => if (d * g < 0) d else 0.0 })
```