# 🧬Ensamble Hibrido de Genoma Bacteriano.🧫
Protocolo para el ensamble híbrido de novo y anotación funcional de genomas bacterianos. Flujo de trabajo optimizado para la integración de lecturas cortas (Illumina) y largas (PacBio) mediante Unicycler, con anotación genómica automatizada usando Prokka.
**Nota**: Este protocolo se realizo considerando el siguente Hardware:
**Máquina:** Asus TUF GAMING A15
**Procesador:** RYZEN 5 7000 series
**RAM:** 24 GB
**Sistema Operativo:** Windows 11
## Instalación de <img width="28" height="28" alt="image" src="https://github.com/user-attachments/assets/76ea8cd6-4024-403d-bcac-64a9dfb2fc66" /> Ubuntu 24.04,1 LTS, instalación de <img width="28" height="28" alt="image" src="https://github.com/user-attachments/assets/3e97a15e-1de2-431a-bb58-df53f894aa07" /> Miniforge3 y creacion de ambiente con los programas requeridos
### Se instala Ubuntu 24.04.1 LTS y se descargan los archivos de Miniforge3 desde la terminal de Ubuntu con el comando wget.
```
wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh
```
### Se ejecuta Miniforge3
```
bash Miniforge3-Linux-x86_64.sh
```
### Después de la instalación, reinicia la terminal y debería aparecer el prefijo de Conda:
`
(base) usuario@USUARIO:~$
`
### Se crea el entorno de trabajo `ensamble_env` con los siguentes programas seqkit, samtools, unicycler, flye, quast, fastqc, nanoplot, fastp y multiqc.
```
mamba create -n ensamble_env -c bioconda -c conda-forge fastqc samtools nanoplot fastp multiqc unicycler quast busco --yes
```
Descripcion de los programas:
| Programa | Función Principal | Tipo de Datos |
| :---: | :---: | :---: |
| <img width="28" height="28" alt="image" src="https://github.com/user-attachments/assets/4729e58a-13aa-4d2b-a9f4-a1fd06558aed" /> **FastQC** | Control de calidad de lecturas cortas | Illumina |
| **Samtools** | Procesamiento de archivos biológicos y conversión de formatos (BAM a FASTQ) | Pacbio / Oxford Nanopore |
| <img width="28" height="28" alt="image" src="https://github.com/user-attachments/assets/598d5cb8-e36b-401c-a323-bc7f6e9e8cf0" /> **NanoPlot** | Estadísticas y calidad de lecturas largas | PacBio / Oxford Nanopore |
| <img width="28" height="28" alt="image" src="https://github.com/user-attachments/assets/ddeacd49-4d04-4b4e-9266-f5f80bbe11af" /> **fastp** | Control de calidad, filtrado y corrección | Lecturas cortas |
| <img width="29" height="29" alt="image" src="https://github.com/user-attachments/assets/38b1e605-9a4f-46ca-9cdb-ff004feee2b7" /> **MultiQC** | Consolidación de reportes en un solo HTML | Todos los anteriores |
| <img width="28" height="28" alt="image" src="https://github.com/user-attachments/assets/17a08db0-15b2-40f5-a7e6-164d10555e10" /> **Unicycler** | Ensamble híbrido *de novo* asistido por grafos | Illumina + PacBio/ONT |
| <img width="28" height="28" alt="image" src="https://github.com/user-attachments/assets/3e4b62c9-16ac-4c0b-83aa-cf228c2aaaf2" /> **QUAST** | Evaluación de métricas del ensamble (N50, L50) | Contigs / Scaffolds |
| <img width="28" height="28" alt="image" src="https://github.com/user-attachments/assets/f4fad507-2586-497b-b1f0-12e10f449faa" /> **Busco** | Evaluación de completitud biológica mediante la búsqueda de genes esenciales | Contigs / Scaffolds |
---
Para activar tu entorno de trabajo usa el siguiente comando:
```
conda activate ensamble_env
```
**🗂️Nota: Si quieres organizar tu trabajo puedes usar la siguiente estructura usando mkdir**
```
mkdir -p Proyecto_Ensamble/{00_Secuencias_crudas/{Illumina,Pacbio},01_Evaluacion_Secuencias/{Reportes_Secuencias_Crudas/{Illumina,Pacbio},Reportes_Post_Filtrado/{Illumina,Pacbio}},02_Filtrado,03_Ensamble,04_Evaluacion_Ensamble,05_Anotacion,Scripts_Bitacora}
```
`[Nombre_del_modelo_de_estudio]` Aqui debes de colocar el nombre de tu modelo de estudio.
| **Nombre de carpeta** | **Descripción** |
| :---: | :---: |
| **00_Secuencias_crudas** | Secuencias crudas directo del secuenciador (cada una dentro de sus respectivas carpetas)|
| **01_Evaluacion_Secuencias** | Reportes iniciales de calidad (FastQC/NanoPlot) |
| **02_Filtrado** | Lecturas procesadas y limpias |
| **03_Ensamble** | Resultados finales del ensamble genómico |
| **04_Evaluacion_Ensamble** |Análisis de calidad del ensamble con QUAST |
| **05_Anotacion** | Resultados de la anotación funcional |
| **scripts / bitacora** | Registro de comandos y scripts personalizados |
---
Entra a tu carpeta con el comando `cd`
## Control de calidad y filtrado de secuencias crudas
**Objetivo**: Revisar la calidad de las secuencias crudas que se obtuvieron por medio de las plataformas Illumina y Pacbio.

### Control de calidad de secuencias cortas (Illumina) con FastQC.
```
fastqc 00_Secuencias_crudas/Illumina/*.fastq -o 01_Evaluacion_Secuencias/Reportes_Secuencias_Crudas/Illumina
```
### Antes de iniciar el control de calidad, los datos crudos de PacBio en formato `.bam` eben convertirse a `fastq`. Para ello usaremos Samstool.
```
samtools fastq 00_Secuencias_crudas/Pacbio/*.bam > 00_Secuencias_crudas/Pacbio/[Nombre_del_Archivo].fastq
```
`[Nombre_del_modelo_de_estudio]` Aqui debes de colocar el nombre que desees.
### Posteriormente se evalua su calidad con Nanoplot.
```
NanoPlot --threads 10 --fastq 00_Secuencias_crudas/Pacbio/*.fastq -o 01_Evaluacion_Secuencias/Reportes_Secuencias_Crudas/Pacbio/
```
### Unificamos los reportes con multiQC.
```
multiqc 01_Evaluacion_Secuencias/ -o 01_Evaluacion_Secuencias/ --title "Reporte Inicial"
```
### Verificamos la calidad y filtramos Lecturas Cortas (Illumina).
Eliminamos secuencias de adaptadores, bases de baja calidad (Q < 30) y lecturas cortas resultantes del proceso de secuenciación de Illumina. Usaremos fastp v0.23.x
```
fastp -i 00_Secuencias_crudas/Illumina/[Nombre_de_Archivo].fastq \
      -I 00_Secuencias_crudas/Illumina/[Nombre_de_Archivo].fastq \
      -o 02_Filtrado/[Nombre_de_Archivo]_clean.fastq.gz \
      -O 02_Filtrado/[Nombre_de_Archivo]_clean.fastq.gz \
      --html 02_Filtrado/reporte_fastp.html \
      --json 02_Filtrado/reporte_fastp.json \
      --qualified_quality_phred 30 \
      --length_required 50 \
      --detect_adapter_for_pe \
      --thread 10
```
| Parámetro | Nombre | Descripción Técnica |
| :---: | :---: | :---: |
| **`-i`** | *Input 1* | Ruta del archivo original de Illumina **R1** (Forward). |
| **`-I`** | *Input 2* | Ruta del archivo original de Illumina **R2** (Reverse). |
| **`-o`** | *Output 1* | Nombre del archivo de salida limpio para **R1**. El sufijo `.gz` indica compresión. |
| **`-O`** | *Output 2* | Nombre del archivo de salida limpio para **R2**. |
| **`--html`** | *Reporte HTML* | Genera un reporte visual interactivo de la calidad antes y después del proceso. |
| **`--json`** | *Reporte JSON* | Genera un reporte en formato de datos, útil para procesos automatizados. |
| **`-q 30`** | *Qualified Quality* | Define que una base es "buena" si tiene un Q-score ≥ 30 (99.9% de precisión). |
| **`-u 40`** | *Unqualified Percent* | Máximo porcentaje permitido de bases que no cumplen con el requisito de calidad (por default es 40%). |
| **`-l 50`** | *Length Required* | Elimina lecturas que, tras el corte, tengan una longitud menor a 50 bp. |
| **`--detect_adapter_for_pe`** | *Adapter Trimming* | Detecta y elimina automáticamente las secuencias de adaptadores en lecturas pareadas (Paired-End). |
| **`--trim_poly_g`** | *PolyG Trimming* | Elimina colas de Guaninas (G) que suelen aparecer como ruido en secuenciadores de dos colores (como NovaSeq/NextSeq). |
| **`--thread 10`** | *Hilos de procesamiento* | Utiliza 10 núcleos de tu procesador |

> **Nota de Robustez:** Al usar `-q 30`, estamos siendo más estrictos que el estándar común (Q20), lo que garantiza que solo las bases de más alta confianza entren al proceso de ensamble con Unicycler.
Filtrado de PacBio con `seqkit`
```
seqkit seq -m 1000 --threads 10 00_Secuencias_crudas/Pacbio/[Nombre_de_Archivo].fastq > 02_Filtrado/[Nombre_de_Archivo]_filt.fastq
```
### Filtrado de Lecturas Largas (PacBio HiFi)
A diferencia de Illumina, el filtrado de PacBio se enfoca en la longitud de los fragmentos para maximizar la capacidad del ensamblador de resolver regiones repetitivas.

| Parámetro | Función | Descripción |
| :---: | :---: | :---: |
| **`seq`** | Comando de SeqKit | Indica que realizaremos una transformación o filtrado de secuencias. |
| **`-m 1000`** | *Minimum length* | Elimina cualquier lectura que mida menos de 1,000 pares de bases. |
| **`--threads 10`** | Multiprocesamiento | Utiliza 10 núcleos del procesador |
| **`>`** | Redirección | Guarda el resultado filtrado en un nuevo archivo FASTQ. |

> **Nota de Robustez:** Las lecturas HiFi ya vienen corregidas de fábrica. Filtrar por longitud mínima evita el "ruido" de fragmentos pequeños que podrían generar ensambles fragmentados o erróneos.

***Repetimos los pasos de calidad con Fastqc, Nanoplot y MultiQC, con las secuencias filtradas***
```
fastqc 02_Filtrado/[Nombre_de_Archivo]_R*_clean.fastq.gz -o 01_Evaluacion_Secuencias/Reportes_Post_Filtrado/Illumina/
```
```
NanoPlot --threads 10 --fastq 02_Filtrado/[Nombre_de_Archivo]_filt.fastq -o 01_Evaluacion_Secuencias/Reportes_Post_Filtrado/Pacbio/
```
```
multiqc 01_Evaluacion_Secuencias/Reportes_Post_Filtrado/ -o 01_Evaluacion_Secuencias/Reportes_Post_Filtrado/ --title "Control de Calidad Post-Filtrado"
```
## Ensamblado del Genoma
## Se ensambla el genoma usando Unycicler
```
unicycler -1 02_Filtrado/[Nombre_de_Archivo]_R1_clean.fastq.gz \
          -2 02_Filtrado/[Nombre_de_Archivo]_R2_clean.fastq.gz \
          -l 02_Filtrado/[Nombre_de_Archivo]_filt.fastq \
          -o 03_Ensamble/ \
          --threads 10 \
          --mode bold \
          --keep 2
```
| Parámetro | Función |	Importancia en tu proyecto |
| :---: | :---: | :---: |
| **`-1 / -2`** |	Lecturas Illumina |	Proporcionan la alta precisión para que no haya errores de indels |
| **`-l`** |	Lecturas PacBio |	Actúan como "puentes" para unir los contigs y cerrar el cromosoma |
| **`-o`** |	Output |	Crea la carpeta 03_Ensamble/ con todos los resultados |
| **`--mode bold`** |	Modo de ensamble | Prioriza la conectividad del genoma (produce menos contigs, pero más largos) |
| **`--threads 10`** |	Procesamiento |	 Utiliza 10 núcleos de tu procesador  (dejando algunos núcleos libres para el sistema) |
| **`--keep 2`** |	Retención de archivos |	Guarda archivos intermedios útiles para depurar si algo falla |
---
## Análisis de Calidad con Quast
```
quast 03_Ensamble/assembly.fasta -o 04_Evaluacion_Ensamble/quast_results/
```
## Análisis de calidad con Busco
```
busco -i 03_Ensamble/assembly.fasta \
      -o 04_Evaluacion_Ensamble/busco_results/ \
      -m genome \
      -l lactobacillales_odb10 \
      --cpu 10
```
| Parámetro | Función | Importancia |
| :---: | :---: | :---: |
| **`-i`** | Input | Tu archivo de ensamble final (assembly.fasta) |
| **`-o`** | Output | Carpeta donde se guardarán los resultados de evaluación |
| **`-m`** | genome | Mode Indica que vas a evaluar un genoma completo (no un transcriptoma) |
| **`-l`** | Lineage | lactobacillales_odb10 es el set de genes esenciales para lactobacilos |
| **`--cpu 10`** | Procesamiento |  Utiliza 10 núcleos de tu procesador  |
