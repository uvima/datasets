En este cuaderno vamos a procesar un conjunto de datos para
posteriormente realizar un Análisis Factorial a partir de los microdatos
del informe PISA(informe del Programa para la Evaluación Internacional
de los Estudiantes) en España en el año 2018.

Para ello, usaremos datos de la prueba PISA del año 2018 que pueden
descargarse en el siguiente link:
<https://www.educacionyfp.gob.es/inee/bases-datos/evaluaciones-internacionales/pisa.html>.
Se pueden descargar en el siguiente link en formato SPSS o SAS y
nosotros transformaremos a un excel los que nos interesan.

Tomamos el fichero *“Datos_PISA_ESP.csv”*

Específicamente, estos datos proceden del Cuestionario de contexto del
alumno, cuestionario que deben rellenar todos los alumnos que pasan PISA
y que trata de medir la existencia de variables socioeconómicas,
metacognitivas, motivacionales e, incluso, emocionales, que pueden tener
impacto sobre el rendimiento académico. Entre las variables relacionadas
con la motivación podemos encontrar:

-   La afición por la lectura.
-   La actitud hacia la Educación.
-   La competitividad.
-   La perseverancia.
-   El miedo al fracaso.
-   La autoeficacia.
-   La orientación a metas de aproximación a la maestría.

Pero, para simplificar más las cosas y no trabajar con un número
considerable de factores, nos vamos a centrar simplemente en tres: la
competitividad, la perseverancia y el miedo al fracaso.

Los items (preguntas) que miden respectivamente la competitividad, al
perseverancia y el miedo al fracaso de los estudiantes son:

1.  **¿Hasta qué punto estás de acuerdo con las siguientes afirmaciones
    sobre ti mismo?**

-   ST181Q02HA: Disfruto trabajando en situaciones que requieren
    competir con los demás.
-   ST181Q03HA: Es importante para mí hacerlo mejor que los demás al
    realizar una tarea.
-   ST181Q04HA: Me esfuerzo mucho cuando estoy compitiendo contra los
    demás.

1.  **¿Hasta qué punto estás de acuerdo con las siguientes afirmaciones
    sobre ti mismo?**

-   ST182Q03HA: Me siento satisfecho cuando me esfuerzo todo lo que
    puedo.
-   ST182Q04HA: Cuando inicio una tarea continúo hasta terminarla.
-   ST182Q05HA: Cuando hago algo, parte de mi satisfacción se debe a que
    he mejorado mis resultados anteriores.
-   ST182Q06HA: Si algo no se me da bien, prefiero seguir esforzándome
    para mejorar, en lugar de hacer otra cosa que sí se me da bien.

1.  **¿Hasta qué punto estás de acuerdo con las siguientes
    afirmaciones?**

-   ST183Q01HA: Cuando me he equivocado, me preocupa lo que otras
    personas piesen de mí.
-   ST183Q02HA: Cuando me he equivocado, me preocupa no tener el talento
    suficiente.
-   ST183Q03HA: Cuando me he equivocado, dudo sobre mis planes para el
    futuro.

La escala de respuesta para estos tres conjuntos de ítems es la misma:
1 - Totalmente en desacuerdo, 2 - En desacuerdo, 3 - De acuerdo y 4 -
Totalmente de acuerdo.

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
# Fichero Personas
datos <- read.csv("/Users/davpero/Downloads/Datos_PISA_ESP.csv", sep=";", dec=",")
#Este csv esta separado por ; y los decimales son ,
dim(datos) #Tenemos 35943 observaciones y 37 columnas
```

    ## [1] 35943    37

``` r
head(datos)
```

    ##   CNTRYID CNT CNTSCHID CNTSTUID ST160Q01IA ST160Q02IA ST160Q03IA ST160Q04IA
    ## 1     724 ESP 72400001 72400490          2          2          2          2
    ## 2     724 ESP 72400001 72401482          3          1          2          3
    ## 3     724 ESP 72400001 72402362          3          1          1          2
    ## 4     724 ESP 72400001 72402959          1          1          1          1
    ## 5     724 ESP 72400001 72403316          1          3          2          1
    ## 6     724 ESP 72400001 72403522          3          1          2          3
    ##   ST160Q05IA JOYREAD ST036Q05TA ST036Q06TA ST036Q08TA ATTLNACT ST181Q02HA
    ## 1          3 -0.1042          2          2          2  -0.6583          3
    ## 2          3 -0.8176          1          1          1   1.0844          3
    ## 3          3 -0.9016          1          1          1   1.0844          4
    ## 4          1 -0.1119          3          2          1  -0.6506          4
    ## 5          1  0.9011          2          1          1   0.4667          3
    ## 6          3 -0.8176          2          2          1   0.0083          3
    ##   ST181Q03HA ST181Q04HA COMPETE ST182Q03HA ST182Q04HA ST182Q05HA ST182Q06HA
    ## 1          3          3  0.1956          3          3          3          3
    ## 2          2          3 -0.2661          4          4          4          3
    ## 3          2          3  0.1244          4          4          4          3
    ## 4          2          4  0.7262          3          3          2          3
    ## 5          1          2 -1.0214          4          3          3          3
    ## 6          2          3 -0.2661          4          3          3          3
    ##   WORKMAST ST183Q01HA ST183Q02HA ST183Q03HA GFOFAIL ST188Q01HA ST188Q02HA
    ## 1  -0.4540          2          2          3 -0.3700          3          3
    ## 2   1.7124          2          2          2 -0.6870          2          3
    ## 3   1.7124          3          3          3  0.4637          3          3
    ## 4  -0.9415          4          3          4  1.0671          3          2
    ## 5   0.2119          3          2          2 -0.4342          3          3
    ## 6   0.2119          2          3          2 -0.1300          4          3
    ##   ST188Q03HA ST188Q06HA ST188Q07HA RESILIENCE ST208Q01HA ST208Q02HA ST208Q04HA
    ## 1          2          2          3    -0.8109          3          3          3
    ## 2          3          3          3    -0.4531          4          3          5
    ## 3          3          2          2    -0.8738          5          5          5
    ## 4          3          3          3    -0.4909          4          4          4
    ## 5          3          3          3    -0.0614          2          2          3
    ## 6          3          4          3     0.7512          4          4          4
    ##   MASTGOAL
    ## 1  -0.4347
    ## 2   0.4317
    ## 3   1.8524
    ## 4   0.5761
    ## 5  -1.0266
    ## 6   0.5761

``` r
#Vamos a extraer simplemente las columnas de identificador del alumno y las de los items que nos interesan (los que miden la competitividad, la perseverancia y el miedo al fracaso):
datos <- datos[,c("CNTSTUID", "ST181Q02HA", "ST181Q03HA", "ST181Q04HA", "ST182Q03HA", "ST182Q04HA", "ST182Q05HA", "ST182Q06HA", "ST183Q01HA", "ST183Q02HA", "ST183Q03HA")]
```

``` r
# Creamos excel con datos
library("writexl")
write_xlsx(datos, "/Users/davpero/Desktop/jaja.xlsx")
```

Este dataset será el que se proporcione para el estudiante para hacer
sus análisis.
