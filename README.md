# Data Storytelling con Spotify España 2025

**Máster en Data Science e Inteligencia Artificial · EBIS Business School**  
Tema 23 — Data Storytelling

---

## Descripción

Ejercicio práctico de Data Storytelling aplicado a datos reales de **Spotify Charts España 2025**.  
Los alumnos asumen el rol de analistas de datos en una discográfica independiente y construyen una narrativa visual para apoyar decisiones estratégicas de lanzamiento.

El ejercicio integra los principios de *Storytelling with Data* (Cole Nussbaumer Knaflic, 2015):
- Análisis explicativo vs. exploratorio
- Elección del visual adecuado
- Eliminación de clutter
- Atributos preatentivos
- Estructura narrativa: setup → conflicto → resolución

---

## Estructura del repositorio

```
├── Lab04_Spotify/
│   ├── Lab04_Spotify_Storytelling.ipynb     # Notebook principal (Python)
│   ├── Lab04 - Data Storytelling con Spotify.html   # Enunciado del ejercicio
│   ├── Lab04 - Data Storytelling con Spotify.md     # Enunciado en Markdown
│   ├── Data/
│   │   └── regional-es-daily-YYYY-MM-DD.csv  # Rankings diarios Spotify España
│   └── Images/
│       └── *.jpg / *.webp / *.png            # Imágenes de referencia
└── README.md
```

---

## Los datos

| Campo | Descripción |
|---|---|
| `rank` | Posición en el top 50 ese día |
| `artist_names` | Artista/s |
| `track_name` | Título de la canción |
| `streams` | Reproducciones ese día en España |
| `days_on_chart` | Días acumulados en el ranking |
| `peak_rank` | Mejor posición histórica |
| `previous_rank` | Posición el día anterior |

**Cobertura:** Enero – Octubre 2025 · ~290 días · Top 50 diario de España

---

## Los tres casos del ejercicio

### Caso A — El Reloj de Arena
> *¿Cuánto tiempo tienes para capitalizar un lanzamiento?*

Analiza la curva de vida de un hit: cómo evolucionan los streams desde el debut hasta la caída, y en cuántos días se pierde el 50% del pico.

### Caso B — Boom o Slow Burner
> *¿Todos los hits siguen el mismo patrón?*

Clasifica las canciones en dos arquetipos según su patrón de crecimiento y compara sus curvas medias para derivar estrategias de lanzamiento diferenciadas.

### Caso C — Los Inmortales
> *¿Tiene sentido apostar por el catálogo clásico?*

Analiza la distribución de longevidad del chart y cuantifica el valor económico estimado del catálogo longevo frente a los lanzamientos virales.

---

## Cómo ejecutar el notebook

### Prerequisitos

```bash
pip install pandas numpy matplotlib
```

### Ejecución

```bash
# Desde la carpeta Lab04_Spotify/
jupyter notebook Lab04_Spotify_Storytelling.ipynb
# o con JupyterLab:
jupyter lab Lab04_Spotify_Storytelling.ipynb
# o directamente en VS Code
```

El notebook genera 4 visualizaciones exportadas como PNG:
- `viz_A_reloj_de_arena.png`
- `viz_B_boom_vs_slow.png`
- `viz_C_longevidad_distribucion.png`
- `viz_D_valor_catalogo.png`

---

## Marco teórico

Este ejercicio se basa en *Storytelling with Data* (Knaflic, 2015). El libro completo está disponible en la carpeta `Documentation/` del proyecto de clase.

Los principios aplicados en cada visualización están documentados en las celdas Markdown del notebook.

---

## Enunciado para estudiantes

Abrir directamente en el navegador:

```
Lab04_Spotify/Lab04 - Data Storytelling con Spotify.html
```

---

*Datos: Spotify Charts · Licencia: solo uso educativo*
