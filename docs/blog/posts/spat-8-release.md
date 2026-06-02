---
date:
  created: 2025-01-20
authors:
  - davide
tags:
  - spatializzazione
  - panning
  - plugin
categories:
  - Plugin
---

# spat_8: spatializzatore a 8 canali

Primo plugin serio: un panner circolare a 8 canali con
controllo di azimuth, distanza e spread.

<!-- more -->

## Design della patch

Il cuore è un array di 8 coefficienti calcolati con coseno
rialzato. La posizione angolare determina il peso di ogni canale.

| Parametro | Range     | Funzione |
|-----------|-----------|----------|
| azimuth   | 0 – 360° | Posizione angolare |
| distance  | 0 – 1    | Attenuazione dal centro |
| spread    | 0 – 1    | Apertura della sorgente |

## Problemi incontrati

Il primo build aveva NaN nell'output — causato da `[pow~]` con
argomenti negativi. Risolto con un clipping preventivo.

!!! tip "Lezione imparata"
    Sempre testare con valori estremi prima di compilare con hvcc.
    I NaN si propagano e sono difficili da debuggare nel plugin
    compilato.

## Download

Disponibile nella pagina [Releases](https://github.com/dvddmg/spat_plugin/releases).
