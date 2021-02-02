---
layout: post
title: Deriving the Normal Equation from an Optimization perspective
---
I was watching through the lecture video about Optimization methods taught by Nando de Freitas and learnt something interesting that I'd like to share. Since I'm predominantly self taught, I've watched through many different videos about Machine Learning topics on Youtube. Nando has a rare quality of being able to distill complicated topics and explain them in a very intuitive way. I've thoroughly benefited from his materials and would highly recommend watching his video lectures on Youtube if you're interested in Machine Learning.

Back to the topic of the post. In the video I was watching, Nando was explaining the different optimization methods used for Machine Learning. What I found really interesting was that when you applied Newton's method on Linear Regression, you end up with the same form as the Maximum Likelihood Estimate solution.

So before we go into spoiler territory, let's review what we want to achieve in model optimization. \
When we are training a Machine Learning model, what we want is to find a set of $$\theta$$ or in plain english, a collection of "settings" for our model that minimizes some cost/loss function we define. For the case of Linear Regression, we know that we can derive the closed form solution or the Normal Equation by taking the log of the likelihood function, equating to zero and solving for $$\theta$$. \
What we end up with is: $$\theta=(X^{T}X)^{-1}X^{T}y$$

## First-Order Method
Now, if there doesn't exist a closed form solution for the model as above. The usual approach is to apply Gradient Descent where we take steps(controlled by the learning rate, $$\eta$$) opposite the gradient vector, $$g_{k}$$(since gradients are in the direction of largest change, we want to go opposite direction in order to minimize).
This looks like the following: $$\theta_{k+1}=\theta_{k}-\eta g_{k}$$ \
For Linear Regression, this would look like the following: \
Squared Loss cost function in matrix form, $$(y-X\theta)^{T}(y-X\theta)$$ \
Expanding the cost function out and taking the gradient, $$g_{k}=\frac{\partial{f(\theta)}}{\partial{\theta}}(y^{T}y-2y^{T}X\theta+\theta^{T}X^{T}X\theta)$$ \
$$g_{k}=2X^{T}y+2X^{T}X\theta$$ \
$$\theta_{k+1}=\theta_{k}-\eta (2X^{T}y+2X^{T}X\theta)$$

So for the Gradient Descent method for finding the minimum, we would keep repeating this calculation by first calculating the gradient, using that information to take an opposite step in $$\theta$$ to obtain the new $$\theta_{k+1}$$ and repeat until the gradient settles to zero or as close as possible (since our choice of $$\eta$$ might cause overshoot).

## Second-Order Method
The idea of Newton's method is relatively simple. Instead of using a fixed scalar for $\theta$, can we perhaps use the second-order of the function to help us optimize? In a physics setting, the second derivative tells us if the function or motion is accelerating or coming to a stop.
Instead of a fixed $$\eta$$, we instead use the inverse of the Hessian ($$H^{-1}$$) to obtain $$\theta_{k+1}=\theta_{k}-H_{k}^{-1}g_{k}$$

**Derivation** \
We can think of this as doing a 2nd order Taylor Series approximation of $$f(\theta)$$ and $$\theta_{k}$$. \
So if $$f(\theta)=f(\theta_{k})+g_{k}^{T}(\theta-\theta_{k})+\frac{1}{2}(\theta - \theta_{k})^{T}H_{k}(\theta - \theta_{k})$$ \
The first part $$g_{k}^{T}$$ can be treated as a 1st order approximation. \
The second part $$H_{k}$$ can be treated as a 2nd order approximation. \
$$\Delta \frac{f(\theta_{k})}{d \theta} = 0 + g_{k} + H_{k}(\theta - \theta_{k}) = 0$$ \
$$H_{k}(\theta-\theta_{k}) = -g_{k}$$ \
$$H_{k}\theta - H_{k}\theta_{k} = -g_{k}$$ \
$$\theta - \theta_{k} = -H_{k}^{-1}g_{k}$$ \
$$\theta = \theta_{k} - H_{k}^{-1}g_{k}$$ \

**Applying Newton's method on Linear Regression** \
$$\theta_{k+1} = \theta_{k}-H_{k}^{-1}g_{k}$$ \
$$\theta_{k} - (2X^{T}X)^{-1}[-2X^{T}Y+2X^{T}X \theta_{k}]$$ \
Expanding the terms out, note that since $$2^{-1}=\frac{1}{2}$$ when we expand out the $$2$$ cancels. \
$$\theta_{k} + (X^{T}X)^{-1}X^{T}Y - (X^{T}X)^{-1}X^{T}X \theta_{k}$$ \
Since $$(X^{T}X)^{-1}X^{T}X$$ becomes 1, $$\theta_{k} - \theta_{k} = 0$$ and we end up with: \
$$(X^{T}X)^{-1}X^{T}Y$$ \
So when we apply Newton's method on Linear Regression we end up with the same closed form answer from M.L.E.

---
#### References:
[1] Deep Learning Lecture 6: Optimization, Nando de Freitas. (https://youtu.be/0qUAb94CpOw)
[2] Pattern Recognition and Machine Learning, C.Bishop.

