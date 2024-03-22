En este cuaderno vamos a procesar un conjunto de datos para
posteriormente analizar la existencia del llamado “ascensor social” en
España a partir de los microdatos de la Encuesta de Condiciones de Vida
(ECV) del año 2019. Dicho análisis nos sirve como excusa para tratar de
mostrar en qué consiste una regresión logística y cómo llevarla a cabo
en R.

En primer lugar, debemos definir qué es eso del “ascensor social” y cómo
vamos a tratar de analizarlo nosostros. Éste se define como la
posibilidad de ascender o descender de clase social. Aunque podemos
considerar que la pertenencia a una determinada clase social - si es que
éstas existen de forma estanca y perfectamente distinguible - se explica
por una combinación de aspectos: nivel económico, cultural, de estudios,
etc, nosotros nos vamos a centrar simplemente en el nivel económico.

Para ello, usaremos datos transversales de la ECV del año 2019. Este es
el último año - hasta la fecha (2021) - que la ECV ha contado con el
modulo temático Transmisión intergeneracional de la pobreza - batería de
preguntas que nos permiten realizar el análisis deseado.

Los microdatos de la ECV 2019 pueden descargarse en el siguiente link:
<https://www.ine.es/dyngs/INEbase/es/operacion.htm?c=Estadistica_C&cid=1254736176807&menu=resultados&idp=1254735976608#!tabs-1254736195153>.
Dichos microdatos están formados por cuatro archivos:

-   **Fichero D**: datos básicos del hogar (esudb19d.csv).
-   **Fichero R**: datos básicos de la persona (esudb19r.csv).
-   **Fichero H**: datos detallados del hogar (esudb19h.csv).
-   **Fichero P**: datos detallados de los adultos (esudb19p.csv). En
    dicho fichero se encuentran las preguntas del módulo Transmisión
    intergeneracional de la pobreza.

Para poder unir los ficheros contamos con una variable de identificación
por fichero. Las variables de identificación de los ficheros de hogares
(DB030 y HB030) son idénticas; lo mismo para las variables de
identificación de los ficheros de personas (RB030 y PB030). Éstas
últimas se componen del identificador del hogar y el nº de orden, a dos
dígitos, de la persona dentro del hogar.

Asimismo, al descargarse los microdatos, en la misma carpeta comprimida,
encontramos un documento Word que contiene la explicación de todas las
variables de la encuesta.

## ¿Cómo determinamos si una persona ha mejorado o no su situación económica?

Para responder a esta pregunta vamos a utilizar las respuestas de los
encuestados a dos cuestiones concretas:

-   **Situación económica del hogar cuando el adulto era adolescente**.
    Las opciones de respuesta son:

1.  Muy mala
2.  Mala
3.  Moderadamente mala
4.  Moderadamente buena
5.  Buena
6.  Muy buena

Esta pregunta pertenece al módulo Trasmisión intergeneracional de la
pobreza y, por tanto, se encuentra en el fichero P. Es la pregunta
PT190.

-   **Capacidad del hogar para llegar a fin de mes**. Cuyas opciones de
    respuesta son:

1.  Con mucha dificultad.
2.  Con dificultad.
3.  Con cierta dificultad.
4.  Con cierta facilidad.
5.  Con facilidad.
6.  Con mucha facilidad.
7.  Esta pregunta se refiere al hogar y se encuentra, específicamente,
    en el fichero H. Es la HS120.

Como vemos, las preguntas no son idénticas, ni siquiera sus opciones de
respuesta lo son. Pero sí resultan muy similares y podemos utizarlas
como proxy para determinar si una persona ha mejorado o no su situación
económica.

Vamos a superponer, pues, lo siguiente: una persona ha mejorado su
situación económica, es decir, ha logrado subir en el ascensor social,
si ha mejorado en el número de su respuesta de la pregunta 1 a la 2. Por
ejemplo: una persona que responde Muy mala (opción 1) en la pregunta 1,
y Con cierta dificultad (opción 3) en la segunda pregunta, aunque su
situación econónomica siga sin calificarla de “buena”, sí que ha
mejorado. Por el contrario si el número de la respuesta es el mismo o
menor en la pregunta 2 que en la 1, dicha persona no habrá mejorado.

De esta manera, clasificaremos a los encuestados en dos categorías: 1)
Personas que han mejorado - han subido con el ascensor social; 2)
Personas que no han mejorado - no han subido con el ascensor social.
Dicha sencilla categorización puede resultar simplista, pero nos
permitirá llevar a cabo una regresión logística binaria, la más sencilla
y, por ello, la más fácil de entender y realizar.

Esta variable binaria será la variable dependiente de nuestra regresión
logística.

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
datos_Fichero_P <- read.csv("/Users/davpero/Desktop/Trial/esudb19p.csv", sep = ",")
# Fichero Hogares
datos_Fichero_H <- read.csv("/Users/davpero/Desktop/Trial/esudb19h.csv", sep = ",")
```

Vamos a crear una variable en el conjunto de datos datos_Fichero_P que
sea igual a la variable identificadora del hogar HB030. Para ello,
debemos quitar los dos últimos dígitos a la variable PB030. De esta
forma, tendremos dos variables identificadoras idénticas que nos
permitirán realizar la unión entre tablas.

``` r
# Para quitar los dos últimos dígitos dividimos entre 100, quedándonos solo con la parte entera:
datos_Fichero_P$HB030 <- datos_Fichero_P$PB030 %/% 100

# unión de dataframes usando las variables HB030
datos_conjuntos <- merge(x = datos_Fichero_P, y = datos_Fichero_H, by = "HB030")

cat("El número de filas del dataframe datos_conjuntos es", nrow(datos_conjuntos), "y el de columnas es", ncol(datos_conjuntos))
```

    ## El número de filas del dataframe datos_conjuntos es 33376 y el de columnas es 405

Como vemos, tenemos un nuevo dataframe cuyo número de filas es igual al
número de filas del fichero P (el más grande, ya que hay más personas
que hogares), y cuyo número de columnas es igual a la suma de las
variables de los ficheros P y H.

A continuación, creamos la variable MSE (Mejora_Situación_Económica).
Dicha variable tendrá valor 1 si la situación económica de cada persona
ha mejorado con los años y valor 0, si no lo ha hecho:

``` r
datos_conjuntos$MSE <- ifelse(datos_conjuntos$PT190 < datos_conjuntos$HS120, 1, 0)

cat("El número de personas que han mejorado su situación económica es de", nrow(filter(datos_conjuntos, MSE == 1)), ". Mientras que el resto,", nrow(filter(datos_conjuntos, MSE == 0)), ", no han mejorado su situación económica. \n")
```

    ## El número de personas que han mejorado su situación económica es de 3168 . Mientras que el resto, 14299 , no han mejorado su situación económica.

``` r
cat("El número de valores missing de la variables MSE es", sum(is.na(datos_conjuntos$MSE)), "La grandísima mayoría -", sum(is.na(datos_conjuntos$PT190)), "- se deben a los valores missing que presentaba la variable PT190.")
```

    ## El número de valores missing de la variables MSE es 15909 La grandísima mayoría - 15900 - se deben a los valores missing que presentaba la variable PT190.

``` r
#Creamos un pequeño dataset con las variables que vamos a utilizar:
datos1 = datos_conjuntos[,c("MSE","PB040","PB190","PE040","PB140")]
```

Ahora convertimos el **Nivel de estudios** (variable PE040) a 3 clases:
0,1,2. Estas corresponde con nivel bajo, medio, alto. Haremos lo mismo
con **Estado civil** (PB190) que le daremos un valor 1 si el individuo
está casado y 0 en caso contrario. Por último también vamos a incluir la
variable **Año de Nacimiento** (PB140) en el dataset.

``` r
datos1 <- datos1 %>%
  mutate(PE040 = case_when(
    PE040 == 100 ~ 0, # Educacion primaria nivel bajo
    PE040 == 200 ~ 0, # Educacion media hasta abajo
    PE040 == 300 ~ 1,
    PE040 == 353 ~ 1,
    PE040 == 344 ~ 1,
    PE040 == 354 ~ 1,
    PE040 == 400 ~ 1,
    PE040 == 450 ~ 1,
    PE040 == 500 ~ 2, # Educación superior
    TRUE ~ PE040
  ))


# Cambiamos el nombre de las variables para que quede más claro lo que vamos a hacer:
datos1 <- rename(datos1, Factor_de_elevacion = PB040, Estado_civil = PB190, Ano_Nacimiento = PB140, Nivel_Estudios = PE040)

# Recodificamos la variable Estado_civil: todas las categorias diferentes a 2, recibiran el valor 0; y la categoria 2 (Casado), el valor 1
datos1$Estado_civil <- ifelse(datos1$Estado_civil != 2, 0, 1)



# Omitimos las observaciones con NA
datos1 <- na.omit(datos1)


# Convertir a factor
datos1$Estado_civil <- factor(datos1$Estado_civil)
datos1$MSE <- factor(datos1$MSE)
datos1$Nivel_Estudios <- factor(datos1$Nivel_Estudios)


# Creamos excel con datos
library("writexl")
write_xlsx(datos1, "/Users/davpero/Library/CloudStorage/GoogleDrive-davidperez202223@gmail.com/Mi unidad/4th Quarter/INE/2Datasets/Datos/Nuevos/ECV.xlsx\\jaja.xlsx")
```

Este dataset será el que se proporcione para el estudiante para hacer
sus análisis.

``` r
lmod2 <- glm(formula = MSE ~ Estado_civil+Ano_Nacimiento+Nivel_Estudios,family = binomial(link = logit),data=datos1)
summary(lmod2)

library(Epi)
#The ROC function
ROC(form=MSE ~ Estado_civil+Ano_Nacimiento+Nivel_Estudios, data=datos1,plot="ROC",lwd=3,cex=1.5)


table(datos1$Ano_Nacimiento)
```
