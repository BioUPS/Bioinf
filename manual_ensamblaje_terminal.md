# Manual práctico de ensamblaje genómico y visualización
## Desde secuencias FASTQ hasta visualización en JBrowse

---

# Objetivo de la práctica

En esta práctica aprenderás a desarrollar un pipeline bioinformático básico para:

1. Evaluar calidad de lecturas FASTQ
2. Filtrar y recortar secuencias
3. Ensamblar un genoma de novo
4. Evaluar el ensamblaje
5. Realizar alineamiento de lecturas
6. Generar archivos BAM
7. Visualizar el ensamblaje y alineamientos
8. Visualizar anotaciones en JBrowse

El flujo de trabajo simula un análisis bioinformático real aplicado en:

- Genómica bacteriana
- Metagenómica
- Vigilancia epidemiológica
- Genómica comparativa
- Anotación genómica

---

# Flujo general del pipeline

```text
FASTQ crudo
   ↓
Control de calidad (FastQC)
   ↓
Trimming (Trimmomatic)
   ↓
Ensamblaje (SPAdes)
   ↓
Evaluación del ensamblaje
   ↓
Alineamiento contra scaffolds (BWA)
   ↓
Conversión SAM → BAM (SAMtools)
   ↓
Indexación BAM
   ↓
Visualización (IGV/JBrowse)
```

---

# Requisitos del sistema

## Sistema operativo
Ubuntu/Linux

## Programas necesarios

Instalar:

```bash
sudo apt update
```

## FastQC

```bash
sudo apt install fastqc
```

## Java

```bash
sudo apt install default-jre
```

## Trimmomatic

```bash
sudo apt install trimmomatic
```

## SPAdes

```bash
sudo apt install spades
```

## BWA

```bash
sudo apt install bwa
```

## SAMtools

```bash
sudo apt install samtools
```

## IGV

```bash
sudo apt install igv
```

## seqkit (opcional)

```bash
sudo apt install seqkit
```

---

# PARTE 1 — Descarga de secuencias FASTQ

## Crear directorio de trabajo

```bash
mkdir ensamblaje_practica
cd ensamblaje_practica
```

---

## Descargar dataset FASTQ

Ejemplo:

```bash
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR000/SRR000001/SRR000001.fastq.gz
```

---

## Descomprimir

```bash
gunzip SRR000001.fastq.gz
```

---

## Visualizar contenido

```bash
head SRR000001.fastq
```

---

# ¿Qué es un archivo FASTQ?

Formato utilizado para almacenar:

- Secuencia nucleotídica
- Calidad de cada base

Estructura:

```text
@Identificador
ATCGATCG
+
IIIIHHHH
```

---

# PARTE 2 — Control de calidad con FastQC

## Ejecutar FastQC

```bash
fastqc SRR000001.fastq
```

---

## Archivos generados

```bash
ls
```

Aparecerán:

```text
SRR000001_fastqc.html
SRR000001_fastqc.zip
```

---

## Abrir reporte

```bash
xdg-open SRR000001_fastqc.html
```

---

# Interpretación básica FastQC

## 1. Basic Statistics

Muestra:

- Número de lecturas
- Longitud
- %GC

---

## 2. Per base sequence quality

Evalúa calidad por posición.

Interpretación:

- Verde → buena calidad
- Amarillo → aceptable
- Rojo → mala calidad

Ideal:

```text
Q ≥ 30
```

---

## 3. Per sequence GC content

Evalúa distribución GC.

Ideal:

- Distribución tipo campana

Puede detectar:

- Contaminación
- Mezcla de organismos

---

## 4. Adapter Content

Detecta adaptadores residuales.

Si aparece alto:

→ realizar trimming

---

# PARTE 3 — Trimming con Trimmomatic

## Verificar ubicación

```bash
dpkg -L trimmomatic
```

Ruta típica:

```text
/usr/share/java/trimmomatic-0.39.jar
```

---

## Ejecutar trimming

```bash
java -jar /usr/share/java/trimmomatic-0.39.jar SE -phred33 \
SRR000001.fastq SRR000001_trimmed.fastq \
LEADING:3 TRAILING:3 SLIDINGWINDOW:4:20 MINLEN:36 \
2>&1 | tee trimming.log
```

---

# Explicación del trimming

## LEADING:3

Elimina bases al inicio con calidad < 3

---

## TRAILING:3

Elimina bases finales con calidad < 3

---

## SLIDINGWINDOW:4:20

Revisa ventanas de 4 bases.

Si calidad promedio < 20:

→ corta lectura.

---

## MINLEN:36

Descarta lecturas menores a 36 pb.

---

# Reevaluar calidad

```bash
fastqc SRR000001_trimmed.fastq
```

Comparar ambos reportes.

---

# Trimming adicional con CROP

```bash
java -jar /usr/share/java/trimmomatic-0.39.jar SE -phred33 \
SRR000001.fastq SRR000001_trimmed2.fastq \
CROP:280 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:20 MINLEN:36
```

---

# ¿Qué hace CROP?

Conserva únicamente:

- primeras 280 bases

Elimina el resto.

---

# PARTE 4 — Ensamblaje genómico

# ¿Qué es el ensamblaje?

Proceso computacional donde:

- múltiples reads
- son reconstruidos
- para formar secuencias más largas.

---

# Conceptos importantes

## Read

Fragmento corto de ADN.

---

## Contig

Secuencia continua sin gaps.

---

## Scaffold

Conjunto ordenado de contigs.

Puede contener:

```text
NNNNN
```

(gaps)

---

# Tipos de ensamblaje

## Ensamblaje de novo

No utiliza referencia.

---

## Ensamblaje por referencia

Usa un genoma conocido.

---

# PARTE 5 — Ensamblaje con SPAdes

## Crear directorio

```bash
mkdir ensamblaje
```

---

## Mover archivo

```bash
mv SRR000001_trimmed2.fastq ensamblaje/
```

---

## Ejecutar SPAdes
Fuera de la carpeta ensamblaje
```bash
spades -o ensamblaje/ -s ensamblaje/SRR000001_trimmed2.fastq
```
Dentro de la carpeta ensamblaje
```bash
spades -o . -s SRR000001_trimmed2.fastq
```
---

# Explicación

## -o

Directorio de salida.

---

## -s

Lecturas single-end.

---

# Archivos importantes

Entrar:

```bash
cd ensamblaje
```

---

## Visualizar archivos

```bash
ls
```

Archivo más importante:

```text
scaffolds.fasta
```

---

## Ver secuencias

```bash
head scaffolds.fasta
```

---

# PARTE 6 — Evaluación del ensamblaje

## Estadísticas básicas

```bash
seqkit stats scaffolds.fasta
```

---

# Parámetros importantes

## num_seqs

Número de scaffolds.

---

## sum_len

Longitud total ensamblada.

---

## max_len

Scaffold más largo.

---

## GC

Contenido GC.

---

## N50

Longitud donde:

- 50% del ensamblaje
- está contenido
- en scaffolds iguales o mayores.

<!--
NOTA INVISIBLE
N50 alto:

→ ensamblaje más continuo.
-->
---

# PARTE 7 — Alineamiento de lecturas

# Objetivo

Validar el ensamblaje.

Preguntas:

- ¿Las lecturas vuelven a alinearse?
- ¿Hay regiones sin cobertura?
- ¿Existen errores?

---

# Indexar scaffolds

```bash
bwa index scaffolds.fasta
```

---

# Crear carpeta resultados

```bash
mkdir resultados_bwa
cd resultados_bwa
```

---

# Alinear lecturas

```bash
bwa mem ../scaffolds.fasta ../SRR000001_trimmed2.fastq -o bwaFile.sam
```

---

# ¿Qué hace BWA?

Busca:

- mejor posición
- de cada lectura
- sobre el ensamblaje.

---

# PARTE 8 — Conversión SAM → BAM

## Convertir SAM a BAM

```bash
samtools view -bS bwaFile.sam > bwaFile.bam
```

---

# Diferencia entre formatos

## SAM

Texto.

Muy grande.

---

## BAM

Binario comprimido.

Más eficiente.

---

# Ordenar BAM

```bash
samtools sort -o bwaFilesorted.bam bwaFile.bam
```

---

# Indexar BAM

```bash
samtools index bwaFilesorted.bam
```

Genera:

```text
bwaFilesorted.bam.bai
```

---

# PARTE 9 — Visualización en IGV

## Abrir IGV

```bash
igv
```

---

# Cargar genoma

Ruta:

```text
Genomes → Load Genome from File
```

Seleccionar:

```text
scaffolds.fasta
```

---

# Cargar alineamiento

Ruta:

```text
File → Load from File
```

Seleccionar:

```text
bwaFilesorted.bam
```

Importante:

El archivo:

```text
.bai
```

Debe estar en la misma carpeta.

---

# ¿Qué observar?

## Cobertura

Cantidad de lecturas alineadas.

---

## Huecos

Regiones sin reads.

---

## SNPs

Bases diferentes.

---

## Regiones repetidas

Cobertura anormal.

---

# PARTE 10 — Visualización en JBrowse

# ¿Qué es JBrowse?

Visualizador genómico web.

Permite explorar:

- scaffolds
- genes
- cobertura
- alineamientos
- anotaciones

---

# Instalación básica de JBrowse 2

## Instalar NodeJS

```bash
sudo apt install nodejs npm
```

---

## Instalar JBrowse CLI

```bash
npm install -g @jbrowse/cli
```

---

# Crear proyecto

```bash
jbrowse create jbrowse_project
```

---

## Entrar al directorio

```bash
cd jbrowse_project
```

---

# Agregar genoma

```bash
jbrowse add-assembly ../ensamblaje/scaffolds.fasta --name ensamblaje
```

---

# Agregar BAM

```bash
jbrowse add-track ../resultados_bwa/bwaFilesorted.bam --load copy
```

---

# Ejecutar servidor

```bash
python3 -m http.server 8080
```

---

# Abrir en navegador

```text
http://localhost:8080
```

---

# ¿Qué visualizar en JBrowse?

## Tracks

Capas de información.

---

## Cobertura

Profundidad de secuenciación.

---

## Alineamientos

Lecturas alineadas.

---

## Genes

Si existe anotación GFF/GTF.

---

# Conclusiones de la práctica

Al finalizar esta práctica el estudiante podrá:

✔ Evaluar calidad de lecturas

✔ Aplicar trimming bioinformático

✔ Ensamblar secuencias genómicas

✔ Evaluar métricas de ensamblaje

✔ Realizar alineamientos

✔ Manipular archivos SAM/BAM

✔ Interpretar cobertura genómica

✔ Visualizar resultados JBrowse

---

# Referencias

- Babraham Institute. FastQC
https://www.bioinformatics.babraham.ac.uk/projects/fastqc/

- Bankevich et al., 2012. SPAdes Genome Assembler
Nature Methods.

- Li & Durbin, 2009. BWA
Bioinformatics.

- Danecek et al., 2021. SAMtools and BCFtools
GigaScience.

- Buels et al., 2016. JBrowse
Genome Biology.

- Nagarajan & Pop, 2013.
Sequence assembly demystified.
Nature Reviews Genetics.

