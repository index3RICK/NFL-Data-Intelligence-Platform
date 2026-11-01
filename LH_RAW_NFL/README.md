# LH_RAW_NFL

## Zona de aterrizaje de datos NFL

### Descripción

`LH_RAW_NFL` es el entorno encargado de centralizar los conjuntos de datos históricos y actualizados de [nflverse](https://github.com/nflverse/nflverse-data).

Su propósito es funcionar como una zona de aterrizaje (*Data Landing Zone*), donde se almacenan los archivos originales de la NFL en formato Parquet, sin aplicar transformaciones, limpieza o reglas de negocio.

El alcance de esta zona comprende los datos históricos desde 1999 hasta la temporada NFL 2026, con información recopilada hasta la semana 8 en el estado inicial documentado del proyecto.

## ¿Por qué Microsoft Fabric y no Azure Data Lake Storage Gen2?

La idea original del proyecto era implementar una zona de almacenamiento utilizando **Azure Data Lake Storage Gen2 (ADLS Gen2)**, siguiendo un enfoque habitual en soluciones de ingeniería de datos: separar la recepción de datos de los procesos posteriores de transformación y análisis.

Sin embargo, para desarrollar el proyecto y practicar con las herramientas del ecosistema Microsoft, se decidió implementar esta funcionalidad directamente en **Microsoft Fabric**, utilizando OneLake y un Lakehouse como almacenamiento.

Esta decisión permite trabajar con una estructura de aterrizaje de datos y mantener la separación entre la ingesta y el procesamiento, sin necesidad de desplegar una cuenta de almacenamiento independiente en Azure.

**Importante:** `LH_RAW_NFL` no es una implementación de ADLS Gen2. Es una emulación conceptual de su función como zona de aterrizaje, desarrollada sobre Microsoft Fabric y OneLake.

## Representación conceptual

![Representación conceptual de la zona de aterrizaje NFL](../docs/img/nfl_data_landing.png)

La imagen representa la idea original de contar con una zona de almacenamiento dedicada a recibir los datos de nflverse. En este proyecto, esa función se implementa mediante Microsoft Fabric.

## Alcance y responsabilidades

Este entorno tiene una responsabilidad específica: **recibir y conservar los datos originales**.

Sus principales funciones son:

- Recopilar conjuntos de datos históricos y actualizados de nflverse.
- Almacenar los archivos originales en formato Parquet.
- Conservar los nombres y el contenido de los archivos recibidos.
- Organizar los archivos por tipo de conjunto de datos.
- Permitir nuevas ingestas para incorporar información reciente.
- Comprobar que los archivos descargados estén completos y tengan una integridad básica antes de almacenarlos.

### ¿Por qué no se implementa una arquitectura aquí?

`LH_RAW_NFL` no implementa una arquitectura Medallion ni contiene capas Bronze, Silver o Gold.

Tampoco realiza limpieza, normalización, deduplicación ni transformaciones de negocio.

Las validaciones existentes están orientadas exclusivamente a comprobar que la ingesta se complete correctamente y que los archivos descargados no estén incompletos o dañados.

La validación de calidad de los datos y su preparación para el análisis corresponden al entorno de procesamiento `LH_NFL_PLATFORM`.

## Flujo general

1. **Fuente:** nflverse proporciona los conjuntos de datos de la NFL.
2. **Ingesta:** los notebooks de este entorno generan los enlaces y descargan los archivos correspondientes.
3. **Validación de integridad:** se realizan comprobaciones básicas sobre los archivos recibidos.
4. **Almacenamiento:** los archivos se conservan en OneLake, dentro del Lakehouse `LH_RAW_NFL`.
5. **Disponibilidad:** los datos quedan disponibles para su consumo desde el entorno de procesamiento mediante un OneLake Shortcut.

## Estructura de almacenamiento

```text
LH_RAW_NFL/
└── Files/
    └── nflverse/
        ├── play_by_play/
        ├── rosters/
        ├── weekly_rosters/
        ├── stats_player/
        ├── stats_team/
        ├── schedules/
        ├── players/
        └── teams/
```

La estructura permite organizar los conjuntos de datos sin modificar su contenido original.

## Relación con el entorno de procesamiento

Los datos almacenados en `LH_RAW_NFL` son consumidos por el segundo entorno del proyecto: `LH_NFL_PLATFORM`.

La conexión se realiza mediante un **OneLake Shortcut**, que permite acceder a los archivos del Lakehouse de aterrizaje desde el entorno de procesamiento sin generar una segunda copia física de los datos.

El procesamiento, las transformaciones y las capas de la arquitectura Medallion se desarrollan en `LH_NFL_PLATFORM`, no en este entorno.

## Fuente de datos

- [nflverse-data - GitHub](https://github.com/nflverse/nflverse-data)
- [nflverse - Documentación](https://nflverse.nflverse.com/)

## Estado del entorno

| Elemento | Estado |
|---|---|
| Plataforma de almacenamiento | Microsoft Fabric - OneLake |
| Almacenamiento | Lakehouse `LH_RAW_NFL` |
| Función | Data Landing Zone |
| Formato principal | Parquet |
| Cobertura histórica | Desde 1999 |
| Referencia de datos documentada | Temporada 2026, semana 8 |
| Transformaciones de negocio | No implementadas |
| Arquitectura Medallion | No implementada |
| Consumo posterior | `LH_NFL_PLATFORM` |
