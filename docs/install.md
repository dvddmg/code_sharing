# Installazione

## Download binari

I plugin precompilati per macOS (Apple Silicon) sono disponibili
nella pagina [Releases](https://github.com/dvddmg/spat_plugin/releases/latest)
su GitHub.

1. scarica il `.zip` della release
2. decomprimi
3. copia il file `.vst3` o `.component` nella cartella appropriata

### Percorsi di installazione

=== "VST3"

    ```
    ~/Library/Audio/Plug-Ins/VST3/
    ```

=== "AU"

    ```
    ~/Library/Audio/Plug-Ins/Components/
    ```

!!! note "Gatekeeper"
    Al primo avvio macOS potrebbe bloccare il plugin. Per sbloccarlo:

    ```bash
    xattr -dr com.apple.quarantine ~/Library/Audio/Plug-Ins/VST3/nome_plugin.vst3
    ```

## Compilazione da sorgente

### Requisiti

- macOS con Xcode Command Line Tools
- Python 3.10+
- [hvcc](https://github.com/Wasted-Audio/hvcc)

### Procedura

```bash
# clona con submodule
git clone --recursive https://github.com/dvddmg/spat_plugin.git
cd spat_plugin

# installa hvcc
pip install hvcc

# compila un plugin
./build.sh spat_8

# il plugin compilato si trova in:
# build/spat_8/bin/spat_8.vst3
```

!!! tip "Script di build"
    `./build.sh` accetta il nome del plugin come argomento.
    Senza argomenti compila tutti i plugin.
