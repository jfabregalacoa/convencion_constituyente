---
title: "ConvencionalesConstituyentes"
author: "Jorge Fábrega"
date: "18-05-2021"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
require(readxl)
library(shape)
```

## Estimación de la ideología de los miembros de la Convención Constituyente

El 15 y 16 de mayo del 2021, Chile fue a las urnas para elegir a las 155 personas que deberán redactar un nuevo texto constitucional. Al finalizar la jornada, muchos se llevaron una gran sorpresa al constatar que un número significativo de las personas electas no estaban adscritas a ningún partido político ¿Qué piensan y cómo las ubicamos en los tradicionales ejes ideológicos? 

A medida que las semanas pasen, esa incertidumbre se irá despejando. :smiley:

No obstante, como un esfuerzo para contribuir a reducir la incertidumbre, en este sitio presento una estimación ideológica de los Convencionales Constituyentes, incluidos los independientes. Se trata de un proyecto que se irá actualizando en la medida que se disponga de información nueva y relevante. 

En su versión inicial, se cuenta con una estimación ideológica de 135 de los 155 miembros. En la siguiente sección explico cómo se hizo la estimación y los supuestos que se realizaron. Posteriormente presento algunos gráficos ilustrativos de las posiciones ideológicas de miembros de la convención. Finalmente, presento las estimaciones para que puedan ser usadas por investigadores y público en general que desee construir nuevos análisis a partir de ellos. 

Las bases de datos con las estimaciones están depositadas en el siguiente repositorio de Dataverse.

# El método
El Diario La Tercera hizo una encuesta a los candidatos a la convención constitucional (932 postulantes respondieron su encuesta) y entre los finalmente electos, 120. Los datos fueron utilizados para elaborar un sitio interactivo denominado [Tu March Constituyente](https://interactivo.latercera.com/tu-match-constituyente/candidatos-constituyentes/). La encuesta contenía set de preguntas tales como:  
> ¿Qué se debe introducir en la nueva Constitución respecto a este tema?
> ¿Qué cambios se le deben realizar al TC?
> ¿El Congreso Nacional debe tener una o dos cámaras?
  
Las alternativas de respuesta contenían afirmaciones que tienen la virtud de discriminar posturas políticamente relevantes. Por ejemplo, en relación a la composición del congreso, las alternativas eran:

> a. Debe haber un Congreso bicameral como actualmente existe: Senado y Cámara de Diputadas y Diputados.
> b. Se debe mantener la estructura de las dos Cámaras pero dejando una de ellas para la representación territorial y otra para la representación política.
> c. Congreso unicameral: una sola Cámara.

El ejercicio de estimación convirtió las posibles respuestas en dummies. Ello puede interpretarse del siguiente modo: "¿selecciona el/la candidato/a la alternativa $z$?: Sí o No". Con esta transformación, se contruye una matriz de respuestas que pueden ser interpretadas como votaciones en un cuerpo colegiado. Y, debido a que se trata de preguntas políticamente relevantes, se puede asumir que las diferencias en los patrones observados de respuesta deben responder a concepciones ideológicas que distinguen a unos de otros.  

Bajo esa premisa, se puede proceder a estimar cuál es el mejor vector o conjunto de vectores que mejor refleja dichas diferencias de visión entre los miembros de la convención. Para hacer la estimación se obtuvieron los datos de las respuestas de todos los constituyentes a todas las alternativas y se aplicaron métodos frecuentistas (W-NOMINATE) y bayesianos de estimación de ideología. 

El siguiente gráfico muestra que ambos métodos de estimación generan resultados similares con algunas variaciones en las estimaciones hacia los extremos de la distribución.

```{r metodos, echo=FALSE,warning=FALSE,message=FALSE,error=FALSE}
require(here)
library(pscl)
require(wnominate)
aqui <- here()
archivo <- read.csv(paste0(aqui,"/data/MatchLaTercera.csv"),sep=";", encoding = "UTF-8")
colnames(archivo)[1:2] <- c("nombre","candidato")
candidatos <- as.list(archivo$candidato)
archivo.origen <- archivo[,12:NCOL(archivo)]
z <- NA
x <- as.numeric(matrix(NA,nrow=NROW(archivo.origen), ncol=1))
for(i in 1:NCOL(archivo.origen)){ # para cada pregunta
  alternativas <- unique(archivo.origen[,i])
  alternativas.num <- length(unique(archivo.origen[,i]))
  for(j in 1:alternativas.num){ # para cada alternativa de cada pregunta
    z.aux <- paste("p",i,"_",j,sep="")
    z <- cbind(z,z.aux)
    x.aux <- matrix(NA,nrow=NROW(archivo.origen), ncol=1)
    for(k in 1:NROW(archivo.origen)){ # para cada candidato
      x.aux[k] <- ifelse((archivo.origen[k,i]==alternativas[j])==T,1,0)
    }
    x <- cbind(x,x.aux)
  }
}

x <- as.data.frame(x[,2:NCOL(x)])

votos <- cbind(archivo$candidato,x)
colnames(votos) <- z
colnames(votos)[1] <- "candidato"
candidatos <- votos[,1]
write.csv(votos,paste0(aqui,"/data/votos_total_pais.csv"))
votos <- votos[,2:NCOL(votos)]

rc_total <- rollcall(votos,             
                   yea=c(1), # reduce los valores a dos grupos yea/nay
                   nay=c(0),
                   notInLegis=NULL, # vector de ausentes en que seccion
                   legis.names=candidatos,
                   legis.data=NULL,
                   desc="Todo Chile")

# usando NOMINATE
result_total <- wnominate(rc_total, dims=2, polarity=c(26,26))

mcmc_e <- pscl::ideal(rc_total, codes = rc_total$codes,
                      maxiter = 10000, burnin = 1000, thin = 250,
                      normalize = T) # ver help para detalles

```

```{r metodos_comparados, echo=FALSE,warning=FALSE,message=FALSE,error=FALSE}
plot(-mcmc_e$xbar[,1], result_total$legislators$coord1D, 
     xlab="Estimacion Bayesiana (1Dim)", ylab="W-NOMINATE (1Dim)",
     main ="Comparación entre métodos",
     sub="Estimación propia en base a datos de TuMatchConstituyente, La Tercera")
text(1.4,-0.9,"@jorgefabrega", cex=0.5)
```

Dado el resultado anterior, el resto del análisis se hizo considerando la estimación bayesiana puesto que permite generar intervalos de confiabilidad sobre los resultados obtenidos. 

```{r electos, echo=FALSE,warning=FALSE,message=FALSE,error=FALSE}
archivo <- read.csv(paste0(aqui,"/data/estimaciones_ideologia.csv"),sep=",",
                    encoding = "unknown")
archivo.complemento <- read.csv(paste0(aqui,"/data/estimaciones_ideologia_matchlistas.csv"),
                                sep=";", encoding = "unknown")

base <- merge(archivo,
              archivo.complemento, 
              by.x = "candidatura",
              by.y = "candidatura")

# base con nombre agregado de las listas
archivo.complemento <- read.csv(paste0(aqui,"/data/estimaciones_ideologia_matchlistas.csv"),
                                sep=";", encoding = "unknown")

# base con los resultados
base_electos.aux <- read_excel(paste0(aqui,"/data/cc_electos_real.xlsx"))
base_electos <- read.csv(paste0(aqui,"/data/elegidos_con_estimacion_4.csv"),sep=";")
base_elegidos <- merge(base_electos.aux,base_electos, all = T)
#base_elegidos <- base[base$candidatura %in% base_electos$candidatura,]

# base con ideología filtrados.

# los elegidos que no están en la bbdd con estimaciones y hay que agregar
#aux <- base_electos[!(base_electos %in% base$candidatura)] 
#aux.2 <- base_electos[base_electos$candidatura %in% aux,]
#write.csv(aux.2,paste0(aqui,"/data/para_imputar_media.csv"))




# base ordenada por ideologia
base_elegidos <- base_elegidos[order(base_elegidos$media),]
base_elegidos$candidatura <- factor(base_elegidos$candidatura, 
                                    levels = base_elegidos$candidatura[order(base_elegidos$media)])
colnames(base_elegidos)[NCOL(base_elegidos)] <- "Lista"
base_elegidos$id <- seq(1:NROW(base_elegidos))


# grafico
titulo <- paste0("Estimación ideológica de Convencionales Constituyentes")
# ggplot(base_elegidos, aes(x=media, y=candidatura, color=Lista)) + geom_point(size = 3) +
#   geom_errorbarh(aes(xmin = err_min, xmax = err_max),
#                  height = 0) + ggtitle(titulo) +
#   xlab("Posición ideológica estimada") +
# #  scale_fill_brewer(palette="Dark2") + 
#   scale_x_continuous(limits = c(-3, 3)) + 
#   theme_minimal()

# metodo 2 para establecer rango (rango)

v <- 19 # casos sin ideologia estimada

a <- (155 - 1 -v)*2/3  # pivotal hacia la derecha
b <- (155 - 1 -v) - a # pivotal hacia la izquierda
r <- 10 # rango +/-

a.u1 <- a + 1
a.u2 <- a + r
a.d1 <- a - 1
a.d2 <- a - r

b.u1 <- b + 1
b.u2 <- b + r
b.d1 <- b - 1
b.d2 <- b - r

base_elegidos$grupo <- "otros"
base_elegidos$grupo[a.u1:a.u2] <- "en rango"
base_elegidos$grupo[a.d1:a.d2] <- "en rango"
base_elegidos$grupo[b.u1:b.u2] <- "en rango"
base_elegidos$grupo[b.d1:b.d2] <- "en rango"
base_elegidos$grupo[a] <- "pivotal"
base_elegidos$grupo[b] <- "pivotal"

# agregado por listas
res <- aggregate(base_elegidos[, 21:26], list(base_elegidos$sigla), mean)
res.min <- aggregate(base_elegidos$err_min, list(base_elegidos$sigla), min)
res.max <- aggregate(base_elegidos$err_max, list(base_elegidos$sigla), max)
res$mintot <- res.min$x
res$maxtot <- res.max$x
res$Lista <- factor(res$Group.1,levels = res$Group.1[order(res$media)])
res <- res[is.na(res$media)==FALSE,]
```

El siguiente gráfico resume los resultados de la estimación. Cada punto representa un miembro de la convención. La línea horizontal que cruza cada punto, es el intervalo de confiabilidad estimado. Dado que son muchas personas, sólo se incluyen los nombres de algunas de ellas en el gráfico o de lo contrario, no se podrían distinguir. 

Ahora bien, lo más relevante de la figura es lo que está representado en los colores. En rojo se destaca a los votantes pivotales bajo una regla de 2/3. Mirando de arriba hacia abajo, el primer punto rojo es el votante pivotal para incluir en la constitución algún texto que es preferido desde miembros ubicados a la izquierda de la distribución. El punto rojo de más abajo es el votante pivotal cuando dichos textos son promovidos desde la derecha de la distribución. Los puntos en amarillo contienen un rango de veinte personas en torno al punto rojo. Ese rango refleja el hecho que hay aún veinte personas para las que no tenemos una estimación ideológica. Por ende, ese es el rango en que puede variar la ubicación del votante pivotal en cada caso.

Por último, es importante aclarar que ésta es una escala normalizada (esto es, centrada en cero). Por ende, no debe interpretarse el gráfico como si los números positivos significa que las personas allí representadas son de derecha y los números negativos, de izquierda. Al respecto no debe olvidarse que las candidaturas políticamente identificadas como de derecha (Lista Vamos por Chile) no alcanzaron un tercio del total de escaños.   

```{r estimacion, echo=FALSE,warning=FALSE,message=FALSE,error=FALSE}
require(ggplot2)

base_elegidos.x <- base_elegidos[is.na(base_elegidos$media)==F,]
q <- 5
nombres <- c(base_elegidos.x$nombre0[1],"","","","",
                            base_elegidos.x$nombre0[1+q],"","","","",
                            base_elegidos.x$nombre0[1+2*q],"","","","",
                            base_elegidos.x$nombre0[1+3*q],"","","","",
                            base_elegidos.x$nombre0[1+4*q],"","","","",
                            base_elegidos.x$nombre0[1+5*q],"","","","",
                            base_elegidos.x$nombre0[1+6*q],"","","","",
                            base_elegidos.x$nombre0[1+7*q],"","","","",
                            base_elegidos.x$nombre0[1+8*q],"","","","",
                            base_elegidos.x$nombre0[1+9*q],"","","","",
                            base_elegidos.x$nombre0[1+10*q],"","","","",
                            base_elegidos.x$nombre0[1+11*q],"","","","",
                            base_elegidos.x$nombre0[1+12*q],"","","","",
                            base_elegidos.x$nombre0[1+13*q],"","","","",
                            base_elegidos.x$nombre0[1+14*q],"","","","",
                            base_elegidos.x$nombre0[1+15*q],"","","","",
                            base_elegidos.x$nombre0[1+16*q],"","","","",
                            base_elegidos.x$nombre0[1+17*q],"","","","",
                            base_elegidos.x$nombre0[1+18*q],"","","","",
                            base_elegidos.x$nombre0[1+19*q],"","","","",
                            base_elegidos.x$nombre0[1+20*q],"","","","",
                            base_elegidos.x$nombre0[1+21*q],"","","","",
                            base_elegidos.x$nombre0[1+22*q],"","","","",
                            base_elegidos.x$nombre0[1+23*q],"","","","",
                            base_elegidos.x$nombre0[1+24*q],"","","","",
                            base_elegidos.x$nombre0[1+25*q],"","","","",
                            base_elegidos.x$nombre0[1+26*q],"","","",
                            base_elegidos.x$nombre0[135])

ggplot(base_elegidos.x, aes(x=media, y=candidatura, color=grupo)) + geom_point(size = 3) +
  geom_errorbarh(aes(xmin = err_min, xmax = err_max), height = 0) + 
  ggtitle(titulo) +
  xlab("Posición ideológica estimada") +
  ylab("Algunos casos seleccionados") +
  scale_x_continuous(limits = c(-3, 3)) + 
  scale_color_manual(values = c("otros" = "grey", "pivotal" = "red", "en rango" = "gold")) +
  theme_minimal() + 
  scale_y_discrete(labels=nombres)
```
Ahora bien,una de las preguntas que más interesa sabes es dónde están ubicados ideológicamente las personas elegidas que no provienen de los partidos y grupos políticos tradicionales. Particularmente, ¿dónde se ubican ideológicamente las personas de la lista del pueblo? En el siguiente gráfico se presenta la misma información del gráfico precedente pero agrupados por lista. Como puede verse, la lista del pueblo se ubica en un punto intermedio de Apruebo Dignidad y la Lista del Apruebo. 

```{r gr_lista, echo=FALSE,warning=FALSE,message=FALSE,error=FALSE}
pueblo <- read.csv(paste0(aqui,"/data/del_pueblo.csv"),sep=";", encoding = "UTF-8")
ap_dignidad <- read.csv(paste0(aqui,"/data/apruebo_dignidad.csv"),sep=";", encoding = "UTF-8")
apruebo <- read.csv(paste0(aqui,"/data/l_apruebo.csv"),sep=";", encoding = "UTF-8")
no_neutrales <- read.csv(paste0(aqui,"/data/no_neutrales.csv"),sep=";", encoding = "UTF-8")
chile_vamos <- read.csv(paste0(aqui,"/data/chile_vamos.csv"),sep=";", encoding = "UTF-8")
otros_fuera_pacto <- read.csv(paste0(aqui,"/data/otros_fuera_pacto.csv"),sep=";", encoding = "UTF-8")

base_elegidos.y1 <- base_elegidos[base_elegidos$nombre0 %in% as.list(pueblo$nombre),]
base_elegidos.y2 <- base_elegidos[base_elegidos$nombre0 %in% as.list(ap_dignidad$nombre),]
base_elegidos.y3 <- base_elegidos[base_elegidos$nombre0 %in% as.list(apruebo$nombre),]
base_elegidos.y4 <- base_elegidos[base_elegidos$nombre0 %in% as.list(no_neutrales$nombre),]
base_elegidos.y5 <- base_elegidos[base_elegidos$nombre0 %in% as.list(chile_vamos$nombre),]
base_elegidos.y6 <- base_elegidos[base_elegidos$nombre0 %in% as.list(otros_fuera_pacto$nombre),]
base_elegidos.y6 <- base_elegidos.y6[is.na(base_elegidos.y6$media)==F,]

base_elegidos.y1$vertical <- -0.05
plot(base_elegidos.y1$media,base_elegidos.y1$vertical,
     axes = F, 
     ylab = "", 
     xlab = "",
     pch = 19, 
     col = "purple",
     ylim = c(-1.3,2),
     xlim = c(-2.3,2.3),
     main = "Posiciones ideológicas estimadas por lista")
for(i in 1:NROW(base_elegidos.y2)){
  points(base_elegidos.y2$media[i],-0.6, col = "red", pch = 19, cex = 1.1)
}
for(i in 1:NROW(base_elegidos.y3)){
  points(base_elegidos.y3$media[i],0.8, col = "orange", pch = 19, cex = 1.1)
}
for(i in 1:NROW(base_elegidos.y4)){
  points(base_elegidos.y4$media[i],1.4, col = "gold", pch = 19, cex = 1.1)
}
for(i in 1:NROW(base_elegidos.y5)){
  points(base_elegidos.y5$media[i],1.8, col = "blue", pch = 19, cex = 1.1)
}
for(i in 1:NROW(base_elegidos.y6)){
  points(base_elegidos.y6$media[i],0.3, col = "green", pch = 19, cex = 1.1)
}
text(mean(base_elegidos.y1$media),-0.19,"Lista del Pueblo")
text(mean(base_elegidos.y2$media),-0.81,"Aprueblo Dignidad")
text(mean(base_elegidos.y3$media),0.67,"Apruebo")
text(mean(base_elegidos.y4$media),1.2,"No Neutrales")
text(mean(base_elegidos.y5$media),1.6,"Vamos por Chile")
text(mean(base_elegidos.y6$media),0.2,"Otros")

segments(min(base_elegidos.y2$media), -1.2, max(base_elegidos.y5$media),-1.2, col = "black")
Arrows(min(base_elegidos.y2$media), -0.5, min(base_elegidos.y2$media),-0.9, col = "black")
Arrows(max(base_elegidos.y5$media), 1.39, max(base_elegidos.y5$media),-0.9, col = "black")

# for(j in 1:23){
#   s <- (-1)^j + (-0.1)^j 
# segments(base_elegidos.y$media[j], s, base_elegidos.y$media[j], y1 = 0, col = "lightblue")
# text(base_elegidos.y$media[j],s,base_elegidos.y$nombre0[j], y1 = 0,srt = 45)
# }


```

Finalmente, volviendo sobre la identificación de actores pivotales. En el gráfico siguiente se destacan los actores que serían claves en la discusión en la convención constituyente. Actores que por su ubicación en la distribución ideológica están son los que con mayor probabilidad dan los mínimos necesarios para cumplir con los quórums que la convención establezca en su reglamento.

En azul están los votantes medianos. Es decir, los que tienen al enumerar hacia su derecha o izquierda se contabiliza la misma cantidad de personas. Dado eso, ellos son los actores que dan las mayorías simples para proyectos. Y en rojo se presentan los actores pivotales de 2/3 explicado anteriormente. Ahora bien, a estas alturas que el proceso constituyente está empezando, más importante que saber los nombres específicos, es entender las visiones que están en una posicioón de ser pivotales en una discusión. Ello por dos motivos, primero porque la propia estimación tiene un margen de variación que ya expliqué en un gráfico anterior y, por ende, para efectos estadísticos cuando dos miembros de la convención tienen estimaciones traslapadas, no podemos afirmar cuál de ellos está más a la derecha o más a la izquierda. Y segundo, porque no hay que olvidar que hay aún 20 personas para las cuales este ejercicio no ha hecho una estimación de su posición ideológica. 

```{r rangos,echo=FALSE,warning=FALSE,message=FALSE,error=FALSE}
# plot horizontal
pivotal_1 <- base_elegidos$media[a]
pivotal_2 <- base_elegidos$media[b]
enrango1 <- base_elegidos$media[a.d1:a.d2]
enrango2 <- base_elegidos$media[a.u1:a.u2]
enrango3 <- base_elegidos$media[b.d1:b.d2]
enrango4 <- base_elegidos$media[b.u1:b.u2]

base_elegidos$vertical <- 0
res$vertical <- 0
```

```{r gr_casos_seleccionados,echo=FALSE,warning=FALSE,message=FALSE,error=FALSE}
plot(base_elegidos$media,base_elegidos$vertical,
     axes = F, 
     ylab = "", 
     xlab = "",
     pch = 19, 
     col = "grey",
     ylim = c(-1.5,1.5),
     xlim = c(-2.3,2.3),
     main = "Actores pivotales para los Convencionales Constituyentes")
for(i in 1:r){
  points(enrango1[i],0, col = "gold", pch = 19, cex = 1.1)
  points(enrango2[i],0, col = "gold", pch = 19, cex = 1.1)
  points(enrango3[i],0, col = "gold", pch = 19, cex = 1.1)
  points(enrango4[i],0, col = "gold", pch = 19, cex = 1.1)
  }
points(pivotal_1, 0, col = "red", pch = 19, cex = 1.4)
points(pivotal_2, 0, col = "red", pch = 19, cex = 1.4)
points(base_elegidos$media[77],0, col = "blue", pch = 19, cex = 1.4)
points(base_elegidos$media[78],0, col = "blue", pch = 19, cex = 1.4)

segments(max(base_elegidos$media),-0.2,base_elegidos$media[b],-0.2)
segments(min(base_elegidos$media),0.2,base_elegidos$media[a],0.2)
text((max(base_elegidos$media)+base_elegidos$media[b])/2,-0.4,"2/3")
text((min(base_elegidos$media)+base_elegidos$media[b])/2,0.4,"2/3")

Arrows(pivotal_1, 1, pivotal_1, y1 = 0.1, col = "blue",arr.type="simple", arr.width=0.1)
Arrows(pivotal_2, -1, pivotal_2, y1 = -0.1, col = "blue",arr.type="simple", arr.width=0.1)
Arrows(base_elegidos$media[67], 0.5, base_elegidos$media[77], y1 = 0.1, col = "blue",arr.type="simple", arr.width=0.1)
Arrows(base_elegidos$media[68], -0.5, base_elegidos$media[78], y1 = -0.1, col = "blue",arr.type="simple", arr.width=0.1)
text(pivotal_1,1.1,base_elegidos$nombre_corto[a])
text(pivotal_2,-1.1,base_elegidos$nombre_corto[b])
text(base_elegidos$media[67],0.6,base_elegidos$nombre0[67])
text(base_elegidos$media[68],-0.6,base_elegidos$nombre0[68])
```

Los datos que sustentan los gráficos precedentes se encuentran disponible [aquí](https://dataverse.harvard.edu/file.xhtml?fileId=4656268&version=1.0)

Si usted tiene comentarios o consultas, por favor dirigirlas al email jfabrega@udd.cl.

Las personas para las cuales no se ha estimado una posición política son las siguientes:

```{r datos2,echo=FALSE,warning=FALSE,message=FALSE,error=FALSE}
require(kableExtra)

z <- base_elegidos$nombre0[is.na(base_elegidos$media)==T]
kable(z)

```