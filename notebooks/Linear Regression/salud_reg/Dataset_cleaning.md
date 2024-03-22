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

Aquí se muestran las variables que vamos a escoger para el estudio de
una regresión lineal. En ella se pretende predecir el peso de una
persona en función de su edad, altura, sexo y IMC. Si se conociera el
valor exacto del IMC, entonces se podría saber el peso de manera exacta,
ya que
*I**M**C* = *P**e**s**o*/*A**l**t**u**r**a*<sup>2</sup>
, por lo que no sería necesario hacer un modelo para estimar la altura.

Por el contrario, no tenemos el valor del IMC exacto, sino una
clasificación del IMC entorno a 4 niveles. Este estudio demostrará como
a patir de esos niveles, ya se puede estimar el peso con bastante
exactitud, sin conocer el valor exacto del IMC.

-   **EDADa**: Identificación del adulto seleccionado: Edad.

-   **SEXOa**: Identificación del adulto seleccionado: Sexo.

-   **S109**: Altura en cm.

-   **S110**: Peso en kg.

-   **IMC**: Índice de masa corporal (IMC) del adulto. Toma los
    siguientes valores

    -   1: Peso insuficiente
    -   2: Normopeso
    -   3: Sobrepeso
    -   4: Obesidad

# Procesamos el dataset para adaptarlo a lo de arriba

``` r
# Microdatos
library(readxl)
datos<-read_excel("/Users/davpero/Downloads/BECA/datos_ensalud17_xlsx/MICRODAT.CA.xlsx")
datos = datos[,c("EDADa","SEXOa","S109","S110","IMCa")]
library(dplyr)
datos=na.omit(datos)




# Conevrtimos a factor variable sexo y la renombramos
datos$SEXO<-as.factor(ifelse(datos$SEXOa==1,1,0))

#Numéricas las demas y las renombramos
datos$EDAD<-as.numeric(datos$EDADa)
datos$Altura<-as.numeric(datos$S109)
datos$Peso<-as.numeric(datos$S110)
datos$IMC<-as.factor(datos$IMCa)


# Los valores atípicos los quitamos, los que tienen un peso y una altura que no tiene sentido
datos$Peso[datos$Peso>220]<-NA
datos$Altura[datos$Altura>220]<-NA
datos=na.omit(datos)


datos = datos[,c("EDAD","SEXO","Altura","Peso","IMC")]

# Creamos excel con datos
library("writexl")
write_xlsx(datos, "/Users/davpero/Desktop/salud.xlsx")
```

Este dataset será el que se proporcione para el estudiante para hacer
sus análisis.
