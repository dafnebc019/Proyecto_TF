#  🧬 Proyecto_TF
## 🗃️ Análisis mediante formato .gff
## Curso: Principios de Programación - Bioinformatica

## AUTORES :
* Amelia Bolaños 
* Dayra Espinar Catro

## Profesores:
* Frank Lino Guzman Escudero
* Manuel Alain Ramirez Saenz
  
**Universidad:** UPC - Facultad de Ciencias de la Salud 

# Analizador Genómico de *Nematostella vectensis* 🧬

Este repositorio contiene un robusto script de automatización en Bash diseñado para procesar, limpiar y analizar archivos de anotación genómica en formato **GFF (General Feature Format)** provenientes de la NCBI. El proyecto se enfoca en el estudio de la arquitectura génica y la complejidad estructural/regulatoria (splicing alternativo) de la anémona de mar *Nematostella vectensis*.

## 📌 Características del Proyecto
* **Modularidad:** Estructurado en 3 opciones independientes (splicing, arquitectura génica, correlación estructura-complejidad), cada una accesible desde un menú interactivo.
* **Jerarquía biológica real:** los módulos 2 y 3 reconstruyen la relación `gen → transcrito → exón` a partir de los atributos `ID=` / `Parent=` del GFF, en vez de depender de atajos como el tag `gene=`, lo que evita perder exones de tipos de transcrito no-`mRNA` (`tRNA`, `rRNA`, `lnc_RNA`, `snRNA`, `snoRNA`, `ncRNA`, `transcript`).
* **Detección universal de transcritos:** una sola regla (`Parent=gene-...`) identifica cualquier tipo de transcrito sin necesidad de listarlos uno por uno — funciona igual para otros organismos con anotaciones distintas.
* **Optimización en Memoria:**  Procesamiento en una sola pasada por archivo con arrays asociativos de `awk`, sin generar archivos temporales pesados innecesarios.
* **Control de Duplicados Genómicos:** Algoritmo bioinformático calibrado para mapear exones basados en coordenadas físicas únicas, previniendo la inflación artificial de datos por isoformas compartidas.

---

## 📊 Módulos de Análisis

### Módulo 1: Splicing Alternativo
* **Identificación de Isoformas:** Extrae los transcritos (`mRNA`) y los asocia con su gen de origen mediante el atributo `Parent=gene-`.
* **Cuantificación de Isoformas:** Calcula el número total de genes que presentan isoformas anotadas en el archivo GFF3.
* **Ranking de Diversidad Transcriptómica:** Extrae los genes con mayor número de isoformas para identificar candidatos con mayor evidencia de splicing alternativo.
* **Reporte Automático:** Genera el archivo `reporte_splicing.txt` con el listado completo de genes e isoformas.
 > **Pregunta biológica:** ¿qué genes tienen mayor diversidad transcripcional?

### Módulo 2: Arquitectura Génica 
Reconstruye la jerarquía biológica real `gen → transcrito → exón` para describir cómo está construido el genoma a nivel estructural.
* **Métricas de Genes:** Calcula el número total de genes, así como sus longitudes promedio, máxima y mínima (pb).
* **Detección universal de transcritos:** en lugar de limitarse a `mRNA`, identifica cualquier tipo de transcrito hijo directo de un gen (`tRNA`, `rRNA`, `lnc_RNA`, `snRNA`, `snoRNA`, `ncRNA`, `transcript`) mediante la regla `Parent=gene-...`, sin necesidad de listar cada tipo manualmente.
* **Paisaje Exón/Intrón:** Cuantifica el número total de exones, estima los intrones y calcula el promedio de exones e intrones por gen.
* **Caracterización del Paisaje Génico:** Determina la proporción de regiones codificantes (exones) y no codificantes (intrones) dentro de la arquitectura génica.
* **Control de Calidad:** Verifica la correcta asociación entre transcritos y exones antes de realizar los cálculos estructurales.
* **Ranking de Dimensiones:** Extrae automáticamente el TOP 10 de los genes de mayor longitud presentes en el genoma.
* **Reporte Automático:** Genera el archivo `arquitectura_gff.txt.`
  > **Pregunta biológica:** ¿cómo está construido el genoma en términos de tamaño, exones e intrones?

### Módulo 3: Correlación entre Arquitectura Génica y Complejidad Transcriptómica
* **Integración Estructural:** Relaciona cada gen con sus transcritos, exones e isoformas mediante la jerarquía biológica del archivo GFF3.
* **Análisis Estadístico:** Calcula el coeficiente de correlación de Pearson entre la longitud génica y el número de isoformas.
* **Clasificación Transcriptómica:** Agrupa los genes en cuatro categorías según su tamaño y complejidad transcriptómica (compactos complejos, gigantes complejos, compactos simples y gigantes simples).
* **Identificación de Casos Biológicos Relevantes:** Detecta genes con alta densidad transcriptómica y genes extensos con baja diversidad de isoformas ("Gigantes Dormidos").
* **Reporte Automático:** Genera el archivo `reporte_correlacion.txt.`
  > **Pregunta biológica:** ¿el tamaño del gen predice cuánta complejidad regulatoria (splicing alternativo) tiene?

* El Módulo 3 no depende de los reportes de texto de los Módulos 1 y 2 — vuelve a leer el archivo GFF original y recalcula sus propias métricas, por lo que puede ejecutarse de forma independiente.

> **Nota metodológica:** los **pseudogenes** (766 en este genoma) se excluyen de los tres módulos, ya que no participan de forma significativa en procesos activos de splicing regulado — es una decisión biológica deliberada, no una limitación técnica del script.

---

## 🛠️ Requisitos 

| Requisito | Detalle |
|---|---|
| **Sistema operativo** | Linux / Unix (probado en Google Cloud Shell, Debian/Ubuntu) |
| **Versión de Bash** | ≥ 4.0 (usa `read -p`, arrays asociativos vía `awk`, `<<<` here-strings). Probado y confirmado en **Bash 5.2.21(1)-release** (GNU bash, Google Cloud Shell) |
| **Dependencias externas** | `awk` (gawk o mawk estándar — no requiere extensiones GNU), `grep`, `sed`, `sort`, `cut`, `wc` — todas incluidas por defecto en la mayoría de distribuciones Linux |
| **Archivo de entrada** | Archivo de anotación genómica en formato `.gff` o `.gff3` (NCBI RefSeq) |

Verifica tu versión de bash con:
```bash
bash --version
```

> El script comienza con la línea `#!/bin/bash` (shebang), que le indica al sistema operativo qué intérprete usar para ejecutarlo. Por eso, aunque le des permisos de ejecución (`chmod +x`), el script **debe correrse en un sistema con Bash instalado en `/bin/bash`** — no es compatible con `sh` puro (Dash), ya que usa características propias de Bash como `read -p`, `<<<` (here-strings) y `[[ ]]`.

---

## 📥INSTALACIÓN

* 1. Clona el repositorio:

```bash    
git clone https://github.com/dafnebc019/Proyecto_TF.git
   cd Proyecto_TF
```

* 2. Otorga permisos de ejecución al script:

```bash
chmod +x script_AnalisisGFF.sh
```

**Coloca tu archivo .gff de entrada en el mismo directorio (o anota su ruta completa para usarla en el siguiente paso).**

 * 3. Descarga el archivo GFF de *Nematostella vectensis* desde NCBI y descomprímelo en el directorio del proyecto:
   ```bash
   wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/932/526/225/GCF_932526225.1_jaNemVect1.1/GCF_932526225.1_jaNemVect1.1_genomic.gff.gz
   gunzip GCF_932526225.1_jaNemVect1.1_genomic.gff.gz
   mv GCF_932526225.1_jaNemVect1.1_genomic.gff nematostella.gff
   ```
----

## ▶️ Uso

```bash
./script_AnalisisGFF.sh <archivo.gff>
```

El archivo GFF se pasa como **argumento posicional** ($1) al ejecutar el script. Luego el programa solicita datos de forma interactiva y despliega un menú de opciones.

### Parámetros / Opciones

El script no usa flags tipo `-f`/`-o`/`-h`; en su lugar funciona con un **menú interactivo**:

| Entrada | Descripción |
|---|---|
| `<archivo.gff>` | (argumento obligatorio) ruta al archivo GFF/GFF3 a analizar |
| Prompt: `Nombre del organismo en estudio` | texto libre, para encabezar los reportes |
| Prompt: `Introduce nombre de estudiante` | texto libre, para encabezar los reportes |
| Opción `1` | Análisis de Splicing Alternativo |
| Opción `2` | Análisis de Arquitectura Génica |
| Opción `3` | Análisis de Correlación Estructura–Complejidad |
| Opción `4` | Salir del programa |

### Ejemplo de ejecución

```bash
./script_AnalisisGFF.sh nematostella.gff
Nombre del organismo en estudio: Nematostella vectensis
Introduce nombre de estudiante: Amelia Dafne Cruz

 MENU ANALISIS BIOINFORMATIC:
-------------------------------------------------
1. Analisis Splicing Alternativo
2. Arquitectura Genica
3. Analisis de Correlacion
4. Salir del programa
---------------------------------------
Ingrese una opcion [1-4]: 1
```
---

## 📄 Entrada esperada

Archivo `.gff`/`.gff3` con las 9 columnas estándar (`seqid`, `source`, `type`, `start`, `end`, `score`, `strand`, `phase`, `attributes`), donde `attributes` contiene pares `clave=valor` separados por `;` (ej. `ID=`, `Parent=`, `gene=`).

Verificación rápida del formato antes de correr el script:
```bash
head -5 nematostella.gff
tail -5 nematostella.gff
```
Ejemplo real de una línea válida (tipo `exon`):
```
NC_064034.1  Gnomon  exon  623791  623986  .  -  .  ID=exon-XR_007307167.1-1;Parent=rna-XR_007307167.1;gbkey=misc_RNA;gene=LOC116613690;transcript_id=XR_007307167.1
```

El script valida automáticamente (con mensajes de error en rojo) que:
- Se haya proporcionado un archivo como argumento.
- El archivo exista en la ruta indicada.
- El archivo no esté vacío (0 bytes).
- La extensión sea estrictamente `.gff` o `.gff3`.

---

-----

## 📤 Salida esperada (resultados)

Cada opción imprime el resultado en consola **y** lo guarda en un archivo de texto en el directorio de ejecución:

| Opción | Archivo generado |
|---|---|
| 1. Splicing Alternativo | `reporte_splicing.txt` |
| 2. Arquitectura Génica | `arquitectura_gff.txt` |
| 3. Correlación Estructura-Complejidad | `reporte_correlacion.txt` |

**Opción 1 — Splicing Alternativo (salida real):**
```
Ejecutando Analisis 1: SPLICING ALTERNATIVO....
Splicing Alternativo de Nematostella Vectnesis
TOP 5 genes con mas variantes de splicing:
------------------------
Gen: gene-LOC5516164 -> Isoformas: 50
Gen: gene-LOC5521105 -> Isoformas: 49
Gen: gene-LOC5511589 -> Isoformas: 45
Gen: gene-LOC5507355 -> Isoformas: 39
Gen: gene-LOC116614186 -> Isoformas: 35
------------------------
Total de genes con Splicing Alternativo: 19231
OK reporte completo se guardo en 'reporte_splicing.txt'
```

**Opción 2 — Arquitectura Génica (salida real, confirmada):**
```
== CONTROL DE CALIDAD DE TRANSCRITOS ==
Transcritos almacenados: 53358
Transcritos con exones: 53354
===============================

METRICAS DE GENES:
Numero total de genes: 37996
Longitud promedio de los genes: 5383.56 pb
Longitud MAXIMA de gen encontrada: 192681 pb
Longitud Minim de gen encontrada: 62 pb
---------------------------------
ESTRUCTURA DE EXONES E INTRONES:
Numero total de exones anotados : 408925
Numero estimado de intrones : 355571
Promedio de exones por gen: 10.76 exones/gen
Promedio de intrones por gen : 9.36 intrones/gen
• Longitud promedio de exones : 272.72 pb
• Longitud promedio de intrones    : 261.64 pb
---------------------------------
PROPORCIONES DEL PAISAJE GÉNICO:
Regiones Codificantes (exones): 54.52%
Regiones No Codificantes (Intron): 45.48%
---------------------------------
RANKING DE LOS 10 GENES MÁS LARGOS:
---------------------------------
1. Gen: gene-LOC5516654    -> Tamaño: 192681 pb
2. Gen: gene-LOC5521826    -> Tamaño: 171210 pb
3. Gen: gene-LOC5512112    -> Tamaño: 156894 pb
4. Gen: gene-LOC5502675    -> Tamaño: 155702 pb
5. Gen: gene-LOC116617962  -> Tamaño: 150992 pb
6. Gen: gene-LOC5513895    -> Tamaño: 145441 pb
7. Gen: gene-LOC116613873  -> Tamaño: 140608 pb
8. Gen: gene-LOC5520309    -> Tamaño: 138668 pb
9. Gen: gene-LOC5522255    -> Tamaño: 137609 pb
10. Gen: gene-LOC5503308   -> Tamaño: 136135 pb
---------------------------------
[OK] Módulo 2 finalizado. Reporte guardado en 'arquitectura_gff.txt'.
```
Nota: `Transcritos almacenados` (53,358) es ahora igual o mayor que `Transcritos con exones` (53,354), como debe ser biológicamente — confirma que la detección universal de transcritos (`Parent=gene-`) quedó corregida.

**Opción 3 — Correlación Estructura-Complejidad (salida real, confirmada):**
```
-- ANALISIS DE CORRELACION: ESTRUCTURA VS COMPLEJIDAD --
------------------------------------------------------
Genes analizados: 37996
Longitud mediana de gen: 1817 pb
Isoformas medianas por gen: 1

COEFICIENTE DE CORRELACION DE PEARSON (r):
Longitud genica vs Numero de isoformas
r = 0.4555 -> Correlacion: MODERADA

Interpretacion: a mayor longitud, TIENDE A HABER MAS splicing.
------------------------------------------------------
DISTRIBUCION DE CUADRANTES:
Genes compactos con alta complejidad transcriptómica  : 105 genes
Gigantes y complejos (lo esperado)                    : 6661 genes
Genes extensos con baja complejidad transcriptómica   : 18894 genes
Gigantes y Simples (espacio 'desperdiciado')          : 12336 genes
------------------------------------------------------
TOP 10 CAMPEONES DE EFICIENCIA REGULATORIA:
(mas isoformas por cada 1000 pb -> maxima complejidad en minimo espacio)
1. gene-LOC125568235   Long:1723 Exones:73 Isoformas:10 Densidad:5.8038 iso/kb
2. gene-LOC125568329   Long:1359 Exones:9  Isoformas:4  Densidad:2.9433 iso/kb
3. gene-LOC125560641   Long:715  Exones:5  Isoformas:2  Densidad:2.7972 iso/kb
4. gene-LOC116616307   Long:726  Exones:4  Isoformas:2  Densidad:2.7548 iso/kb
5. gene-LOC125560700   Long:846  Exones:4  Isoformas:2  Densidad:2.3641 iso/kb
6. gene-LOC116614473   Long:851  Exones:6  Isoformas:2  Densidad:2.3502 iso/kb
7. gene-LOC125730640   Long:861  Exones:5  Isoformas:2  Densidad:2.3229 iso/kb
8. gene-LOC125566676   Long:1792 Exones:13 Isoformas:4  Densidad:2.2321 iso/kb
9. gene-LOC125563030   Long:898  Exones:4  Isoformas:2  Densidad:2.2272 iso/kb
10. gene-LOC116618786  Long:901  Exones:5  Isoformas:2  Densidad:2.2198 iso/kb
------------------------------------------------------
>>> GEN CON MAYOR DENSIDAD TRANSCRIPTÓMICA
gene-LOC125568235  (10 isoformas en solo 1723 pb)
>>> GIGANTE DORMIDO (gen mas largo con splicing casi nulo)
gene-LOC5522255  (137609 pb, solo 1 isoforma(s))
====================================================
[OK] Modulo 3 finalizado. Reporte guardado en 'reporte_correlacion.txt'.
```

---

## 🧪 Pruebas (`bash script_AnalisisGFF.sh`)

El script incluye validación robusta de errores. Casos de prueba cubiertos:

1. **Sin argumentos** → `./script_AnalisisGFF.sh` → Error: "No se proporciona ningún archivo"
2. **Archivo inexistente** → `./script_AnalisisGFF.sh noexiste.gff` → Error: "no existe en la ruta especifica"
3. **Archivo vacío (0 bytes)** → `./script_AnalisisGFF.sh vacio.gff` → Error: "archivo esta vacio"
4. **Extensión inválida** → `./script_AnalisisGFF.sh datos.txt` → Error: "Extensión inválida. Debe terminar en .gff o .gff3"

---

## 📚 Referencias

- BioBAM — [Differences between GTF and GFF files in genomic data analysis](https://www.biobam.com/differences-between-gtf-and-gff-files-in-genomic-data-analysis/)
- QIAGEN CLC Genomics Workbench — [GFF3 format manual](https://resources.qiagenbioinformatics.com/manuals/clcgenomicsworkbench/1204/index.php?manual=GFF3_format.html)
- NCBI — [About NCBI GFF3](https://www.ncbi.nlm.nih.gov/datasets/docs/v2/reference-docs/file-formats/annotation-files/about-ncbi-gff3/)
- Dumortier et al. — genómica comparada, splicing alternativo y metilación en *Nematostella vectensis*: https://link.springer.com/article/10.1186/1471-2164-13-480
- Alternative Splicing Across the Tree of Life (correlación AS vs contenido codificante): https://elifesciences.org/reviewed-preprints/94802
