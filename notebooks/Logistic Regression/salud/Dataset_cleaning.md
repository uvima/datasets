En este cuaderno vamos a procesar un conjunto de datos para
posteriormente discriminar el sexo de una persona a partir de los
microdatos de la **Encuesta Nacional de Salud. Resultados**
Concretamente, se han tomado los datos relativos a **2017**.

Los microdatos de la *Encuesta 2017* pueden descargarse en el siguiente
link:
<https://www.ine.es/dyngs/INEbase/es/operacion.htm?c=Estadistica_C&cid=1254736176783&menu=resultados&secc=1254736195295&idp=1254735573175#!tabs-1254736195295>.
Tomamos el fichero relativo a **Adultos (15 años y más)**. De dichos
microdatos leemos el fichero en formato .RData y vamos a tomar las
siguientes variables:

# Variables de Intés

Aquí se muestran las variables que vamos a escoger para el estudio de un
anaálisis de discriminante y el nombre que tomará en el dataset
exportado.

-   **EDADa**: Identificación del adulto seleccionado: Edad.
-   **SEXOa**: Identificación del adulto seleccionado: Sexo.
-   **S109**: Altura en cm.
-   **S110**: Peso en kg.

# Procesamos el dataset para adaptarlo a lo de arriba

``` r
# Microdatos
library(readxl)
datos<-read_excel("~/Downloads/datos_ensalud17_xlsx/MICRODAT.CA.xlsx")
datos = datos[,c("EDADa","SEXOa","S109","S110")]
library(dplyr)
datos=na.omit(datos)




# Conevrtimos a factor variable sexo
datos$SEXO<-as.factor(ifelse(datos$SEXOa==1,1,0))

#Numéricas las demas
datos$EDAD<-as.numeric(datos$EDADa)
datos$Altura<-as.numeric(datos$S109)
datos$Peso<-as.numeric(datos$S110)

# Los valores atípicos los quitamos, los que tienen un peso y una altura que no tiene sentido
datos$Peso[datos$Peso>250]<-NA
datos$Altura[datos$Altura>250]<-NA
datos=na.omit(datos)

# Creamos excel con datos
library("writexl")
write_xlsx(datos2, "/Users/davpero/Desktop/salud.xlsx")
```

Este dataset será el que se proporcione para el estudiante para hacer
sus análisis.
