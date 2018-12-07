# tarea5

---
title: "R Tarea 5"
output:
  html_document:
    df_print: paged
---

```{r setup, include=TRUE}
knitr::opts_chunk$set(echo = TRUE, warning= FALSE, message=FALSE)

```


Pregunta 2
-----------------------------------------------------------------------


* Descarga de las librerias a utilizar 
```{r}
library(pdfetch)
library(ggplot2)
library(dplyr)
library(gridExtra)
```

* Descarga de las datas base

```{r}
msft=pdfetch_YAHOO("MSFT",fields=c("open","adjclose"),from="2000-01-01", to="2018-08-31", interval="1mo")
msft= as.data.frame(msft)
appl=pdfetch_YAHOO("AAPL",fields=c("open","adjclose"),from="2000-01-01", to="2018-08-31", interval="1mo")
appl = as.data.frame(appl)

```


* Creación de la funcion

```{r}

function_finance = function (x, return = c("yes", "no"), graph = c("type1", "type2"), norm = c("yes", "no")) {
 
 z <- { 
   if (return == "yes") { 
    a <-   matrix(diff(log(x[,2]), lag = 1))
    b <- rbind(0,a)
    x$retorno <- b
    print(head(x,6))
     }
  else if (return == "no") { 
    a <-   matrix(diff(log(x[,2]), lag = 1)/x[-1,2])
    b <- rbind(0,a)
    x$retorno <- b
    print(head(x,6))
  }
   }
 y <- { 
   if (graph == "type1") {
    c <-   matrix(diff(log(x[,2]), lag = 1))
    d <- rbind(0,c)
    x$retorno <- d
    x %>% ggplot(aes(y = retorno, x= (1:length(retorno)))) +
            geom_line(color= "blue", lwd= .8) +
            labs(
              x=  "Dias",
              y= "Retornos",
              title = "Retornos Diarios",
              caption = "Fuente: Yahoo Finance"
            ) +
              theme_gray()
 }
  else if (graph == "type2") {
    c <-   matrix(diff(log(x[,2]), lag = 1))
    d <-   rbind(0,c)
    x$retorno <- cumsum(d)
    x %>% ggplot(aes(y=retorno, x = (1:length(retorno)))) +
      geom_line(color="black", lwd =.8) +
      labs(
        x=  "Dias",
        y= "Retornos",
        title = "Retornos Acumulados",
        caption = "Fuente: Yahoo Finance"
      ) +
      theme_gray()
  } 
   }
  w <- {
    if (norm == "yes") {
    c <-   matrix(diff(log(x[,2]), lag = 1))
    d <- rbind(0,c)
    x$retorno <- d
    skew <- {(1/length(x[,1]))* sum((x[,3]-mean(x[,3])^3))}/{(((1/length(x[,1]))* sum((x[,3]-mean(x[,3])^2))))^(3/2)}
    kur <- {(1/length(x[,1]))* sum((x[,3]-mean(x[,3])^4))}/{(((1/length(x[,1]))* sum((x[,3]-mean(x[,3])^2))))^(2)}
    JB <- length(x[,1]) * {((skew^2)/6) + ((kur-3)^2)/24}
    cat(JB,"jarque-bera")
  } 
    else if (norm=="no"){ }
    }
  z
  w
  y
 }

function_finance(msft, return = "no", graph = "type2", norm = "yes")  

```

Pregunta 3
------------ 

Parte a 


```{r}
set.seed(123)
reps = 10000
betas = matrix(NA, nrow = reps, ncol = 8)
beta0 = 2
beta1 = 2.5
beta2 = 1

n = c(50, 100, 500, 1000) 
for (j in 1:length(n)) {
  X1=rnorm(n[j],20,1)
  X2= (0.8*X1) + rnorm(n[j],0,1) #Caso 1
  
  for (i in 1:reps) {
    u= rnorm(n[j],0,1)
    v=(beta2*X2 + u )
    Ycs = beta0 + beta1*X1 + v   #con sesgo
    Yss = beta0 + beta1*X1 + beta2*X2 + u   #sin sesgo
    modelcs = lm(Ycs~X1)  
    betas[i,j] = modelcs$coef[2]
    modelss = lm(Yss~X1+X2) 
    betas[i,j+4] = modelss$coef[2]
  }
}
betas_dataC1 <- data.frame(betas)

```

Esperanza de los Betas

```{r}
eb1 <- apply(betas_dataC1[5:8],2,mean)
knitr::kable(
eb1 
)
```

Varianza de los Betas

```{r}
knitr::kable(
apply(betas_dataC1[5:8],2,var)
)
```


Parte b

```{r}
g1 <- ggplot(betas_dataC1) + 
  geom_histogram(aes(betas_dataC1[,5], y=..density..), col="blue", bins = 30) +
  stat_function(fun=dnorm, args=list(mean=mean(betas_dataC1[,5]), sd=sd(betas_dataC1[,5])), 
                geom="line", colour="red", size=1) +
  ylab("Densidad") +   ggtitle("n=50") + xlab(expression(hat(beta)[1])) +
  theme_minimal()

g2 <- ggplot(betas_dataC1) + 
  geom_histogram(aes(betas_dataC1[,6], y=..density..), col="blue", bins = 30) +
  stat_function(fun=dnorm, args=list(mean=mean(betas_dataC1[,6]), sd=sd(betas_dataC1[,6])), 
                geom="line", colour="red", size=1) +
  ylab("Densidad") +   ggtitle("n=100") + xlab(expression(hat(beta)[1])) +
  theme_minimal()

g3 <- ggplot(betas_dataC1) + 
  geom_histogram(aes(betas_dataC1[,7], y=..density..), col="blue", bins = 30) +
  stat_function(fun=dnorm, args=list(mean=mean(betas_dataC1[,7]), sd=sd(betas_dataC1[,7])), 
                geom="line", colour="red", size=1) +
  ylab("Densidad") +   ggtitle("n=500") + xlab(expression(hat(beta)[1])) +
  theme_minimal()

g4 <- ggplot(betas_dataC1) + 
  geom_histogram(aes(betas_dataC1[,8], y=..density..), col="blue", bins = 30) +
  stat_function(fun=dnorm, args=list(mean=mean(betas_dataC1[,8]), sd=sd(betas_dataC1[,8])), 
                geom="line", colour="red", size=1) +
  ylab("Densidad") +   ggtitle("n=1000") + xlab(expression(hat(beta)[1])) +
  theme_minimal()
grid.arrange(g1, g2, g3, g4, nrow=2, ncol=2)
```

Parte c 

```{r}
set.seed(123)
reps = 10000
betas = matrix(NA, nrow = reps, ncol = 8)
beta0 = 2
beta1 = 2.5
beta2 = 1

n = c(50, 100, 500, 1000)
for (j in 1:length(n)) {
  X1=rnorm(n[j],20,1)
  X2=runif(n[j],0,1)   #Caso 2
  
  for (i in 1:reps) {
    u= rnorm(n[j],0,1)
    v=(beta2*X2 + u )
    Ycs = beta0 + beta1*X1 + v 
    Yss = beta0 + beta1*X1 + beta2*X2 + u
    modelcs = lm(Ycs~X1)  
    betas[i,j] = modelcs$coef[2]
    modelss = lm(Yss~X1+X2) 
    betas[i,j+4] = modelss$coef[2]
  }
}
betas_dataC2 <- data.frame(betas)

```

Esperanza de los Betas

```{r}
eb1 <- apply(betas_dataC2[5:8],2,mean)
knitr::kable(
eb1 
)
```

Varianza de los Betas

```{r}
knitr::kable(
apply(betas_dataC2[5:8],2,var)
)
```

Grafica

```{r}
#graficos Caso 2

g5 <- ggplot(betas_dataC2) + 
  geom_histogram(aes(betas_dataC2[,5], y=..density..), col="black", bins = 30) +
  stat_function(fun=dnorm, args=list(mean=mean(betas_dataC2[,5]), sd=sd(betas_dataC2[,5])), 
                geom="line", colour="red", size=1) +
  ylab("Densidad") +   ggtitle("n=50") + xlab(expression(hat(beta)[1])) +
  theme_minimal()

g6 <- ggplot(betas_dataC2) + 
  geom_histogram(aes(betas_dataC2[,6], y=..density..), col="black", bins = 30) +
  stat_function(fun=dnorm, args=list(mean=mean(betas_dataC2[,6]), sd=sd(betas_dataC2[,6])), 
                geom="line", colour="red", size=1) +
  ylab("Densidad") +   ggtitle("n=100") + xlab(expression(hat(beta)[1])) +
  theme_minimal()

g7 <- ggplot(betas_dataC2) + 
  geom_histogram(aes(betas_dataC2[,7], y=..density..), col="black", bins = 30) +
  stat_function(fun=dnorm, args=list(mean=mean(betas_dataC2[,7]), sd=sd(betas_dataC2[,7])), 
                geom="line", colour="red", size=1) +
  ylab("Densidad") +   ggtitle("n=500") + xlab(expression(hat(beta)[1])) +
  theme_minimal()

g8 <- ggplot(betas_dataC2) + 
  geom_histogram(aes(betas_dataC2[,8], y=..density..), col="black", bins = 30) +
  stat_function(fun=dnorm, args=list(mean=mean(betas_dataC2[,8]), sd=sd(betas_dataC2[,8])), 
                geom="line", colour="red", size=1) +
  ylab("Densidad") +   ggtitle("n=1000") + xlab(expression(hat(beta)[1])) +
  theme_minimal()
grid.arrange(g5, g6, g7, g8, nrow=2, ncol=2)
```



