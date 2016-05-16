

# Regression Models: Week 2: Simple Linear Models


## Linear model:

    Yi = B0 + B1 * Xi + Ei

* B0 = y-intercept
* B1 = correlation factor between changes-in-X and changes-in-Y
* Ei is assumed to be iid N(0,sd^2)
    * iid: "independent and identically disributed"
    * i.e. that the errors (residuals) are normally distributed
    * if they weren't, that would suggest patterns/correlations in the data that we're not accounting for
* NOTE: don't confuse Ei (random variable for errors/residuals) with E[X] (expected value)
* `E[Ei] = 0`, because it's (assumed to be) normally distributed with mean=0

.

    E[Yi | Xi = xi] = ui = B0 + B1 * xi     # because we're conditioning on x, and E[Ei] = 0
    Var(Yi | Xi = xi) = sd^2                # TODO
    
    ^B1 = Cor(Y,X) sd(y) / sd(x)
    ^B0 = Y' - B1 * X'                      # at the means of X and Y

The hats ^ are the estimates.
Theoretically there are REAL values for B0 and B1
but we don't know them, cuz we only have a sample to estimate from



## Interpreting coefficients 

* B0 is the expected response (Y) when the predictor (X) is 0
    * sometimes this isn't useful because predictor can't be 0
    * so shift it by X' ("centering" the variable)
* B1 is the expected change in response (Y) for a 1 unit change in the predictor (X)
* Note: if you want to change the units of X, need to adjust the coefficient:
    * `Yi = B0 + B1 * Xi + Ei   =   B0 + B1/a * Xi * a + E1`

R Example: 

    library(usingR)
    data(diamond)
    fit <- lm(price ~ carat, data=diamond)
    - non-useful intercept, so let's shift:
    fit2 <- lm(price ~ I(carat ~ mean(carat)), data=diamond)
    
    newx <- c(0.16, 0.27, 0.34)
    predict(fit, newdata = data.frame(carat=newx))
    summary(fit)


## Residuals


* `^Yi = ^B0 + ^B1 * Xi`
    * ^Yi = estimate based on the model
* `ei = Yi - ^Yi` 
    * ei = residual = diff between estimate (^Yi) and actual (Yi)
    * Note: NOT Ei, which is the true error, which we don't know
    * ei is an outcome within the Ei random variable


### Residual variation

    # Linear model
    Yi = B0 + B1 * Xi + Ei

    # Error term (random variable, assumed to be normally distributed)
    Ei ~ N(0,sd^2)

    # Error term population variance, aka residual variance:
    sd^2 =  SUM(1..n)[ ei^2 ]   
            -----------------
                    n

    # Residual sample variance: 
    sd^2 =  SUM(1..n)[ ei^2 ]
            -----------------
                 n-2        # subtract 2 degrees of freedom for estimated ^B0 and ^B1


R Example:

    e <- resid(fit)
    plot(e)


TODO: heteroskedasticity


## Total Variation and R^2

Total Variation = Residual Variation + Regression Variation

* Total Variation: variance of the response variable Y (alone)
* Residual Variation: variance of the residuals
    * how far off are the predictions from the actuals
* Regression Variation: variance of the regression model
    * Regression Variation = Total Variation - Residual Variation
* R^2: compares Residual Variation with Total Variation
    * "goodness of fit"
    * if small residual variation, then R^2 will be high (close to 1)
        * indicates the model is a good fit of the actual data
    * if large residual variation, then R^2 will be low (close to 0)
        * indicates the model is not such a good fit of the actual data
        * lots of eror in the predictions

.

    R^2 = % of total variation that's explained by the model

    R^2 = Regression Variation
          --------------------
            Total Variation 

    R^2 = (Total Variation - Residual Variation) 
          --------------------------------------
            Total Variation 

    R^2 = 1 -   Residual Variation 
               ----------------------
                Total Variation 

    R^2 = 1 -   SSres           # sum-of-squares(residuals)
               --------
                SStot           # sum-of-squares(total) = sum-of-squares(Y)

    
    # For simple linear models only (one regressor),
    # R^2 = r^2 = the correlation between (X,Y) squared
    R^2 = r^2 = Cor(Y,X)^2 



## Inference in Regression

Consider our simple linear model:

* Yi = B0 + B1 * Xi + Ei
* E ~ N(0,sd^2) - iid normal
* ^B1 = Cor(Y,X) sd(y) / sd(x)
* ^B0 = Y' - ^B1 * X'


#### Estimated coefficients: variance, standard error, and confidence intervals

                                 
    Var(^B1) =          sd^2 
                ------------------------
                 SUM(1..n)[ (Xi-X')^2 ]          

    # where: sd^2 = residual variation?

    # TODO:
    SUM(1..n)[ (Xi-X')^2 ] is sort of Var(X), without dividing by n (sum-of-squares(X))

    so this equation is sort of like the var of the outcome (sd^2)
    divided by the var of the predictor.


                         1                  X'^2 
    Var(^B0) = sd^2 * { --- + ----------------------------- }
                         n          SUM(1..n)[ (Xi-X')^2 ]


Under iid guassian errors:

    ^B1 - B1
    ----------       follows a t distribution with n-2 degrees of freedom.
    sd(B1)           (note this is true for all coefficients, Bj)



#### Confidence intervals for the coefficients:

    ^B1 +- qt(0.95,n-2) * sd(B1)

* we assume ^B1 is at the center of the t (0 error), 
* the t quantiles gives us the probability distribution of the error
* t is z-scaled, so have to multiply by the sd(B1) to "re-inflate" it into the units of B1


R Example:

    fit <- lm(y ~ x)
    summary(fit) // gives you all the relevant se data: p, R^2, etc.
    
    // or, compute manually:
    beta1 <- cor(y,x) * sd(y) / sd(x)
    beta0 <- mean(y) - beta1 * mean(x)
    e <- y - (beta0 + beta1 * x) // actual - estimate
    // or: e <- resid(fit)
    sigma <- sqrt( sum(e^2) / (n-2) )
    ssx <- sum( (x-mean(x))^2 )
    beta0.se <- sigma * sqrt( 1/n + mean(x)^2 / ssx )
    beta1.se <- sigma / sqrt(ssx)
    beta0.tstat <- beta0 / beta0.se // z-scaling: (beta0 - 0) / beta0.se
    beta1.tstat <- beta1 / beta1.se // where H0 is beta0 == 0, Ha is beta0 != 0
    beta0.p <- 2 * pt(abs(beta0.tstat), df=n-2, lower.tail=F) // mult by 2 for both tails
    beta1.p <- 2 * pt(abs(beta1.tstat), df=n-2, lower.tail=F) 
    // note: pt => the probability of observing beta1.tstat 
    // under the H0 assumption that beta1 = 0 (beta1.tstat = 0)
    
    summary(fit)$coefficients
    Estimate Std. Error t value Pr(>|t|)
    (Intercept) 0.1884572 0.2061290 0.9142681 0.39098029
    x 0.7224211 0.3106531 2.3254912 0.05296439
    
    beta0 <- summary(fit)$coefficients[1,1]
    beta0.se <- summary(fit)$coefficients[1,2]
    beta1 <- summary(fit)$coefficients[2,1]
    beta1.se <- summary(fit)$coefficients[2,2]
    
    // confidence interval for the coefficients:
    beta0 + c(-1,1) * qt(0.975, df=fit$df) * beta0.se
    beta1 + c(-1,1) * qt(0.975, df=fit$df) * beta1.se



## Predicted outcomes: Confidence Intervals

* predicted estimate: y0 = ^B0 + ^B1 * x0
* distinction between confidence intervals:
    * regression line (slope) conf interval
    * outcome (y value) conf interval

.

                                          1              (x0-X')^2 
    line/slope SE at x0: se^2 = sd^2 * { --- + ----------------------------- }
                                          n         SUM(1..n)[ (Xi-X')^2 ]

* similar to Var(^B0)
* sd^2 = residual variance

.

                                                      1           (x0-X')^2 
    prediction/outcome SE at x0: se^2 = sd^2 * { 1 + --- + ----------------------------- }
                                                      n       SUM(1..n)[ (Xi-X')^2 ]


* note that the extra "1" in the prediction interval is essentially the residual se (sd^2 * 1)
* i.e the se of the diffs between ^y and y (estimated and actual)
* as n gets large, the other two terms go to 0, leaving just the se


R Example: 

    fit <- lm(y~x)
    sigma <- summary(fit)$sigma // residual se
    se.line <- sigma * sqrt( 1/ n + (x0 - mean(x))^2 / ssx ) // line. ssx calc'ed above
    se.pred <- sigma * sqrt( 1 + 1/ n + (x0 - mean(x))^2 / ssx ) // prediction interval
    
    beta0 <- summary(fit)$coefficients[1,1]
    beta1 <- summary(fit)$coefficients[2,1]
    
    // predicted value
    yp <- beta0 + beta1 * x0
    
    // prediction interval:
    yp + c(-1,1) * qt(0.975, df=fit$df) * se.pred
    
    // confidence interval (line)
    yp + c(-1,1) * qt(0.975, df=fit$df) * se.line
    
    
    // or...
    predict(fit, newdata=data.frame(x=x0), interval="confidence") // line
    predict(fit, newdata=data.frame(x=x0), interval="prediction") // prediction interval




## Preview: Multi-variable Regression 

    Yi = B1 * X1i + B2 * X2i + ... + BpXpi + Ei

* X1i = 1 typically = "Y-intercept term".
    * not associated with any other regressor variable
* not all models have a Y-intercept term.


#### Least squares minimizes:

    SUM(i=1..n) ( Yi - SUM(k=1..p) Xki * Bk ) ^2

* minimize the diff between..
    * the actual (Yi) 
    * and the predicted (SUM(k=1..p) Xki * Bk) 





