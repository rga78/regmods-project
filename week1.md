
# Regression Models: Week 1: Covariance, Correlation, and Linear Least Squares

## Normalization:

* "z-scale"
* translate data points (outcomes/trials/?) into units of standard deviation
    * by subtracting the data point from the mean and dividing by stddev
* consistent comparison across random variables (with possibly different stddevs)

.

    Zi = Xi - X'
         ------
           sd
    
    R: z <- (x - mean(x)) / sd(x)


## Covariance:

* **Covariance** measures how much two random variables "vary together" from their respective means
* calculated similar to variance and sample variance (how one variable varies)
* assumes the random variable outcomes are correlated in some way that makes it worth measuring covariance
    * e.g. two variables measured from the same subject (e.g. the weight and height of a person)
* if X outcomes vary far from the mean X' at the same time Y outcomes vary far from the mean Y'..
    * then X and Y have large covariance
    * which means they are correlated
    * which means there _might_ be a causal relation between them
    
.

    # variance of a full population
    Var(X) =  sum (i=1..n) [ (Xi-X') (Xi-X') ]
              --------------------------------
                            n

    # variance of a sample 
    # account for one degree of freedom (the mean X', which is an estimate)
    Sample Var(X) =  sum (i=1..n) [ (Xi-X') (Xi-X') ]
                     --------------------------------
                                 n-1             

    # Co-variance of a sample
    Sample Cov(X,Y) =  sum (i=1..n) [ (Xi-X') (Yi-Y') ]
                       -----------------------------------
                                  n-1

    # correlation = covariance / stddevs  
    # "normalized", like z-scores
    Cor(X,Y) = Cov(X,Y)
               --------
                sdx sdy


* -1 <= Cor(X,Y) <= 1
* -1,1 is a perfect linear relationship
* 0 implies no linear relationship


## Linear Least Squares

* find the line thru the data such that...
    * the sum of the square diffs between 
        * the actual data point/outcome, `Yi`
        * and the value predicted by the linear model `^Y = B0 + B1*Xi`
        * is MINIMIZED
    * `sum (Yi - B0 + B1*Xi)^2`
* note that the mean of a random variable is the value that..
    * minimizes the square diffs between 
        * the actual data values 
        * and the mean, 
    * aka the variance.

.

    B1 = Cor(Y,X) * sdy / sdx       # units of Y/X
    B0 = Y' - B1 * X'               # units of Y

    ^Y = B0 + Cor(Y,X) * sdy/sdx * X

* Y is the *outcome* variable 
* X is the *predictor/regressor* variable 
* slope (B1) is the same if you centered the data and did regression thru origin
* if you normalize the data, the slope == the correlation
    * B1 = Cor(Y,X)
* the line always passes thru the point X',Y'

.

    # assuming B1= 0
    B0 = Y'

    # assuming B0 = 0,
    B1 = sum(i=1..n) Yi*Xi  =  SUM Y*X
         -----------------     -------
         sum(i=1..n) Xi*Xi     SUM X^2


