# Ejemplo de la organización del proyecto

**Autores:** Ivonne Vaca

**Fecha:**  23 de marzo de 2024

## Introducción

El informe será un documentos en Github en formato Markdown (método de escritura, basado en un formato de texto plano).

Aquí vemos la diferencia entre un procesador de texto (tipo Word) vs Markdown, abiertos en un editor de texto plano. 

![Procesador de texto vs Markdown](/Proyecto_ejemplo/imagenes/editor_texto_plano.png)


Les dejo algunos formatos para el uso :+1: :

## 1. Titulos
```
# Título primer nivel
## Título segundo nivel
###  Título tercer nivel
```
Se visualiza así:
# Título primer nivel
## Título segundo nivel
###  Título tercer nivel

## 2. Texto en negrita
```
**Hola**
```
**Hola**

## 3. Texto en cursiva

```
*Hola*
```
*Hola*

## 4. Superíndice y subíndice
```
Este es un <sub>subíndice</sub> 
Este otro es un <sup>superíndice</sup> 
```
Este es un <sub>subíndice</sub> 

Este otro es un <sup>superíndice</sup>

## 5. Adicionar línea de comando

````
```
Mira, puedes ver las comillas y formato
```
````
Se ve así:

```
Mira, puedes ver las comillas y formato
```
## 6. Links
```
Este sitio fue construido usando [GitHub](https://pages.github.com/)
```
Este sitio fue construido usando [GitHub](https://pages.github.com/)

## 7. Listas
```
Usa * - o + por ejemplo:
* Empezamos en 3
+ Empezamos en 2
- Empezamos en 1
* 0
```

Visualizamos así:
* Empezamos en 3
+ Empezamos en 2
- Empezamos en 1
* 0

## 8. Creaciones de diagramas

Creando un diagrama parcial  de pipeline:
````
```mermaid
graph TD;
    Lecturas_crudas--> Control_calidad;
    Control_calidad-->FastQC;
    Control_calidad-->Falco;
    FastQC -->Trimmomatic;
    Falco -->Trimmomatic;
    Trimmomatic --> Ensamblaje;
```
````

```mermaid
graph TD;
     Lecturas_crudas--> Control_calidad;
    Control_calidad-->FastQC;
    Control_calidad-->Falco;
    FastQC -->Trimmomatic;
    Falco -->Trimmomatic;
    Trimmomatic --> Ensamblaje;
```

Puede aprender sobre Github en la [Guía de inicio rápido](https://docs.github.com/es/get-started/start-your-journey)
