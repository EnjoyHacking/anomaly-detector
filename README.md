# anomaly-detector
A streaming anomaly detection system built with Oryx.

# Robust PCA
This is a work-in-progress for a time series anomaly detector based
on Robust Principal Component Analysis. Robust Principal Component Analysis
seeks to decompose a matrix into a low rank component and a sparse component.
The sparse component is the part of interest for us, those are the anomolous
inputs! We will find this decomposition using the principal component pursuit
algorithm defined in [Candes Et al.](http://statweb.stanford.edu/~candes/papers/RobustPCA.pdf).

The gist of principal component pursuit is that we are trying to decompose a matrix M in to its
low rank component L and its sparse component S, M = L + S. Traditional PCA is easily corrupted by
outliers in the matrix. In PCA used for dimensionality reduction we usually think of retrieving
the low rank approximation where M = L + N, L is the low rank approximation fo M using the top K
singular vectors and N is made up of small noise. In the sparse situation, components of S can
have arbitrarily large magnitude but are distributed among the indices of the matrix in a sparse
way.

We do this decomposition by framing it s an optimization problem where we seek to minimize
||L||_{Nuclear Norm} + \lambda||S||_{1}
subject to the constraint that L+S=M. This can be solved with computationally intensive
optimization methods. We can get updates using this method at long intervals
but this does not solve the problem of estimating the sparse component on the fly,
or keeping our estimates of the principal directions up-to-date.

[Real-time Robust Principal Components Pursuit](http://arxiv.org/pdf/1010.0608v3.pdf)
describes an algorithm for streaming sparse signal detection as well as a method for
continuously updating your estimates of the singular values. It seems best to build this
incrementally, first building the batch and serving layers and seeing how that works on
real data. Then implementing noise corrected robust pca as well as the continuous model
update portion of the algorithm if we need improvements or are having trouble separating the
sparse component fromt he low rank one.

## Batch Layer Update

## Serving Model Manager

## Speed Model Manager
