---
date:
  created: 2026-10-06
authors:
  - davide
tags:
  - puredata
  - hvcc
  - dpf
  - plugins
  - dsp
  - vst3
categories:
  - Plugin
---

# Sviluppo plugins in PureData con HVCC e DPF

Framework per creare pluging con PureData.

<!-- more -->

## Introduzione

Questo post mostra come è strutturata e funziona questa <a href="https://github.com/dvddmg/dev_plugin" target="_blank">repository</a> per lo sviluppo di **plugin** partendo da patch scritte in **PureData** o **PlugData**.

L'output finale sono tre plugin per la spazializzazione in formato `VST3`, disponibili per tutti e tre i sistemi operativi più diffusi: **macOS universal**, **Windows 64 bit** [^1], **Linux 64 bit** [^1].

I comandi per la compilazione e parte del codice `C++` sono stati sviluppati con l'ausilio di **Claude Code**. La parte teorica e di DSP è stata sviluppata personalmente.

Questo ambiente è stato scritto e testato su macOS facendo una cross compilazione con ambienti virtuali per Windows e Linux.

[^1]: approfondire il capitolo [cross compilazione](#compilazione-e-cross-compilazione) per maggiori dettagli su compatibilità ed eventuali errori.

## Obbeittivi

L'obbiettivo principale di questo ambiente di sviluppo è quello di creare facilmente dei plugin audio per DAW, nei formati disponbili. Lo stato dell'arte riporta grandi novità a riguardo; infatto l'ambiente che andrò a presentare non è from scratch, ma ha delle dipendenze importanti sopratutto per quanto riguardo la conversione della patch Pd in CPP per compilazione di `DSP` e `GUI`.

## Framework

L'ambiente di lavoro è strutturato come segue

```bash
.
├── README.md           // informazioni sull'ambiente
├── config.sh           // file di configurazione (autore, percorsi dipendenze)
├── new.sh              // comando per creare template nuovo plugin
├── build.sh            // compilazione plugin
├── dep                 // dipendenze e submodules
├── src                 // codice sorgente e patch di tutti i plugin
├── common              // codice C++ in comune (colori UI e funzioni grafiche)
├── templates           // file di partenza nuovo plugin (*.pd, *.json, UI.cpp)
├── docker              // file configurazione per la cross compilaizone in Linux
└── venv                // virtual enviroment python
├── requirements.txt    // registro di tutte le librerie contenute nel VENV python
├── parse_max_pd        // studio conversione da Pd a Max.
```

Le dipendenze del progetto sono le seguenti:

- [Heavy Compiler Collection (hvcc)](https://github.com/Wasted-Audio/hvcc/tree/32483a8fd348e793be0bcbcedd626e6335c61a6c)

    `HVCC` è un compilatore basato su **Python** che genera codice **C/C++** e una varietà di wrapper specifici per framework audio. E' sviluppato per molte <a href="https://github.com/Wasted-Audio/hvcc/blob/develop/docs/getting-started/index.md#supported-platforms" target="_blank">piattaforme</a> e <a href="https://github.com/Wasted-Audio/hvcc/blob/develop/docs/getting-started/index.md#supported-frameworks" target="_blank">frameworks</a>.
    Il concetto principale è che prende patch scritte con **PureData**[^2] e le compila per un determinato sistema operativo con il framework selezionato. Noi useremo `DPF` che permette di esportare plugin in formato `LV2`, `VST`, `VST3`, `CLAP` e `JACK`.

[^2]: **PlugData** viene distribuito con questo compilatore già incluso nel software. A primo impatto è molto utile ma limitato per una configurazione più precisa per grafica, dipedenze o altre necessità.

- [DPF - DISTRHO Plugin Framework](https://github.com/DISTRHO/DPF/tree/4238e1c7f0351bbe488d79f0899c540543ac7583)

    `DPF` è il cuore del progetto, interpreta il codice tradotto da `hvcc` e lo compila in vari formati disponibili elencati sopra. Inoltre crea un'interfaccia grafica di base che viene collegata con la parte di DSP in entrambe le direzioni; questo collegamento avviene grazie ad una `API` in `C++` che lavora scambiando messaggi di `chiave - valore`.

- [DPF Widgets](https://github.com/DISTRHO/DPF-Widgets)

    Infine questa dipendenza può tornare utile per sviluppare `UI` personalizzate partendo da esempi pubblicati online all'interno di questa community. Rimane un submodule necessario per la compilazione e il progetto.

Nella repository è anche inclusa la **Heavy Lib**, una collezione di abstraction per `Pd` scritte dagli autori di `hvcc`.

### HVCC e DPF

Il primo step è creare il virtual enviroment con python, installare i `requirements.txt` e attivare il venv.

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
source venv/bin/activate        // per disattivare lanciare il comando `deactivate`
```

Fatti questi primi step abbiamo a disposizione il comando `hvcc` all'interno del terminale. Questo comando ci permette di convertire una patch in codice C++. Ci sono diverse opzioni che possiamo passare a questo comando per personalizzare la compilazione. <a href="https://github.com/Wasted-Audio/hvcc/tree/32483a8fd348e793be0bcbcedd626e6335c61a6c#usage" target="_blank">comandi</a>.

Il seguente comando indica le informazioni principali che servono a noi per questo progetto.

```bash
hvcc plugin_name.pd -g dpf -m plugin.json -o /output/directory
```

Specifichiamo il nome della patch da compilare, il framwork utilizzato (`dpf`) e la cartella di output dove mettere il file compilato una volta terminato il processo. L'opzione `-m plugin.json` è una personalizzazione disponibile solo se compiliamo da terminale: permette di specificare delle impostazioni aggiuntive che sostiuiscono quelle di default inserite dagli sviluppato di `hvcc`. Il suo contenuto di base può essere il seguente:

```json
{
    "name": "@NAME@",          // nome plugin
    "nosimd": true,            // Single Instruction, Multiple Data: isola problemi di compilazione
    "dpf": {                   // opzioni di DPF
        "dpf_path": "../../dep/",                   // percorso al submodule DPF
        "enable_ui": true,                          // attiva la UI
        "ui_size": {"width": 480, "height": 320},   // grandezza finestra
        "description": "@DESCRIPTION@",             // descrizione
        "maker": "@MAKER@",                         // sviluppatore
        "brand_id": "@BRAND_ID@",                   // codice brand
        "unique_id": "@UNIQUE_ID@",                 // codice univoco (4 cifre)
        "homepage": "@HOMEPAGE@",                   // webpage
        "plugin_uri": "@URI@",                      // webpage plugin
        "version": "1, 0, 0",                       // versione
        "license": "@LICENSE@",                     // licenza
        "midi_input": 0,                            // input midi
        "midi_output": 0,                           // output midi
        "plugin_formats": @FORMATS@                 // array dei formati desiderati
    }
}
```

Questo file viene copiato come default quando si crea un nuovo plugin. In particolare il comando `new.sh $NOME_PLUGIN` avvia un prompt che chiede tutte le informazioni necessarie tra cui il formato. Una volta completato dentro la cartella `/src` vediamo il tempalte base per il nuovo plugin con i file `.pd`, `.json` e la cartella `/ui` in cui vengono renderizzati i parametri con la palette e una disposizione automatica in forma di Knob (vedremo più avanti).

![PureData template](./test-baisc-pd.png){width="100%" align=left}

Un po' come i **MaxForLive**, vediamo qui soopra una patch di default già pronta per la compilazione. Possiamo aggiungere diversi parametri da esporre ad alto livello per il plugin come viene mostrato nell'immagine. Questo plugin è già pronto per essere compilato, lanciando il comando che segue possiamo vederene il risultato

```bash
./build.sh test-basic --install
```

L'opzione `--install` mette in autoamtico il plugin all'interno della cartella specificata nel file `config.sh`.

![VST3 template](./test-basic-vst3.png){width="100%" align=left}

Così è come appare il plugin appena compilato. <a href="https://wasted-audio.github.io/hvcc/latest/getting-started/patching/#exposing-parameters" target="_blank">Qui</a> c'è la documentazion di `hvcc` per la definizione di altri parametri. La documentazione approfondisce anche maggiormente come viene creata l'interfaccia grafica, per questo progetto è stata utilizzata l'API di <a href="https://github.com/ocornut/imgui" target="_blank">ImGui</a>.  E' un sistema interessante che ci permette facilmente di inserire variabili di vario tipo in breve tempo. facendo le seguenti modifiche al plugin `test-basic` otteniamo un'altro tipo di interfaccia.

![PureData template 2](./test-basic-pd-2.0.png){width="100%" align=left}

File **`./src/test-basic/plugin.json`**

```json
{
    "name": "test_basic",
    "nosimd": true,
    "dpf": {
        //...
        "enumerators": {
            "mode": ["0", "1723", "848"]
        }
    }
}
```

File **`./src/test-basic/ui/HeavyDPF_test_basic_UI.cpp`**

```cpp
#include "PluginUIBase.hpp"

START_NAMESPACE_DISTRHO

class PluginUI : public PluginUIBase
{
protected:
    void drawContent() override
    {
        const float pad   = kPad * fZ;
        const float width = getWidth() - 2.0f * pad;

        ImGui::SetCursorPos(ImVec2(pad, pad));
        drawTitle("test-basic", "test e info");

        ImGui::SetCursorPosX(pad);
        sectionHeader("PARAMETERS", width);

        ImGui::SetCursorPosX(pad);
        static const char* const modeItems[] = { "uno", "due", "tre" };

        ImGui::SetCursorPosX(pad);
        comboParam("mode", parammode, modeItems, 3, 140.0f * fZ);

        ImGui::SetCursorPosX(pad);
        knobParam("gain", paramgain, 0.0f, 1.0f, "%.2f");
    }
};

UI* createUI()
{
    return new PluginUI();
}

END_NAMESPACE_DISTRHO
```

Queste modifiche portano al seguente risultato

![Vst3 template 2](./test-basic-vst3-2.0.png){width="100%" align=left}

### Compilazione e Cross compilazione

Per compilare abbiamo a disposizione il comando `./build.sh <nome-plugin> |opzioni|` che con diverse opzioni ci permette di compilare il nostro plugin.

| Opzione | Cosa fa |
|---|---|
| *(nessuna)* o `--native` | compila per il sistema su cui stai lavorando |
| `--universal` | macOS universal: Apple Silicon + Intel (solo da Mac) |
| `--win` | Windows 64 bit: da Mac con MinGW, nativo su Windows |
| `--linux` | Linux 64 bit: da Mac con Docker, nativo su Linux |
| `--all` | `--universal` + `--win` + `--linux`: un solo bundle per i tre sistemi |
| `--install` | copia il bundle nella cartella VST3 indicata in `config.sh` |
| `--clean` | cancella `build*` e `bin` del plugin prima di compilare |

- L'opzione `--wind` ha la dipendenza da `mingw-w64` che è possibile installare con `brew`. Un terminale windows che viene avviato dal comando `build` se specificato come opzione.
- L'opzione `--linux` richiede l'avvio di `Docker` su cui avviamo sempre tramito comando un ambiente virtuale Linux.

## Output

Con questa infrastruttura sono stati scritti i plugin `Orbita`, `Lontananza` e `Diffuser`. 

[DOWNLOAD](https://github.com/dvddmg/dev_plugin/releases/latest)