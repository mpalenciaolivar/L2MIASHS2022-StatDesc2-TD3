---
title: "TD 3 de Statistique descriptive 2"
author: "Miguel PALENCIA-OLIVAR"
date: '2022-02-14'
output: 
  html_document:
    keep_md: true
    df_print: paged
---
# Préambule

Le présent document a pour objectif de présenter synthétiquement - et au fur et
à mesure des séances - la solution du TD 3 correspondant au cours de Statistique
descriptive 2 dispensé en L2 MIASHS à l'Université Lumière Lyon 2 par [Stéphane CHRÉTIEN](https://sites.google.com/site/stephanegchretien/enseignement/l2-miashs-statistiques-descriptives/l2-statistiques-descriptives-2-regression-et-classification). Les consignes et le corrigé sont trouvables dans le répertoire `doc` ; `data` contient pour sa part les jeux de données utilisés dans le cadre du TD dans des formats simples d'usage.

*Ce document n'est pas un tutoriel pour R, et n'a pas pour but de remplacer le CM. En revanche, c'est la suite directe du TD 2, [trouvable ici](https://github.com/mpalenciaolivar/L2MIASHS2022-StatDesc2-TD2)*.

# Ressources utiles
## Ouvrages
- [Cours de Ricco RAKOTOMALALA sur la régression logistique (davantage de Statistique)](http://eric.univ-lyon2.fr/~ricco/cours/cours_regression_logistique.html)
- [Cours de Julien AH-PINE (davantage général)](https://eric.univ-lyon2.fr/~jahpine/cours/m2_sise-dm/cm.pdf)

## Ressources internet
- De manière générale, tout ce qui est dans la galaxie [WikiStat](http://wikistat.fr/), mais en particulier :

  - [Statistique descriptive élémentaire](https://www.math.univ-toulouse.fr/~baccini/zpedago/asde.pdf)
  
  - [Pratique de la modélisation statistique](https://www.math.univ-toulouse.fr/~besse/pub/modlin.pdf)
  
  - [Exploration statistique multidimensionnelle](https://www.math.univ-toulouse.fr/~besse/pub/Explo_stat.pdf)

- Autrement :

  - [Decision Tree Algorithm, Explained](https://www.kdnuggets.com/2020/01/decision-tree-algorithm-explained.html)
  
  - [LOGIT REGRESSION | R DATA ANALYSIS EXAMPLES](https://stats.oarc.ucla.edu/r/dae/logit-regression/)
  
  - [Tutoriel Titanic (repris partiellement)](https://medium.com/analytics-vidhya/a-beginners-guide-to-learning-r-with-the-titanic-dataset-a630bc5495a8)
  
# Exercice
Ce TD est un peu différent ; il s'agit de vous mettre un pied à l'étrier sur la pratique de la classification. Cet exercice est inédit, en ce sens qu'il n'est pas compris dans le sujet de TD. À ce titre, on mettra l'accent sur les ressources suivantes et sur le langage R :

- [Introduction à la data science – Du data mining au big data analytics](https://eric.univ-lyon2.fr/~ricco/cours/slides/intro_ds_from_dm_to_bd.pdf). On pourra trouver une version complémentaire [ici](https://eric.univ-lyon2.fr/~ricco/cours/slides/Introduction_au_Data_Mining.pdf)

- [Apprentissage supervisé](https://eric.univ-lyon2.fr/~ricco/cours/slides/Apprentissage_Supervise.pdf)

- [Régression logistique façon Machine Learning](http://eric.univ-lyon2.fr/~ricco/cours/slides/logistic_regression_ml.pdf)

- [Classifieur bayésien naïf](http://eric.univ-lyon2.fr/~ricco/cours/slides/naive_bayes_classifier.pdf)


**Il ne sert à rien de courir : il faut d'abord lire les ressources PUIS faire le TD, et pas l'inverse.**

## Contexte
On cherche à prédire quels sont les individus qui survivront au nauffrage du [Titanic](https://fr.wikipedia.org/wiki/Titanic) à partir d'un certain nombre de variables.

## Dictionnaire de données
![](img/datadict.png)

Avant même de considérer le code, il va falloir faire un petit travail de qualification des données. Ce n'est pas parce qu'une variable prend des valeurs "chiffre" que c'est une variable quantitative continue (ex : un rang).

## Pré-requis
Le projet va nécessiter un certain nombre de packages externes. Pour les installer, on ouvrira `requirements.R` et on cliquera sur `Run All` (CTRL+ALT+R).

## Chargement des données

```r
# na.string est initialisé de la sorte afin que l'on puisse mieux gérer les données manquantes par la suite.
titanic <- read.csv(file.path("data", "titanic", "train.csv"), na.strings = "")
test <- read.csv(file.path("data", "titanic", "test.csv"), na.strings = "")
```

## Préparation des données
Comme souvent lorsque l'on travaille avec des données, il y a des valeurs manquantes, des colonnes mal typées, etc. Il faut y remédier avant toute chose. Commençons par jeter un oeil à nos données chargées :

```r
# Attention : cette fonction est faite pour fonctionner avec RStudio. Elle est utile pour faire des tris "à la Excel". Elle est commentée pour pouvoir exécuter le notebook sans interruption.
# View(titanic)
```

Voyons voir s'il y a des données manquantes :

```r
library(Amelia)  # Chargement de lib
```

```
## Warning: package 'Amelia' was built under R version 4.1.3
```

```
## Loading required package: Rcpp
```

```
## ## 
## ## Amelia II: Multiple Imputation
## ## (Version 1.8.0, built: 2021-05-26)
## ## Copyright (C) 2005-2022 James Honaker, Gary King and Matthew Blackwell
## ## Refer to http://gking.harvard.edu/amelia/ for more information
## ##
```

```r
missmap(titanic, col = c("black", "grey"))
```

![](TD_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

Nous avons une variable Cabin qui a beaucoup de données manquantes, et une variable PassengerId qui est une pseudo-variable d'identifiants. Nous les excluerons de l'analyse. Nous suivrons notre tutoriel de référence dans un souci de simplicité (pour la pédagogie, donc). Mais la décision de faire d'abandonner d'autres variables que celles mentionnées précédemment qui a été prise par l'auteur du tuto me semble peu justifiée à certains égards. Par exemple, on enlève le prix payé et le port d'embarquement, sans justification autre
que l'intuition, sans vérification. Mais, en l'état, qui nous dit que parmi les survivants, il n'y a pas davantage de gens qui ont payé cher, et que ces gens ayant payé cher ont embarqué à un endroit plutôt qu'à un autre ? C'est une explication un peu recherchée et sans forcément de fondement, mais l'idée est là : le choix est méthodologiquement discutable pour faire de la sélection de variables (en l'absence d'analyse).

Puisque nous parlons de ce type de sélection, parlons également des outliers et des données aberrantes. De mon point de vue, une donnée aberrante est une valeur dont il est manifeste qu'il s'agit d'une erreur : une taille négative, par exemple. Si l'on peut appliquer un correctif, il faut le faire. Un outlier est pour sa part une valeur qui sort du lot. On peut souvent voir que les outliers sont retirés des données, sans plus de considération. C'est une erreur de jugement : tout d'abord, pourquoi le point que l'on cherche à retirer serait-il à retirer ? N'appartient-il pas à une sous-distribution (modèle de mélange par ex), n'est-il pas symptomatique d'autre chose ? Un exemple concret : prenons une chaîne de cafés, dont certains (mettons 1 ou 2) sont en centre-ville, la plupart dans le reste de la ville voire en périphérie. On étudie le chiffre d'affaires. Il semble assez logique que ceux en centre-ville fassent bien davantage de chiffre que ceux en périphérie... cela veut surtout dire qu'il s'agit de prendre en compte le lieu d'implantation, en l'espèce. Je préfère tenir compte des cas de ce type dans ma modélisation plutôt que de supprimer l'info, même lorsque je n'ai pas forcément la localisation parmi mes variables, mais juste un élément de connaissance métier (cela se fait en Statistique bayésienne). Et même si l'on choisissait de retirer une observation, il y a des techniques savantes (Statistique par ex) pour faire de la détection d'outlier. Il en va de même sur l'imputation de données manquantes, qui n'est parfois pas même désirable en fonction du contexte d'étude.


```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
titanic <- select(titanic, Survived, Pclass, Age, Sex, SibSp, Parch)
test <- select(test, Survived, Pclass, Age, Sex, SibSp, Parch)
```

Il ne restera plus qu'à retirer les valeurs manquantes pour savoir ce que l'on traitera.


```r
titanic <- na.omit(titanic)
test <- na.omit(test)
```

Il s'agit maintenant de savoir comment sont codées nos variables.


```r
str(titanic)
```

```
## 'data.frame':	714 obs. of  6 variables:
##  $ Survived: int  0 1 1 1 0 0 0 1 1 1 ...
##  $ Pclass  : int  3 1 3 1 3 1 3 3 2 3 ...
##  $ Age     : num  22 38 26 35 35 54 2 27 14 4 ...
##  $ Sex     : chr  "male" "female" "female" "female" ...
##  $ SibSp   : int  1 1 0 1 0 0 3 0 1 1 ...
##  $ Parch   : int  0 0 0 0 0 0 1 2 0 1 ...
##  - attr(*, "na.action")= 'omit' Named int [1:177] 6 18 20 27 29 30 32 33 37 43 ...
##   ..- attr(*, "names")= chr [1:177] "6" "18" "20" "27" ...
```

Survived et Pclass sont considérées comme étant des variables numériques. Or, elles sont respectivement catégorielle et catégorielle *ordinale*. Transformons les.


```r
titanic$Survived <- factor(titanic$Survived)
titanic$Pclass <- factor(titanic$Pclass, order=TRUE, levels = c(3, 2, 1))

test$Survived <- factor(test$Survived)
test$Pclass <- factor(test$Pclass, order=TRUE, levels = c(3, 2, 1))
```

## Exploration des données
Il est temps de commencer à regarder nos données. Allons-y !

### Corrélations

```r
library(GGally)
```

```
## Warning: package 'GGally' was built under R version 4.1.3
```

```
## Loading required package: ggplot2
```

```
## Registered S3 method overwritten by 'GGally':
##   method from   
##   +.gg   ggplot2
```

```r
ggcorr(titanic,
       nbreaks = 6,
       label = TRUE,
       label_size = 3,
       color = "grey50")
```

```
## Warning in ggcorr(titanic, nbreaks = 6, label = TRUE, label_size = 3, color =
## "grey50"): data in column(s) 'Survived', 'Pclass', 'Sex' are not numeric and
## were ignored
```

![](TD_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

### Comptage du nombre de survivants

```r
library(ggplot2)

ggplot(titanic, aes(x = Survived)) +
  geom_bar(width=0.5, fill = "coral") +
  geom_text(stat='count', aes(label=stat(count)), vjust=-0.5) +
  theme_classic()
```

![](TD_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

### Comptage du nombre de survivants en fonction du sexe

```r
ggplot(titanic, aes(x = Survived, fill = Sex)) +
 geom_bar(position = position_dodge()) +
 geom_text(stat='count', 
           aes(label=stat(count)), 
           position = position_dodge(width=1), vjust=-0.5)+
 theme_classic()
```

![](TD_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

### Comptage du nombre de survivants en fonction de Pclass

```r
ggplot(titanic, aes(x = Survived, fill = Pclass)) +
 geom_bar(position = position_dodge()) +
 geom_text(stat='count',
           aes(label=stat(count)), 
           position = position_dodge(width=1), 
           vjust=-0.5)+
 theme_classic()
```

![](TD_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

### Densité de l'âge

```r
ggplot(titanic, aes(x = Age)) +
 geom_density(fill='coral')
```

![](TD_files/figure-html/unnamed-chunk-12-1.png)<!-- -->

### Survie en fonction de l'âge

```r
# Discrétisation de l'âge
titanic$Discretized.age <- cut(titanic$Age, c(0, 10, 20, 30, 40 ,50, 60, 70, 80, 100))

ggplot(titanic, aes(x = Discretized.age, fill = Survived)) +
  geom_bar(position = position_dodge()) +
  geom_text(stat='count', aes(label=stat(count)), position = position_dodge(width=1), vjust=-0.5)+
  theme_classic()
```

![](TD_files/figure-html/unnamed-chunk-13-1.png)<!-- -->

```r
titanic$Discretized.age <- NULL
```

## Constitution des jeux pour l'apprentissage

```r
# Attention: il n'y a pas de randomisation ici puisque le jeu de données est déjà randomisé. CETTE RANDOMISATION EST NÉCESSAIRE !
# S'il y en avait, il aurait fallu exécuter la fonction avec une seed, tel que :

# set.seed(2022)

train_validation_split <- function(data, fraction = 0.8, train = TRUE) {
  total_rows <- nrow(data)
  train_rows <- fraction * total_rows
  sample <- 1:train_rows
  if (train == TRUE) {
    return (data[sample, ])
  } else {
    return (data[-sample, ])
  }
}


train <- train_validation_split(titanic, 0.8, train = TRUE)
validation <- train_validation_split(titanic, 0.8, train = FALSE)
```

## Arbre de décision

```r
library(rpart)
library(rpart.plot)
```

```
## Warning: package 'rpart.plot' was built under R version 4.1.3
```

```r
set.seed(2022)
dtree <- rpart(Survived ~ ., data = train, method = 'class')
rpart.plot(dtree, extra = 106)
```

![](TD_files/figure-html/unnamed-chunk-15-1.png)<!-- -->

On évalue le modèle:


```r
library(MLmetrics)
```

```
## Warning: package 'MLmetrics' was built under R version 4.1.3
```

```
## 
## Attaching package: 'MLmetrics'
```

```
## The following object is masked from 'package:base':
## 
##     Recall
```

```r
y_pred <- predict(dtree, validation, type = 'class')
y_true <- validation$Survived

dtree_precision <- Precision(y_true, y_pred, positive = 1)
dtree_recall <- Recall(y_true, y_pred, positive = 1)
dtree_f1 <- F1_Score(y_true, y_pred, positive = 1)
dtree_auc <- AUC(y_true, y_pred)

paste0("Precision: ", dtree_precision)
```

```
## [1] "Precision: 0.737704918032787"
```

```r
paste0("Recall: ", dtree_recall)
```

```
## [1] "Recall: 0.803571428571429"
```

```r
paste0("F1 Score: ", dtree_f1)
```

```
## [1] "F1 Score: 0.769230769230769"
```

```r
paste0("AUC: ", dtree_auc)
```

```
## [1] "AUC: 0.801779288284686"
```



```r
library(ROSE)
```

```
## Warning: package 'ROSE' was built under R version 4.1.3
```

```
## Loaded ROSE 0.0-4
```

```r
roc.curve(y_pred, y_true, plotit = TRUE, add.roc = FALSE, 
          n.thresholds=100)
```

![](TD_files/figure-html/unnamed-chunk-17-1.png)<!-- -->

```
## Area under the curve (AUC): 0.802
```

On affine notre arbre (fine tuning) :


```r
set.seed(2022)

control <- rpart.control(minsplit = 8,
                         minbucket = 2,
                         maxdepth = 6,
                         cp = 0)
dtree_tuned_fit <- rpart(Survived ~ ., data = train, method = 'class', control = control)
y_pred <- predict(dtree_tuned_fit, validation, type = 'class')

dtree_tuned_fit_precision <- Precision(y_true, y_pred, positive = 1)
dtree_tuned_fit_recall <- Recall(y_true, y_pred, positive = 1)
dtree_tuned_fit_f1 <- F1_Score(y_true, y_pred, positive = 1)
dtree_tuned_fit_auc <- AUC(y_true, y_pred)

paste0("Precision: ", dtree_tuned_fit_precision)
```

```
## [1] "Precision: 0.806451612903226"
```

```r
paste0("Recall: ", dtree_tuned_fit_recall)
```

```
## [1] "Recall: 0.892857142857143"
```

```r
paste0("F1 Score: ", dtree_tuned_fit_f1)
```

```
## [1] "F1 Score: 0.847457627118644"
```

```r
paste0("AUC: ", dtree_tuned_fit_auc)
```

```
## [1] "AUC: 0.866188769414576"
```

```r
rpart.plot(dtree_tuned_fit, extra = 106)
```

![](TD_files/figure-html/unnamed-chunk-19-1.png)<!-- -->



```r
roc.curve(y_pred, y_true, plotit = TRUE, add.roc = FALSE, 
          n.thresholds=100)
```

![](TD_files/figure-html/unnamed-chunk-20-1.png)<!-- -->

```
## Area under the curve (AUC): 0.866
```

## Régression logistique
Même si la régression logistique est ici présentée sous l'angle de la classification, elle est fondamentalement, intrinsèquement, décidément - notez que j'insiste - une régression. On renverra surtout aux ressources mises en ligne par R. RAKOTOMALALA pour la vision Statistique de la chose.


```r
set.seed(2022)

# On standardise les variables numériques
data_rescale <- mutate_if(titanic,
                          is.numeric,
                          list(~as.numeric(scale(.))))
train <- train_validation_split(data_rescale, 0.7, train = TRUE)
validation <- train_validation_split(data_rescale, 0.7, train = FALSE)
logreg <- glm(Survived ~ ., data = train, family = "binomial")
summary(logreg)
```

```
## 
## Call:
## glm(formula = Survived ~ ., family = "binomial", data = train)
## 
## Deviance Residuals: 
##     Min       1Q   Median       3Q      Max  
## -2.5920  -0.6739  -0.4134   0.6367   2.3344  
## 
## Coefficients:
##              Estimate Std. Error z value Pr(>|z|)    
## (Intercept)  1.419361   0.196762   7.214 5.45e-13 ***
## Pclass.L     1.555536   0.228015   6.822 8.97e-12 ***
## Pclass.Q    -0.063754   0.217026  -0.294 0.768940    
## Age         -0.532113   0.138546  -3.841 0.000123 ***
## Sexmale     -2.569211   0.251982 -10.196  < 2e-16 ***
## SibSp       -0.316053   0.135620  -2.330 0.019784 *  
## Parch       -0.009988   0.127804  -0.078 0.937709    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 677.21  on 498  degrees of freedom
## Residual deviance: 461.88  on 492  degrees of freedom
## AIC: 475.88
## 
## Number of Fisher Scoring iterations: 4
```

Cette sortie est la même que pour une régression linéaire. Encore une fois, c'est parce que la *régression logistique est une régression et non un classifieur en soi*. Voyons comment cela se passe sur la significativité globale :


```r
LR <- logreg$null.deviance - logreg$deviance
p <- logreg$df.null - logreg$df.residual
pchisq(LR, p, lower.tail = F)
```

```
## [1] 1.031983e-43
```

En supposant un risque à 5%, on est bien en-deçà. On a donc bien un modèle informatif.


```r
y_true <- validation$Survived
y_pred <- predict(logreg, validation, type = 'response')
y_pred <- as.factor(ifelse(y_pred > 0.5, 1, 0))

logreg_precision <- Precision(y_true, y_pred, positive = 1)
logreg_recall <- Recall(y_true, y_pred, positive = 1)
logreg_f1 <- F1_Score(y_true, y_pred, positive = 1)
logreg_auc <- AUC(y_true, y_pred)
paste0("Precision: ", logreg_precision)
```

```
## [1] "Precision: 0.756410256410256"
```

```r
paste0("Recall: ", logreg_recall)
```

```
## [1] "Recall: 0.710843373493976"
```

```r
paste0("F1 Score: ", logreg_f1)
```

```
## [1] "F1 Score: 0.732919254658385"
```

```r
paste0("AUC: ", logreg_auc)
```

```
## [1] "AUC: 0.790613887329216"
```


```r
roc.curve(y_pred, y_true, plotit = TRUE, add.roc = FALSE, 
          n.thresholds=100)
```

![](TD_files/figure-html/unnamed-chunk-24-1.png)<!-- -->

```
## Area under the curve (AUC): 0.791
```

## Classifieur bayésien naïf

```r
library(e1071)

set.seed(2022)
nbClassifier <- naiveBayes(Survived ~., data = train)
y_pred <- predict(nbClassifier, validation)
y_true <- validation$Survived

nbClassifier_precision <- Precision(y_true, y_pred, positive = 1)
nbClassifier_recall <- Recall(y_true, y_pred, positive = 1)
nbClassifier_f1 <- F1_Score(y_true, y_pred, positive = 1)
nbClassifier_auc <- AUC(y_true, y_pred)
paste0("Precision: ", nbClassifier_precision)
```

```
## [1] "Precision: 0.774647887323944"
```

```r
paste0("Recall: ", nbClassifier_recall)
```

```
## [1] "Recall: 0.662650602409639"
```

```r
paste0("F1 Score: ", F1_Score(y_true, y_pred, positive = 1))
```

```
## [1] "F1 Score: 0.714285714285714"
```

```r
paste0("AUC: ", nbClassifier_auc)
```

```
## [1] "AUC: 0.79010172143975"
```


```r
roc.curve(y_pred, y_true, plotit = TRUE, add.roc = FALSE, 
          n.thresholds=100)
```

![](TD_files/figure-html/unnamed-chunk-26-1.png)<!-- -->

```
## Area under the curve (AUC): 0.790
```

## Comparaison et sauvegarde d'un modèle
Nous avons 4 modèles. L'heure est à la comparaison des performances. Mon indice
préféré est le F1-Score, car il s'agit d'une moyenne (harmonique). En fonction
des cas, on pourra placer davantage d'importance sur la précision ou sur le
rappel, et tout de même synthétiser cela au sein du Beta-F1-Score. Faisons un
dataframe de nos indices.


```r
dtree_perfs <- list(precision = dtree_precision,
                    recall = dtree_recall,
                    f1_score = dtree_f1,
                    auc = dtree_auc)

dtree_tuned_fit_perfs <- list(precision = dtree_tuned_fit_precision,
                              recall = dtree_tuned_fit_recall,
                              f1_score = dtree_tuned_fit_f1,
                              auc = dtree_tuned_fit_auc)

logreg_perfs <- list(precision = logreg_precision,
                     recall = logreg_recall,
                     f1_score = logreg_f1,
                     auc = logreg_auc)

nbClassifier_perfs <- list(precision = nbClassifier_precision,
                           recall = nbClassifier_recall,
                           f1_score = nbClassifier_f1,
                           auc = nbClassifier_auc)

# Pas très élégant, mais ça marche
perfs <- as.data.frame(t(do.call(rbind, Map(data.frame,
                       dtree = dtree_perfs,
                       dtree_tuned = dtree_tuned_fit_perfs,
                       logreg = logreg_perfs,
                       nbClassifier = nbClassifier_perfs))))

# On trie par f1-score décroissant
# Il faudra télécharger le notebook pour voir les scores correctement
attach(perfs)
perfs[order(-f1_score),]
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":[""],"name":["_rn_"],"type":[""],"align":["left"]},{"label":["precision"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["recall"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["f1_score"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["auc"],"name":[4],"type":["dbl"],"align":["right"]}],"data":[{"1":"0.8064516","2":"0.8928571","3":"0.8474576","4":"0.8661888","_rn_":"dtree_tuned"},{"1":"0.7377049","2":"0.8035714","3":"0.7692308","4":"0.8017793","_rn_":"dtree"},{"1":"0.7564103","2":"0.7108434","3":"0.7329193","4":"0.7906139","_rn_":"logreg"},{"1":"0.7746479","2":"0.6626506","3":"0.7142857","4":"0.7901017","_rn_":"nbClassifier"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
detach(perfs)
```

Notre gagnant est l'arbre de décision avec tuning. Puisque nous avons notre
vainqueur, il peut être judicieux de le sauvegarder pour le réutiliser. C'est
très facile à faire (décommenter les fonctions):


```r
# saveRDS(dtree_tuned_fit, "dtree_tuned_fit.rds")
# dtree_tuned_fit <- readRDS("dtree_tuned_fit.rds")
```

## Test du modèle
Nous avons entraîné/tuné un modèle ; il est maintenant temps de le tester sur
des données totalement inédites du point de vue de l'entraînement. Il faut
traiter les données nouvelles de la même façon que les données d'entraînement
au préalable. Cela a été fait en même temps que le reste des données, dans les
sections dédiées. Voyons maintenant la performance "réelle", sur données test:


```r
y_pred <- predict(dtree_tuned_fit, test, type = 'class')
y_true <- test$Survived

dtree_tuned_fit_precision <- Precision(y_true, y_pred, positive = 1)
dtree_tuned_fit_recall <- Recall(y_true, y_pred, positive = 1)
dtree_tuned_fit_f1 <- F1_Score(y_true, y_pred, positive = 1)
dtree_tuned_fit_auc <- AUC(y_true, y_pred)

paste0("Precision: ", dtree_tuned_fit_precision)
```

```
## [1] "Precision: 0.797101449275362"
```

```r
paste0("Recall: ", dtree_tuned_fit_recall)
```

```
## [1] "Recall: 0.866141732283465"
```

```r
paste0("F1 Score: ", dtree_tuned_fit_f1)
```

```
## [1] "F1 Score: 0.830188679245283"
```

```r
paste0("AUC: ", dtree_tuned_fit_auc)
```

```
## [1] "AUC: 0.85473629164799"
```
