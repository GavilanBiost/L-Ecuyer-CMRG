# L'Ecuyer-CMRG

## Reproducibilidad en computación paralela en R

**Autor:** Jesús F García Gavilán

---

### Descripción

Protocolo para garantizar la reproducibilidad de análisis estadísticos, modelos de machine learning y simulaciones ejecutados en paralelo en R. Especialmente relevante en proyectos de salud pública y biomedicina, donde el rigor metodológico es crítico.

Se utiliza el algoritmo **L'Ecuyer-CMRG** (Combined Multiple-Recursive Generator), integrado en el paquete `parallel` de R base, sin necesidad de instalar paquetes externos.

---

### Requisitos previos

- R instalado (cualquier versión reciente)
- No se requieren paquetes externos
- Paquete `parallel` incluido en R base

---

### Protocolo de implementación

#### Paso 1: Configurar el motor de números aleatorios

1. Carga la librería `parallel`
2. Cambia el motor de generación al algoritmo L'Ecuyer-CMRG mediante `RNGkind`
3. Fija una semilla global para el entorno principal

#### Paso 2: Instanciar el clúster de procesamiento

4. Calcula el número de núcleos disponibles restando al menos uno (`detectCores() - 1`). Se recomienda liberar 2 o 3 si se quiere seguir trabajando en otras tareas simultáneamente
5. Crea el clúster con `makeCluster`

#### Paso 3: Sincronizar las semillas en los nodos *(paso clave)*

6. Usa `clusterSetRNGStream` para enviar una semilla específica (`iseed`) a cada nodo, garantizando secuencias independientes y deterministas en cada hilo

#### Paso 4: Ejecutar y liberar recursos

7. Lanza la función paralela con `parLapply` (o equivalentes de la familia `apply`)
8. Cierra siempre el clúster con `stopCluster` para liberar memoria

---

### Código de implementación

```r
# 1. Cargar el paquete para computación paralela
library(parallel)
library(ggplot2)

# 2. Configurar el algoritmo generador de números aleatorios
RNGkind("L'Ecuyer-CMRG")
set.seed(123)

# 3. Detectar núcleos y crear el clúster (dejando 1 núcleo libre para el SO)
num_cores <- detectCores() - 1
cl <- makeCluster(num_cores)

# 4. Distribuir secuencias de números independientes a cada núcleo
clusterSetRNGStream(cl, iseed = 123)

# 5. Ejecutar el análisis en paralelo
# (Ejemplo: media de 100 números aleatorios normales, iterado 1000 veces)
resultados <- parLapply(cl, 1:1000, function(x) mean(rnorm(100)))

# 6. Detener el clúster y liberar los recursos del procesador
stopCluster(cl)

# 7. Visualización de los resultados
datos <- data.frame(medias = unlist(resultados))

ggplot(datos, aes(x = medias)) +
  geom_histogram(aes(y = after_stat(density)), bins = 30,
                 fill = "steelblue", color = "white", alpha = 0.7) +
  geom_density(color = "darkred", linewidth = 1) +
  theme_minimal() +
  labs(title = "Distribución de Medias Muestrales",
       subtitle = "1000 iteraciones en paralelo",
       x = "Valor de la Media",
       y = "Densidad")
```

---

### Nota metodológica

> La reproducibilidad computacional lograda con este esquema no es opcional al escalar análisis: es la base sobre la que se construye cualquier proyecto de ciencia de datos fiable aplicado a la salud pública o la biomedicina.

---

### Licencia

Ver [LICENSE](LICENSE)
