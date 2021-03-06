
# Bayesian Data Analysis

## Model Building

## Inference

### Basic Quantities {-}

The mechanics of Bayesian inference follow directly from Bayes's rule.
To fix notation, let $y$ represent observed quantities such as data
and let $\theta$ represent unknown quantities such as parameters and
future observations.  Both $y$ and $\theta$ will be modeled as random.
Let $x$ represent known, but unmodeled quantities such as constants,
hyperparameters, and predictors.

### Probability Functions {-}

The probability function $p(y,\theta)$ is the joint probability
function of the data $y$ and parameters $\theta$.  The constants and
predictors $x$ are implicitly understood as being part of the
conditioning.  The conditional probability function $p(y|\theta)$ of
the data $y$ given parameters $\theta$ and constants $x$ is called the
sampling probability function; it is also called the likelihood
function when viewed as a function of $\theta$ for fixed $y$ and $x$.

The probability function $p(\theta)$ over the parameters given the
constants $x$ is called the prior because it characterizes the probability
of the parameters before any data are observed.  The conditional
probability function $p(\theta|y)$ is called the posterior because
it characterizes the probability of parameters given observed data $y$
and constants $x$.

### Bayes's Rule {-}

The technical apparatus of Bayesian inference hinges on the following
chain of equations, known in various forms as Bayes's rule (where
again, the constants $x$ are implicit).

$$
\begin{array}{rcll}
p(\theta|y)  & =  & \displaystyle \frac{p(\theta,y)}{p(y)}
&  \ \ \ \ \ \mbox{ [definition of  conditional probability]}
\\[16pt]
& = & \displaystyle \frac{p(y|\theta) \, p(\theta)}{p(y)}
& \ \ \ \ \ \mbox{ [chain rule]}
\\[16pt]
& = & \displaystyle \frac{p(y|\theta) \, p(\theta)}
                        {\int_{\Theta} p(y,\theta) \, d\theta}
& \ \ \ \ \ \mbox{ [law of total probability]}
\\[16pt]
& = & \displaystyle \frac{p(y|\theta) \, p(\theta)}
                        {\int_{\Theta} p(y|\theta) \, p(\theta) \, d\theta}
& \ \ \ \ \ \mbox{ [chain rule]}
\\[16pt]
& \propto & \displaystyle p(y|\theta) \, p(\theta)
\end{array}
$$

Bayes's rule "inverts" the probability of the posterior
$p(\theta|y)$, expressing it solely in terms of the likelihood
$p(y|\theta)$ and prior $p(\theta)$ (again, with constants and
predictors $x$ implicit).  The last step is important for Stan, which
only requires probability functions to be characterized up to a
constant multiplier.

### Predictive Inference {-}

The uncertainty in the estimation of parameters $\theta$ from the data
$y$ (given the model) is characterized by the posterior $p(\theta|y)$.
The posterior is thus crucial for Bayesian predictive inference.

If $\tilde{y}$ is taken to represent new, perhaps as yet unknown,
observations, along with corresponding constants and predictors
$\tilde{x}$, then the posterior predictive probability function is
given by

$$
p(\tilde{y}|y)
= \int_{\Theta} p(\tilde{y}|\theta)
                \, p(\theta|y) \, d\theta.
$$
Here, both the original constants and predictors $x$ and the new
constants and predictors $\tilde{x}$ are implicit.  Like the posterior
itself, predictive inference is characterized probabilistically.
Rather than using a point estimate of the parameters $\theta$,
predictions are made based on averaging the predictions over a range
of $\theta$ weighted by the posterior probability $p(\theta|y)$ of
$\theta$ given data $y$ (and constants $x$).

The posterior may also be used to estimate event probabilities.  For
instance, the probability that a parameter $\theta_k$ is greater than
zero is characterized probabilistically by

$$
\mbox{Pr}(\theta_k > 0)
= \int_{\Theta} \mbox{I}(\theta_k > 0) \, p(\theta|y) \, d\theta.
$$

The indicator function, $\mbox{I}(\phi)$, evaluates to one if the
proposition $\phi$ is true and evaluates to zero otherwise.

Comparisons involving future observables may be carried out in
the same way.  For example, the probability that $\tilde{y}_n >
\tilde{y}_{n'}$ can be characterized using the posterior predictive
probability function as
$$
\mbox{Pr}(\tilde{y}_n > \tilde{y}_{n'})
= \int_{\Theta} \int_{Y} \mbox{I}(\tilde{y}_n > \tilde{y}_{n'}) \,
p(\tilde{y}|\theta) p(\theta|y) \, d\tilde{y} \, d\theta.
$$

## Model Checking and Evaluation

### Fake-Data Simulation {-}

### Posterior Predictive Checking {-}

After the parameters are fit to data, they can be used to simulate a
new data set by running the model inferences in the forward
direction.  These replicated data sets can then be compared to the
original data either visually or statistically to assess model fit
[@GelmanEtAl:2013, Chapter 6].

In Stan, posterior simulations can be generated in two ways.  The
first approach is to treat the predicted variables as parameters and
then define their distributions in the model block.  The second
approach, which also works for discrete variables, is to generate
replicated data using random-number generators in the generated
quantities block.

<!--
### Exploratory Data Analysis as Model Checking {-}

### Cross Validation {-}

## Model Expansion

## Decision Analysis
-->
