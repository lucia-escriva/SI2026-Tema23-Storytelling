# Lab 04 — Data Storytelling con Spotify España
### Máster en Data Science e Inteligencia Artificial · EBIS Business School

---

## Punto de partida

Eres analista de datos en **SurLabel**, una discográfica independiente con sede en Madrid. Tu directora artística acaba de entrar en tu despacho con una pregunta directa:

> *"Necesito entender cómo funciona el éxito en Spotify España. Tenemos tres lanzamientos previstos para el próximo trimestre y no sé si estamos planificando bien las campañas. Dame algo que pueda enseñarle al equipo el lunes."*

Tienes a tu disposición los rankings diarios de Spotify España de 2025: **más de 200 días de datos**, con las posiciones del top 50 y los streams reales de cada canción.

Tu trabajo no es hacer un análisis exploratorio completo. Es construir una historia que ayude a tomar una decisión concreta.

---

## Los datos

**Fuente:** Spotify Charts – España (archivos `regional-es-daily-YYYY-MM-DD.csv`)

| Columna | Tipo | Descripción |
|---|---|---|
| `rank` | int | Posición en el ranking ese día (1–50) |
| `artist_names` | string | Artista/s |
| `track_name` | string | Título de la canción |
| `streams` | int | Reproducciones ese día en España |
| `days_on_chart` | int | Días acumulados en el top 50 hasta esa fecha |
| `peak_rank` | int | Mejor posición histórica alcanzada |
| `previous_rank` | int | Posición el día anterior (−1 si es nueva entrada) |

**Preprocesamiento mínimo necesario (R o Python):**
```r
# R
files <- list.files("Data/", pattern = "*.csv", full.names = TRUE)
df <- map_dfr(files, ~read_csv(.x) |> mutate(fecha = as.Date(str_extract(.x, "\\d{4}-\\d{2}-\\d{2}"))))
```
```python
# Python
import pandas as pd, glob, re
files = glob.glob("Data/*.csv")
df = pd.concat([pd.read_csv(f).assign(fecha=re.search(r'\d{4}-\d{2}-\d{2}', f).group()) for f in files])
df['fecha'] = pd.to_datetime(df['fecha'])
```

---

## Marco teórico

Antes de empezar con los datos, interioriza los tres ejes del análisis **explicativo** según *Storytelling with Data* (Knaflic, 2015):

1. **Contexto primero** — El análisis empieza por la audiencia, no por los datos. Pregunta: *¿quién escucha esto, qué decisión tiene que tomar, qué tono corresponde?*
2. **El gráfico es el argumento** — No ilustras una conclusión; el visual *es* la conclusión. El título debe decir qué concluir, no qué ver.
3. **La atención es escasa, el ruido es gratis** — Todo elemento visual que no contribuye a la historia resta. Eliminar es diseñar.

Cada uno de los tres casos que siguen activa un conjunto específico de estas herramientas. Léelos todos antes de elegir el tuyo.

---

---

# CASO A — "El Reloj de Arena"
## *¿Cuánto tiempo tienes para aprovechar un hit?*

---

### La pregunta de negocio

La directora quiere saber cuánto dura el momento de máxima atención sobre una canción. Tiene dos lanzamientos simultáneos previstos y no sabe si tiene recursos para dar soporte a los dos al mismo tiempo o si debería escalonarlos.

**Big Idea** *(Knaflic, cap. 2)*: Una sola frase que resume la historia antes de abrir ningún gráfico. Escríbela antes de hacer el análisis:

> *"Un hit en Spotify España pierde la mitad de sus streams en menos de [X] días: hay una ventana corta y predecible para capitalizar el lanzamiento."*

El análisis debe confirmar o refutar esa hipótesis, y concretar el valor de X.

---

### Teoría SWD aplicada

| Principio | Cómo se activa en este caso |
|---|---|
| **Elige el visual adecuado** | Gráfico de líneas: ideal para mostrar evolución temporal y comparar curvas |
| **Elimina el clutter** | Sin leyenda si las líneas están etiquetadas; sin grid horizontal; ejes mínimos |
| **Atributos preatentivos** | Color: canciones individuales en gris claro, curva media en verde Spotify (#1DB954) y negrita |
| **El título cuenta la historia** | No "Evolución de streams"; sí "El 70% de los streams de un hit ocurren en sus primeros 21 días" |
| **Llamada a la acción** | La última diapositiva recomienda el calendario de lanzamientos |

---

### Guía de análisis

#### Paso 1 — Construye la curva de vida de un hit

Filtra canciones que hayan llegado al **top 10** en algún momento. Para cada una, normaliza los streams respecto a su pico máximo:

```r
# R
top10_songs <- df |>
  filter(rank <= 10) |>
  distinct(track_name, artist_names)

curve_data <- df |>
  semi_join(top10_songs, by = c("track_name", "artist_names")) |>
  group_by(track_name, artist_names) |>
  mutate(streams_norm = streams / max(streams),
         days_since_debut = as.numeric(fecha - min(fecha))) |>
  ungroup()
```

```python
# Python
top10_songs = df[df['rank'] <= 10][['track_name','artist_names']].drop_duplicates()
merged = df.merge(top10_songs, on=['track_name','artist_names'])
merged['streams_norm'] = merged.groupby(['track_name','artist_names'])['streams'].transform(lambda x: x / x.max())
merged['days_since_debut'] = merged.groupby(['track_name','artist_names'])['fecha'].transform(lambda x: (x - x.min()).dt.days)
```

#### Paso 2 — Calcula la curva media y el percentil 80

La curva media (línea negra o verde) es tu argumento visual principal. Añade los percentiles para mostrar variabilidad sin abrumar:

```r
summary_curve <- curve_data |>
  filter(days_since_debut <= 60) |>
  group_by(days_since_debut) |>
  summarise(
    media = mean(streams_norm, na.rm = TRUE),
    p20   = quantile(streams_norm, 0.20, na.rm = TRUE),
    p80   = quantile(streams_norm, 0.80, na.rm = TRUE)
  )
```

#### Paso 3 — Calcula la "vida media" del hit

¿En cuántos días cae por primera vez por debajo del 50% del pico?

```r
half_life <- curve_data |>
  arrange(track_name, days_since_debut) |>
  group_by(track_name, artist_names) |>
  filter(streams_norm <= 0.5) |>
  summarise(half_life_day = first(days_since_debut)) |>
  ungroup()

median(half_life$half_life_day, na.rm = TRUE)  # Este número va al título
```

---

### La visualización

**Gráfico principal:** líneas de streams normalizados por días desde el debut.

```
Streams (% del pico)
100% ─┐
      │ ╲  (canciones individuales: gris claro, alpha 0.2)
 75%  │  ╲
      │   ─── (banda p20–p80: verde muy tenue)
 50%  │    ╲──────── ← MARCA AQUÍ LA MEDIANA DE HALF-LIFE
      │         ╲───────────────
 25%  │               ╲─────────────────────────────────
  0%  └──────────────────────────────────────────────────
      0    7   14   21   30          60
                              Días desde el debut
```

**Elementos a incluir:**
- Líneas grises (alpha = 0.15–0.25) para cada canción: representan el ruido, no el argumento
- Banda sombreada entre percentil 20 y 80
- Línea media en verde (#1DB954), grosor 2–2.5
- Línea vertical punteada en el día que la media cruza el 50%: *ese* es el dato del título
- Anotación directa sobre la línea media (sin leyenda separada)

**Lo que NO debe aparecer:**
- Leyenda con 50 nombres de canciones
- Grid vertical
- Decimales en el eje Y
- Título genérico

---

### La narrativa (estructura de 3 diapositivas)

**Diapositiva 1 — Setup: el contexto que crea tensión**
> Título: *"Cada semana se lanzan decenas de canciones en España. La mayoría desaparece antes de que te des cuenta."*
> Contenido: dato de contexto (número de canciones que entran y salen del top 50 por semana en el dataset)

**Diapositiva 2 — El gráfico: el argumento**
> Título: *"Un hit pierde el 50% de sus streams en [X] días. La ventana es corta."*
> Contenido: la visualización descrita arriba

**Diapositiva 3 — La recomendación: la resolución**
> Título: *"Lanza los dos singles con [Y] semanas de diferencia, no simultáneamente."*
> Contenido: 2–3 bullets concretos derivados del análisis. La directora debe poder leer esta slide sin haber visto las anteriores.

---

### Entregable

Un documento de **3 slides** (PowerPoint, Canva o equivalente) + un párrafo de 5 líneas explicando la elección del visual y qué elementos de clutter eliminaste y por qué.

---

---

# CASO B — "Boom o Slow Burner"
## *Naces hit o te conviertes en uno: dos estrategias de lanzamiento*

---

### La pregunta de negocio

La directora tiene un artista nuevo sin base de fans consolidada y uno ya establecido con leal comunidad. ¿Deberían lanzar sus singles de la misma manera? Los datos sugieren que no: existen dos patrones de éxito radicalmente distintos, y cada uno requiere una estrategia diferente.

**Big Idea:**

> *"En Spotify España conviven dos tipos de hit: los que arrasan en el debut y se desinflan rápido (Boom), y los que crecen en silencio hasta convertirse en referentes (Slow Burner). Confundir los dos es tirar el presupuesto."*

---

### Teoría SWD aplicada

| Principio | Cómo se activa en este caso |
|---|---|
| **Contexto y audiencia** | La directora no sabe qué es un "percentil de streams"; necesita una metáfora, no una estadística |
| **Visual adecuado** | Dos curvas medias superpuestas sobre fondo de ruido: comparación directa entre patrones |
| **Atributos preatentivos: color** | Rojo para Boom (urgencia, explosión), azul para Slow Burner (calma, constancia); gris para el resto |
| **Atributos preatentivos: posición** | Separar los dos tipos en pequeños múltiplos si la superposición genera confusión |
| **Elimina el clutter** | Sin ejes de título, sin leyenda lateral; etiqueta directamente las dos curvas en el gráfico |

---

### Guía de análisis

#### Paso 1 — Clasifica cada canción

Una canción es **Boom** si más del 50% de sus streams totales ocurren en los primeros 7 días en el chart. Es **Slow Burner** si tarda más de 7 días en alcanzar el top 10.

```r
# R
debut_info <- df |>
  group_by(track_name, artist_names) |>
  arrange(fecha) |>
  summarise(
    debut_rank    = first(rank),
    debut_date    = min(fecha),
    total_streams = sum(streams),
    streams_7d    = sum(streams[days_on_chart <= 7]),
    first_top10   = min(fecha[rank <= 10], na.rm = TRUE),
    .groups = "drop"
  ) |>
  mutate(
    prop_early = streams_7d / total_streams,
    days_to_top10 = as.numeric(first_top10 - debut_date),
    hit_type = case_when(
      prop_early >= 0.5              ~ "Boom",
      days_to_top10 > 7              ~ "Slow Burner",
      TRUE                           ~ "Otros"
    )
  )
```

```python
# Python
def classify(g):
    total = g['streams'].sum()
    early = g[g['days_on_chart'] <= 7]['streams'].sum()
    debut = g['rank'].iloc[0]
    top10 = g[g['rank'] <= 10]['fecha'].min()
    debut_date = g['fecha'].min()
    days_to_top10 = (top10 - debut_date).days if pd.notna(top10) else 999
    prop_early = early / total if total > 0 else 0
    if prop_early >= 0.5:
        return 'Boom'
    elif days_to_top10 > 7:
        return 'Slow Burner'
    return 'Otros'

debut_info = df.sort_values('fecha').groupby(['track_name','artist_names']).apply(classify).reset_index()
debut_info.columns = ['track_name','artist_names','hit_type']
df = df.merge(debut_info, on=['track_name','artist_names'])
```

#### Paso 2 — Construye las curvas medias por tipo

```r
curve_by_type <- df |>
  inner_join(debut_info |> select(track_name, artist_names, hit_type, debut_date),
             by = c("track_name","artist_names")) |>
  filter(hit_type %in% c("Boom","Slow Burner")) |>
  group_by(track_name, artist_names, hit_type) |>
  mutate(streams_norm      = streams / max(streams),
         days_since_debut  = as.numeric(fecha - debut_date)) |>
  ungroup() |>
  filter(days_since_debut <= 60) |>
  group_by(hit_type, days_since_debut) |>
  summarise(media = mean(streams_norm, na.rm = TRUE), .groups = "drop")
```

#### Paso 3 — Calcula los KPIs comparativos para la narrativa

Estos números van en las anotaciones del gráfico y en la slide de recomendación:

```r
debut_info |>
  filter(hit_type %in% c("Boom","Slow Burner")) |>
  group_by(hit_type) |>
  summarise(
    n_canciones       = n(),
    streams_medios    = median(total_streams),
    duracion_mediana  = median(days_to_top10, na.rm = TRUE)
  )
```

---

### La visualización

**Opción 1 — Curvas superpuestas** *(recomendada si la separación es clara)*

```
Streams (% del pico)
100% ─┐
      │╲  BOOM ────────────── (rojo, etiqueta directa)
 75%  │ ╲╲
      │  ╲╲──
 50%  │    ──────╲
      │      ╲    ──────────────────────
      │       ╲  SLOW BURNER ─── (azul, etiqueta directa)
 25%  │        ╲────╱─────────────────
      │              ╲───────────────────────────────────
  0%  └──────────────────────────────────────────────────
      0    7   14   21   30          60  días desde debut
```

**Opción 2 — Pequeños múltiplos** *(recomendada si se superponen demasiado)*

Dos paneles idénticos en eje, uno por tipo, con el fondo gris de canciones individuales visible en ambos para que la comparación sea honesta.

**Elementos críticos:**
- El fondo de líneas grises (canciones individuales) es *obligatorio*: da credibilidad estadística al argumento
- Las curvas medias son el argumento, el fondo es el contexto
- Anota con flechas o cajas de texto: *"Pierde el 60% del pico en 10 días"* para Boom y *"Tarda 3 semanas en alcanzar su pico"* para Slow Burner

---

### La narrativa (estructura de 4 diapositivas)

**Diapositiva 1 — El problema**
> Título: *"¿Boom o largo recorrido? Cada artista necesita una estrategia diferente."*
> Gancho: dos portadas de canciones opuestas del dataset con sus curvas de evolución

**Diapositiva 2 — El patrón Boom**
> Título: *"Los Boom concentran el [X]% de sus streams en la primera semana."*
> Gráfico: sólo la curva Boom + fondo gris + anotaciones clave

**Diapositiva 3 — El patrón Slow Burner**
> Título: *"Los Slow Burners tardan más, pero su audiencia es más fiel."*
> Gráfico: sólo la curva Slow Burner + anotaciones

**Diapositiva 4 — La recomendación**
> Título: *"Artista nuevo = estrategia Boom. Artista consolidado = apostar por el largo recorrido."*
> Tabla de 2 columnas: qué hacer en cada caso (budget de promoción, ventana de campaña, KPIs a medir)

---

### Reflexión obligatoria

Antes de entregar, responde por escrito (5–8 líneas):

1. ¿Por qué has elegido normalizar los streams en lugar de usar valores absolutos?
2. ¿Qué sesgo puede introducir la clasificación binaria Boom/Slow Burner?
3. ¿Cambiaría tu recomendación si la directora tuviera presupuesto limitado?

Estas preguntas no tienen una respuesta única. Se evalúa la capacidad de identificar limitaciones del análisis propio.

---

### Entregable

**4 slides** + respuesta a las 3 preguntas de reflexión.

---

---

# CASO C — "Los Inmortales"
## *Cuando una canción deja de ser un hit y se convierte en un himno*

---

### La pregunta de negocio

La directora está pensando en la estrategia de catálogo a largo plazo. Le han dicho que "Soldadito Marinero" de Fito y los Fitipaldis lleva más de 6 años entre las canciones más escuchadas de España sin haber sido un lanzamiento reciente. No lo termina de creer. ¿Es un caso aislado o hay un patrón?

**Big Idea:**

> *"Un 3% de las canciones del top 50 español llevan más de 4 años en el ranking. No son hits: son activos permanentes. Y el catálogo clásico puede valer más que el próximo lanzamiento viral."*

---

### Teoría SWD aplicada

| Principio | Cómo se activa en este caso |
|---|---|
| **Comprende el contexto** | Audiencia no técnica que necesita sentir la magnitud del outlier, no verlo en una tabla |
| **Visual adecuado** | Distribución (histograma o dot plot) para mostrar que el outlier *es* el argumento |
| **Atributos preatentivos: color e intensidad** | La masa gris hace que los outliers coloreados "salten" sin necesidad de señalización extra |
| **Anotación directa** | Etiqueta los outliers con nombre de canción + años en el chart: ese dato es la historia |
| **El título es la conclusión** | El título no describe el gráfico; resume el insight que la directora debe llevarse |

---

### Guía de análisis

#### Paso 1 — Calcula la longevidad máxima de cada canción

```r
# R
longevity <- df |>
  group_by(track_name, artist_names) |>
  summarise(
    max_days      = max(days_on_chart, na.rm = TRUE),
    mean_streams  = mean(streams, na.rm = TRUE),
    avg_rank      = mean(rank, na.rm = TRUE),
    .groups = "drop"
  ) |>
  mutate(
    years_on_chart = max_days / 365,
    categoria = case_when(
      max_days >= 1825 ~ "Eterno (5+ años)",     # 5 * 365
      max_days >= 1095 ~ "Resistente (3–5 años)",
      max_days >= 365  ~ "Longevo (1–3 años)",
      TRUE             ~ "Efímero (< 1 año)"
    )
  )
```

```python
# Python
longevity = df.groupby(['track_name','artist_names']).agg(
    max_days=('days_on_chart','max'),
    mean_streams=('streams','mean'),
    avg_rank=('rank','mean')
).reset_index()

longevity['years_on_chart'] = longevity['max_days'] / 365

def categorize(d):
    if d >= 1825: return 'Eterno (5+ años)'
    if d >= 1095: return 'Resistente (3–5 años)'
    if d >= 365:  return 'Longevo (1–3 años)'
    return 'Efímero (< 1 año)'

longevity['categoria'] = longevity['max_days'].apply(categorize)
```

#### Paso 2 — Construye la distribución

El histograma o dot plot de `max_days` para todas las canciones del dataset. La forma de la distribución (muy sesgada a la derecha, con cola larga) *es* el mensaje visual.

```r
# Identifica los outliers para anotar
immortals <- longevity |>
  filter(max_days >= 1825) |>
  arrange(desc(max_days))

# Calcula el percentil 95 para contextualizar
quantile(longevity$max_days, 0.95)
```

#### Paso 3 — Conecta longevidad con valor económico

Este paso transforma el análisis técnico en argumento de negocio. Un activo de catálogo genera streams constantes sin coste de promoción.

```r
# Streams acumulados estimados por categoría
longevity |>
  group_by(categoria) |>
  summarise(
    n = n(),
    streams_medios_diarios = median(mean_streams),
    # Streams totales estimados = streams diarios * días en chart
    valor_estimado = median(mean_streams * max_days)
  ) |>
  arrange(desc(valor_estimado))
```

> **Nota metodológica para incluir en el entregable:** los streams del dataset son sólo los días en que la canción estuvo en el top 50. El valor real del catálogo es mayor, porque una canción puede seguir generando streams fuera del top 50. Señala esta limitación explícitamente.

---

### La visualización

**Gráfico 1 — La distribución (el argumento principal)**

Dot plot o histograma horizontal de `max_days`:

```
Días en el top 50
 
 < 30  ████████████████████████████████████████  (la mayoría)
30–90  ██████████████████
90–180 ████████
180–1y ████
 1–3y  ██
 3–5y  █
 5y +  •  "Soldadito Marinero" (2.430 días · 6,7 años)
       •  "Me Rehúso" (2.285 días · 6,3 años)
              ↑
       Anotar directamente con nombre + años
```

Colores:
- Barra mayoritaria: gris claro
- Barra 1–3 años: verde tenue (#85D6A5)
- Barras 3y+: verde Spotify (#1DB954), negrita
- Outliers 5y+: punto destacado con etiqueta directa

**Gráfico 2 — El valor del catálogo (la recomendación)**

Gráfico de barras horizontal con los `n` artistas con más días acumulados y sus streams medios diarios. El mensaje: una canción inmortal genera más valor total que diez lanzamientos virales.

---

### La narrativa (estructura de 4 diapositivas)

**Diapositiva 1 — El gancho**
> Título: *"¿Sabías que España lleva escuchando 'Soldadito Marinero' más de 6 años seguidos?"*
> Visual: portada de la canción + número de días en grande, sin más contexto todavía

**Diapositiva 2 — La escala del fenómeno**
> Título: *"No es casualidad. Un puñado de canciones desafía por completo la lógica del streaming."*
> Visual: distribución completa de longevidad. El outlier ya no parece raro: es la cola de una distribución que tiene forma.

**Diapositiva 3 — El valor económico**
> Título: *"Una canción eterna genera más streams que [N] lanzamientos virales juntos."*
> Visual: comparación de streams acumulados estimados por categoría

**Diapositiva 4 — La recomendación**
> Título: *"Invertir en catálogo es invertir en activos, no en campañas."*
> Contenido: 3 acciones concretas que SurLabel puede tomar (p. ej.: reactivar catálogo propio, buscar sincronizaciones, campaña de nostalgia)

---

### Reflexión obligatoria

1. El dataset sólo cubre el top 50 diario. ¿Qué sesgos introduce esto en el análisis de longevidad?
2. ¿Por qué un histograma comunica mejor que una tabla ordenada en este caso concreto? Argumenta usando los principios de carga cognitiva de Knaflic.
3. Si tuvieras que reducir las 4 slides a 1, ¿cuál conservarías y por qué?

---

### Entregable

**4 slides** + respuesta a las 3 preguntas de reflexión + una frase que resuma el *Big Idea* en menos de 25 palabras.

---

---

## Rúbrica de evaluación (común a los tres casos)

| Criterio | Insuficiente (0–4) | Suficiente (5–7) | Excelente (8–10) |
|---|---|---|---|
| **Big Idea** | No existe o es vaga | Existe pero no es accionable | Una frase, una audiencia, una decisión concreta |
| **Elección del visual** | El tipo de gráfico no se justifica o es inadecuado | El gráfico es correcto pero genérico | El gráfico es el argumento; se justifica la elección |
| **Eliminación de clutter** | Leyendas, grids y decoración innecesaria | Reducción parcial del ruido visual | Sólo existe lo que añade información |
| **Atributos preatentivos** | Color/tamaño aleatorio o excesivo | Un atributo usado correctamente | Jerarquía visual clara: el ojo llega solo al insight |
| **Narrativa** | Las slides son independientes | Hay hilo conductor pero sin tensión | Setup → conflicto → resolución; cada slide depende de la anterior |
| **Reflexión crítica** | No identifica limitaciones | Identifica una limitación evidente | Cuestiona el propio análisis con argumentos metodológicos |

---

## Criterio de selección

No tienes que desarrollar los tres casos. Elige **uno**. La nota no depende del caso elegido, sino de la profundidad con la que lo trabajes.

Si terminas antes del tiempo estimado, el reto opcional es: *¿puedes combinar dos historias en una sola narrativa coherente para la directora?*

---

*Duración estimada: 50–60 minutos de análisis + 20–30 minutos de diseño de slides*
*Herramientas: R, Python, Tableau, Power BI, o cualquier combinación*
