
# Prior Distributions and Models for Data

## Prior, likelihood, posterior densities; Bayes as information aggregation

As discussed in the context of the Hello World example in chapter 1, a
Stan model is constructed by adding to the objective function or
log-posterior density; each line in the model block with "target +="
or a "~" symbol adds a term.  The increments can come in any order, as
addition is commutative ($X+Y = Y+X$).  So your Stan model can start
with the log-prior and then add the log-likelihood, or the other way
around.  Or the two can be interspersed.  The structure of Stan can be
thought of as an implementation of the general idea of Bayesian
inference as information aggregation: Prior information, data
information, all of it is just information that is fed into the
inference engine.

## Example:  apparently duplicate priors

Start with the following simple Stan program:

```
data {
  real y;
}
parameters {
  real theta;
}
model {
  theta ~ normal(0, 1);
  y ~ normal(theta, 1);
}
```

Here, the prior and data happen to provide the same amount of
information about $\theta$, and the posterior mean of $\theta$ will
fall halfway between the data, $y$, and the prior mean, 0.

Now suppose we add a new line to the model block, apparently
duplicating the prior:

```
model {
  theta ~ normal(0, 1);
  theta ~ normal(0, 1);
  y ~ normal(theta, 1);
}
```

This looks weird. We have specified a model for $\theta$ twice.  Does
this cause an error in the Stan program?  Does Stan simply ignore the
duplicate code?  Does the second "prior" overwrite the first?
Actually, no.  The above model block is legal Stan code, as we can see
by replacing each line by its equivalent augmentation of the objective
function:

```
model {
  target += normal(theta | 0, 1);
  target += normal(theta | 0, 1);
  target += normal(y | theta, 1);
}
```

Each line adds a piece of information to the log posterior density.
In this simple example, all three lines of code contain the equivalent
amount of information---each is a normal density with scale 1---and so
the posterior mean of $\theta$ will be the average of the three
numbers, 0, 0, and $y$.

Indeed, the above code is equivalent to this:

```
model {
  theta ~ normal(0, 1/sqrt(2));
  y ~ normal(theta, 1);
}
```

We cannot in general do this sort of simplification but it happens
that with the normal distribution these computations can be performed
analytically, as the product of two normal densities is itself normal.

As a practical matter, we do _not_ recommend code like this:

```
  theta ~ normal(0, 1);
  theta ~ normal(0, 1);
```

as it just looks confusing.  But understanding how it works gives
insight into the way that priors and data combine to express
information.


## Concentration of the likelihood

For any fixed model, as the sample size increases, the likelihood
becomes more informative.  This can be seen, for example, in the
mdoels above, where each additional data point adds another term to
the likelihood in the implicit vector addition of the `target +=` or
`~` statement.  See Chapter 4 of _Bayesian Data Analysis_ for more on
this.

## Problems with so-called noninformative priors

Before discussing prior distributions more generally, we share a simple example.

You have a single parameter $\theta$ which we estimate using an
experiment which returns an estimate $y$.  For simplicity, assume the
estimate is unbiased, normally distributed, and with known standard
deviation that happens to equal 1.  Thus, the data model is:
$$
y \sim \mbox{normal}(\theta,1).
$$
Now suppose you perform Bayesian inference using a uniform prior on
$\theta$.  Then your posterior distribution is simply (see, for
example, chapter 2 of _Bayesian Data Analysis_),
$$
\theta|y \sim \mbox{normal}(y,1).
$$

OK, fine.  Now suppose your estimate happens to be $y=1$.  There are
two ways of thinking about this:

* The observed data, 1 standard error away from 0, is completely
  consistent with a true value of $\theta=0$, or of low values of
  $\theta$ that could be either positive or negative.  From our usual
  statistical perspective, we would say that the data are consistent
  with pure noise, and we cannot learn much from these data alone.

* From the Bayesian posterior distribution, we can compute that
  $\mbox{Pr}(\theta>0) = 0.84$ (in R, `pnorm(1)`), thus you should be
  willing to bet with 5-to-1 odds that $\theta$ is positive.

This seems wrong, the idea that something recognizable as pure noise
can lead to 5:1 posterior odds.

The problem is coming from the prior distribution. We can see this in
two ways. First, just directly, effects near zero are more common than
large effects.

Jakulin et al. (2008) argue that logistic regression coefficients are
usually less than 1. So let's try combining normal(1,1) data with a
Cauchy(0,1) prior. It's easy enough to do in Stan:

```
data {
  real y;
}
parameters {
  real theta;
}
model {
  theta ~ cauchy (0, 1);
  y ~ normal(theta, 1);
}
```

If we fit this model supplying data $y=1$, and then from the resulting
simulations compute the posterior probability that $\theta > 0$, we
get 0.77, that is, roughly a 3:1 posterior probability that the effect
is positive.

Just to check that we're not missing anything, let's re-run using the
flat prior. New Stan model:

```
data {
  real y;
}
parameters {
  real theta;
}
model {
  y ~ normal (theta, 1);
}
```
and re-run with the same R code. This time, indeed, 84% of the posterior simulations of $\theta$ are greater than 0.

So far so good. Although one might argue that the posterior probability of 0.77 (from the inference given the unit Cauchy prior) is still too high. Perhaps we want a stronger prior? This sort of discussion is just fine. If you look at your posterior inference and it doesn't make sense to you, this "doesn't make sense" corresponds to additional prior information you haven't included in your analysis.

OK, so that's one way to consider the unreasonableness of a noninformative prior in this setting. It's not so reasonable to believe that effects are equally likely to be any size. They're generally more likely to be near zero.

The other way to see what's going on with this example is to take that flat prior seriously. Suppose theta really could be just about anything---or, to keep things finite, suppose you wanted to assign theta a uniform prior distribution on $[-1000,1000]$, and then you gather enough data to estimate $\theta$ with a standard deviation of 1. Then, a priori, you're nearly certain to gather very very strong information about the sign of $\theta$. To start with, there's a 0.998 chance that your estimate will be more than 2 standard errors away from zero so that your posterior odds about the sign of $\theta$ will be at least 20-to-1. And there's a 0.995 chance that your estimate will be more than 5 standard errors away from zero.

So, in your prior distribution, this particular event---that $y$ is so close to zero that there is uncertainty about $\theta$'s sign---is extremely unlikely. And it would be irrelevant that $y$ is not statistically significantly different from 0.

Modeling can be difficult. In some settings the data are strong and prior information is weak, and it's not really worth the effort to think seriously about what external knowledge we have about the system being studied. Often, though, we do know a lot, and we're interested in various questions where data are sparse, and we could be putting more effort into quantifying our prior distribution.

Upsetting situations---for example, the estimate of $1 \pm 1$ which leads to a seemingly too-strong claim of 5-to-1 odds in favor of a positive effect---are helpful in that they can reveal that we have prior information that we have not yet included in our models.



## Priors {#general-priors.section}

The prior only matters where the likelihood is nonzero (and vice-versa).  One way to think of the prior distribution is as a "regularization" or "soft constraint" on the parameters of the model.  We discuss prior choice more fully on the [prior choice wiki page](https://github.com/stan-dev/stan/wiki/Prior-Choice-Recommendations).

We consider 5 levels of priors:

- Flat prior;

- Super-vague but proper prior: normal(0, 1e6);

- Weakly informative prior, very weak: normal(0, 10);

- Generic weakly informative prior: normal(0, 1);

- Specific informative prior: normal(0.4, 0.2) or whatever. Sometimes this can be expressed as a scaling followed by a generic prior: theta = 0.4 + 0.2*z; z ~ normal(0, 1);

In Stan, if you don't supply any prior at all, you are implicitly using a flat prior.  In general we don't recommend this, but for many cases in this book, such as the Hello World example above, we use flat priors for simplicity and because we are focuseing on the expression of the data model or likelihood.

Setting up priors goes along with setting up the model.  You want to set up your parameters so they are understandable, and then you will typically have some sense of their possible range, and you can use this to set up priors.  If you have a lot of data, the prior won't matter much, but there are lots of real examples where data are weak and it it is helpful to use an informative prior.

### Background Reading {-}

See @Gelman:2006 for an overview of choices for priors for
scale parameters, @ChungEtAl:2013 for an overview of choices
for scale priors in penalized maximum likelihood estimates, and
@GelmanJakulinPittauEtAl:2008 for a discussion of prior choice
for regression coefficients.

### Improper Uniform Priors {-}

The default in Stan is to provide uniform (or "flat") priors on
parameters over their legal values as determined by their declared
constraints.  A parameter declared without constraints is thus given a
uniform prior on $(-\infty,\infty)$ by default, whereas a scale
parameter declared with a lower bound of zero gets an improper uniform
prior on $(0,\infty)$.  Both of these priors are improper in the sense
that there is no way formulate a density function for them that
integrates to 1 over its support.

Stan allows models to be formulated with improper priors, but in order
for sampling or optimization to work, the data provided must ensure a
proper posterior.  This usually requires a minimum quantity of data,
but can be useful as a starting point for inference and as a baseline
for sensitivity analysis (i.e., considering the effect the prior
has on the posterior).

Uniform priors are specific to the scale on which they are formulated.
For instance, we could give a scale parameter $\sigma > 0$ a uniform
prior on $(0,\infty)$, $q(\sigma) = c$ (we use $q$ because the
"density" is not only unnormalized, but unnormalizable), or we could
work on the log scale and provide $\log \sigma$ a uniform prior on
$(-\infty,\infty)$, $q(\log \sigma) = c$.  These work out to be
different priors on $\sigma$ due to the Jacobian adjustment necessary
for the log transform.

Stan automatically applies the necessary Jacobian adjustment for
variables declared with constraints to ensure a uniform density on the
legal constrained values.  This Jacobian adjustment is turned off when
optimization is being applied in order to produce appropriate maximum
likelihood estimates.

### Proper Uniform Priors: Interval Constraints {-}

It is possible to declare a variable with a proper uniform prior by
imposing both an upper and lower bound on it, for example,

```
real<lower=0.1, upper=2.7> sigma;
```

This will implicitly give `sigma` a $\mathsf{Uniform}(0.1, 2.7)$
prior.

#### Matching Support to Constraints {-}

As with all constraints, it is important that the model
provide support for all legal values of `sigma`.  For example,
the following code constraints `sigma` to be positive, but then
imposes a bounded uniform prior on it.

```
parameters {
  real<lower=0> sigma;
  ...
model {
  // *** bad *** : support narrower than constraint
  sigma ~ uniform(0.1, 2.7);
```

The sampling statement imposes a limited support for `sigma` in
(0.1, 2.7), which is narrower than the support declared in the
constraint, namely $(0, \infty)$.  This can cause the Stan program to
be difficult to initialize, hang during sampling, or devolve to a
random walk.

#### Boundary Estimates {-}

Estimates near boundaries for interval-constrained parameters
typically signal that the prior is not appropriate for the model.  It
can also cause numerical problems with underflow and overflow when
sampling or optimizing.

### "Uninformative" Proper Priors {-}

It is uncommon to see models with priors on regression coefficients
such as $\mathsf{normal}(0,1000)$.^[The practice was common in BUGS
and can be seen in most of their examples @LunnEtAl:2012.]

If the prior scale, such as 1000, is several orders of magnitude
larger than the estimated coefficients, then such a prior is
effectively providing no effect whatsoever.

We actively discourage users from using the default scale priors
suggested through the BUGS examples [@LunnEtAl:2012], such as
$$
\sigma^2 \sim \mathsf{InvGamma}(0.001, 0.001).
$$

Such priors concentrate too much probability mass outside of
reasonable posterior values, and unlike the symmetric wide normal
priors, can have the profound effect of skewing posteriors; see
@Gelman:2006 for examples and discussion.

### Truncated Priors {-}

If a variable is declared with a lower bound of zero, then assigning
it a normal prior in a Stan model produces the same effect as
providing a properly truncated half-normal prior.  The truncation at
zero need not be specified as Stan only requires the density up to a
proportion.  So a variable declared with

```
real<lower=0> sigma;
```

and given a prior
```
sigma ~ normal(0, 1000);
```

gives `sigma` a half-normal prior, technically

$$
p(\sigma)
\ = \
\frac{\mathsf{normal}(\sigma | 0, 1000)}
     {1 - \mathsf{NormalCDF}(0 | 0, 1000)}
\ \propto \
\mathsf{normal}(\sigma | 0, 1000),
$$

but Stan is able to avoid the calculation of the normal cumulative
distribution (CDF) function required to normalize the half-normal density.
If either the prior location or scale is a parameter or if the
truncation point is a parameter, the truncation cannot be dropped,
because the normal CDF term will not be a constant.


### Weakly Informative Priors {-}

Typically a researcher will have some knowledge of the scale of the
variables being estimated.  For instance, if we're estimating an
intercept-only model for the mean population height for adult women,
then we know the answer is going to be somewhere in the one to three
meter range.  That gives us information around which to form a weakly
informative prior.

Similarly, a logistic regression with predictors on the standard scale
(roughly zero mean, unit variance) is unlikely to have a
coefficient that's larger than five in absolute value.  In these
cases, it makes sense to provide a weakly informative prior such as
$\mathsf{normal}(0,5)$ for such a coefficient.

Weakly informative priors help control inference computationally and
statistically.  Computationally, a prior increases the curvature
around the volume where the solution is expected to lie, which in turn
guides both gradient-based like L-BFGS and Hamiltonian Monte Carlo
sampling by not allowing them to stray too far from the location of a
surface.  Statistically, a weakly informative prior is more sensible
for a problem like women's mean height, because a  diffuse prior
like $\mathsf{normal}(0,1000)$ will ensure that the vast majority of
the prior probability mass is outside the range of the expected
answer, which can overwhelm the inferences available from a small data
set.

### Bounded Priors {-}

Consider the women's height example again.  One way to formulate a
proper prior is to impose a uniform prior on a bounded scale.  For
example, we could declare the parameter for mean women's height to
have a lower bound of one meter and an upper bound of three meters.
Surely the answer has to lie in that range.

Similarly, it is uncommon to see priors for scale parameters that
impose lower bounds of zero and upper bounds of large numbers, such as
10,000.^[This was also a popular strategy in the BUGS example models
[@LunnEtAl:2012], which often went one step further and set the lower
bounds to a small number like 0.001 to discourage numerical underflow
to zero.]

This provides roughly the same problem for estimation as a 
diffuse inverse gamma prior on variance.  We prefer to leave
parameters which are not absolutely physically constrained to float
and provide them informative priors.  In the case of women's height,
such a prior might be $\mathsf{normal}(2,0.5)$ on the scale of meters;
it concentrates 95% of its mass in the interval $(1,3)$, but still
allows values outside of that region.

In cases where bounded priors are used, the posterior fits should be
checked to make sure the parameter is not estimated at or  close
to a boundary.  This will not only cause computational problems, it
indicates a problem with the way the model is formulated.  In such
cases, the interval should be widened to see where the parameter fits
without such constraints, or boundary-avoid priors should be used (see
the [hierarchical priors section](#hierarchical-priors.section).)


### Fat-Tailed Priors and Default Priors {-}

A reasonable alternative if we want to accommodate outliers is to use a
prior that concentrates most of mass around the area where values are
expected to be, but still leaves a lot of mass in its tails.  One choice in such a situation is to use a Cauchy distribution for a
prior, which can concentrate its mass around its median, but has tails
that are so fat that the variance is infinite.

Without specific information, the Cauchy prior can be a reasonable default
parameter choice for regression coefficients
@GelmanJakulinPittauEtAl:2008 and the half-Cauchy (coded
implicitly in Stan) a reasonable default choice for scale parameters
@Gelman:2006.



### Informative Priors {-}

Ideally, there will be substantive information about a problem that
can be included in an even tighter prior than a weakly informative
prior.  This may come from actual prior experiments and thus be the
posterior of other data, it may come from meta-analysis, or it may
come simply by soliciting it from domain experts.  All the goodness of
weakly informative priors applies, only with more strength.

### Conjugacy {-}

Unlike in Gibbs sampling, there is no computational advantage to
providing conjugate priors (i.e., priors that produce posteriors in
the same family) in a Stan program.^[BUGS and JAGS both support
conjugate sampling through Gibbs sampling.  JAGS extended the range of
conjugacy that could be exploited with its GLM module.  Unlike Stan,
both BUGS and JAGS are restricted to conjugate priors for constrained
multivariate quantities such as covariance matrices or simplexes.]  It
is possible in some cases to exploit conjugacy to marginalize out
parameters, which can lead to efficiency gains.

Neither the Hamiltonian Monte Carlo samplers or the optimizers make
use of conjugacy, working only on the log density and its derivatives.

