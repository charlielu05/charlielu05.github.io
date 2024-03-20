---
layout: single
title: Calculating the Posterior for Bayesian Regression Model (Part 3)
---

This post seeks to write down the explicit steps and reasoning involved to come up with the posterior representation in a Bayesian Linear Regression model.

Complete the square trick is used to derive the solution to the posterior of a Bayesian Linear Regression model.

The posterior seeks to obtain the best probability estimate for the $$\theta$$ or coefficients given evidence (new data).

### Posterior weights: $$p(\theta|y)$$
This is modelled as a Gaussian,
$$\mathcal{N}(\theta|\mu_{\theta |y}, \Sigma_{\theta | y})$$ \
This says that the Gaussian is conditioned or are the result of $$y$$ (data).

### Prior weights: $$p(\theta)$$
Modelled as a Gaussian,
$$\mathcal{N}(\theta|\mu_{\theta}, \Sigma_{\theta})$$ \
We model the $$\theta$$ as a Gaussian, for a single univariate predictor this could just be a simple Normal Distribution. Extending it to multiple predictors/features we simply use a Multivariate normal distribution. Note that this could be a source of confusion since some articles or textbook examples might use a Normal distribution and some might use a Multivariate normal distribution.

### Likelihood: $$p(y|\theta)$$
As before, modelled by a Gaussian.
$$\mathcal{N}(y|X\theta, \Sigma_{y})$$ \
The mean of the Gaussian is the product of the predictors/independent variables $$X$$ and the $$\theta$$/weights.

### Prediction/Marginal of y: $$p(y)$$
$$\mathcal{N}(y|X\mu_{\theta},\Sigma_{y}+X\Sigma_{\theta}X^{T})$$ \
This is your prediction after you have calculated the posterior weights. Simply the new test points $$X$$ multiplied by the mean value of your posterior weight $$\theta$$. The unique part is tha the uncertainity on your prediction or the variance of $$y$$ is the sum of the uncertainity/noise on $$y$$ in addition to the uncertainity/noise on your $$\theta$$/weights.

### Deriving Proof
The entire step to obtaining the posterior, conditioned Gaussian.
We first want to dervie the joint distribution: $$p(\theta,y)$$=likelihood $$\times$$ prior = $$p(y|\theta)p(\theta)$$ \
Replacing the MVN form by taking the log (cancelling the exponential and turning the multiplication into addition) and disregard the constant before the exponential we get the form: \
$$log(p(\theta,y))=$$
$$-\frac{1}{2}(y-X\theta)^{T}\Sigma_{y}^{-1}(y-X\theta)-\frac{1}{2}(\theta-\theta_{0})^{T}\Sigma_{\theta}^{-1}(\theta-\theta_{0})$$

$$\theta_{0}$$: prior theta \
$$\theta_{\theta}$$: new theta

**Expanding out $$p(\theta)$$** \
$$-\frac{1}{2}((\theta-\theta_{0})^{T}(\Sigma_{\theta}^{-1}\theta-\Sigma_{\theta}^{-1}\theta_{0}))$$ \
Expanding out further: \
$$-\frac{1}{2}(\theta^{T}\Sigma_{\theta}^{-1}\theta-\theta^{T}\Sigma_{\theta}^{-1}\theta_{0}-\theta_{0}^{T}\Sigma_{\theta}^{-1}\theta+\theta_{0}^{T}\Sigma_{\theta}^{-1}\theta_{0})$$
\
Notice that the 2nd and 3rd group of terms are equivalent, therefore we can re-write as: \
$$-\frac{1}{2}(\theta^{T}\Sigma_{\theta}^{-1}\theta-2\theta^{T}\Sigma_{\theta}^{-1}\theta_{0}+\theta_{0}^{T}\Sigma_{\theta}^{-1}\theta_{0})$$
\
Factoring out the $$\Sigma$$ we obtain: \
$$-\frac{\Sigma_{\theta}^{-1}}{2}(\theta^{T}\theta-2\theta^{T}\theta_{0}+\theta_{0}^{T}\theta_{0})$$

Just note that if you specified your prior mean centred at 0, this expression would simplify down to only the first group of terms since everything else would be multiplying by $$0$$: \
$$-\frac{\Sigma_{\theta}^{-1}}{2}(\theta^{T}\theta)$$

**Expanding out $$p(y|X\theta)$$** \
Note that the matrix transpose rule: $$(BC)^{T}=C^{T}B^{T}$$ is used here and the $$\Sigma_{y}$$ has been factored out to skip a few steps. \
$$-\frac{\Sigma_{y}^{-1}}{2}(y^{T}y-y^{T}X\theta-\theta^{T}X^{T}y+\theta^{T}X^{T}X\theta)$$

Similar to $$p(\theta)$$, the second and third collection of terms are also equivalent. Also because $$y$$ is given it's treated as a constant. So rewriting the equation this becomes: \
$$-\frac{\Sigma_{y}^{-1}}{2}(2\theta^{T}X^{T}y+\theta^{T}X^{T}X\theta)$$

**Combining the Likelihood and the Prior** \
Now we have both $$p(\theta)$$ and $$p(y|X\theta)$$ as: \
$$-\frac{\Sigma_{\theta}^{-1}}{2}(\theta^{T}\theta-2\theta^{T}\theta_{0}+\theta_{0}^{T}\theta_{0})-\frac{\Sigma_{y}^{-1}}{2}(y^{T}y-y^{T}X\theta-\theta^{T}X^{T}y+\theta^{T}X^{T}X\theta)$$

**Complete the square trick** \
There is only one last step and it's probably the most important and strangely commonly skipped step in most blogs or textbooks I read. Basically, we want to re-arrange the above expression by factoring and the end result will look like a quadratic expansion of a MVN expression.
To be more concerete, since we know as a fact that the product of two Gaussians (the prior and the likelihood) is a Gaussian. We are therefore trying to make the above complicated expression look like a "Gaussian" in its expanded form. By doing this, we can identify the parts that makes up the $$\mu$$ and $$\Sigma$$ of this new Gaussian.

So just to cover the basic, if we only look at the exponential part of a MVN p.d.f (since the terms before are constant) we have: \
$$-\frac{1}{2}(x-u)^{T}\Sigma^{-1}(x-u)$$ \
Expanding this out we get: \
$$-\frac{1}{2}(x^{T}\Sigma^{-1}x-2x^{T}\Sigma^{-1}u+u^{T}\Sigma^{-1}u)$$

So just to recap, once we have re-arranged the terms for our likelihood and prior, what we are hoping to see is something similar in form to the above.

Re-arranging the posterior by factoring out the $$\frac{1}{2}$$ and $$\theta^{T}$$: \
$$-\frac{1}{2}(\theta^{T}(\Sigma_{\theta}^{-1}+\Sigma_{y}^{-1}x^{T}x)\theta-2\Sigma_{y}^{-1}\theta^{T}x^{T}y-2\theta^{T}\Sigma_{\theta}^{-1}\theta_{0}+\theta_{0}^{T}\Sigma_{\theta}^{-1}\theta_{0})$$

By looking at the expression above, we can see that the first collection of terms contains $$\Sigma$$ for our new Gaussian. \
For notation simplicity we will replace $$(\Sigma_{\theta}^{-1}+\Sigma_{y}^{-1}x^{T}x)$$ with $$\Sigma$$. \
This can be further simplified by factoring out the $$2\theta^{T}$$ to give: \
$$\frac{1}{2}(\theta^{T}\Sigma^{-1}\theta-2\theta^{T}(\Sigma_{y}^{-1}x^{T}y-\Sigma_{\theta}^{-1}\theta_{0})+\theta_{0}^{T}\Sigma_{\theta}^{-1}\theta_{0})$$ \
Now, we can see that this looks very similar to the expanded form of a MVN.
So since we already know our $$\Sigma$$, this really only leaves the $$\mu$$ term to be found for our new Gaussian.
Looking at the second collection of terms we can say that: \
$$\Sigma^{-1}\mu=\Sigma_{y}^{-1}x^{T}y-\Sigma_{\theta}^{-1}\theta_{0}$$ \
Multiplying both sides by $$\Sigma$$ finally leaves us with: \
$$\mu=\Sigma(-\Sigma_{\theta}^{-1}\theta_{0}+\Sigma_{y}^{-1}x^{T}y)$$

This is the exact form as formula 3.50 and 3.51 in Bishop's PRML book for Bayesian Linear Regression.

---
#### References:
[1] Pattern Recognition and Machine Learning, Bishop. (p.85,86,91)

[2] "Computing posterior in probablistic linear regression", Piyushi Rai, CS771 (https://www.cse.iitk.ac.in/users/piyush/courses/ml_autumn18/material/plr_posterior.pdf)

[3] Machine Learning, Murphy. (formula 4.150)
