En este cuaderno vamos a procesar un conjunto de datos para predecir el número de matrimonios en las ciudades española a partir del número de mujeres que las habitan y el número de nacimientos que ha habido ese año. Concretamente, se han tomado los datos relativos a **2022**. Este análisis parece razonable a simple vista ya que el matrimonio la mayor parte de las veces va a acompañado de un nacimiento en los meses anteriores o posteriores.

Los datos del año *2022* necesitan ser combinados para obtener unos datos consientes para usar. Concretamente tomaremos:

1.  Cifras oficiales del padrón por municipios en 2022.
2.  Nacimientos por municipios en el año 2022.
3.  Matrimonios por municipios en el año 2022.

En todo caso trataremos las poblaciones con un número de habitantes comprendido entre 50.000 y 300.000, habiendo aproximadamente 150 poblaciones en España de este tipo. Los datos pueden descargarse en .csv (Formato delimitado por comillas) en:

1.  [Pincha aquí. Población total](https://www.ine.es/jaxiT3/Datos.htm?t=29005)
2.  [Pincha aquí. Población femenina](https://www.ine.es/jaxiT3/Datos.htm?t=29005)
3.  [Pincha aquí. Nacimientos](https://www.ine.es/jaxiT3/Datos.htm?t=31934)
4.  [Pincha aquí. Matrimonios](https://www.ine.es/jaxiT3/Datos.htm?t=37645).

# Preproceso

Procedemos a leer los cuatro ficheros de datos ya que vienen de orígenes distintos (consultar en los links del apartado anterior). Todos ellos se leerán directamente como un csv desde su dirección web. Concretamente se busca extraer de cada uno:

1.  **Población total**: Se busca extraer los códigos de las ciudades y su población para posteriormente poder filtrar las ciudades por número de habitantes.
2.  **Población femenina**: Código de ciudades y número de habitantes femeninas.
3.  **Nacimientos**: Códigos de ciudades y número de nacimientos en dicho año en inscritos en la ciudad.
4.  **Matrimonios**: Códigos de ciudades y número de matrimonios en dicho año en inscritos en la ciudad.

Los datos de las diferentes fuentes se van a juntar mediante el uso de la variable código de ciudad (JOIN).

``` r
library(readr)
library(dplyr)
# Intuimos el encoding que siguen los ficheros
guess_encoding("https://www.ine.es/jaxiT3/files/t/es/csv_bdsc/29005.csv?nocab=1")


guess_encoding("https://www.ine.es/jaxiT3/files/t/es/csv_bdsc/31934.csv?nocab=1")

# Población mujeres
poblmuj <- read_delim("https://www.ine.es/jaxiT3/files/t/es/csv_bdsc/29005.csv?nocab=1",
  delim = ";", escape_double = FALSE, locale = locale(decimal_mark = ",", encoding = "ISO-8859-1", asciify = TRUE),
  trim_ws = TRUE
)
poblmuj <- filter(poblmuj, Periodo == "2022")

# Población total
pobltotal <- read_delim("https://www.ine.es/jaxiT3/files/t/es/csv_bdsc/29005.csv?nocab=1",
  delim = ";", escape_double = FALSE, locale = locale(decimal_mark = ",", encoding = "ISO-8859-1", asciify = TRUE),
  trim_ws = TRUE
)

pobltotal <- filter(pobltotal, Periodo == "2022")


# Nacimientos
nacim22 <- read_delim("https://www.ine.es/jaxiT3/files/t/es/csv_bdsc/31934.csv?nocab=1",
  delim = ";", escape_double = FALSE, locale = locale(decimal_mark = ",", encoding = "ISO-8859-1", asciify = TRUE),
  trim_ws = TRUE
)

nacim22 <- as.data.frame(nacim22)
colnames(nacim22)[3] <- "Edaddelamadre"
colnames(nacim22)[4] <- "Mesdelnacimiento"
nacim22 <- filter(nacim22, Periodo == "2022", Sexo == "Mujeres", Mesdelnacimiento == "Total", Edaddelamadre == "Todas las edades")


# Matrimonios
matrim22 <- read_delim("https://www.ine.es/jaxiT3/files/t/es/csv_bdsc/37645.csv?nocab=1",
  delim = ";", escape_double = FALSE, locale = locale(decimal_mark = ",", encoding = "ISO-8859-1", asciify = TRUE),
  trim_ws = TRUE
)

matrim22 <- as.data.frame(matrim22)
colnames(matrim22)[3] <- "Mesdecelebracion"
colnames(matrim22)[4] <- "Formadecelebracion"
matrim22 <- filter(matrim22, Periodo == "2022", Mesdecelebracion == "Total", Formadecelebracion == "Total")
```

En cada conjunto de datos nos quedamos únicamente con los nombres de los municipios y la variable “Total” que indica la magnitud de cada fenómeno.

``` r
poblmuj <- poblmuj[, c("Municipios", "Total")]
nacim22 <- nacim22[, c("Municipios", "Total")]
matrim22 <- matrim22[, c("Municipios", "Total")]
pobltotal <- pobltotal[, c("Municipios", "Total")]


# Convertimos a data frame
poblmuj <- as.data.frame(poblmuj)
nacim22 <- as.data.frame(nacim22)
matrim22 <- as.data.frame(matrim22)
pobltotal <- as.data.frame(pobltotal)




# Extraemos códigos de ciudades de la variable municipio
# Ej 20003 Albacete
poblmuj$Ciudad <- sub(".* ", "", poblmuj$Municipios)
poblmuj$Municipios <- sub(" .*", "", poblmuj$Municipios)
nacim22$Municipios <- sub(" .*", "", nacim22$Municipios)
matrim22$Municipios <- sub(" .*", "", matrim22$Municipios)
pobltotal$Municipios <- sub(" .*", "", pobltotal$Municipios)
```

Juntamos ahora todos datos en una tabla mediante el uso de la variable `Municipios`, que contiene los códigos de los municipios.

``` r
# Juntamos los matrimonios con la pobalción de mujeres de cada ciudad
inner1 <- inner_join(matrim22, poblmuj, by = "Municipios", suffix = c(".matr", ".muj"))
inner1 <- as.data.frame(inner1)

# Juntamos el anterior con los nacimientos
inner2 <- inner_join(inner1, nacim22, by = "Municipios")
inner2 <- as.data.frame(inner2)
```

Creamos un dataset con los nombres de las variables correctos y volvemos a hacer un join con la Población Total de cada ciudad.

``` r
data <- data.frame(Municipios = inner2$Municipios, Ciudad = inner2$Ciudad, Matrimonios = inner2$Total.matr, Mujeres = inner2$Total.muj, Nacimientos = inner2$Total)
data <- as.data.frame(data)
pobltotal <- as.data.frame(pobltotal)

data <- inner_join(data, pobltotal, by = "Municipios")


# Filtramos por ciudades entre 50.000 y 300.000 habitantes
data <- filter(data, Total < 300000)

data <- filter(data, Total > 50000)
```

# Variables finales

Es decir, finalmente tenemos un dataset conteniendo para las ciudades españolas de entre 50.000 y 300.000 habitantes la siguiente información:

-   **Municipios**: Códigos de los municipios.
-   **Ciudad**: Nombres de los municipios.
-   **Matrimonios**: Número de matrimonios en cada municipio.
-   **Mujeres**: Número total de habitantes mujeres en cada municipio.
-   **Nacimientos**:Número de nacimientos en cada municipio.
-   **Total**: Número total de habitantes en cada municipio.

# Exportamos datos

Teniendo en cuenta las variables anteriores, se procede a exportar el conjutno de datos.

``` r
# Creamos excel con datos
library("writexl")
write_xlsx(data, "../../../files/matrimonios_reg.xlsx")
```

Este dataset será el que se proporcione para el estudiante para hacer sus análisis. Se puede encontrar en [*matrimonios_reg.xlsx*](../../../files/matrimonios_reg.xlsx).
