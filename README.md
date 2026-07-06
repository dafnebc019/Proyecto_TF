# Proyecto_TF
# Analizador Genómico de *Nematostella vectensis* 🧬

Este repositorio contiene un robusto script de automatización en Bash diseñado para procesar, limpiar y analizar archivos de anotación genómica en formato **GFF (General Feature Format)** provenientes de la NCBI. El proyecto se enfoca en el estudio de la arquitectura génica y la complejidad estructural/regulatoria (splicing alternativo) de la anémona de mar *Nematostella vectensis*.

## 📌 Características del Proyecto
* **Modularidad:** Estructurado en opciones independientes para análisis global, arquitectura genómica y splicing.
* **Optimización en Memoria:** Desarrollado utilizando procesamiento directo con bloques avanzados de `awk`, evitando la generación excesiva de archivos temporales pesados.
* **Control de Duplicados Genómicos:** Algoritmo bioinformático calibrado para mapear exones basados en coordenadas físicas únicas, previniendo la inflación artificial de datos por isoformas compartidas.

---

## 📊 Módulos de Análisis

### Módulo 1: Control de Calidad y Sanity Check
* Realiza un escrutinio inicial del archivo GFF.
* Valida la integridad de las líneas de comentarios y la estructura de las columnas.

### Módulo 2: Arquitectura Génica Global
* **Métricas de Genes:** Calcula el número total de loci génicos, junto con las longitudes promedio, máximas y mínimas (en pares de bases).
* **Paisaje Exón/Intrón:** Cuantifica el total de exones anotados, estima el número de intrones y extrae el promedio real de exones por gen.
* **Proporciones del Paisaje Ómico:** Determina el porcentaje exacto de regiones codificantes (exones) frente a regiones no codificantes (intrones) dentro de los límites genómicos.
* **Ranking de Dimensiones:** Extrae automáticamente el TOP 10 de los genes físicamente más largos del genoma.

### Módulo 3: Análisis de Complejidad Estructural y Splicing Alternativo
Aplica un **Índice de Complejidad Genómica** basado en la arquitectura y capacidad de transcripción del locus:
$$\text{Puntuación} = \text{Exones Únicos} + (\text{Isoformas} \times 5)$$

* **Clasificación Automatizada:**
  * **BAJA Complejidad:** $\le 3$ Exones y 1 sola Isoforma (baja actividad de splicing).
  * **ALTA Complejidad:** $> 10$ Exones o $> 3$ Isoformas (alta tasa de regulación post-transcripcional).
  * **MEDIA Complejidad:** Rango intermedio de reordenamiento molecular.
* **Extremos Regulatorios:** Identifica el gen con mayor dinamismo de splicing alternativo (múltiples mRNAs) y el locus estructuralmente más simple.
* **Perfil de Regulación:** Genera un TOP 20 con los genes de mayor complejidad regulatoria del organismo.

---

## 🛠️ Requisitos e Instalación

El script se ejecuta directamente sobre cualquier entorno Unix/Linux (incluyendo Google Cloud Shell) y solo requiere utilidades nativas de la terminal:

```bash
# Asegurar permisos de ejecución en la terminal de Cloud Shell
chmod +x script_AnalisisGFF.sh
