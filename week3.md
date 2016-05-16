


# Regression Models: Week 3:  Multi-Variable Regression


## Multi-variable Regression 

    Yi = B1 * X1i + B2 * X2i + ... + BpXpi + Ei

* X1i = 1 typically = "Y-intercept term".
    * not associated with any other regressor variable
* not all models have a Y-intercept term.
* interpreting coefficients: 
    * B2 is the expected change in Y per unit change in X2
        * **holding all other variables CONSTANT**
    * Bp is the expected change in Y per unit change in Xp
        * **holding all other variables CONSTANT**


#### Least squares minimizes:

    SUM(i=1..n) ( Yi - SUM(k=1..p) Xki * Bk )^2

* minimize the diff between..
    * the actual (Yi) 
    * and the predicted (SUM(k=1..p) Xki * Bk) 


## Residual variation

    Yi = X1i * B1 + X2i * B2 + ... + Xpi * Bp + Ei
    Ei = iid N(0,sd^2)
    ei = Yi - ^Yi = diff between actual and predicted = Yi - ( X1i * ^B1 + X2i * ^B2 + ... + Xpi * ^Bp)

    sd^2 = residual variation = sum[ ei^2 ]
                                ------------
                                    n-p

* (n-p): p is the number of predictor/regressor variables (X's) in the regression model
* represents the number of estimated coefficients (degrees of freedom)
* "residual variation" in a linear model is similar conceptually to the regular ol' variance of a random variable
    * the model tries to explain the response variable (Y) with a line
        * using multiple regressor variables (X's) to estimate the line
    * basically the same as taking the mean of a random variable
        * the mean tries to "explain" the random variable with a line
        * the "mean residuals" are the "errors" between the mean line and the actuals 
    * same can be said for a regression's residuals
        * the "regression model residuals" are the "errors" between the "mean line" (the model) and the actuals


R Example:

    fit <- lm(Y ~ . -1)         # . = include all regressors?  
                                # -1 = exclude y-intercept term
    plot(predict(fit), resid(fit))  # easier to use predicted values in multi-variable regression


## Adjustment

* Add confounding variables to the linear model (i.e.multi-variable regression)
* "adjustment" can cause coefficients to flip signs!
    * e.g. if two regressor variables themselves are correlated
    * we can try to remove the effect of one of those variables
        * consider Y = B1 * X1 + B2 * X2
        * by plotting 
            * y= resid( lm( Y ~ X2 ) )
            * x= resid( lm( X1 ~ X2 ) )
* "adjusted" means include a confounding variable
* "unadjusted" means dont

TODO


## Dummy variables

* for factor (non-continuous) variables
* Each factor *value* gets its own "dummy variable"
* The dummy variable is "ON" (set to 1 or "true")...
    * when the actual outcome (xi) equals...
    * the factor value associated with the dummy variable
    * all other dummy variables are 0 for that factor value
    * i.e. only 1 dummy variable will ever be 1
* Y = B1 * X1 + B2 * X2 + B3 * X3
    * where X1, X2, X3 are dummy variables
    * take values 0 or 1, based on a SINGLE factor value
        * e.g X1 = 1 when FACTOR variable = "A"
        * e.g X2 = 1 when FACTOR variable = "B"
        * e.g X3 = 1 when FACTOR variable = "C"
    * so basically the ^Yi value would be the mean of Y for that factor value
        * which would also be the coefficient
        * cuz ^Yi = B1 * X1, where X1=1
* things change when you add an intercept (B0)
* Y = B0 + B1 * X1 + B2 * X2 + B3 * X3
    * a "base" factor value is used as the intercept 
    * the rest of the X coefficients are tested against the base factor value/coefficient
        * ^Yi = B0 + B1 * X1, where X1=1



## Interpretting a continuous interaction between two variables

* in order to measure the effect of the interaction of..
    * two variables (X1 and X2) on the outcome variable (Y)
    * need to create another regressor variable...
    * that is the multiplication of X1 * X2
    * which contains all info for the interaction
* this new variable gets its own coefficient

.

    Yi = B0 + B1 * X1i + B2 * X2i + B3 * X1i * X2i + Ei

* B0 = the Y intercept (when all X's are 0)
* B1 = change in Y per X1 where X2=0
* B2 = change in Y per X2 where X1=0
* B3 = change in [change in Y per X1] per X2
    * i.e. a change of the change 
    * i.e. change of the slope
    * i.e. second derivative


R Example: 

    # Confounders, factor variables:
    fit <- lm(mpg ~ wt + I(as.factor(cyl)), data=mtcars)
    
    # note: variable order doesn't matter 
    # same as: fit <- lm(mpg ~ I(as.factor(cyl)) + wt, data=mtcars)
    
    
    # Interaction:
    fit <- lm(mpg ~ wt + I(as.factor(cyl)) + I(as.factor(cyl)) * wt, data=mtcars)
    # same as : fit <- lm(mpg ~ I(as.factor(cyl)) * wt, data=mtcars)



## Comparing Models

TODO: Likelihood ratios.

R Example:

    fit <- lm(mpg ~ factor(cyl) + wt, data=mtcars)
    fit2 <- update(fit, mpg ~ factor(cyl) * wt)
    anova(fit, fit2)

    # if p < 0.05, indicates the additional variables (or interactions) 
    # are significant (should be included in the model)



## Leverage, Influence of Outlier data points

* "influence": outliers that are not around the other data points along the predictor (X) axis
    * how does including vs excluding this data point impact the model?
    * dfbetas

* "leverage": outliers that can strongly influence the model
    * leverage measures == hat values
    * only depend on X values, not Y
    * measure *potential* for leverage
    * useful for diagnosing data entry errors 

* so an influential outlier (away from the X cloud) might have low leverage if it conforms to the model
    * i.e. lies nicely along the slope created by the non-outlying data


R Example:

    ?influence.measures
    # hatvalues: measures of leverage
    # resid(fit) / (1 - hatvalues(fit)) == PRESS residuals 
    # "leave one out cross-validation residuals"
    # leave out the ith data point from the model, compare the predicted result for Yi
    
    x <- c(0.586, 0.166, -0.042, -0.614, 11.72)
    y <- c(0.549, -0.026, -0.127, -0.751, 1.344)
    fit <- lm(y ~ x)

    dfbetas(fit)
    # the diff in coefficients when the ith point is deleted

    hatvalues(fit)
    # the "leverage" exerted by each data point



## Model selection

* **omitting variables** 
    * results in bias in the coefficients of interest
    * UNLESS their regressors/variables are uncorrelated with the ommitted ones
    * randomize treatments to uncorrelate the treatment indicator
* **including variables that we shouldnt** 
    * increases standard errors of the regression variables
    * the model tends toward perfect fit as number of non-redundant regressors -> n
    * R^2 increases monotonically as more regressors/variables are included
    * the SSE decreases monotonically as more refressors/variables are included
* TODO: variance inflation
    * worse when variables are correlated with each other
    * when other variables are orthogonal, there's no variance inflation


