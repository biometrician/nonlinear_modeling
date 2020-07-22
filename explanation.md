Visualisation of non-linear modeling
=====================================


## Aim of this shiny applicaton
For eleven different explanatory variables, the user can visualise non-linear effects applying **fractional polynomials**, **natural (restricted cubic) splines**, or **linear b-splines**. For each method the relevant parameters can be changed and changes in the non-linear effect can be graphically assessed. Users can visualise first- or second-degree fractional polynomials, linear linear b-splines with 4 degrees of freedom and  natural splines with 3 degrees of freedom. Additionally, if one assumes a specific residual standard error, one can visualise how the outcome $y$ could look like. Are such data realistic?
               
## Data used in this app
The data used to vizualize non-linear modeling stem from the [National Health and Nutrition Helath Survey (NHANES)](<https://www.cdc.gov/nchs/nhanes/index.htm>). For each variable one thousand randomly selected men and women are used.
               
***

## Modeling non-linear effects


### 1. The linearity assumption

Recall the definition of a linear regression model: $y = \beta_0 + \beta_1 x + \epsilon$, 

where $y$ is the outcome, $\beta_0$ is the intercept, $\beta_1$ is the regression coefficient of the explanatory variable $x$ and $\epsilon$ is the error term of the model. 
The linear regression model is equivalent to $E(Y) = \beta_0 + \beta_1 x$ with $E(Y)$ being the expected value of $y$.

**The linearity assumption states that with each one-unit difference in $x$, there should be a $\beta_1$ difference in $y$.** Put mathematically, $\partial E(Y) / \partial x = \partial (\beta_0 + \beta_1 x)/\partial x = \beta_1$.

*But what actually happens if this assumption is violated?* **A violation of the linearity assumption would imply that in different regions of $x$ the impact of $x$ is not $\beta_1$, but larger or smaller than on average. This means that inclusion of $x$ as a single explanatory variable is not sufficient to model the assocation with $y$.** 

In medical applications, the linearity assumption is frequently violated.



### 2. Relaxing the linearity assumption

If the association between the explanatory variable $x$ and the outcome $y$ is obviously not linear, one can relax the linearity assumption. The above mentioned model can be extended to accomdate non-linear effects using polynomials. A polynomial of a continous variable $x$ of degree $k$ is defined as $x, x^2, \ldots, x^k$. If such a polynomial of is used to represent the variable in a regression model, then for each term of the polynomial a separate regression coefficient is estimated, and the dependent variable is then modeled by $E(Y) = \beta_0 + \beta_1 x + \beta_2 x^2 + \ldots + \beta_k x^k$. $x$ (=$x^1$), $x^2$ and $x^3$ are called base functions of $x$. For a raw polynomial of order 3 of a variable $x$, they can be computed by, e.g.:

| Variable value $x$ | $x^1$ | $x^2$ | $x^3$
|:---------------------:|:--------:|:-------:|:--------:|
|1 | 1 | 1 | 1 |
|2 | 2 | 4 | 8 |
|3 | 3 | 9 | 27 
|... | ... | ... | ... |

The `poly()` function is a convenient way to compute polynomial terms in R. 

Such a model is now flexible enough, even with $k$ chosen as 2 or 3, to relax the assumption that each increase in $x$ is associated with a difference in $y$ of $\beta_1$. Note, the regression model described above is still a linear model, despite that it provides a non-linear function of the explanatory variable.


In the last decades, a couple of techniques were developed that allow in very flexible ways to relax the linearity assumption. Among them are **fractional polynomials**, and various types of 'splines', like **restricted cubic splines** or **B-splines**. The use of these methods allow investigation of non-linear effects of continuous variables. In all these proposals, the continuous variable $x$ is first transformed into a few base functions, and the base functions are then used for modeling. 

#### Fractional polynomials

Fractional polynomials select one or two base functions from a predefined catalogue of eight possible transformations, which is:

$\{x^{-2}, x^{-1}, x^{-1/2}, log(x), x^{1/2}, x, x^{2}, x^{3}\}$. 

The so-called 'powers' ($-2, -1, ..., 3$) in those transformations are selected by using those one or two transformations that yield the best model fit. Models requiring one power are called first-degree (FP1) function and models requirung two powers are second-degree (FP2) functions. If the same power is selected twice (i.e., ‘repeated powers’), e.g. x^{2} is selected twice, the FP2 function is defined as $\beta_1 x^{2} + \beta_2 log(x)$. This defines 8 FP1 and 36 FP2 models. These 44 models are sufficient for most biomedical applications as they include very different types of non-linear functions. 

A suitable pretransformation is applied to make the base functions independent, but for pragmatic reasons the pretransformation will only shift the values to the positive range (if necessary) and then divide them by a multiple of 10. In this way, the pretransformation stays simple enough to be written down in a report.  

To select the best model among those 44 candidates, a closed testing procedure, called the function selection procedured (FSP) has be introduced:

1. Test of overall association of the outcome $y$ with $x$: The best-fitting FP2 model for $x$ at the $\alpha$ level is tested against a model without $x$ (using four degrees of freedom). If the test is not significant, stop, concluding that $x$ is not significant at $\alpha$. $x$ is not included in the final model. Otherwise continue.
2. Test the evidence of non-linearity: The best-fitting FP2 model for $x$ is tested against a straight line (linear function) at the $\alpha$ level (using 3 degrees of freedom). If the test is not significant, stop, concluding that the linear function of $x$ is sufficient. Otherwise continue.
3. Test between a simple or a more complex non-linear model: The best-fitting FP2 model for $x$ is tested against the best-fitting FP1 model at the $\alpha$ level (using two degrees of freedom). If the test is not significant, the final model includes the best-fitting FP1 function, otherwise the final model includes the best-fitting FP2 function. 

This procedure preserves the overall (i.e. familywise) type I error probability at a chosen $\alpha$ level. A typical choice for the nominal $p$-value is $\alpha = 0.05$.

The approach is implemented in the R package `mfp`, in which it can be combined with variable selection and used with continuous, binary or time-to-event outcome variables.

For more information on fractional polynomials we refer to the [multivariable fractional polynomials website](<http://mfp.imbi.uni-freiburg.de/fp>).

#### Splines
Splines are a common method of modelling nonlinear effects of continuous variables. A spline function is defined by a set of piecewise polynomial functions of a continuous variable that are joined smoothly at a set of knots. 

In [Perperoglou A, Sauerbrei W, Abrahamowicz M, Schmid M. BMC Med Res Methodol. 2019](<https://bmcmedresmethodol.biomedcentral.com/articles/10.1186/s12874-019-0666-3/>) the idea of splines is very eloquently explained as: *The term ‘spline’ refers to a craftsman’s tool, a flexible thin strip of wood or metal, used to draft smooth curves. Several weights would be applied on various positions so the strip would bend according to their number and position. This would be forced to pass through a set of fixed points: metal pins, the ribs of a boat, etc. On a flat surface these were often weights with an attached hook and thus easy to manipulate. The shape of the bended material would naturally take the form of a spline curve. Similarly, splines are used in statistics in order to mathematically reproduce flexible shapes. Knots are placed at several places within the data range, to identify the points where adjacent functional pieces join each other. Instead of metal or wood stripes, smooth functional pieces (usually low-order polynomials) are chosen to fit the data between two consecutive knots. The type of polynomial and the number and placement of knots is what then defines the type of spline.*

[This paper](<https://bmcmedresmethodol.biomedcentral.com/articles/10.1186/s12874-019-0666-3/>) also includes a simple-to-read review of splines methodology and splines functions procedures in R.

##### Linear b-splines

B-splines transform the original variable $x$ into base functions which are greater than 0 for specific subranges of $x$ and 0 otherwise. The number of degrees of freedom defines the number of base functions, and the degree defines the type of transformation. The subranges are defined by the location of so-called 'knots'. These are usually set automatically, but are part of the definition of the spline transformation. 
The individual spline bases have no simple interpretation and can only be used together as a set of independent variables repesenting variable $x$ in a regression model.

##### Restricted (natural) cubic splines

Restricted cubic splines (also known as natural splines) are cubic transformations (third-order polynomials) of the explanatory variable $x$ in the interior of its range (within the outermost knots), and are linear at the edges (outside the outermost knots). Typically, a small number of knots (e.g., 3 to 5) is sufficient to model most data in biomedical applications. Knots are located at various percentiles of $x$. For example, Harrell recommends in his book (Harrell Jr. FE. Regression modeling strategies. With applications to linear models, logistic regression, and survival analysis. Springer 2015, chapter 2.4.6) the following settings:

number of knots | percentiles |
----------------|:------------:|
3 | 0.10 0.5 0.90 |
4 | 0.05 0.35 0.65 0.95 |
5 | 0.05 0.275 0.5 0.725 0.95 |

B-splines and restricted (natural) cubic splines are implemented in the R package `splines`.