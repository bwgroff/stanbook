
# Introduction

## Bayesian Data Analysis

### Bayesian inference and Stan {-}

Bayesian inference is a statistical power tool.  It enables you to use prior knowledge and observed data to build a probability model. Your data and unknowns are embedded in this probability model and you construct a "posterior distribution". 

The prior distribution represents the knowledge that you have about the world from past experiments, research or simply scientific truths. The observed data is represented by the ubiquitous likelihood function. We then employ Bayes' rule to combine the prior and likelihood and approximate the posterior distribution. Once you have this posterior you can make inferences and predictions about everything.

Stan is a platform for statistical modeling and high-performance
statistical computation. Given the prior and the data or likelihood function, Stan uses computational methods to approximate the posterior distribution. 

A big challenge for applied Bayesian inference is computation and scalability. Converting the mathematical expression of the posterior distribution into specific inferences or predictions (e.g the probability that some coefficient of interest is postive or the 90% predictive interval for some future outcome) can be difficult. For even moderately large or complex problems these quantities are expressed in terms of high-dimensional integrals with no closed-form expressions.


Over the past fifty years, a series of advances in
computational statistics have allowed these integrals to be computed
using approximations and simulations.  The simulations use random
numbers and are called "Monte Carlo methods," named after the city in
Europe that is famous for its gambling casinos.  These methods were
originally developed in the 1940s for aiding in large computations for
the military, and in the 1980s it became clear how to apply them for
general problems in Bayesian inference.

So: since the 1970s--1980s, methods have been developed to perform
approximate computations for Bayesian inferences that would otherwise
REQUIRE INTRACTABLE INTERVALS.  These approximations needed to be
developed one model at a time.  In the 1990s--2000s, the WinBugs
software was developed, which allowed automatic computation for a
large class of Bayesian models.  WinBugs (and its successors, OpenBugs
and Jags) can be slow, and starting in 2011 we developed Stan, which
uses more efficient computations (Hamilton Monte Carlo, the no-U-turn
sampler, and algorithmic autodifferentiation) so that automatic
Bayesian computation can be applied to larger and more complex
problems.

Where we stand now is that, for a fairly broad class of models and
data of moderate size, we can transparently program our Bayesian
models in Stan and perform inference automatically.  This represent
the culmination of decades of work in computational statistics, along
with corresponding decades of experience fitting and understanding
these models.  The challenge is not just fitting the model; it is
also deciding what models to fit.

Future work, by ourselves and others, will increase the speed and
scalability of Stan in various ways, including more seamless
implantation of parallel processing.

### Appealing features of Bayesian inference {-}

Here are some reasons we like to use Bayesian methods:

* Integration of data and prior information

* Quantification of uncertainty, including probabilistic predictions

* Ability to pipe inference directly into decision analysis

* Ability to handle uncertainty in large numbers of parameters

It is said that the most important aspect of a statistical analysis is
not what you do with the data, it's what data you use.  A key
advantage of modern statistical methods (including Bayesian methods
but also various non-Bayesian or semi-Bayesian approaches in machine
learning) is that they allow you to incorporate different sorts of
information into your analysis.


### Some things that Bayesian inference and Stan can't do {-}

Bayesian inference does not solve all statistical problems, though.
One important class of problems where it is not currently possible to
perform fully Bayesian inference is nonlinear classification and
optimization with large datasets: familiar examples include language
processing, speech and image recognition, and those computer programs
that play Go or ping-pong.  These problems are often attacked using
Bayesian models, but the inferences used are typically only rough
approximations to the mathematical Bayesian posterior distribution:
the required calculations are simply too involved, and the posterior
distributions tend to be multimodal and essentially impossible to
fully navigate using any existing algorithm.  Stan is not the best
tool for these problems.  We do think, however, that Stan is the best
tool for fitting continuous-parameter models that arise in many
application areas, including astronomy, ecology, economic forecasting,
earth science, insurance, public health, survey sampling, to just name
a few.  A wide-ranging set of case studies is available on the Stan
website at: http://mc-stan.org/users/documentation/case-studies
and in the conference proceedings from every StanCon:
https://github.com/stan-dev/stancon_talks.


## Hello World

_Bayesian inference_ is a framework for estimating parameters and
constructing predictions given probability models and data.  _Bayesian
data analysis_ is the larger process of building, fitting, and
checking probability models.  _Stan_ is an open-source computer
program for Bayesian inference and simulation.  Stan can be run from
R, Python, Julia, or other scientific/statistical software.  In the
examples in this book, we set up data and run Stan from R, but our
focus is on Stan, not the R code.

### Getting started {-}

Go to the Stan webpage (http://mc-stan.org) and navigate to users and
interfaces.  Instructions for setting up Stan for use within R are
here: http://mc-stan.org/users/interfaces/rstan.html.  Follow all the
steps on that page.

### A Stan program for simple linear regression {-}

Our "Hello World" example is a linear regression,
$$y_i=a+bx_i +\mbox{error}_i, \mbox{ for } i=1,\dots,N,$$ with errors
independent and normally distributed with mean 0 and standard
deviation $\sigma$.

Just one little thing to be careful of: in statistical notation, the
normal distribution with mean $\mu$ and standard deviation $\sigma$ is
written as $N(\mu,\sigma^2)$.  In Stan, we write it as `normal(mu,
sigma)` (_not_ `sigma^2`).
Also, note that when we are talking about the elements of a Stan program
we use `typewriter font` and when we are talking about the elements
of a statistical model we use Greek letters and math notation.

Here is the full model written in Stan:


```
data {
  int<lower=0> N;
  vector[N] x;
  vector[N] y;
}
parameters {
  real a;
  real b;
  real<lower=0> sigma;
}
model {
  y ~ normal(a + b * x, sigma);
}
```

The program has three named blocks: the data block which
declares the inputs to the model, the parameters block which
declares the parameters to be estimated, and the model block
which presents the statistical model.
In this program the variables `y` and `x` are vectors of length
`N` which come as input data and the variables `a`, `b`, and `sigma`
are scalar values which are estimated jointly by the model.

In order to run the program, you must provide inputs for all
variables declared in the data block. 
Therefore we need to simulate some data
for the vector length `N` and a vector of values for both
`x` and `y` using known values for model parameters
`a`, `b`, and `sigma`.
We can do this using Stan or directly in R:


```r
N <- 100
a <- 10
b <- 4
sigma <- 5
x <- runif(N, 0, 10)
y <- rnorm(N, a + b*x, sigma)
hello_data <- list(N=N, x=x, y=y)
```

The last statement creates a single R data structure `hello_data`
which contains the names and values of all inputs.
This object is passed into the Stan algorithm which
fits the the model with the simulated data.
In this example we use Stan's default sampling algorithm:


```r
fit <- stan("stan/simplest-regression.stan", data=hello_data)
```


To evaluate the fit,
we can print the estimates of the model parameters
`a`, `b`, and `sigma`:


```r
print(fit, pars=c("a","b","sigma"), probs=c(0.025,0.975))
Inference for Stan model: simplest-regression.
4 chains, each with iter=2000; warmup=1000; thin=1; 
post-warmup draws per chain=1000, total post-warmup draws=4000.

      mean se_mean   sd 2.5%  98% n_eff Rhat
a     11.4    0.02 0.97  9.4 13.2  1955    1
b      3.8    0.00 0.16  3.5  4.1  1921    1
sigma  4.9    0.01 0.36  4.3  5.7  2275    1

Samples were drawn using NUTS(diag_e) at Wed Feb  6 21:44:39 2019.
For each parameter, n_eff is a crude measure of effective sample size,
and Rhat is the potential scale reduction factor on split chains (at 
convergence, Rhat=1).
```

To simulate the data `y` we picked `10`, `4` and `5` as
the values of `a`, `b`, and `sigma`, respectively.
As the true values are within the inferential ranges,
we have reason to trust our computation.
In the [workflow chapter](#workflow-in-action) we show more ways to
assess parameter recovery.

## Bayesian workflow for model checking and model improvement {-}

In Bayesian inference we make a sort of deal with the devil: we commit
to a strong model, and from this we get strong inferences.  But, as
the saying goes, with great power comes great responsibility.  We need
to vigilantly _check_ the fit of our models, following this up with
model _improvement_.  As a result, Bayesian workflow does not involve
fitting just one model to data.  We typically fit multiple models,
including some models that we know are too simple (to get a sense of
what is lost by not including certain features in our analysis) and
others that we suspect are too complex (to get a sense of the
boundaries of what we can learn given the resolution of the our
available data).

_Model checking_ consists of the following steps:

1. _Simulate fake data._ Specify sizes and values for all predictors
in a model and choose a set of values for all model parameters
and generate the corresponding data values $y$.

2. _Fit the model._ Express the model in Stan, pass the simulated data
into the program, and estimate the parameters.

3. _Evaluate the fit._ Compare the estimated parameters (or, more
fully, the posterior distribution of the parameters) to their true
values, which in this simulated-data scenario are known.

These are exactly the steps that we just carried out in our
"Hello World" example in the previous section.

_Model improvement_ follows the idealized steps listed on the
first page of _Bayesian Data Analysis_:

1. _Set up a full probability model._  Define a joint probability
distribution for all observable and unobservable quantities in a
problem. The model should be consistent with knowledge about the
underlying scientific problem and the data collection process.

2. _Condition on observed data._   Calculate and interpret the
appropriate posterior distribution -- the conditional probability
distribution of the unobserved quantities of ultimate interest, given
the observed data.

3. _Evaluate the fit of the model and the implications of the
resulting posterior distribution._  How well does the model fit the
data, are the substantive conclusions reasonable, and how sensitive
are the results to the modeling assumptions? In response,
one can alter or expand the model and repeat.

This workflow helps us to think generatively about
the underlying scientific problem expressed by the model.
While our models are imperfect compared to
the true data generating process, the above procedure
provides a clear specification of the
data generating process encoded by our model.

### Idealized plan for Bayesian case studies {-}

The [workflow chapter](#workflow-in-action) of this book
includes several case studies to give some
sense of Bayesian modeling on some fairly simple problems.
Many more case studies are available on the Stan website at: 
http://mc-stan.org/users/documentation/case-studies
and in the StanCon conference proceedings at:
https://github.com/stan-dev/stancon_talks.

Presentation of these examples vary,
but the paradigmatic format for a case study would follow these steps:

1.  Applied example to give context

2.  Discussion of reasonable parameter values and fake-data simulation
in Stan or in your scientific computing environment of choice

3.  Graph of fake data

4.  Stan program

5.  Fit fake data in Stan; discuss convergence, model diagnostics,
and parameter estimates and uncertainties

6.  Graph the fitted model along with the data

7.  Fit real data in Stan

8.  Graph the fit

9.  Model checking

10.  Directions for model expansion


## Writing a Stan program {-}

When you write a Stan program, you're
writing C++ code that gives instructions for computing an "objective
function."  In this book we will be using Stan for Bayesian inference,
and the objective function is interpreted as the logarithm of the
posterior density, up to an arbitrary constant.

A Stan program includes various blocks to declare data and parameters
and make transformations, but the heart of a Stan program, where it
computes the objective function, is in the model block.  The Stan
program above has the following model block:

```
model {
  y ~ normal(a + b * x, sigma);
}
```

In this case, `y` and `x` are vectors of length `N` that are
input to the Stan program as data and `a`, `b`, and `sigma`
are the parameters estimated jointly by the model.
The above code is mathematically (but not computationally)
equivalent to:
```
model {
  for (n in 1:N) {
    y[n] ~ normal(a + b * x[n], sigma);
  }
}
```
Each line inside the loop adds a term to the objective function with
the logarithm of the corresponding normal density; thus,
$$\log(\frac{1}{\sqrt{2\pi}\sigma}\exp(-\frac{1}{2}(\frac{y_n - (a +
bx_n)}{\sigma})^2)) = - \frac{1}{2}\log(2\pi) - \frac{1}{2}\log\sigma
- \frac{1}{2}(\frac{y_n - (a + bx_n)}{\sigma})^2$$
For most purposes,
we do not care about arbitrary multiplicative constants in the
posterior density or, equivalently, arbitrary additive constants in
the log-posterior density, so it does not matter if the
$-\frac{1}{2}\log(2\pi)$ term is present.  We _do_ need to include
$-\frac{1}{2}\log\sigma$, however, because $\sigma$ is a parameter in
the model and thus we cannot consider this term as constant.

For reasons that we shall discuss later, the above code is more
efficient in vectorized form (without the loop).

The relevant point of the above discussion is that the model block is
where the objective function is computing, with distributional
statements corresponding to increments to the objective function.  We
can make this explicit by rewriting the above model block as:
```
model {
  target += normal_lpdf(y | a + b * x, sigma);
}
```
or
```
model {
  for (n in 1:N) {
    target += y[n] ~ normal_lpdf(y[n] | a + b * x[n], sigma);
  }
}
```
Here, "target" is the objective function, "lpdf" stands for "log
probability density function," and the vertical bar is statistics
notation for conditioning: thus, we are adding to the objective
function the normal log density function of `y`, given mean `a+bx` and
standard deviation `sigma`.

Here is a slightly more elaborate version, in which we include
$\mbox{normal}(0,1)$ prior distributions for `a`, `b`, and `sigma`
(actually the prior for `sigma` is half-normal as this parameter has
been constrained to be positive).  We will talk more about priors in a
bit; for here, you can just think of these extra statements as
representing additional information that the parameters `a`, `b`, and
`sigma` are likely to be not too far from 0:
```
model {
  y ~ normal(a + b * x, sigma);
  a ~ normal(0, 1); b ~ normal(0, 1);
  sigma ~ normal(0, 1);
}
```
Or, equivalently:
```
model {
  target += normal_lpdf(y | a + b * x, sigma);
  target += normal_lpdf(a | 0, 1);
  target += normal_lpdf(b | 0, 1);
  target += normal_lpdf(sigma | 0, 1);
}
```
Every line in the model block with a tilde (~) corresponds to an
augmentation of the target, or objective function.

We can also include lines in the code that do _not_ augment the
objective function.  For example:
```
model {
  // expected value of y when x=2 y ~ normal(a + b * x, sigma);
  real a_shifted = a + 2 * b;
  a_shifted ~ normal(0, 1);
  b ~ normal(0, 1);
  sigma ~ normal(0, 1);
}
```
Here we wanted to assign a prior distribution not to the parameter `a`
but to the shifted parameter `a+2b`.  The above code is executed
directly, with an augmentation of "target" for every line with a
tilde.

### Interpreting the objective function as the log posterior density {-}

That is, the objective function is interpreted as being the log
posterior density of the parameters, plus some arbitrary constant.
Mathematically, if the objective function computed by Stan is
$g(\theta,y)$, then the implied posterior density is
$p(\theta|y)=\frac{1}{Z(y)}e^{g(\theta,y)}$, where $Z(y)=\int
g(\theta,y)d\theta$.  To perform inference using this posterior
distribution, is not necessary to actually evaluate the integral $Z$;
it is enough to be able to compute $g$.

### Stan's fitting algorithms {-}

As described above, a Stan program can be viewed as instructions for
computing an objective function.
When you run a Stan program, it performs optimization or approximation
or sampling of this objective function, using one of the algorithms
described below.


#### Option 1: Sampling {-}

Usually when we run Stan we use it to _sample_ from the posterior
distribution that is proportional to the exponential of the objective
function.  Currently available in Stan are two different sampling
algorithms: Hamiltonian Monte Carlo (HMC) and the no-U-turn sampler
(NUTS).  For background on HMC, see Chapter 12 of _Bayesian Data
Analysis_.  NUTS is an adaptive version of HMC
and is the algorithm that we use by default when fitting models in Stan.
For more information on NUTS 
see Chapter "MCMC Sampling" of the _Stan Reference Manual_.
When running NUTS (or HMC, or any other sampling algorithm),
the output of Stan is a list of posterior simulations.

We have already seen an example of this above:


```r
fit <- stan("stan/simplest-regression.stan", data=hello_data)
```

The output of the Stan run, saved in R as the object "fit", contains
posterior simulations and also some other information regarding the
settings of the fitting algorithm.  Here, for example, we access some
simulations:


```r
sims_as_matrix <- as.matrix(fit)
print(sims_as_matrix[1:5,])
          parameters
iterations  a   b sigma lp__
      [1,] 12 3.7   5.1 -207
      [2,] 12 3.8   4.8 -206
      [3,] 12 3.9   4.8 -207
      [4,] 11 3.8   4.6 -206
      [5,] 12 3.8   4.8 -206
sims_as_list <- extract(fit)
print(sims_as_list$a[1:5])
[1] 11 12 13 12 12
```

Each row of the above matrix is a different posterior simulation: we
see the parameters `a`, `b`, and `sigma`, along with the objective
function (or log-posterior density), "lp__".

#### Option 2: Optimization {-}

The simplest way to run Stan is on optimize setting; it then uses the
BFGS algorithm to find (or attempt to find) the maximum of the
objective function, and Stan returns the value of the parameters at
this estimated mode along with the computed value of the objective
function.  Here is an example:


```r
simplest <- stan_model("stan/simplest-regression.stan")
fit_2 <- optimizing(simplest, data=hello_data)
```


In this case we broke the computation into two steps, first compiling
and saving the compiled model into the R object "simplest" and then
performing the optimizer on this program.  And here is the result:


```
$par
    a     b sigma 
 11.4   3.8   4.8 

$value
[1] -207

$return_code
[1] 0
```

For this simple model, the values of the parameters at the optimum are
close to the posterior median estimates obtained from to the NUTS fit
earlier.  We also get the value of the objective function, which has
no direct interpretation in this case, and a return code which
represents the stability of the optimizer.  (A return code of 0 is
good news.)

Interpreting the objective function as the log posterior density, the
result of the optimizer is the posterior mode.  If the model has a
uniform prior distribution, then this is also the maximum likelihood
estimate.

In classical statistics, the maximum likelihood is accompanied by an
uncertainty estimate based on the curvature of the log-likelihood
function; see Sections 4.1 and 13.3 of _Bayesian Data Analysis_ for a
corresponding Bayesian interpretation based on the posterior mode.
There are times when this mode-based approximation makes
sense---notably when datasets are so large that full Bayesian
computation is too slow to be practical---but usually we fit our Stan
models using simulation, so that we get posterior uncertainties
directly, with no need for any normal approximation.

#### Option 3: Distributional approximations {-}

Stan also allows you to fit models using ADVI (autodifferentiated
variational inference).  As an approximation to the posterior
distribution, ADVI can work better than simple mode-based
approximations and it can be much faster for large problems, but
currently the way ADVI is implemented in Stan, there are concerns
about its convergence.  We have applied ADVI to a few hundred small
example models where we can compare to the full posterior
distribution: in many of these examples, ADVI works well, but in some
examples, ADVI gives apparent convergence but to the wrong answer that
is not a good approximation to the posterior.  Hence, when using ADVI,
it is important to do some fake-data simulation to check that the
algorithm can recover something close to the underlying model.  For
now, it is probably best to think of ADVI as an experimental tool that
you should only try for problems that are too large to solve using
regular NUTS.

### Iterations and steps {-}

Stan's fitting algorithms are iterative.  The algorithm starts with
some initial values for all the parameters (these can be
user-specified and fed into the call to Stan, or else Stan will
simulate them from a default distribution; (see
Chapter "MCMC Sampling", section "Initialization"
of the _Stan Reference Manual_)
and then, at each _iteration_, the parameter values are updated.
After enough iterations, the parameter values converge to an estimate
of the optimum (if Stan is being set to optimization) or to an
estimate of the center of the distribution (if Stan is being set to
distributional approximation) or the iterations will wander through
the posterior distribution (if Stan is being set to sample).

In the HMC and NUTS algorithms (and recall that NUTS is Stan's
default), each iteration includes some number of _steps_.  In each
step, Stan computes the gradient of the objective function, which is
used in the "leapfrog algoritm" to determine the direction in which
the algorithm moves (see Section 12.4 of _Bayesian Data Analysis_ and
the "MCMC Sampling" chapter of the _Stan Reference Manual_).
An iteration might have tens or hundreds of steps.
The information from each step is not saved by Stan; what is
returned is the value of the parameters at the end of each iteration.

As a user, you typically don't need to worry about steps or
iterations; all that is relevant is that Stan returns posterior
simulation draws of the parameters.  But when Stan has convergence
problems or when it takes a long time to fit a model, then it can be
helpful to understand what is happening under the hood, as often it
can be possible to reparameterize or reprogram the model to run more
efficiently.

### Adaptation and warmup {-}

The iterative algorithms have _tuning parameters_.  These are not
statistical parameters in the model to be estimated; rather, they are
settings that need to be adjusted for the algorithms to run
efficiently.  The sampling algorithms in Stan have _warmup_ and
_production_ stages.  In the warmup stage, the tuning parameters are
altered using various heuristics with the goal of efficient movement
during the production stages, whose iterations are saved.  In the
current default settings, Stan runs for 1000 warmup iterations and
1000 production iterations, but we would like to change these defaults
to become more adaptive, running fewer iterations if that is all that
is needed, and automatically running more when 2000 iterations are not
enough.

### Multiple chains, mixing, R-hat, effective sample size {-}

To have trust in the results of an iterative algorithm, it can be
helpful to run it from different starting points and check that the
different runs of the algorithm reach the same endpoint.  Each run of
the algorithm is called a "chain" because the algorithms can often be
described mathematically as Markov chains; hence we speak of _multiple
chains_.  By default, when run from R, Stan simulates 4 chains in
parallel, which is convenient on many laptop computers which have 4
processors.

For optimization or distributional approximation, we literally want
the different chains to converge to the same spot.  For sampling, we
want the different chains to trace out the same distribution, which we
measure by comparing the simulations from different chains to each
other.  For each parameter or quantity of interest saved, Stan reports
"R-hat," a numerical measure which is larger than 1 when chains have
not mixed well and becomes close to 1 when the chains have mixed.  We
typically run Stan for the default number of iterations and, if R-hat
is less than 1.1 for all parameters, we just use the simulations to
represent the posterior distribution.  If R-hat is greater than 1.1
for any parameters, the algorithm is slow to converge and we might run
it longer and then, if mixing remains poor, consider
reparameterization of the problem.

Stan also computes for each parameter, transformed parameter,
and generated quantity an "effective sample size" which represents
the number of uncorrelated draws in the sample.
For more information see section "Effective Sample Size"
in Chapter "Posterior Analysis" of the _Stan Reference Manual_.

### Blocks and declarations {-}

As the language is currently configured, a Stan program is divided
into _blocks_:

- Data.  Here you declare all the information that must be supplied
  for the program to run.  For example, in a regression model, the
  data would be x, y, and N.  Each data object in a Stan program must
  be given a type, such as int (integer), real, vector, matrix, etc.
  Explanations of all the types are in chapter on
  [matrices, vectors, and arrays](#matrices-vectors-and-arrays) of this book.

- Transformed data.  These are mathematical operations that are
  performed once when the Stan program is called.

- Parameters.  These are the unknown quantities that are estimated
  when the Stan program is run.  Parameters also need to be declared,
  and they need to be continuous.  You cannot have an integer-valued
  parameter.  See the [latent discrete parameters](#latent-discrete.chapter)
  chapter for further discussion of this point.

- Transformed parameters.  Computations that depend on parameters as
  well as data, thus must be computed in each step when Stan runs.

- Model.  This is the part of the Stan program where the objective
  function is computed.  As discussed above, at each step, Stan
  computes the objective function by starting at zero and then adding
  to it with each statement that includes "target +=" or "~".

- Generated quantities.  These are computed at the end of each
  iteration (not each step) and are saved.

### Vectors and arrays {-}

Here is an example of a Stan model with a vector parameter:


```
data {
  int<lower=0> N;
  int<lower=0> K;
  matrix[N,K] X;
  vector[N] y;
}
parameters {
  vector[K] b;
  real<lower=0> sigma;
}
model {
  y ~ normal(X*b, sigma);
}
```

To fit this model, we simply reconfigure the data by constructing the
matrix of predictors and then run from R:


```r
ones <- rep(1, N)
X <- cbind(ones, x)
K <- 2
data_3 <- list(N=N, K=K, X=X, y=y)
fit_3 <- stan("stan/vector-regression.stan", data=data_3)
```


Here is the result:


```
Inference for Stan model: vector-regression.
4 chains, each with iter=2000; warmup=1000; thin=1; 
post-warmup draws per chain=1000, total post-warmup draws=4000.

        mean se_mean   sd   2.5%    25%    50%    75%    98% n_eff Rhat
b[1]    11.4    0.02 0.99    9.4   10.7   11.3   12.0   13.3  1635    1
b[2]     3.8    0.00 0.16    3.5    3.7    3.8    3.9    4.1  1660    1
sigma    4.9    0.01 0.35    4.3    4.7    4.9    5.2    5.7  2507    1
lp__  -207.0    0.04 1.23 -210.2 -207.6 -206.7 -206.1 -205.6  1237    1

Samples were drawn using NUTS(diag_e) at Wed Feb  6 21:45:30 2019.
For each parameter, n_eff is a crude measure of effective sample size,
and Rhat is the potential scale reduction factor on split chains (at 
convergence, Rhat=1).
```

And here is what happens if we grab the matrix or list of posterior
simulations:


```r
sims_as_matrix <- as.matrix(fit_3)
print(sims_as_matrix[1:5,])
          parameters
iterations b[1] b[2] sigma lp__
      [1,]   12  3.7   4.5 -206
      [2,]   11  3.8   4.8 -206
      [3,]   11  3.9   5.1 -206
      [4,]   10  3.9   5.2 -207
      [5,]   12  3.8   5.2 -206
sims_as_list <- extract(fit_3)
print(sims_as_list$b[1:5,])
          
iterations [,1] [,2]
      [1,]   12  3.6
      [2,]   11  3.8
      [3,]   12  3.8
      [4,]   11  3.9
      [5,]   11  3.7
```

The parameter `b` is a vector, and so the posterior simulations are a
2-dimensional array.  Similarly, if `b` is a matrix or array of
vectors, the posterior simulations will become a 3-dimensional array,
and so on.  We shall encounter such parameter structures when fitting
hierarchical models.

## Software environment

### Working in R and Stan {-}

In this book, we will work with data and fit Stan models using the
Rstan package in R, as demonstrated above.  Our focus, though, is on
statistical modeling and writing Stan programs, so in most of the book
we suppress the R.  All the code is available (see below) so that you
can reproduce any of the computations we perform.

### Other Stan interfaces: Python, Julia, command line, and more {-}

Stan can also be run from the command line or from other statistical
environments such as Python and Julia.  The Stan website contains
a list of interfaces and installation instructions at
http://mc-stan.org/users/interfaces/index.html.

### Source code for this book and its programs, scripts, data {-}

You may be reading this book as an html or pdf file or even as a
printed document.  The underlying content lives in the GitHub repository
https://github.com/stan-dev/docs in the subdirectory
https://github.com/stan-dev/docs/tree/master/src/bayes-stats-stan.
Each chapter of the book is a "knitr file," which means
that it can be run in R, most conveniently using Rstudio.
You can also go to the raw knitr docs (again, most conveniently using the
editor in Rstudio): these are the files with .Rmd extensions.  These
documents have the text of the book (e.g., the words you are reading
now) along with R code for analysis and visualization,
some of which is hidden in this html or pdf document that
you are reading but is visible in the raw .Rmd documents.

The Stan programs used in this book are all in `.stan` files in
the directory https://github.com/stan-dev/docs/tree/master/src/bayes-stats-stan/stan.

## Bayes and Stan

Bayesian inference is a general approach for learning about unknown
parameters and making predictions about missing or latent data, given
observed data and a probability model.  For the theory and practice of
Bayesian inference, see our book _Bayesian Data Analysis_.  Or, for a
more introductory treatment, the book _Statistical Rethinking: A
Bayesian Course with Examples in R and Stan_ by Richard McElreath.  We
recommend you read those books along with this one.

Stan is a computer program that performs Bayesian inference on any
model with continuous parameters whose log-posterior density can be
computed (up to an arbitrary additive constant, as discussed above).

If you want to fit a Bayesian model _not_ using Stan, you can perform
computations analytically or by direct simulation for some simple
models (see the early chapters of _Bayesian Data Analysis_ for many
examples), or write your own customized simulation algorithm (see Part
III of _Bayesian Data Analysis_), or use some other general-purpose
software.  We like Stan, both for fitting Bayesian models in our
applied work, and for teaching the general principles of Bayesian
modeling.

### What _can't_ be fit in Stan? {-}

There are several sorts of models that can't be fit, or can't be fit
well, using Stan.  Other software has problems with such models too.

- Models with discrete parameters.  Stan can only fit
  continuous-parameter models.  If you have a model with discrete
  parameters, you have to fit it using some other approach.  Very
  simple discrete-parameter models can be fit using direct computation
  of the posterior distribution; we give two such examples in Section
  1.4 of _Bayesian Data Analysis_.  For more complicated examples, we
  can sometimes average over the discrete parameters analytically; see
  the [mixture modeling](#mixture-modeling.chapter) chapter
  of this book for general discussion and examples of
  finite mixture and hidden Markov models.  More generally, posterior
  distributions for models with discrete unknowns can be explored
  using various stochastic algorithms, but except in certain special
  cases these can be very slow to converge: the challenge is to
  perform a tour of a discrete space without simply going through all
  the possible values, which would result in a combinatorial
  explosion.  Thus, we sometimes say that Stan can't walk through
  discrete spaces, but no other algorithm can do so effectively
  either.

In any case, if you want to perform Bayesian inference for a model
with discrete unknowns, you need to either average over the discrete
variables as described in
the [latent discrete parameters](#latent-discrete.chapter) chapter,
or fit the model outside of Stan.  Either way, we recommend fake-data
simulation to check that the
fit can recover the parameter values under reasonable assumptions.

- Models with likelihoods or priors that contain uncomputable
  normalizing constants.  This comes up sometimes with spatial or
  network models. We won't discuss these further here; you can see
  Section 13.10 of _Bayesian Data Analysis_ for a brief general
  discussion of the problem.

- Models for which the NUTS algorithm mixes very slowly.  This can
  include continuous models with discrete aspects to their geometry,
  such as multimodal posterior distributions.  Sometimes we can get
  these models to run more smoothly by reparameterizing; see the
  [optimization](#optimization.chapter) chapter of this book
  for the important example of the use of the
  non-centered parameterization to avoid the "funnel problem" with
  hierarchical models.

- Models for which the objective function takes a long time to
  compute.  This includes some models with differential equations (see
  the [ordinary differential equations](#ode-solver.chapter)
  chapter) or even simple models, if the size of the data or the
  number of parameters is large enough.  There are various approaches
  to managing large and slow computations in Stan, including data
  subsetting, and posterior approximations, and parallel computing.
  For most of this book we focus on small and moderate-sized problems;
  we discuss big-data strategies in the [map-reduce](#map-reduce.chapter)
  chapter.


## More information about Bayesian data analysis and Stan

Here is a list of resources, roughly in order from most to least
formal:

- The Stan Reference Manual:
  http://mc-stan.org/users/documentation/index.html which has details
  on the algorithms and implementations used by Stan, along with all
  of Stan's functions.

- _Bayesian Data Analysis_ by Gelman, Carlin, Stern, Dunson, Vehtari,
  and Rubin (third edition, 2014)

- _Statistical Rethinking: A Bayesian Course with Examples in R and
  Stan_ by McElreath (2015)

- _Regression and Other Stories_ by Gelman, Hill, and Vehtari
  (forthcoming)

- Stan case studies:
  http://mc-stan.org/users/documentation/case-studies.html and
  https://github.com/stan-dev/stancon_talks

- Stan Discourse list: https://discourse.mc-stan.org
