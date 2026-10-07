================================================================================
INDICE DE RURALIDAD DE ESCUELAS (CENSO 2024)
Scripts de limpieza de datos, descriptivos y calculo del indice
================================================================================
 
Autores: Valentina Giaconi, Marco Pérez, Ngaire Honey, Manuela Mendoza
Ultima actualizacion: 2026-10-07
  
--------------------------------------------------------------------------------
1. QUE HACE ESTE PROYECTO
--------------------------------------------------------------------------------
 
Construye un indice de ruralidad para cada escuela de Chile. El indice es el
puntaje de un modelo factorial confirmatorio de un factor (CFA) estimado con
variables del Censo 2024 del sector donde esta la escuela, la densidad
poblacional de ese sector y la distancia de la escuela al limite urbano.
 
Un "sector" es una zona censal (area urbana) o una localidad censal (area
rural). Cada escuela se ubica en un sector con sus coordenadas, usando la
cartografia del Censo 2024. 

Resultado final: un puntaje por escuela, de mayor a menor ruralidad
(ind_rural, ind_rural_100 e ind_rural_q5).
  
--------------------------------------------------------------------------------
2. SCRIPTS Y ORDEN DE EJECUCION
--------------------------------------------------------------------------------
 
Los scripts se ejecutan en este orden. Cada uno usa la salida del anterior.
 
  1. B0_Limpieza_datos_geograficos.Rmd
     Ubica cada escuela en su sector censal y calcula area, densidad y
     distancia al limite urbano.
     Entradas : Directorio Oficial de Escuela 2024, tres capas parquet del Censo
                2024 (zonal, localidades y limite urbano) y
                correccion_coordenadas_manual.R.
     Salida   : escuelas_con_longitud_latitud_completadas_ID.csv
 
  2. B1_Limpieza_de_datos_general.Rmd
     Prepara las variables censales por sector (conteos y porcentajes) y las
     une a cada escuela.
     Entradas : Base_zona_localidad_CPV24.csv (censo) y la salida de B0.
     Salida   : escuelas_con_longitud_latitud_completadas_IDok.csv
 
  3. A0_Descriptivos.Rmd  (opcional, no genera archivos)
     Descriptivos univariados y bivariados de los indicadores y comparacion
     entre sectores urbanos y rurales (AREA_C).
     Entrada  : salida de B1.
     Salida   : solo resultados en el documento (HTML).
 
  4. A1_Calculo_indice.Rmd
     Ajusta el CFA y calcula el puntaje de ruralidad de cada escuela.
     Entrada  : salida de B1.
     Salidas  : indice_ruralidad_escuelas.csv e indice_ruralidad_escuelas.xlsx
 
--------------------------------------------------------------------------------
3. REQUISITOS
--------------------------------------------------------------------------------
 
Programas: R y RStudio. "R version 4.5.3 (2026-03-11 ucrt)"
 
Paquetes de R:
 
  install.packages(c("dplyr", "tidyr", "ggplot2", "knitr", "GGally",
                     "writexl", "scales", "lavaan", "sf", "arrow", "sfarrow"))
 
El CFA se corrio con lavaan 0.6-21. Los scripts B0 y A1 pueden tardar varios
minutos (cruces espaciales y ajuste del modelo).
 
 
--------------------------------------------------------------------------------
4. ESTRUCTURA DE CARPETAS
--------------------------------------------------------------------------------
 
Todos los archivos viven bajo una carpeta base (path_in). Los scripts estan en
la subcarpeta Scripts_indice_censo2024. De donde descargar los datos de
entrada: ver seccion 11.
 
  Proyecto Indice de Ruralidad de Escuelas/        <- path_in
  |-- Scripts_indice_censo2024/
  |     B0_Limpieza_datos_geograficos.Rmd
  |     B1_Limpieza_de_datos_general.Rmd
  |     A0_Descriptivos.Rmd
  |     A1_Calculo_indice.Rmd
  |     correccion_coordenadas_manual.R
  |     README.txt
  `-- Base de datos/
        |-- Directorio Oficial EE 2024/
        |-- Bases Censo Distrito 2024/
        |     Base_zona_localidad_CPV24.csv
        |-- Cartografia_censo2024_Pais/
        |     Cartografia_censo2024_Pais_Zonal.parquet
        |     Cartografia_censo2024_Pais_Localidades.parquet
        |     Cartografia_censo2024_Pais_Limite_Urbano.parquet
        |-- escuelas_con_longitud_latitud_completadas_ID.csv        (salida B0)
        |-- escuelas_con_longitud_latitud_completadas_IDok.csv      (salida B1)
        |-- indice_ruralidad_escuelas.csv                           (salida A1)
        `-- indice_ruralidad_escuelas.xlsx                          (salida A1)
 
 
--------------------------------------------------------------------------------
5. COMO EJECUTAR
--------------------------------------------------------------------------------
 
1. Descargar la carpeta completa de Dropbox y esperar a que los archivos
   esten descargados (no "solo en linea").
 
2. En cada script, ajustar la linea path_in al usuario y computador:
 
     path_in <- "TU/RUTA/Proyecto Indice de Ruralidad de Escuelas/"
 
   Los scripts traen dos lineas path_in, una por usuario; dejar activa solo la
   que corresponde y comentar la otra. Usar "/" (no "\") en las rutas.
 
3. Ejecutar en orden B0, B1, A0 (opcional) y A1. En RStudio: abrir el script y
   usar "Knit" para ejecutarlo completo, o correr los chunks uno a uno.
 
4. Revisar los chequeos de cada script antes de pasar al siguiente.
 
Si se cambia algun archivo de entrada, volver a correr desde el primer script
afectado: cada salida alimenta al script siguiente.
 
 
--------------------------------------------------------------------------------
6. QUE HACE CADA SCRIPT
--------------------------------------------------------------------------------
 
B0_Limpieza_datos_geograficos
  1. Lee el directorio de escuelas y se queda con los establecimientos en
     funcionamiento (ESTADO_ESTAB = 1).
  2. Reconstruye LATITUD y LONGITUD con arreglar_coord(). En el archivo
     completado las coordenadas traen comas como separadores de miles
     ("-18,487,200,..." equivale a -18.4872). La funcion se queda con los
     digitos y pone el decimal despues de los 2 primeros (3 en longitudes de
     Rapa Nui).
  3. Asigna las coordenadas manuales (correccion_coordenadas_manual.R).
  4. Descarta escuelas sin coordenadas, fuera de Chile o con latitud igual a
     longitud (error de digitacion).
  5. Cruza cada escuela con la capa de zonas y la de localidades. Si cae en una
     zona, queda con ID_ZONA; si cae en una localidad, con ID_LOCALIDAD. Nunca
     con ambos.
  6. Las escuelas que no caen en ningun poligono se asignan a la zona o
     localidad mas cercana si esta a menos de 1.000 m (asignada_cercana).
  7. Calcula el area del sector en m2 con una proyeccion equivalente de Albers
     para Chile, y la densidad en personas por km2. No se usa SHAPE_Area de las
     capas porque viene en grados cuadrados.
  8. Calcula la distancia geodesica (en metros) al limite urbano mas cercano; 0
     si la escuela esta dentro.
  9. Compara la comuna del directorio con la comuna de la cartografia como
     chequeo de ubicacion.
 10. Guarda la base sin geometria.
 
B1_Limpieza_de_datos_general
  1. Lee el censo por zona y localidad y crea el identificador de sector (ID):
     ID_ZONA si existe y, si no, ID_LOCALIDAD. Chequea que sea unico.
  2. Define var_mantener (identificadores) y var_sumar (conteos) y convierte
     los conteos a numero. Los valores "*" (reserva estadistica del INE)
     pasan a NA.
  3. Crea las variables combinadas _nuevavar (alcantarillado dentro y fuera de
     la vivienda; lena, pellet y carbon).
  4. Calcula los porcentajes pct_ (0 a 100): ocupacion y rama dividida por
     personas ocupadas; los demas, por hogares.
  5. Une el censo a cada escuela por ID y guarda la base.
 
A0_Descriptivos
  Tabla resumen, histogramas, violin y boxplot, correlaciones entre
  indicadores y comparacion urbano-rural con tamano de efecto.
 
A1_Calculo_indice
  Ajusta el CFA, calcula los puntajes, los reescala y valida el indice con los
  indicadores y con RURAL_RBD. Guarda el resultado en CSV y Excel.
 
 
--------------------------------------------------------------------------------
7. DECISIONES METODOLOGICAS PRINCIPALES
--------------------------------------------------------------------------------
 
- Unidad de analisis: la escuela. Los indicadores censales son del sector, asi
  que escuelas del mismo sector comparten valor; solo la distancia al limite
  urbano varia entre ellas. Por eso el modelo AFC usa errores estandar agrupados
  por sector (cluster = "ID").
 
- Modelo: un factor latente (rurality_com) con ocho indicadores:
    pct_n_caenes_A                          (agricultura y pesca)
    pct_n_fuente_agua_publica
    pct_n_basura_servicios
    pct_n_serv_internet_fija
    pct_n_serv_hig_alcantarillado_nuevavar
    pct_n_comb_calefaccion_lenaymas_nuevavar
    log_dens_sector_km2                     (log natural de densidad + 1)
    log_dist_limite_m                       (log natural de distancia + 1)
  y una covarianza residual entre agua publica y recoleccion de basura.
  Estimacion: MLR, varianza del factor fijada en 1 y datos perdidos por maxima
  verosimilitud (FIML). La electricidad publica se saco del modelo por baja
  variabilidad y por generar residuos altos.
 
- Orientacion: mayor puntaje = mas rural. Se verifica que las cargas de
  agricultura y lena sean positivas y las de servicios y densidad, negativas.

- Distancia al limite urbano: se calcula con geometria esferica (s2), no con la
  proyeccion de Albers, porque Albers conserva areas y no distancias.
 
- RURAL_RBD (escuela a mas de 5 km de un limite urbano) se usa para validar el
  indice. Como se define por distancia, no es independiente del indicador
  log_dist_limite_m. 
 
--------------------------------------------------------------------------------
8. CHEQUEOS PARA SABER QUE SALIO BIEN
--------------------------------------------------------------------------------
 
B0
  - nrow(escuelas_sf) y nrow(escuelas_ids) son iguales.
  - table(tipo_unidad) no deja escuelas sin sector (o muy pocas).
  - Ninguna escuela tiene ID_ZONA e ID_LOCALIDAD a la vez (resultado 0).
  - La coincidencia entre la comuna del directorio y la de la cartografia es
    alta. Revisar las que no coinciden.
  - Comparar dist_limite_m con RURAL_RBD: las escuelas rurales deberian estar,
    en su mayoria, a mas de 5.000 m.
 
B1
  - El chequeo de ID unico no devuelve filas.
  - sum(is.na(n_per)) en la base final es 0 o muy bajo.
  - Los porcentajes pct_ estan entre 0 y 100.
  - nrow(escuelas_ok) es igual al numero de escuelas de la salida de B0.
 
A1
  - El modelo converge ("ended normally").
  - No quedan residuos de correlacion de 0.10 o mas.
  - length(idx) coincide con nrow(dat_schools) (o se sabe cuantas quedaron
    fuera del modelo).
  - Los signos de las cargas son los esperados.

Valores de referencia (corrida del 07-10-2026) para comprobar que se reproduce:
  - Escuelas en el modelo: 12.026 (11.850 con coordenadas del directorio y 176
    con coordenadas manuales); sectores (clusters): 6.272.
  - CFI robusto 0.967; TLI robusto 0.952; RMSEA robusto 0.123; SRMR 0.024.
  - Cargas estandarizadas: agricultura 0.81, agua publica -0.79, basura -0.60,
    internet fija -0.90, alcantarillado -0.93, lena para calefaccion 0.68,
    densidad -0.97 y distancia al limite urbano 0.96.
  - ind_rural: media 0, minimo -1.07, maximo 2.51.
 
 
--------------------------------------------------------------------------------
9. PROBLEMAS FRECUENTES
--------------------------------------------------------------------------------
 
"No se puede abrir la conexion" / archivo no encontrado
  Revisar con file.exists(ruta) y list.files(carpeta). Causas comunes:
  usuario equivocado en path_in; extension doble (".csv.csv"); falta un
  prefijo en el nombre; archivo de Dropbox disponible solo en linea.
 
Error "Loop is not valid" al cruzar escuelas con las capas
  Geometrias invalidas para s2. Reparar con st_make_valid() antes del cruce.
  Si persiste, sf_use_s2(FALSE) lo evita, pero entonces las distancias dejan de
  ser geodesicas y hay que calcularlas en una proyeccion.
 
"NAs no son permitidos en asignaciones subscritas" en arreglar_coord()
  La columna de coordenadas trae valores perdidos. La condicion de ok debe
  excluir los NA: ok <- !is.na(d) & nchar(d) >= 3.
 
Los IDs aparecen como 1,12E+09 en Excel
  Convertir los IDs a texto antes de guardar (as.character()).
 
Acentos raros en los archivos CSV
  Leer con read.csv2(..., fileEncoding = "latin1") o "UTF-8".
 
 
--------------------------------------------------------------------------------
10. LIMITACIONES Y PENDIENTES
--------------------------------------------------------------------------------
 
- Las coordenadas manuales (176 escuelas) se buscaron una por una. La de la
  escuela RBD 42192 (comuna La Reina) cae en Rancagua y esta marcada para
  revisar en correccion_coordenadas_manual.R.
- Los parquet de cartografia no traen CRS legible; se asigna 4326 (WGS84),
  equivalente a SIRGAS 2000, que es el que declaran sus metadatos.
- Si cambia el numero de escuelas (por ejemplo al agregar coordenadas), hay que
  reajustar el CFA y actualizar los indices de ajuste y el N.
- Los resultados dependen de la version del directorio de escuelas y de la
  cartografia del Censo 2024 usadas.


--------------------------------------------------------------------------------
11. DATOS DE ENTRADA (NO INCLUIDOS EN ESTE REPOSITORIO)
--------------------------------------------------------------------------------

Las bases originales no se incluyen. Descargarlas desde la fuente oficial y
guardarlas en las carpetas indicadas en la seccion 4.
Fecha de descarga: 07-10-2026

1. Directorio Oficial de Establecimientos Educacionales 2024
   Institucion : Ministerio de Educacion, Centro de Estudios (Datos Abiertos)
   Enlace      : https://datosabiertos.mineduc.cl/directorio-de-establecimientos-educacionales/
   Que bajar   : "Directorio Oficial EE 2024"
   Formato     : archivo comprimido (.rar); hay que descomprimirlo. Incluye el
                 .csv del directorio, un PDF de documentacion, una planilla de
                 frecuencias y un README del Mineduc.
   Guardar en  : Base de datos/Directorio Oficial EE 2024/
   Usado por   : B0_Limpieza_datos_geograficos
   Terminos    : no se pudieron confirmar al 07-10-2026 (ver seccion 12).

2. Censo de Poblacion y Vivienda 2024: base a nivel de zona y localidad
   Institucion : Instituto Nacional de Estadisticas (INE)
   Enlace      : https://censo2024.ine.gob.cl/resultados/
   Ruta        : Bases de datos pais > "Base a nivel de zona - localidad Censo
                 2024 (csv)"
   Guardar en  : Base de datos/Bases Censo Distrito 2024/
                 (en este proyecto el archivo se usa como
                 Base_zona_localidad_CPV24.csv)
   Usado por   : B1_Limpieza_de_datos_general

3. Cartografia censal 2024
   Institucion : Instituto Nacional de Estadisticas (INE)
   Enlace      : https://censo2024.ine.gob.cl/resultados/
   Ruta        : Cartografia Censal > "Cartografia Pais Censo 2024 (geoparquet)"
   Archivos usados:
                 Cartografia_censo2024_Pais_Zonal.parquet
                 Cartografia_censo2024_Pais_Localidades.parquet
                 Cartografia_censo2024_Pais_Limite_Urbano.parquet
   Guardar en  : Base de datos/Cartografia_censo2024_Pais/
   Usado por   : B0_Limpieza_datos_geograficos

Nota: el directorio y la base del censo se actualizan. Los resultados dependen
de la version descargada; anotar la fecha de descarga y, en el directorio, las
fechas que trae el nombre del archivo.


--------------------------------------------------------------------------------
12. DATOS PUBLICADOS Y LICENCIAS
--------------------------------------------------------------------------------

Datos publicados en este repositorio:
  indice_ruralidad_escuelas.csv                      Indice por escuela (una fila por RBD).
  Libro de códigos indice_ruralidad_escuelas.pdf    Descripcion de cada variable.
Son un producto derivado y no reemplazan a las fuentes originales.

Licencia de los datos publicados: CC BY-SA 4.0
  https://creativecommons.org/licenses/by-sa/4.0/deed.es_ES
  Los terminos de uso del INE
  (https://www.ine.gob.cl/terminos-de-uso-y-licencia-de-datos-abiertos)
  publican su informacion bajo esa licencia. Exige reconocer la fuente,
  indicar que se hicieron cambios, no sugerir que el INE respalda el
  trabajo y difundir los productos derivados con la misma licencia
  (CompartirIgual).

Como citar:
  Giaconi, V., Pérez, M., Honey, N. y Mendoza, M. (2026). Indice de ruralidad de
  escuelas (Censo 2024) [datos y codigo]. Elaborado con datos del INE (Censo de
  Poblacion y Vivienda 2024 y cartografia censal 2024) y del Ministerio de
  Educacion (Directorio Oficial de Establecimientos Educacionales 2024). Datos
  transformados: porcentajes por sector, densidad poblacional, distancia al
  limite urbano e indice de ruralidad. Este indice no es un producto oficial
  del INE ni del Mineduc.

Licencia del codigo (scripts): MIT
Licencia de la documentacion (README y libro de codigos): CC BY-SA 4.0, igual
  que los datos

Terminos del directorio del Mineduc:
  No se pudieron confirmar al 07-10-2026. El archivo publicado no reproduce el
  directorio: solo incluye el RBD, el identificador del sector y las variables
  calculadas.
