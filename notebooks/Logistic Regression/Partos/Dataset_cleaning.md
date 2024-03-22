En este cuaderno vamos a procesar un conjunto de datos para
posteriormente analizar la presencia de una cesárea en un parto a partir
de los microdatos de la **Estadística de nacimientos. Movimiento natural
de la población.** Concretamente, se han tomado los datos relativos a
**partos, 2022** y los relativos a la Comunidad Autónoma de **Navarra**.

Los microdatos de la *Estadística de nacimientos, Movimiento natural de
la población, 2022* pueden descargarse en el siguiente link:
<https://www.ine.es/dyngs/INEbase/es/operacion.htm?c=Estadistica_C&cid=1254736177007&menu=resultados&secc=1254736195443&idp=1254735573002#!tabs-1254736195443>.
De dichos microdatos leemos el fichero en formato .RData y vamos a tomar
las siguientes variables:

# Variables de Intés

Aquí se muestran las variables que vamos a escoger para el estudio de
una regresión logística y el nombre que tomará en el dataset exportado.

-   **MULTIPLI**: Número de bebés nacidos en el parto. (categórica)
-   **SEMANAS**: Número de semanas del embarazo.
-   **EDADM**: Edad de la madre en años cumplidos.
-   **CESAREA**: Si se ha llevado a cabo una césarea en el parto
    (Categórica. 1=si, 2=no).
-   **NORMA**: Si el parto transcurrió con normalidad. (Categórica.
    1=normal, 2=Complicaciones)

# Procesamos el dataset para adaptarlo a lo de arriba

``` r
# Librería tratamiento dataframes
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
# Microdatos
load("~/Downloads/datos_partos22/R/MNPpartos_2022.RData")
datos1 <- Microdatos[, c("MULTIPLI", "SEMANAS", "EDADM", "CESAREA", "PROMA", "NORMA")]

# Quitamos observaciones con NA
datos1 <- na.omit(datos1)

# Seleccionamos CCAA de Navarra, código 31
datos1 <- datos1 %>% filter(PROMA == 31) # 31

# Cambiamos niveles de cesarea
datos1$CESAREA <- as.factor(ifelse(datos1$CESAREA == 1, 1, 0))

datos1 <- datos1 %>% rename(
  bebes = MULTIPLI,
  semanas = SEMANAS,
  edad_madre = EDADM,
  cesarea = CESAREA,
  parto_normal = NORMA
)
```

Con esto nos queda el siguiente dataset.

-   **bebes**: Número de bebés nacidos en el parto. (categórica)
-   **semanas**: Número de semanas del embarazo.
-   **edad_madre**: Edad de la madre en años cumplidos.
-   **cesarea**: Si se ha llevado a cabo una césarea en el parto
    (Categórica. 0=no, 1=si).
-   **parto_normal**: Si el parto transcurrió con normalidad.
    (Categórica. 1=normal, 2=complicaciones)

``` r
# Creamos excel con datos
library("writexl")
write_xlsx(datos1,"/Users/davpero/Library/CloudStorage/GoogleDrive-davidperez202223@gmail.com/Mi unidad/4th Quarter/INE/2Datasets/Datos/Nuevos/partos.xlsx")
```

Este dataset será el que se proporcione para el estudiante para hacer
sus análisis.
