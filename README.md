# anomaly-detector
A streaming anomaly detection system built with Oryx.

# Robust PCA
This is a work-in-progress for a time series anomaly detector based
on Robust Principal Component Analysis. Robust Principal Component Analysis
seeks to decompose a matrix into a low rank component and a sparse component.
The sparse component is the part of interest for us, those are the anomalous
inputs! We will find this decomposition using the principal component pursuit
algorithm defined in [Candes Et al.](http://statweb.stanford.edu/~candes/papers/RobustPCA.pdf)
and [here](http://perception.csl.illinois.edu/matrix-rank/Files/RPCA_JACM.pdf).


## Robust PCA for outlier detection

We can follow the form outlined in [Robust Feature Selection and Robust PCA for Internet Traffic
Anomaly Detection](http://211.68.127
.238/classes/For11Course/class14/1/POVF12.pdf) on page 6:

    0) Form the matrix of observations. Eventually, this implementation should include robust
    feature selection.
    1) Compute the first k approximation of the robust Principal Components (PCs) and their
   variances.
    2) Calculate the Score Distance (Mahalanobis distance in the PC space) and the Orthogonal
    Distance (error after subtracting projection of input onto low dim PC space).
    3) Calculate appropriate thresholds for the score and orthogonal distances.
    4) Label an observation as an anomaly if either the score distance of orthogonal distance for
     that observations exceeds the corresponding threshold.


## Computing Robust Principal Components

The gist of principal component pursuit is that we are trying to decompose a matrix M in to its
low rank component L and its sparse component S, so that M = L + S. Traditional PCA is easily
corrupted by outliers in the matrix. In PCA used for dimensionality reduction we usually think of
retrieving the low rank approximation where M = L + N, L is the low rank approximation fo M
using the top K singular vectors and N is made up of small noise. In the sparse situation,
components of S can have arbitrarily large magnitude but are distributed among the indices of the
matrix in a sparse way.

The first algorithm described for robustly computing the principal components of a matrix in the
presence of sparse noise was framed as an optimization problem where we seek to minimize
    ||L||_{Nuclear Norm} + \lambda||S||_{1}
subject to the constraint that L+S=M. This problem can be solved with optimization methods that
are very computationally intensive.

### Alternating Directions
[Sparse and Low-Rank Matrix Decompositions via Alternating Directions Methods, Yuan and Yang]
(http://www.optimization-online.org/DB_FILE/2009/11/2447.pdf) describes a method for soling the
optimization problem specified above by separating it in to two different minimization problems that
are solved simultaneously.


I think this is the method employed by [RAD in Surus](https://github.com/Netflix/Surus/blob/master/src/main/java/org/surus/math/RPCA.java)

### Croux-Ruiz (CR) Algorithm

The [CR algorithm](http://feb.kuleuven.be/public/NDBAE06/PDF-FILES/pcaPP.pdf) operates on data is stored in a matrix, X ,where the rows are observations
x_{i}
and the columns are variables. We then defined a projection index S where

    a_{1} = arg max_{||a||=1 } S(a^{t}x_{1}, a^{t}x_{2}, ...)

If S is the variance, this statements recovers the top eigenvector. We can continue to apply this
in the orthogonal complement to our eigenspace to construct successive eigenvectors. Variance is
sensitive to outliers in the data, so the idea here is to pick a projection index that is a
more robust measure of variance. There are two common choices:

* Median Absolute Deviation
* First Quartile of Pairwise Distances between all data points

Both of these measure are resilient to outliers in the data, and help us with our aim of
computing a robust pc estimate in the face of sparse anomalous entries.

The next step in the PCA-like algorithm is to subtract off the "center" of the data, where the
"center" is computers in a robust way -- like the L_{1} median or the coordinate-wise median.
Once the data is appropriately centered, this algorithm iteratively builds out its collection of
robustly estomated eigenvector-like vectors a_{i} by maximizing:

    lambda_{k} = max_{set of search directions} S(~ stuff from previous step ~)

Instead of optimizing over the space of all possible solutions, a set of candidate directions are
 chosen and the direction with the max robust variance estimate is chosen.


# How is this done in Oryx?

##Streaming Robust PCA
We can get updates using this method at long intervals
but this does not solve the problem of estimating the anomalous-ness of an observation on the fly
or keeping our estimate of the principal components up to date. [Real-time Robust Principal
Components Pursuit](http://arxiv.org/pdf/1010.0608v3.pdf)
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
Not worrying about this yet. I think [divide and conquer matrix factorization](http://www.cs.berkeley.edu/~ameet/dfc.pdf) or some variant
of it may work well.
