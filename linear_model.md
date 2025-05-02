# Linear Modelling and Statistical Tests

## A Simple Simulated Example
Consider a simple linear model with two conditions and a linear effect that varies over (e.g.) time.
In the example below, we let the first column encode the baseline, the second columns encodes the task, and the third column is some linear effect of time.
Any haemodynamic effect is ignored to keep things simple.
Try copy/pasting the following into MATLAB.
```
X = [[ones(2,1) zeros(2,1)
      zeros(2,1) ones(2,1)
      ones(2,1) zeros(2,1)
      zeros(2,1) ones(2,1)
      ones(2,1) zeros(2,1)] (0:9)']
```
This will give a matrix that looks like
```
X =

     1     0     0
     1     0     1
     0     1     2
     0     1     3
     1     0     4
     1     0     5
     0     1     6
     0     1     7
     1     0     8
     1     0     9
```

When we fit a linear model, we consider that the model explains how the data were generated.
If we assume independent and identically distributed (i.i.d.) Gaussian noise, then the model of the data ${\bf y}$ is

$$
{\bf y} = {\bf X} \boldsymbol{\beta} + \boldsymbol{\epsilon} \text{,  where } \boldsymbol{\epsilon} \sim \mathcal{N}({\bf 0}, \sigma^2 {\bf I})
$$

In the above, the $\boldsymbol{\epsilon} \sim \mathcal{N}({\bf 0}, \sigma^2 {\bf I})$
indicates that $\boldsymbol{\epsilon}$ comes from a zero-mean i.i.d. Gaussian (normal) distribution where the variance is $\sigma^2$.
We can generate some simulated data (``y``) according to this model, where we use $\boldsymbol{\beta}$ values of 30.0, 35.0 and 2.0
and a variance of 4.0.

!!! info "Gaussian distribution"
    The formula for the probability density of ${\bf x}$ under a multivariate Gaussian distribution of
     mean $\boldsymbol{\mu}$ and variance/covariance $\boldsymbol{\sigma}$ is $
     p({\bf x} | \boldsymbol{\mu}, \boldsymbol{\Sigma}) =
     \frac{\exp\left(
     -\frac{1}{2} ({\bf x} - \boldsymbol{\mu})^T \boldsymbol{\Sigma}^{-1} ({\bf x} - \boldsymbol{\mu})
     \right)}{\sqrt{(2 \pi)^D \det | \boldsymbol{\Sigma} |}}$
     
In MATLAB, i.i.d. Gaussian random values with a variance of 1 can be created using the ``randn()`` function.
To give the values a variance of 4, we multiply the random numbers by the standard deviation (the square root of the variance).
The following snippet can be copy/pasted into MATLAB to simulate some data according to our model.
```
beta_gt = [30; 35; 2];
v_gt    = 4;
y       = X*beta_gt + randn(10,1)*sqrt(v_gt);
```

## Estimating Model Parameters
We can estimate the ``beta`` values from the simulated data as we would for real data.
```
beta = X\y
```
After fitting a model like this in SPM, a set of beta_0001.nii, beta_0002.nii and beta_0003.nii images are created that encode
the estimated ``beta`` values at each voxel.

We can also estimate the variance of the data by computing it from the residuals.
This needs the degrees of freedom, which are computed from the length of ``y`` minus the rank of the design matrix ``X``.
This rank is the number of linearly independent columns (or rows), which in this case is 3.
```
nu = size(X,1) - rank(X); % Degrees of freedom
r  = y - X*beta;          % Residuals
v  = sum(r.^2)/nu
```
If you copy and paste the above snippets into MATLAB, you will see that the estimates (``beta`` and ``v``)
are similar to, but not identical to the ground truth (``beta_gt`` and ``v_gt``).

When a statistical analysis is run in SPM, a file entitled ResMS.nii is created, which contains information about the variance estimated at each voxel.

## Plotting
We can plot the (simulated) data, along with the model fit, by
```
plot((1:10)', y, 'k.',  (1:10)', X*beta, 'b-')
xlabel('Time')
ylabel('Signal')
legend('Data','Estimated fit')
```

A plot of the data and model fit might be clearer without the linear trend in signal intensity.
We can adjust what is plotted by subtracting the estimated linear trend.
```
y_adj   = y - X(:,3)*beta(3);
fit_adj = X(:,1:2)*beta(1:2);
plot((1:10)', y_adj, 'k.',  (1:10)', fit_adj, 'b-')
xlabel('Time')
ylabel('Adjusted signal')
legend('Adjusted data','Adjusted fit')
```

## Contrast Vectors and Statistics
With fMRI, what we are typically interested in is the difference between BOLD signal during the task and BOLD signal during the baseline.
This can be computed from the estimated ``beta``, and is simply ``beta(2)-beta(1)``.
Another way to think about this is as ``-1*beta(1) + 1*beta(2) + 0*beta(3)``.
The weights ``[-1 1 0]`` give us the linear combination of ``beta`` values that are relevant to our question.
We call this a "contrast vector". If this is called ``c``, then the linear combination of ``beta`` values is given by
```
c   = [-1 1 0];
con = c * beta
```
In SPM, the above would generate a con_???.nii file that contains the specified linear combination of ``beta`` values.

### T Statistic
p values are based on probabilities of observing the data under the null hypothesis.
With one sided t tests, this would be the probability of obtaining a value of ``con`` as high or higher than
the one actually obtained.
To do this, we need ``c * beta``, as well as the estimated standard deviation of ``c * beta`` (the standard error).
If we divide one by the other, we obtain the t statistic.
```
t = (c * beta) / sqrt(v * c * inv(X'*X) * c')
```
Without any correction for multiple comparisons, we can obtain the p value for this t statistic by
```
p = spm_Tcdf(-t,nu)
% or
p = 1-spm_Tcdf(t,nu)

```


### F Statistic
F tests are a bit harder to explain than t tests.
One concept we need is the idea of a null space.

First though, we need to understand orthogonality.
If two row vectors, say ``c`` and ``d``, are orthogonal, then ``c * d' == 0``.
We can try this by copy/pasting the following into MATLAB:
```
d = [1 1 0];
d = d/sqrt(d*d'); % Makes the length of d equal to 1 (Pythagorus)
c*d'

e = [0 0 1];
c*e'
```
We also note that in the above example, ``d * e' == 0``.
Because the vectors contains three elements, they represent directions in 3D and can be visualised by:
```
plot3([-c(1) c(1)],[-c(2) c(2)],[-c(3) c(3)],'k', [-d(1) d(1)],[-d(2) d(2)],[-d(3) d(3)],'c', [-e(1) e(1)],[-e(2) e(2)],[-e(3) e(3)],'m')
axis image
rotate3d on
```
If you manually rotate the plot by dragging the curser around on it, then you should see that there is a right angle between all vectors. 
We can say that ``d`` and ``e`` form the null space of ``c``.
In MATLAB, we can obtain the null space for ``c`` by ``Cn = null(c)``.
This can be used to remove information related to the contrast of interest from the design matrix by:
```
X0 = X*null(c)
```

An F test is done by comparing the amount of variance in ``y`` that cannot be explained by the full design matrix ``X``,
against how much of it cannot be explained by the reduced matrix ``X0``.
TO do this, we need to know the degrees of freedom, which for F tests have two values.
The first is the degrees of freedom in the part of the design matrix of interest (1 in this example).
The second is the same value as for the t test (7 in this example).
```
nu  = [rank(X)-rank(X0), size(X,1)-rank(X)]
```

The residuals and degrees of freedom are used to give two variance estimates:
```
r  = y - X*(X\y);
v  = sum(r.^2)/nu(2)
r0 = y - X0*(X0\y);
v0 = (sum(r0.^2) - sum(r.^2))/nu(1)
```

The ratio of these two variances gives the F statistic.
```
F = v0/v
```

Similar to the t test, we can compute a p value from this F statistic by
```
p = 1 - spm_Fcdf(F, nu)
```

You should notice that the p value given from the F statistic is twice as large as that from the t statistic.
The reason for this is that the t static is based on a one-tailed t test,
whereas the p value from the F statistic is equivalent to that from a two-tailed t test.

## Temporal correlations
Suppose we were to double the lengths of ``y`` and ``X`` by replicating their values.
One way to do this is
```
T  = kron(eye(size(y,1)),[1 1]');
y2 = T*y;
X2 = T*X;
```
We can compute a t statistic from these by copy/pasting the following into MATLAB.
```
beta2 = X2\y2
nu2 = size(X2,1) - rank(X2);
r2  = y2 - X2*beta2;          % Residuals
v2  = sum(r2.^2)/nu2
t2 = (c * beta2) / sqrt(v2 * c * inv(X2'*X2) * c')
p2 = 1-spm_Tcdf(t2,nu)
```

You should notice that the new p value ``p2`` is much smaller than the one from the original t test.
We should not be able to get extra statistical significance like this for free, so what is wrong?

The answer is that the statistical analysis does not account for the noise no longer being i.i.d.
There is now covariance in the data.
If we know this covariance, then it can be included within the various matrix operations of the statistical analysis
to correct it.
In fMRI, there are temporal correlations in the time series noise, which needs to be accounted for when doing statistical analyses within subject.




You don't need to know or understand the details, but this could be done as follows:
```
V     = T*T';                                               % Covariance matrix
L     = pinv(V);                                            % Precision matrix
beta2 = (X2' * L * X2) \ X2'* L *y2                         % Parameter estimates
R     = L*(eye(size(X2,1)) - X2*inv(X2'*L*X2)*X2'*L);       % Residual forming matrix
nu2   = trace(R*V);                                         % Degrees of freedom
v2    = y2'*R*y2/nu2                                        % Variance
t2    = (c * beta2) / sqrt(v2 * c * inv(X2' * L * X2) * c') % t statistic
p2    = 1-spm_Tcdf(t2, nu2)                                 % p value
```
An easier way would be to "pre-whiten" the data and design matrix.
```
y3 = pinv(T)*y2;
X3 = pinv(T)*X2;
```


