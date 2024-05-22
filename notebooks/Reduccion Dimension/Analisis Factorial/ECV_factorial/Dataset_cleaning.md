En este cuaderno vamos a procesar un conjunto de datos para posteriormente realizar un Análisis Factorial a partir de los microdatos de la Encuesta de Condiciones de Vida (ECV) del año 2013.

Para ello, usaremos datos **transversales** de la ECV del año 2013. Los microdatos de la ECV 2013 pueden descargarse en el siguiente link: <https://www.ine.es/dyngs/INEbase/es/operacion.htm?c=Estadistica_C&cid=1254736176807&menu=resultados&idp=1254735976608#!tabs-1254736195153>. El *.zip* que se descargue contiene dos subcarpetas; *datos_ecv2013* que será el que usemos; *disreg_ecv2013* que no lo utilizaremos por el momento.

Dentro de la carpeta *datos_ecv2013*, vamos a tomar:

-   **Fichero P**: datos detallados de los adultos (**esudb13p.csv**). En dicho fichero se encuentran las preguntas del módulo Transmisión intergeneracional de la pobreza y con el analizaremos la estructura factorial de la siguiente batería de preguntas relacionadas con el bienestar:

**¿Cuál es su grado de satisfacción global con…**

1.  … su vida en la actualidad? (variable PW010).
2.  … la situación económica en su hogar? (PW030).
3.  … su vivienda? (PW040).
4.  … su trabajo actual? (PW100).
5.  … el tiempo que dispone para hacer lo que le gusta? (PW120).
6.  … sus relaciones personales? (PW160).
7.  … las áreas recreativas o verdes de la zona en la que vive? (PW200).
8.  … la calidad de la zona en la que vive? (PW210).

En todos los casos la escala de respuesta va del 0 (Nada satisfecho) al 10 (Plenamente satisfecho).

# Procesamos el dataset para adaptarlo a lo de arriba

``` r
# Librería tratamiento dataframes
library(dplyr)


# Microdatos
# Fichero Personas
datos <- read.csv("/Users/davpero/Desktop/esudb13p.csv", sep = ",")
```

Vamos a crear una variable en el conjunto de datos datos_Fichero_P que sea igual a la variable identificadora del hogar HB030. Para ello, debemos quitar los dos últimos dígitos a la variable PB030. De esta forma, tendremos dos variables identificadoras idénticas que nos permitirán realizar la unión entre tablas.

``` r
# PB030 identificador único de la persona
datos <- datos[, c("PB030", "PW010", "PW030", "PW040", "PW100", "PW120", "PW160", "PW200", "PW210")]
head(datos)
```

``` r
# Creamos excel con datos
library("writexl")
write_xlsx(datos, "../../../../files/ECV_factorial.xlsx")
```

Este dataset será el que se proporcione para el estudiante para hacer sus análisis. [*ECV_factorial.xlsx*](../../../../files/ECV_factorial.xlsx)
