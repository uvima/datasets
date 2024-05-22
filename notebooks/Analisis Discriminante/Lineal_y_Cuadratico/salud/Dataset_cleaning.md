En este cuaderno vamos a procesar un conjunto de datos en bruto para generar un fichero accesible con el fin de discriminar el sexo de una persona a partir de los microdatos de la **Encuesta Nacional de Salud. Resultados** Concretamente, se han tomado los datos relativos a **2017**.

Los microdatos de la *Encuesta 2017* pueden descargarse en el siguiente link: <https://www.ine.es/dyngs/INEbase/es/operacion.htm?c=Estadistica_C&cid=1254736176783&menu=resultados&secc=1254736195295&idp=1254735573175#!tabs-1254736195295>. Tomamos el fichero relativo a **Adultos (15 años y más)**. De dichos microdatos leemos el fichero en formato .RData y vamos a tomar las siguientes variables:

# Variables de Interés

Aquí se muestran las variables que vamos a escoger para el estudio de un anaálisis de discriminante y el nombre que tomará en el dataset exportado.

-   **EDADa**: Identificación del adulto seleccionado: Edad.
-   **SEXOa**: Identificación del adulto seleccionado: Sexo.
-   **S109**: Altura en cm.
-   **S110**: Peso en kg.

# Procesamos el dataset para adaptarlo a lo de arriba

Destacar que las observaciones correspondientes a personas con más de 220kg y/o altura más de 220cm las hemos eliminado pues parecen errores.

``` r
# Microdatos
library(readxl)
datos <- read_excel("/Users/davpero/Downloads/BECA/datos_ensalud17_xlsx/MICRODAT.CA.xlsx")
datos <- datos[, c("EDADa", "SEXOa", "S109", "S110")]
library(dplyr)

# Quitamos las observaciones que carecen de valor
datos <- na.omit(datos)




# Conevrtimos a factor variable sexo y la renombramos
datos$SEXO <- as.factor(ifelse(datos$SEXOa == 1, 1, 0))

# Numéricas las demas y las renombramos
datos$EDAD <- as.numeric(datos$EDADa)
datos$Altura <- as.numeric(datos$S109)
datos$Peso <- as.numeric(datos$S110)

# Los valores atípicos los quitamos, los que tienen un peso y una altura que no tiene sentido
datos$Peso[datos$Peso > 220] <- NA
datos$Altura[datos$Altura > 220] <- NA
datos <- na.omit(datos)


# Nuevo conjunto de datos con las variables deseadas
datos <- datos[, c("EDAD", "SEXO", "Altura", "Peso")]

# Creamos excel con datos
library("writexl")
write_xlsx(datos, "../../../../files/salud.xlsx")
```

Este dataset será el que se proporcione para el estudiante para hacer sus análisis [*salud.xlsx*](../../../../files/salud.xlsx).
