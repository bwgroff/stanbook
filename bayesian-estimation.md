
# Bayesian Point Estimation

There are three common approaches to Bayesian point estimation based
on the posterior $p(\theta|y)$ of parameters $\theta$ given observed
data $y$: the mode (maximum), the mean, and the median.

## Posterior Mode Estimation

This section covers estimates based on the parameters $\theta$ that
maximize the posterior density, and the next sections continue with
discussions of the mean and median.

An estimate based on a model's posterior mode can be defined by

$$
\hat{\theta} = \mbox{argmax}_{\theta} \, p(\theta|y).
$$

When it exists, $\hat{\theta}$ maximizes the posterior density of the
parameters given the data.  The posterior mode is sometimes called the
``maximum a posteriori" (MAP) estimate.

As discussed in the [problematic posteriors
chapter](#problematic-posteriors.chapter) and the [maximum likelihood
section](#mle.section), a unique posterior mode might not
exist---there may be no value that maximizes the posterior mode or
there may be more than one.  In these cases, the posterior mode
estimate is undefined.  Stan's optimizer, like most optimizers, will
have problems in these situations.  It may also return a locally
maximal value that is not the global maximum.

In cases where there is a posterior mode, it will correspond to a
penalized maximum likelihood estimate with a penalty function equal to
the negation of the log prior.  This is because Bayes's rule,
$$
p(\theta|y) = \frac{p(y|\theta) \, p(\theta)}{p(y)},
$$
ensures that

$$
\begin{array}{rcl}
\mbox{argmax}_{\theta} \ p(\theta|y)
& = &
\mbox{argmax}_{\theta} \ \frac{p(y|\theta) \, p(\theta)}{p(y)}
\\[6pt]
& = &
\mbox{argmax}_{\theta} \ p(y|\theta) \, p(\theta),
\end{array}
$$

and the positiveness of densities and the strict monotonicity of log
ensure that
$$
\mbox{argmax}_{\theta} \ p(y|\theta) \, p(\theta)
\ = \
\mbox{argmax}_{\theta} \ \log p(y|\theta) + \log p(\theta).
$$


In the case where the prior (proper or improper) is uniform, the
posterior mode is equivalent to the maximum likelihood estimate.

For most commonly used penalty functions, there are probabilistic
equivalents.  For example, the ridge penalty function corresponds to a
normal prior on coefficients and lasso to a Laplace prior.  The
reverse is always true---a negative prior can always be treated as a
penalty function.



## Posterior Mean Estimation

A standard Bayesian approach to point estimation is to use the
posterior mean (assuming it exists), defined by

$$
\hat{\theta} = \int \theta \, p(\theta|y) \, d\theta.
$$

The posterior mean minimizes the expected square error of the
estimate.

An estimate of the posterior mean for each parameter is returned by
Stan's interfaces;  see the RStan, CmdStan, and PyStan user's guides
for details on the interfaces and data formats.

Posterior means exist in many situations where posterior
modes do not exist.  For example, in the $\mathsf{Beta}(0.1, 0.1)$
case, there is no posterior mode, but posterior mean is well defined
with value 0.5.

A situation where posterior means fail to exist but posterior modes do
exist is with a posterior with a Cauchy distribution
$\mathsf{Cauchy}(\mu,\tau)$.  The posterior mode is $\mu$, but the
integral expressing the posterior mean diverges.  Such diffuse priors
rarely arise in practical modeling applications; even with a Cauchy
Cauchy prior for some parameters, data will provide enough constraints
that the posterior is better behaved and means exist.

Sometimes when posterior means exist, they are not meaningful, as in
the case of a multimodal posterior arising from a mixture model or in
the case of a uniform distribution on a closed interval.


## Posterior Median Estimation

The posterior median (the 50th percentile or 0.5 quantile) is
another popular point estimate reported for Bayesian models.  The
posterior median minimizes the expected absolute error of estimates.
These estimates are returned in the various Stan interfaces;  see the
RStan, PyStan, and CmdStan user's guides for more information on
format.

Although posterior medians may fail to be meaningful, they often exist
even where posterior means do not, as in the Cauchy distribution.
