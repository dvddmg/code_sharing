---
date:
  created: 2025-04-10
authors:
  - davide
tags:
  - granulare
  - plugin
  - dsp
categories:
  - Plugin
---

# grain_cloud: sintesi granulare in un plugin

Un granulatore compilato da Pure Data: densità variabile,
finestre selezionabili e dispersione controllata.

<!-- more -->

## Architettura

Il granulatore usa un pool di 32 voci. Ogni grano viene
attivato da un metro interno la cui frequenza determina la
densità della nuvola.

```
metro → trigger → voice allocator → grain reader → envelope → out
```

## Scelta della finestra

Quattro tipi di finestra, ciascuno con un carattere diverso:

=== "Hann"

    Morbida, universale. La scelta sicura.

=== "Triangolare"

    Più attacco percepibile. Buona per materiale ritmico.

=== "Trapezoidale"

    Plateau stabile al centro. Ideale per droni.

=== "Gaussiana"

    Massima fusione tra grani. Texture lisce.

## Limitazioni

Il buffer è statico — `[soundfiler]` non è supportato da hvcc.
Il campione va convertito in un array di costanti nel codice C
generato, oppure caricato dall'host via parametro.
