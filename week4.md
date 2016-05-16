

# Regression Models: Week 4:  Generalized Linear Models


## Generalized Linear Models

A GLM involves 3 components:

1. an exponential family model for the response (linear,log,etc)
2. a systematic component via a linear predictor
3. a link function that connects the means of the response to the linear predictor
    * lm:  Y = B0 + B1 * X + e
    * glm: g(Y) = B0 + B1 * X + e
    * g is the "link function" that connects...
        * the linear prediction (B0 + B1*X)
        * to the actual response variable (Y)

TODO: need more here

### E.g: linear models:

1. "Normal" distribution of error term
2. linear predictor (ata) = X1 * B1 + X2 * B2 + ...
3. link function g(u) = linear predictor


### E.g: logistic models:

* Binary Logistic Model predicts binary (bernoulli) outcomes
* "logistic" typically refers to "binary logistic model"
* predicted variable is bernoulli (binary outcome)
* the model gives the **probability** of the "success" outcome
* probability is ranged from 0 to 1, 
    * we use the logistic function to translate model output into the range 0-1

.

    Logistic function: f(x) =   L 
                               -------------------
                                 1 + e^-k(x-x0) 

* L = maximum value (for probabilities, L=1)
* k = steepness of curve
* x0 = midpoint (inflection point of s curve)
* transforms all values of x form -inf to +inf to the range 0 -> L


### For regression models:


    f(x) =    1 
           --------
           1 + e^-t

* t is a linear function of regressor variable x
* t = B0 + B1 * x

.

    F(x) =      1 
           -----------------
            1 + e^-(B0+B1x)

* F(x) is the probability of the predicted value being "success"

.

    # logit "log odds"
    ln [ F(x) / 1 - F(x) ] = B0 + B1x

    # Note: F(x) / 1 - F(x) = e^B0+B1x

    odds = e^B0+B1x

    odds ratio = odds(x+1) / odds(x) = e^B1
    # the odds multiply by e^B1 for every unit increase in x


####  E.g. Bernoulli Response Variables

* Y ~ Bernoulli(u) (binomial?)
* E[Y] = u
* 0 <= u <= 1

The GLM is:

1. ?
2. linear predictor ata = X1 * B1 + X2 * B2 + ...
3. link function g(u) = log( u / 1-u ) == "logit"



#### E.g: Poisson Response Variables

* assume Y ~ Poisson(u)

The GLM is:

1. ?
2. linear predictor ata = X1 * B1 + X2 * B2 + ...
3. link function g(u) = log( u )


#### TODO: quasi-models, quasi-likelihood

Relaxes the variance parameter by multiplying it by a constant phi term


R Example:

    # family="binomial" gets you a logistic regression by default
    logistic.model <- glm( y ~ x, family="binomial") 

    # coefficients are in log scale
    # exponentiate them to get them back in normal scale and get "odds ratios"
    exp(logistic.model$coeff)
    exp(confint(logistic.model))
    
    anova(logistic.model, test="Chisq")



## Poisson regression 

* aka "log-linear" model
* assume Y ~ Poisson(u)

The GLM is:

1. ?
2. linear predictor ata = X1 * B1 + X2 * B2 + ...
3. link function g(u) = log( u )
    * i.e. the logarithm of the predicted value...
    * can be modeled after the linear combination of regressor variables
    * `log(E[Y|x]) = B0 + B1X`


The model predicts the expected value (lambda) of the poisson-distributed response variable,
given a particular value of the regressor variable(s).

Technically this is true for every model.

    lm( y ~ x ) ==> E[Y|x]

Linear: 

    Y = B0 + B1 * X + E
        ..or..
    E[Y|X=x] = B0 + B1 * x


Poisson/log-linear:

    log( E[Y|X=x] ) = B0 + B1 * x

    # .. or ..
    E[Y|X=x] = e^(B0 + B1 * x)
    E[Y|X=x] = e^(B0) * e^(B1 * x) 

    # note: log(E[Y]) is different than E[ log(Y) ]


### Interpret coefficients:

* e^B0: E[Y] when X=0
* e^B1: relative increase in E[Y] per unit increase in X
    * E[Y] is multiplied by e^B1 per unit increase in X
    * "relative" increase



### Modeling RATES using offsets


    log( E[Y|X=x] ) = B0 + B1 * x
    
    E[Y|X=x] = e^(B0 + B1 * x)
    E[Y|X=x] = e^(B0) * e^(B1 * x) 
    
Now introduce rate: t

    E[Y|X] / t = e^(B0 + B1 * X)            # <-- different model
    log( E[Y|X ) - log(t) = B0 + B1 * x
    log( E[Y|X ) = log(t) + B0 + B1 * x

Coefficients are reinterpreted as "<whatever> per t"

R Example:

    glm(count ~ x + offset(log(t)), family = poisson)
    # means: E[count] / t, the count per unit of time t


Now imagine:

    glm(count ~ x + offset(log(t2)), family = poisson) 

* where log(t2) <- log(10) + log(t)?
    * note: log(a*b) = log(a) + log(b)
    * so t2 = 10 * t
    * i.e the rate unit is smaller (think of per day (t) -> per hour (t * 24)
* means: E[count] / t2
    * coefficients are adjusted by subtracting log(10) 


R Example:

    pm <- glm( y ~ x, family="poisson")
    
    pm.rates <- glm( y ~ x, family="poisson", offset=log(x1+1) )
    # offset: converts output (y) to rates
    # offset = log( <denominator-of-rate-equation> )
    # so this gives output in terms of y / (x1+1)
    # note: the +1 is added to handle 0 values in x1 (log(0) is undefined)



## Knots - Discontinuous Linear Models


    Y = B0 + B1 * X + (X-knot)+ * Gk
      = B0 - knot * Gk + (B1 + Gk) * X

* changes the intercept by knot * Gk (0 if knot=0)
* changes the slope by Gk


R Example:

    x <- -5:5
    y <- c(5.12, 3.93, 2.67, 1.87, 0.52, 0.08, 0.93, 2.05, 2.54, 3.87, 4.97)
    splineTerms <- (x > 0) * x
    lm(y ~ x + splineTerms)
    ggplot() + geom_point(aes(y=y, x=x)) + geom_line(aes(y=predict(m), x=x))



