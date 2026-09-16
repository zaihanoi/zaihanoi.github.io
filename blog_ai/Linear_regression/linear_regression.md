---
layout: post
title: "Blog AI Linear_regression"
date: 2026-07-08
---
⚠️ Note: This post was inspired by the book "Machine Learning Cơ bản" (Basic of Machine Learning) by Vu Huu Tiep.
# Linear Regression
Linear Regression is one of the most basic and popular Machine Learning algorithms. Being used mostly in `supervised learning`, Linear Regression could predict the output values independently of the fixed labels.
## 1. Base Knowledge
The most basic problem that could be solved by Linear Regression is predict the house prices. There are many features that could affect housing price such as: the amount of bedrooms ($$x_1$$), the area of the house ($$x_2$$), the number of floors ($$x_3$$). From these features, our model has to predict to house price. The training data contained features ($$x_1,x_2,x_3$$) and prices of other houses in the city. This problem could be describe by this linear equation:

$$
y \approx \hat{y} = x_1. w_1+ x_2. w_2 + x_3.w_3 \tag{1}
$$

where  

- $$y$$ is the real price of the house 
- $$\hat{y}$$ is our model prediction
- $$x_1,x_2,x_3$$ are features
- $$w_1,w_2,w_3$$ are the weight values

I set that $$x=[x_1,x_2,x_3]^T$$ and $$w=[w_1,w_2,w_3]^T$$. So the function (1) could be rewritten as:

$$
y \approx \hat{y} = x_1. w_1+ x_2. w_2 + x_3.w_3 = x^T . w
$$

That is the basic form of Linear Regression with feature vector $$x$$ and weight vector $$w$$.

From the above simple example, we could understand the name of the algorithm. `Linear` means that the base of this algorithm is linear function and when trying to output a real value according to the feature vector, we call it `Regression`.
## 2. Mathematical Analysis
From the example above, we could build a general function of Linear Regression for `one data point`:

$$
f(w) = x_1. w_1+ x_2. w_2 + ... + x_n.w_n = x^T . w 
$$

with $$x$$ is its fixed feature vector and we have to find the weight vector $$w$$. The weight vector shows the `importance` of feature according to the output value and the changes of $$w$$ will lead to the fluctuations of output. So, $$w$$ value is the model parameter of Linear Regression. 
### 2.1 Loss function
The loss function of Linear Regression is the difference between $$y$$ and $$\hat{y}$$. So, to make the algorithm works best, we have to minimize the Loss function, that means we have to find the suitable value of $$w$$. For a single data point, the loss value is: $$|y-\hat{y}|$$ and to remove the `absolute value bars` I will square $$y-\hat{y}$$ and multiply it with $$\frac{1}{2}$$ for later `derivative step`. Because $$\frac{1}{2}$$ is a constant number so basically, it does not affect how the Loss function works. So, the loss value in one data point will become:

$$
\frac{1}{2}(y-\hat{y})^2 = \frac{1}{2}(y-x^T . w)^2 
\tag{2}
$$

If there are `N` data points, $$x_i$$ with ($$1 \leq i \leq N ; i \in Z$$) are their feature vectors, $y_i$ with ($$1 \leq i \leq N ; i \in Z$$) are their real outputs, we could build the function to find the average of `N` loss value:

$$
L(w) = \frac{1}{2N} . \sum_{i=1}^{N} (y_i-x_i^T.w)^2 \tag{3}
$$

Function (3) is the general Loss function of Linear Regression.

The Loss function (3) could also be rewritten as:

$$
L(w) = \frac{1}{2N} . ||y-X^T.w||^2_2 \tag{4}
$$

with $$y=[y_1,y_2,...,y_N]^T$$ is a vector of real outputs and $$X =[x_1,x_2,...,x_N]$$ is a matrix of N data points feature vectors. For example, if each data point has `d` features. So, $$X$$ is `(N x d)` and $$X^T.w$$ is `(N x 1)`. The function (4) uses the Euclid algorithm to calculate the distance between two vectors $$y$$ and $$X^T.w$$.

Our target is to find $$w$$:

$$
w = \argmin_{w} L(w) = \argmin_{w} \frac{1}{2N} . ||y-X^T.w||^2_2
$$


### 2.2 Find the minimum value of Loss function

The function (4) is a quadratic function and it is continuous with respect to $$w$$. So we could find its minimum value with those basic steps:

- Calculate the `first derivative` of function (3):

$$
L(w)' = \frac{1}{N} . X . (X^T.w - y)
$$

- Set $L(w)' = 0$ and find $$w$$:

$$
L(w)' = \frac{1}{N} . X . (X^T.w - y) =0

<=> w = (X.X^T)^{-1} .X.y
$$

- If we cannot calculate the value of $$(X.X^T)^{-1}$$ we will use pseudo inverse value instead. The pseudo inverse of $$(X.X^T)$$ is $$(X.X^T)^{+}$$. There is a function in `numpy` library can calculate pseudo inverse.

### 2.3 Bias trick

In many cases, when the problem is more complicated we will add a `b` (bias) into Linear Regression algorithms on one data point.

$$
\hat{y} = x^T.w +b  \tag{5}
$$

We set that there is a feature called $$x_0 = 1$$. Rewrite the function (5):

$$
\hat{y} = x^T.w +b =  x_1.w_1 +...+ x_n.w_n + x_0.b = \hat{x}^T . \hat{w}
$$

with $$\hat{x}= [x_0,x_1,...,x_d]^T$$ and $$\hat{w} = [b,w_1,w_2,...,w_d]^T$$. 

Set that $$\hat{X}= [x_1,x_2,...,x_N]$$ is a features matrix and $$y=[y_1,y_2,...,y_N]^T$$ is a real output vector. The general function of Linear Regression after adding bias is:

$$
L(\hat{w}) = \frac{1}{2N} . ||y-\hat{X}^T.w||^2_2 
$$

so:

$$
\hat{w} = \argmin_{\hat{w}} \frac{1}{2N} . (y- \hat{X}^T . \hat{w})^2 = (\hat{X}.\hat{X}^T)^{+} .\hat{X}.y
$$

The technique of setting a feature $$x_0 = 1$$ and using it as above is called `bias trick`
## 3. Application of Linear Regression algorithm
The function $$(y \approx f(x) = x^T w)$$ is a linear function with respect to both $$w$$ and $$x$$. We could also use Linear Regression when its function is linear with only $$w$$. For example:

$$
y ≈ w_1.x_1 + w_2.x_2^2 + w_3.sin(x_2) + w_5.x_1.x_2 + w_0
$$

But you can see that the idea of calculating $$X= [x_1, x_2^2,x_2,sin(x_2), x_1.x_2]^T$$ from a feature vector $$x= [x_1,x_2]$$ of a data point is not natural. Instead of that, in this case, we use `polynomial regression`.
## 4. Drawbacks of Linear Regression algorithm and how to mitigate them

- Linear Regression is strongly affected by noisy data point

If there are noisy data points in training dataset, they could strongly affect the performance of Linear Regression because of making the algorithm find the wrong weight value.

This problem could be mitigated by cleaning the dataset carefully before training or using `Huber loss function`. Using `Huber loss` in `Linear Regression` is `Huber Regression` and this algorithm is robust to noise.

- Ridge regression

If we could not calculate the inverse of $$(X.X^T)$$ we could change its form a little bit instead of using pseudo inverse value. Specifically, we will do this changing technique:

$$
(X.X^T) = XX^T + λI
$$

with λ is a very small number and $$I$$ is a unit matrix. By performing this technique, we could find the inverse value of $$(X.X^T)$$. By continuing processing the algorithm in this way, we will have `ridge regression`. `Ridge regression` could minimize `over-fitting` which will be posted on my page soon.

- Huge size of $$(X.X^T)$$

In reality, the $$X$$ matrix is often huge because of large number of features and data points. So, when we calculate the inverse value of $$(X.X^T)$$ we have to use many computer resources.

Instead of using basic method above to minimize the Loss function, we could use a method called `Gradient
descent`. Post about this technique will be on my website soon.

## 5. Conclusion
This post gives you general knowledge about Linear Regression algorithm, one of the most basic algorithm in `Machine Learning` and `Deep Learning`, such as how it really works, its mathematical nature, its application and its drawbacks. I hope that after reading this post, you will gain useful information and knowledge about `Machine Learning` and `Deep Learning` for yourself.