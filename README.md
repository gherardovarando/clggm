
# gclm

`gclm` contains methods to estimate a sparse parametrization of
covariance matrix as solution of a continuous time Lyapunov equation
(CLE):

\[ B\Sigma + \Sigma B^t + C = 0 \]

Solving the following \(\ell_1\) penalized loss minimization
problem:

\[ \arg\min L(\Sigma(B,C)) + \lambda \rho_1(B) + \lambda_C ||C - C_0||^2_F   \]

subject to \(B\) stable and \(C\) diagonal, where \(\rho_1(B)\) is the
\(\ell_1\) norm of the off-diagonal elements of \(B\) and
\(||C - C_0||^2_F\) is the squared frobenius norm of the difference
between \(C\) and a fixed diagonal matrix \(C_0\) (usually the
identity).

## Installation

``` r
devtools::install_github("gherardovarando/gclm")
```

## Usage

``` r
library(gclm)

### define coefficient matrices
B <- matrix(nrow = 4, c(-4, 2,   0,  0, 
                         0, -3,  1,  0,
                         0,  0, -2,  0.5,
                         0,  0,  0, -1), byrow = TRUE)
C <- diag(c(1,4,1,4))

### solve continuous Lyapunov equation 
### to obtain real covariance matrix
Sigma <- clyap(B, C) 

### obtain observations 
sample <- MASS::mvrnorm(n = 100, mu = rep(0,4),Sigma =  Sigma)


### Solve minimization

res <- gclm(cov(sample), lambda = 0.3, lambdac = 0.01)

res$B
```

    ##            [,1]       [,2]       [,3]       [,4]
    ## [1,] -0.9864528  0.2502037  0.0000000  0.0000000
    ## [2,]  0.0000000 -0.6438493  0.0000000  0.0000000
    ## [3,]  0.0000000  0.0000000 -0.9091838  0.1831593
    ## [4,]  0.0000000  0.0000000  0.0000000 -0.3209837

``` r
res$C
```

    ## [1] 0.2803356 0.7787710 0.4599322 1.3226940

The CLE can be freely multiplied by a scalar and thus the \(B,C\)
parametrization can be rescaled. As an example we can impose
\(C_{11} = 1\) as in the true \(C\) matrix, obtaining the estimators:

``` r
C1 <- res$C / res$C[1]
B1 <- res$B / res$C[1]

B1 
```

    ##           [,1]       [,2]      [,3]       [,4]
    ## [1,] -3.518828  0.8925148  0.000000  0.0000000
    ## [2,]  0.000000 -2.2967090  0.000000  0.0000000
    ## [3,]  0.000000  0.0000000 -3.243198  0.6533573
    ## [4,]  0.000000  0.0000000  0.000000 -1.1449982

``` r
C1
```

    ## [1] 1.000000 2.777995 1.640649 4.718252

#### Solutions along a regularization path

``` r
path <- gclm.path(cov(sample), lambdac = 0.01)
t(sapply(path, function(res) c(lambda = res$lambda, 
                                npar = sum(res$B!=0),
                                fp = sum(res$B!=0 & B==0),
                                tp = sum(res$B!=0 & B!=0) ,
                                fn = sum(res$B==0 & B!=0),
                                tn = sum(res$B==0 & B==0),
                                errs = sum(res$B!=0 & B==0) + 
                                  sum(res$B==0 & B!=0))))
```

    ##          lambda npar fp tp fn tn errs
    ##  [1,] 1.0154926    4  0  4  3  9    3
    ##  [2,] 0.9026601    4  0  4  3  9    3
    ##  [3,] 0.7898276    5  0  5  2  9    2
    ##  [4,] 0.6769951    5  0  5  2  9    2
    ##  [5,] 0.5641625    5  0  5  2  9    2
    ##  [6,] 0.4513300    6  0  6  1  9    1
    ##  [7,] 0.3384975    6  0  6  1  9    1
    ##  [8,] 0.2256650    7  1  6  1  8    2
    ##  [9,] 0.1128325    7  1  6  1  8    2
    ## [10,] 0.0000000   16  9  7  0  0    9

## Related code

  - Some inspiration is from the `lyapunov` package
    (<https://github.com/gragusa/lyapunov>).
