# Cómo completar los resultados del LLM en las slides

Las slides (`informe_presentacion.tex`) tienen 3 huecos marcados con `[COMPLETAR...]` en rojo, porque los resultados del recomendador LLM hay que generarlos en tu máquina (con Ollama corriendo). Seguí estos pasos.

## 1. Tener el modelo

```bash
ollama pull qwen2.5:3b
```

## 2. Correr el notebook del LLM

Abrí `Clase_Taller2.ipynb` y corré **todas las celdas de arriba a abajo** (Run All). Esperá a que termine el loop de los 14 usuarios (la variable `resultados` queda en memoria).

## 3. Guardar los resultados

Agregá una celda al final con esto y ejecutala:

```python
import json
with open("data/resultados_llm.json", "w", encoding="utf-8") as f:
    json.dump(resultados, f, ensure_ascii=False, indent=2)
print("guardado en data/resultados_llm.json")
```

## 4. Sacar los 2 casos que van a las slides

En otra celda:

```python
for n in ["Rodrigo", "Paula"]:
    print(f"\n== {n} ==")
    for rec in resultados[n]["recomendaciones"]:
        print(f'- {rec["pelicula"]} — {rec["motivo"]}')
```

## 5. Pegar en las slides

Abrí `informe_presentacion.tex`, buscá `COMPLETAR` (aparece 3 veces) y reemplazá:

1. **Rodrigo** → las 5 películas (alcanza con 1 motivo, el más claro).
2. **Paula** → las 5 películas (con 1 motivo).
3. **La frase de contraste** → una oración tuya comparando: ¿el LLM le pegó mejor a la *intención* de la query que LDA, sobre todo en Paula (perfil ambiguo)?

Formato sugerido para cada caso (reemplazando la línea `\item \textcolor{red}{...}`):

```latex
\item \textit{Película 1}, \textit{Película 2}, \textit{Película 3}, \textit{Película 4}, \textit{Película 5}.
\item \footnotesize Motivo (ej.\ de \textit{Película 1}): ``...''
```

## 6. Recompilar el PDF

```bash
pdflatex informe_presentacion.tex
pdflatex informe_presentacion.tex
```

(Dos veces, para que el tema Madrid arme bien la barra inferior.)

---

**Tip:** si querés comparar las dos estrategias sobre el mismo usuario, abrí `recomendador_lda.ipynb` y mirá las recomendaciones de Rodrigo y Paula ahí; en las slides la diapositiva "LDA frente a LLM" ya resume las diferencias conceptuales.
