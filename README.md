# UWOT

An R implementation of the 
[Uniform Manifold Approximation and Projection (UMAP)](https://arxiv.org/abs/1802.03426) 
method for dimensionality reduction (McInnes and Healy, 2018).

Current status: probably working, but not as efficient as it could be. Also, no 
documentation yet.

## Installing

```R
install.packages("devtools")
devtools::install_github("jlmelville/uwot")
library(uwot)
```

## Example

```R
iris_umap <- umap(iris, n_neighbors = 50, alpha = 0.5, init = "random")

# Load mnist from somewhere, e.g.
# devtools::install_github("jlmelville/snedata")
# mnist <- snedata::download_mnist()
mnist_umap <- umap(mnist, n_neighbors = 15, min_dist = 0.001, verbose = TRUE)
```

## Implementation Details

For small (N < 4096), exact nearest neighbors are found using the 
[FNN](https://cran.r-project.org/package=FNN) package. Otherwise, approximate
nearest neighbors are found using 
[RcppAnnoy](https://cran.r-project.org/package=RcppAnnoy).

Coordinate initialization uses
[RSpectra](https://cran.r-project.org/package=RSpectra) to do the
eigendecomposition of the normalized Laplacian.

The main optimization loop is written in C++ (using 
[Rcpp](https://cran.r-project.org/package=Rcpp) and 
[RcppArmadillo](https://cran.r-project.org/package=RcppArmadillo)), aping
the Python code as closely as possible. It is my first time using Rcpp, so 
let's assume I did a horrible job.

For the datasets I've tried it with, the results look at least
reminiscent of those obtained using the 
[official Python implementation](https://github.com/lmcinnes/umap).
Below are results for the 70,000 MNIST digits (downloaded using the
[snedata](https://github.com/jlmelville/snedata) package). On the left
is the result of using the official Python UMAP implementation 
(via the [reticulate](https://cran.r-project.org/package=reticulate) package).
The right hand image is the result of using `uwot`.

|                                    |                                  |
|------------------------------------|----------------------------------|
| ![mnist-py.png](mnist-py.png)      | ![mnist-r.png](mnist-r.png)      |

On my not-particularly beefy laptop `uwot` took around 12 minutes, while 
the Python UMAP implementation took just under 11 minutes. For comparison, 
the default settings of the R package for
[Barnes-Hut t-SNE](https://cran.r-project.org/package=Rtsne) took 21 minutes, and the
[largeVis](https://github.com/elbamos/largeVis) package took 56 minutes.

## Limitations

* Only Euclidean distances are supported for finding nearest neighbors. You can
pass in a `dist` object instead of a data frame. Sparse matrices are not yet 
supported.
* The smooth k-nearest-neighbor routine (this is separate from the nearest 
neighbor search itself) is currently written in pure R, so is unnecessarily slow.

## License

[GPLv3 or later](https://www.gnu.org/licenses/gpl-3.0.txt).

## See Also

* The [UMAP](https://github.com/lmcinnes/umap) reference implementation and
[publication](https://arxiv.org/abs/1802.03426).
* Other R packages for UMAP: https://github.com/ropenscilabs/umapr and 
https://github.com/tkonopka/umap
