**Partial least squares error (PLS) to predict UV filter Concentrations in Foundations Make-up with SPF claim.**

![image](https://github.com/user-attachments/assets/5176271b-eb30-4e96-b424-4f1092242e5c)


When working with product formulation, it leads to huge responsibility and effort, even more so when we don’t have enough time to develop a new formulation or improve an existing one for the company's requirements or the dynamics of the market; moreover, laboratory tests could be expensive and time consuming wich don’t help us to much. Of course, we need them to ensure that our formulations work as designed and are safe for customers, but while designing or enhancing a formulation, screening tests are useful to interact with different formulas' prototypes to obtain results quickly.

In the context of Foundations Make-up with SPF claim, UV filters are the main ingredients we use to give an SPF level, and they are measured by the HPLC technique. And for SPF, there are two main techniques: In-vitro and In-vivo sun protection Factor, but in-vitro SPF is a technique which can be used to measure the UV spectra of the formulations with UV filters. This technique has its limitations, but it is another topic. For now, let’s see how this approach can be implemented to prevent laborious tests in the design process.

I used PLS, which is a great algorithm to process spectra data, and is widely used in the chemometric field. The mdatools R package is the library that allows us to use this technique, I love working with this library because it is very complete and useful.

First, I simulated the data using Python and ensure that the data makes sense for the analysis and interpretation. The spectral data were obtained through the BAFS SPF simulator. Download data

However the idea is to have our own data obtained from our formulations or chemical analysis. This is just a examplo what we could do in order to give us an Idea how to tackle a similar challenge.

This is what data looks like

```{r}
found <- read.csv('found')
found <- found %>% 
  dplyr::mutate_if(is.character, as.factor) 
str(found)

'data.frame': 300 obs. of  123 variables:
 $ FPS                     : num  22.99 13.82 9.62 20.74 23.85 ...
 $ octinoxate              : num  5.53 2.86 2.47 4.58 5.68 ...
 $ zinc_oxide              : num  1.15 7.49 1.07 1.76 3.03 ...
 $ eficacia_octinoxate_hplc: num  95.5 95.6 99.4 98.2 92 ...
 $ eficacia_zno_aa         : num  97.9 84.5 85.4 83.9 99.4 ...
 $ spf_in_vitro            : num  10.67 10.79 9.67 10.13 11.34 ...
 $ emulsifier              : Factor w/ 2 levels "Natural","Sintetic": 1 1 1 1 2 2 2 2 2 1 ...
 $ emulsifier_type         : Factor w/ 2 levels "ionic","non-ionic": 1 1 2 1 2 2 1 1 2 1 ...
 $ base_type               : Factor w/ 2 levels "Oil","Water": 1 2 2 1 1 1 1 1 2 2 ...
 $ viscosidad              : num  631 178 933 769 4605 ...
 $ CSAT                    : Factor w/ 3 levels "dislike","like",..: 1 1 1 1 3 1 1 1 1 1 ...
 $ formula_price           : num  2.74 3.09 1.69 1.26 12.95 ...
 $ x290                    : num  1.68 1.24 1.14 1.6 1.67 ...
 $ x291                    : num  1.69 1.24 1.14 1.6 1.67 ...
 $ x292                    : num  1.69 1.23 1.14 1.61 1.68 ...
.
.
.
 $ x375                    : num  0.417 0.583 0.422 0.423 0.451 ...
 $ x376                    : num  0.411 0.566 0.417 0.418 0.44 ...
 $ x377                    : num  0.409 0.535 0.414 0.416 0.433 ...
  [list output truncated]

```
These are the variables:

FPSMeasured sun protection factor (SPF) of the sunscreen formula.octinoxateActual concentration (%) of Octinoxate active in the formula.zinc_oxideActual concentration (%) of Zinc Oxide active in the formula.eficacia_octinoxate_hplcEfficiency (%) of Octinoxate determined by HPLC assay.eficacia_zno_aaThe efficiency (%) of zinc oxide is determined by atomic absorption.spf_in_vitroIn-vitro SPF measured using UV spectral methods.emulsifierType of emulsifier used (Natural or Synthetic).emulsifier_typeIonic nature of the emulsifier (ionic or non-ionic).base_typeFormula base type (Oil or Water).viscosidadViscosity of the formulation in cP (centipoise).CSATConsumer satisfaction rating (dislike, like, so_so).formula_priceFinal estimated price of the formula in arbitrary units (e.g. USD).x290 to x400Absorbance values from UV spectra, measured between 290 nm to 400 nm.

The next code splits the dataset into a training set (80%) and a test set (20%) using random sampling.
It selects spectral data (columns 13 to 123) as predictors (Xc, Xt) and active ingredients (octinoxate, zinc_oxide) as responses (yc, yt).
It also adds wavelength info as metadata to the spectral data.

The mdaplot() function then plots the UV spectra: colored by octinoxate concentration, then by zinc oxide concentration, and finally by emulsifier type.

```{r}
set.seed(123)
# Generic Rule for the Split: 80/20 (80% for the train, 20% for the test)
idx <- sample(x = nrow(found), size = as.integer(nrow(found)*0.8))
# The second line creates a random sample of 80% of the rows in the dataset found.
# The sample is stored in the variable idx, which will be used to split the dataset into training and testing sets.

# Training set
Xc <- found[idx,13:123]
yc <- found[idx, c(2,3), drop =F]
attr(Xc, 'xaxis.values') <- wl
attr(Xc, 'xaxis.ntame') <- 'Wavelength, nm'

# Testing set
Xt <- found[-idx,13:123]
yt <- found[-idx, c(2,3), drop =F]
attr(Xt, 'xaxis.values') <- wl
attr(Xt, 'xaxis.name') <- 'Wavelength, nm'

mdaplot(data = Xc, type = 'l', cgroup = yc[,1], ylab = 'Octinoxate concentration', xlab = 'Wavelength')
mdaplot(data = Xc, type = 'l', cgroup = yc[,2], ylab = 'Zinc Oxide concentration', xlab = 'Wavelength')
mdaplot(data = Xc, type = 'l', cgroup = as.factor(found$emulsifier[idx]), ylab = 'Emulsifier type', xlab = 'Wavelength')
```

![image](https://github.com/user-attachments/assets/2c84d2c1-ffe5-449c-86b9-473675771f97)

Some noise in UV spectra from in-vitro SPF (using diffuse reflectance) happens because:

Low light reaching the detector: Diffuse reflectance measures scattered light, which is weaker than direct transmission. This can make the signal small and more affected by detector noise.
Sample properties: Uneven surfaces or the nature of the sunscreen ingredients can scatter light unpredictably, adding noise.
Instrument limitations: The detector itself has a level of inherent electronic noise
However, I will leave the data like that, not to make the review longer. Though we can process the data with different techniques, showcasing on “Correction of spectral baseline” using the mdatools R package, and test a new pls model with the data processed.

PLS MODEL

```{r}
m <- pls(x = Xc, y = yc, ncomp = 5, cv = 5, x.test = Xt, y.test = yt, info = 'Pls model FPS ~ EHM+ZnO')
summary(m)

PLS model (class pls) summary
-------------------------------
Info: Pls model FPS ~ EHM+ZnO
Number of selected components: 5
Cross-validation: random with 5 segments

Response variable: octinoxate
     X cumexpvar Y cumexpvar    R2  RMSE Slope    Bias   RPD
Cal     99.99967    98.55124 0.990 0.183 0.990  0.0000 10.10
Cv            NA          NA 0.989 0.190 0.990 -0.0031  9.72
Test    99.99979    98.41945 0.991 0.166 0.995  0.0205 10.93

Response variable: zinc_oxide
     X cumexpvar Y cumexpvar    R2  RMSE Slope    Bias  RPD
Cal     99.99967    98.55124 0.983 0.331 0.983  0.0000 7.70
Cv            NA          NA 0.982 0.338 0.983 -0.0039 7.53
Test    99.99979    98.41945 0.980 0.373 0.982  0.0005 7.19
```

Breaking Down the Results:

R-squared is high (around 0.98–0.99): This means the UV spectra strongly relate to the actual ingredient amounts.

RMSEP is low (0.166 for Octinoxate, 0.373 for Zinc Oxide): The predictions are usually very close to the real amounts. For example, if the real amount of Octinoxate is 5%, the model will likely predict between 4.834% and 5.166%.

RPD is high (10.93 for Octinoxate, 7.19 for Zinc Oxide): This confirms the model’s strong ability to predict the ingredient amounts accurately.
Some validation models, which allow us to assess our results, can be found at https://mda.tools/docs/pls.html. I will show some of them.

Root Mean Square Error (RMSE)

```{r}
# Calibration
m$res$cal$rmse[,]
```
Comp 1    Comp 2    Comp 3    Comp 4    Comp 5
octinoxate 0.5033462 0.4767648 0.3376902 0.1938482 0.1825654
zinc_oxide 2.5406239 1.1066535 0.7220467 0.4563257 0.3305591


```{r}
# Cross Validation 
m$res$cv$rmse[,]
```
              Comp 1    Comp 2    Comp 3    Comp 4    Comp 5
octinoxate 0.5057529 0.4792548 0.3423432 0.1989353 0.1896071
zinc_oxide 2.5858112 1.1219551 0.7391411 0.4702090 0.3383327

```{r}
# Test
m$res$test$rmse[,]
```
              Comp 1    Comp 2    Comp 3    Comp 4    Comp 5
octinoxate 0.4913311 0.4214715 0.3116171 0.1653401 0.1658370
zinc_oxide 2.7067595 1.2257623 0.6126975 0.3972761 0.3731755

```{r}
plotRMSE(m, ny = 1)
plotRMSE(m, ny = 2)
```
![image](https://github.com/user-attachments/assets/7870699e-5a8f-4441-9976-7ae65f0b1d30)

```{r}
 Variable   |    Cal       |    RMSE      |       RMSE       |
           |  (5 Comp.)   | CV (5 Comp.) |  Test (5 Comp.)  |
-----------|--------------|--------------|------------------|
Octinoxate |    0.182     |   0.189      |       0.165      |
-----------|--------------|--------------|------------------|
Zinc Oxide |    0.330     |   0.351      |       0.373      |
------------------------------------------------------------|
```

RMSE Trend: The tables and the plot RMSE graphs show how the model’s average prediction error (RMSE) changes as we add more PLS components (from 2 to 5). Generally, the error decreases as you add components because the model learns more complex relationships.

Finding the best parameters:

Octinoxate (ny=1): The Test RMSE (your most important error measure) drops significantly up to Component 4 (RMSE = 0.165) and then barely changes (or slightly increases) for Component 5 (RMSE = 0.166). This suggests 4 components are optimal for predicting Octinoxate; adding the 5th doesn’t improve prediction for new samples.

Zinc Oxide (ny=2): The Test RMSE keeps decreasing up to Component 5 (RMSE = 0.373), although the improvement from Component 4 (RMSE = 0.397) to Component 5 is smaller than previous improvements.

Choosing Components: Since we need to predict both ingredients, and Octinoxate prediction seems optimal at 4 components while ZnO prediction is best at 5 (within this range), a choice must be made. Using 4 components looks like a very good compromise. It provides excellent Octinoxate prediction and good ZnO prediction (RMSE < 0.4). Using 5 components slightly improves ZnO prediction but offers no benefit for Octinoxate.

Formulation Screening: This accuracy is likely sufficient to quickly screen new formulations. We can rapidly tell if We are close to your target concentrations, allowing faster iteration cycles compared to waiting for HPLC/AA.

Cost/Time Savings: This achieves the core goal: replacing many slow, expensive tests with a rapid UV scan + model prediction, significantly saving time and resources during R&D and potentially for QC


“X” Variance (spectral data) AND “Y” Variance (target variables)

# X Variance
```{r}
m$res$cal$xdecomp$expvar
m$res$test$xdecomp$expvar

      Comp 1       Comp 2       Comp 3       Comp 4       Comp 5 
89.241682062 10.701092678  0.030985335  0.018936294  0.006971674 
      Comp 1       Comp 2       Comp 3       Comp 4       Comp 5 
89.095266207 10.853716029  0.028321148  0.015100857  0.007390311 

# Y Variance 

m$res$cal$ydecomp$expvar
m$res$test$ydecomp$expvar

   Comp 1    Comp 2    Comp 3    Comp 4    Comp 5 
31.848010 53.400365  8.296346  3.957941  1.048582
    Comp 1     Comp 2     Comp 3     Comp 4     Comp 5 
28.2717812 55.8041209 11.4457624  2.7233560  0.1744295 


plotXVariance(m, show.labels = T, type = 'h')
plotYVariance(m, show.labels = T, type = 'h')
```
![image](https://github.com/user-attachments/assets/b4ee1b45-0a97-45d2-99c5-02cbcb75607c)

X Variance (spectral data):
In both calibration and test: Component 1 explains ~89% | Component 2 adds ~10%

The rest explain very little. Most spectral variation is in the first 2 components.
Y Variance (target variables):

In calibration: Comp 1 explains ~32%, Comp 2 adds ~53%
In test: Similar trend: Comp 1 + 2 explain ~84% total
The first 2 components also explain most of the variation in response.
X variance:
The model uses mostly the first 2 components to describe the spectral data. That means the important information is in these first components.

Y variance:
The first 2 components also explain most of the prediction for the ingredients (Octinoxate and ZnO). So, the model can make good predictions using only 2 components.

Prediction plot

```{r}
for(i in 1:5){
  for(j in 1:2){
    plotPredictions(m, ncomp = i, ny = j)
  }
}

m$res$cal$r2[,]
m$res$test$r2[,]
m$res$cv$r2[,]


                 Comp 1    Comp 2    Comp 3    Comp 4   Comp 5
octinoxate 0.9251253813 0.9328247 0.9662994 0.9888948 0.990150
zinc_oxide 0.0006761936 0.8103954 0.9192847 0.9677615 0.983083
                Comp 1    Comp 2    Comp 3    Comp 4    Comp 5
octinoxate  0.92409325 0.9441442 0.9694666 0.9914042 0.9913524
zinc_oxide -0.03387582 0.7879777 0.9470262 0.9777283 0.9803485
                Comp 1    Comp 2    Comp 3    Comp 4    Comp 5
octinoxate  0.92440765 0.9321212 0.9653643 0.9883043 0.9893755
zinc_oxide -0.03518767 0.8051159 0.9154176 0.9657700 0.9822780

```

![image](https://github.com/user-attachments/assets/1663acf8-3060-4441-9a23-ad0bff13518d)


For Octinoxate: The model needs 4 components to reach very high R² ≈ 0.99. So, prediction is very strong, but it needs 4 components to get there.

For Zinc Oxide: R² is low at first (even negative), but from component 2 it improves a lot. It reaches R² ≈ 0.98 at component 4–5. So, it also becomes very good, but it needs at least 2 components to be useful.

Both can be predicted well. So, we can reduce lab tests and use the model to save time and cost.

Regression Coefficients Summary

```{r}
summary(m$coeffs, ncomp = 4, ny = 1)

Regression coefficients for octinoxate (ncomp = 4)
--------------------------------------------------
         Coeffs  Std. err. t-value p-value        2.5%       97.5%
x290 -0.4871320 0.01471896  -33.12   0.000 -0.52799835 -0.44626558
x291 -0.6058684 0.06788960   -8.94   0.001 -0.79436018 -0.41737668
x292 -1.6749549 0.05367043  -31.20   0.000 -1.82396790 -1.52594190
.
.
.
Degrees of freedom (Jack-Knifing): 4


summary(m$coeffs, ncomp = 4, ny = 2)

Regression coefficients for zinc_oxide (ncomp = 4)
--------------------------------------------------
          Coeffs  Std. err. t-value p-value          2.5%       97.5%
x290  -3.3719360 0.17047779  -19.75   0.000  -3.845258220 -2.89861377
x291   1.1010271 0.10294877   10.67   0.000   0.815195493  1.38685872
x292  -0.1984883 0.33471783   -0.63   0.563  -1.127813947  0.73083742
.
.
.
Degrees of freedom (Jack-Knifing): 4
```

```{r}
plotRegcoeffs(m, ny = 1, show.ci = T, ncomp = 2)
plotRegcoeffs(m, ny = 2, show.ci = T, ncomp = 2)
```

![image](https://github.com/user-attachments/assets/cd8e3e72-43e6-414b-89cd-cfa89e69fdab)
Regression coefficients are sensitive to the number of components.

The coefficients show exactly which parts of the fast UV scan the model uses to accurately estimate Octinoxate and Zinc Oxide. It primarily uses the known absorption region of the filters' UV (positively) and leverages other spectral regions (negatively), likely related to other formulation components.

p-value: A statistical test. Very small p-values (like 0.000 or 0.001) mean the model is very confident that this wavelength is truly important for predicting Octinoxate or Zinc Oxide. If the p-value is large (e.g., > 0.05, like for x296 or x318), the model isn’t sure if that specific wavelength is consistently useful.

Distance plot for X-decomposition and Y-decomposition

```{r}
found[rownames(Xc),][categorize(m, m$res$cal, ncomp = 4) == 'outlier',]
```

![image](https://github.com/user-attachments/assets/953a24aa-a99d-46e4-9bf3-de76d952b665)

We can find which samples are outliers to identify them.

```{r}
plotXResiduals(m, show.labels = T, ncomp = 4,show.legend = T)
plotYResiduals(m, show.labels = T, ncomp = 2,show.legend = T)
```
![image](https://github.com/user-attachments/assets/654e2f13-8207-49d2-ab9f-216bc15441bd)

If a new formula has a spectrum with a high residual, it’s a warning sign: its spectrum was not well represented by the learned patterns. This helps prevent blind trust in the model’s prediction for those formulas. This means greater confidence when automating SPF or concentration prediction.


What does an outlier or extreme value indicate?
An outlier suggests that the model fails to adequately represent that sample in spectral space with the selected components.
An extreme value could indicate: A technical problem (experimental error), A formula that is very different from the rest.
Or even a real, valid sample that the model still cannot adequately explain with the number of components you have chosen. This is the case because the concentrations of octinoxate and zinc oxide for sample 195 are very low; in fact, the SPF is the lowest among the samples, so the model fails to adequately fit or explain this spectrum (atypical sample).

CONCLUSIONS:

Even though this is a brief review of how we can perform and assess a PLS model, there are more technical considerations we need to keep in mind. I only used a few validation methods. You can find a complete guide at https://mda.tools/docs/index.html.

The PLS model with 4–5 components accurately predicts Octinoxate and Zinc Oxide concentrations using only UV spectral data, eliminating the need for slower lab methods like HPLC and Atomic Absorption.

Key Benefits

Fast Screening: New formulations can be scanned and evaluated instantly, speeding up decision-making and reducing wait times for lab results.
Efficient Quality Control: In production, UV scans can verify active levels quickly, minimizing routine chemical testing.
Troubleshooting Tool: Unexpected spectra can help detect formulation issues (e.g., wrong active levels) early.
SPF Confidence: While the model doesn’t predict SPF directly, accurate prediction of Octinoxate and ZnO gives strong confidence in meeting SPF targets.
Time and Cost Savings: UV analysis is much faster and cheaper than HPLC or AA, reducing per-sample costs and freeing lab resources.
Accelerated R&D: Quick feedback allows faster iteration in formulation development, shortening time to market.
Next Step
Build a PLS model to predict SPF directly from the UV spectrum. This would allow estimating SPF without any lab-based SPF tests, making the process even more efficient.

Sources consulted
Kucheryavskiy, S. (2023, July 21). Getting started with mdatools for R. Retrieved April 8, 2025, from https://mda.tools/docs/index.html











