En este cuaderno vamos a procesar un conjunto de datos para
posteriormente analizar el nivel de estudios de una población(con gente
con muchos esudios y gente con muy pocos) a partir de los microdatos de
la **Encuestas de estructura salarial. Resultados** Concretamente, se
han tomado los datos relativos a **2018**.

Los microdatos de la *Encuestas de estructura salarial. Resultados*
pueden descargarse en el siguiente link:
<https://www.ine.es/dyngs/INEbase/es/operacion.htm?c=Estadistica_C&cid=1254736177025&menu=resultados&secc=1254736061996&idp=1254735976596#!tabs-1254736195110>.
De dichos microdatos leemos el fichero en formato .RData y vamos a tomar
las siguientes variables:

# Variables de Intés

Aquí se muestran las variables que vamos a escoger para el estudio de un
anaálisis de discriminante y el nombre que tomará en el dataset
exportado.

-   **ESTU**: Nivel de estudios del encuestado.
-   **RETRINOIN**: Sueldo bruto anual (sin incluir subsidios por
    incapacidad).
-   **RETRIIN**: Sueldo bruto anual por incapacidad.
-   **ANOS2**: Edad en años cumplidos del encuestado.
-   **ANOANTI**: Años de antigüedad.
-   **VAL**: Días de vacaciones al año.

# Procesamos el dataset para adaptarlo a lo de arriba

``` r
# Microdatos
library(haven)
datos <- read_sav("~/Downloads/datos_2018/SPSS/EES_2018.sav")
datos2 <- datos[, c("ESTU", "RETRINOIN", "RETRIIN", "ANOS2", "VAL", "ANOANTI")]

# El salario es la suma del salario de no incapacidad y del de incapacidad
datos2$salario <- as.numeric(datos2$RETRINOIN) + as.numeric(datos2$RETRIIN)

# Los que tiene salarios muy grandes los dejamos en 100k (marca de clase)
datos2$salario <- ifelse(datos2$salario > 100000, 100000, datos2$salario)
datos2$Estudios <- as.character(datos2$ESTU)
datos <- na.omit(datos)

# Filtramos los estudios seleccionando con "1" y "2" la gente que tiene estudios hasta primaria
# y con "6" y "7" los que tienen al menos esudiuos universitarios
datos2 <- dplyr::filter(datos2, ESTU == "1" | ESTU == "2" | ESTU == "6" | ESTU == "7")


# Nuevos nombres de variables
datos2$Estudios <- factor(ifelse(datos2$ESTU == "2", 1, ifelse(datos2$ESTU == "1", 1, 0)))
datos2$Antiguedad <- as.numeric(datos2$ANOANTI)
datos2$Salario <- as.numeric(datos2$salario)
datos2$Edad <- as.numeric(as.character(datos2$ANOS2))
datos2$Vacaciones <- as.numeric(datos2$VAL)
# Solo guardamos las que queremos
datos2 <- datos2[, c("Estudios", "Antiguedad", "Salario", "Edad", "Vacaciones")]


# Creamos excel con datos
library("writexl")
write_xlsx(datos2, "/Users/davpero/Desktop/laboral.xlsx")
```

Este dataset será el que se proporcione para el estudiante para hacer
sus análisis.
