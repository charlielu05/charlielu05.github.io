---
layout: post
---

This post seeks to write down the explicit steps and reasoning involved to come up with the posterior representation in a [[Bayesian Linear Regression]] model.

The goal is to obtain the conditional probability distribution of the weights(theta) given y from the joint distribution (likelihood multiplied by the prior) normalized by the probability of y. In plain english, "What do I think the weights/coefficients should be after I have seen more data?".

$$p(\theta|y)=\frac{p(y|\theta)p(\theta)}{p(y)}$$
$$p(\theta|y)=\frac{p(y,\theta)}{p(y)}$$
Since $p(y|\theta)p(\theta)=p(y,\theta)$
#### Likelihood
Remembering that our likelihood function $p(y|\theta)$ is modelled by a normal distribution with a mean given by a normal distribution with a mean of $\theta^{T}X$ where X is our predictor variables. This is the inner product/linear regression between theta and X. $\mathcal{N}(\theta^{T}X, \Sigma_{y})$. The sigma term, $\Sigma_{y}$ indicates our error or uncertainty in our model, in a practical setting we can think of this as sensor error or sensor variance when we obtain some measurements from a sensor device in real-life.
#### Prior
Our prior describes our probability distribution for our actual weights/coefficients used in our calculation of y. The way I think of the prior is a shape where we define what the most likely values the weights could take.
#### Posterior
Given the fact that the product of two Gaussian is a Gaussian, we can obtain the closed from solution

