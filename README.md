# anomaly-detector
A streaming anomaly detection system built with Oryx.

# Robust PCA
This is a work-in-progress for a time series anomaly detector based
on Robust Principal Component Analysis. Robust Principal Component Analysis
seeks to decompose a matrix into a low rank component and a sparse component.
The sparse component is the part of interest for us, those are the anomolous
inputs! We will find this decomposition using the principal component pursuit
algorithm defined in [Candes Et al.](http://statweb.stanford.edu/~candes/papers/RobustPCA.pdf)
and [here](http://perception.csl.illinois.edu/matrix-rank/Files/RPCA_JACM.pdf).

The gist of principal component pursuit is that we are trying to decompose a matrix M in to its
low rank component L and its sparse component S, M = L + S. Traditional PCA is easily corrupted by
outliers in the matrix. In PCA used for dimensionality reduction we usually think of retrieving
the low rank approximation where M = L + N, L is the low rank approximation fo M using the top K
singular vectors and N is made up of small noise. In the sparse situation, components of S can
have arbitrarily large magnitude but are distributed among the indices of the matrix in a sparse
way.

## Robust PCA for outlier detection

We can follow the form outlined in [Robust Feature Selection and Robust PCA for Internet Traffic
Anomaly Detection](http://211.68.127
.238/classes/For11Course/class14/1/POVF12.pdf) on page 6.

0) Form the matrix of observations. Eventually, this implementation should include robust feature
selection.
1) Compute the first k approximation of the robust PCs and their variances.
2) Calculate the Score Distance (Mahalanobis distance in the pc space) and the Orthogonal Distance
 (error after subtracting projection of input onto low dim pc space).
3) Calculate appropriate thresholds for the score and orthogonal distances.
4) Label an observation as an anomaly if either the score distance of orthogonal distance for that
observations exceeds the corresponding threshold.


## Computing Robust Principal Components

We do this decomposition by framing it s an optimization problem where we seek to minimize
||L||_{Nuclear Norm} + \lambda||S||_{1}
subject to the constraint that L+S=M. This can be solved with computationally intensive
optimization methods. We can get updates using this method at long intervals
but this does not solve the problem of estimating the anomolous-ness of an observation on the fly
or keeping our estimate of the principal components up to date.


# How is this done in Oryx?

##Streaming Robust PCA
[Real-time Robust Principal Components Pursuit](http://arxiv.org/pdf/1010.0608v3.pdf)
describes an algorithm for streaming sparse signal detection as well as a method for
continuously updating your estimates of the singular values. It seems best to build this
incrementally, first building the batch and serving layers and seeing how that works on
real data. Then implementing noise corrected robust pca as well as the continuous model
update portion of the algorithm if we need improvements or are having trouble separating the
sparse component from the low rank one.

The current focus of this implementation is on batch model building and streaming scoring. This
may be extended later.

## Batch Layer Update
The batch layer will need to compute robust estimates of principal components for the data. We
should also do some tuning to determine the best threshold for both distance measures.

## Serving Model Manager
The model applied at serving time will need to be able to take an observation, calculate the score
 distance and the orthogonal distance using the precomputed principal components. It will then
 need to compare these distances to their corresponding thresholds.

## Speed Model Manager
Not worrying about this yet.
