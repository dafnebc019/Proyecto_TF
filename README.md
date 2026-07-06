# Proyecto_TF
AUTORES :
Amelia Bolaños 
Dayra Espin Catro
Curso :
Principios de Programacion - Bioinformatica
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

# ANALISIS 1
ejecucion del scrip
echo -e "\n${VERDE} Ejecutando Analisis 1: SPLICING ALTERNATIVO....${RESET}"
                echo "Splicing Alternativo de Nematostella Vectnesis"
                #filtramos los ARNm, extraemis el ID del gen padre, contamos y ordenamos numericamente
                #Guardar todo limpio en un archivo temporal
                grep -v '^#' "$ARCHIVO_INPUT" | awk '$3 == "mRNA" {print $9}' | grep -o 'Parent=gene-[^;]*' | cut -d'=' -f2 | sort | uniq -c | sort -nr > .temp_splicing.txt

                #Contamos cuantos gnes en total tienen mas de 1 isoforma
                TOTAL_SPLICING=$(wc -l < .temp_splicing.txt)
                
                #Imprimir la salida y leemos las 5 primeras lineas 
                echo " TOP 5 genes con mas variantes de splicing: "
                echo "------------------------"
                head -n 5 .temp_splicing.txt | awk '{print "Gen: " $2 " -> Isoformas: " $1}'
                echo " -------------------------"
                echo -e "Total de genes con Splicing Alternativo: ${AMARILLO}$TOTAL_SPLICING${RESET}"

                # Guardar el reporte final completo y limpiar el archivo temporal 
                mv .temp_splicing.txt reporte_splicing.txt
                echo -e "\n${VERDE} OK reporte completo se guardo en 'reporte_splicing.txt'${RESET}"
                ;;

# ANALISIS 2
 echo -e "\n${VERDE}Ejecutando Analisis 2: Arquitectura genica... ${RESET}"
                echo "Calculando longitudes (Prom, Max, Min), exones, intrones y proporciones "
                echo "-------------------------------------------------------"

                awk -F'\t' '
                $3 == "gene" {
                        n_genes++
                        len = ($5 - $4 + 1)
                        len_genes += len

                        #Rastrear Maximos y minimos de genes
                        if (n_genes == 1) { max_g = min_g = len }
                        if (len > max_g) { max_g = len }
                        if (len < min_g) { min_g = len }
                }
                $3 == "exon" {
                        n_exones++
                        len_exones += ($5 - $4 + 1)
                }
                END {
                        #Calculos Matematicos biologicos
                        n_intrones = n_exones - n_genes
                        if (n_intrones > 0) {
                                len_intrones= len_genes - len_exones
                                prom_intron= len_intrones / n_intrones
                                if (prom_intron < 0){ prom_intron = 0; len_intrones = 0  }
                        } else {
                                n_intrones = 0; prom_intron = 0; len_intrones = 0
                        }

                        print "METRICAS DE GENES:"
                        printf "Numero total de genes: %d\n", n_genes
                        if (n_genes > 0) {
                                printf "Longitud promedio de los genes: %.2f pb\n", len_genes / n_genes
                                printf "Longitud MAXIMA de gen encontrada: %d pb\n", max_g
                                printf "Longitud Minim de gen encontrada: %d pb\n", min_g
                        }
                        print "---------------------------------"
                        print "ESTRUCTURA DE EXONES E INTRONES:"
                        printf "Numero total de exones anotados : %d\n", n_exones
                        printf "Numero estimado de intrones : %d\n", n_intrones
                        if (n_genes > 0) {
                                printf "Promedio de exones por gen: %.2f exones/gen\n", n_exones / n_genes
                        if (n_exones > 0) printf "• Longitud promedio de exones : %.2f pb\n", len_exones / n_exones
                        if (n_intrones > 0) printf "• Longitud promedio de intrones    : %.2f pb\n", prom_intron

                        print "---------------------------------"
                        print "PROPORCIONES DEL PAISAJE GÉNICO: "
                        total_espacio = len_exones + len_intrones
                        if (total_espacio > 0) {
                                printf "Regiones Codificantes (exones): %.2f%%\n", (len_exones / total_espacio) * 100
                                printf "Regiones No Codificantes (Intron): %.2f%%\n", (len_intrones / total_espacio) * 100
                        }
                }' "$ARCHIVO_INPUT" > arquitectura_gff.txt

                cat arquitectura_gff.txt

                echo "-----------------------------------------"
                echo -e "${AMARILLO}RANKING DE LOS 10 GENES MÁS LARGOS:${RESET}"
                echo "---------------------------------"
                grep -v '^#' "$ARCHIVO_INPUT" |\
                awk -F'\t' '$3 == "gene" {print $5-$4+1, $9}' |\
                grep -oE '^[0-9]+ .*ID=[^;]*' |\
                sed 's/ID=//' |\
                sort -k1,1nr |\
                head -n 10 |\
                awk '{printf "%d. Gen: %-30s -> Tamaño: %s pb\n", NR, $2, $1}'
                echo "---------------------------------------"

                echo -e "${VERDE}[OK] Módulo 2 finalizado. Reporte guardado en 'arquitectura_gff.txt'.${RESET}"
                ;;

