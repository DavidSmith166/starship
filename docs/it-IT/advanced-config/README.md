# Configurazione Avanzata

Nonostante Starship sia una shell versatile, a volte devi fare qualche modifica in più in `starship.toml` per ottenere alcune cose. Questa pagina descrive alcune tecniche di configurazione avanzate utilizzate in Starship.

::: Attenzione

Le configurazioni in questa sezione sono soggette a modifiche nelle future versioni di Starship.

:::

## Comandi personalizzati di pre-prompt e pre-esecuzione per Bash

Bash non ha un framework preexec/precmd formale come la maggior parte delle altre shell. Per questo motivo, è difficile fornire hook completamente personalizzabile in `bash`. Tuttavia, Starship dà la limitata possibilità di inserire le tue funzioni nella procedura prompt-rendering:

- Per eseguire una funzione personalizzata a destra del prompt prima che venga disegnato, definisci una nuova funzione e assegna il suo nome a `starship_precmd_user_func`. Per esempio, per visualizzare l'icona di un razzo prima del prompt, si può usare il codice seguente

```bash
function blastoff(){
    echo "🚀"
}
starship_precmd_user_func="blastoff"
```

- To run a custom function right before a command runs, you can use the [`DEBUG` trap mechanism](https://jichu4n.com/posts/debug-trap-and-prompt_command-in-bash/). Tuttavia, **devi** intrappolare il segnale DEBUG *prima di* inizializzare Starship! Starship può preservare il valore trappola di DEBUG, ma se la trappola viene sovrascritta dopo l'avvio di Starship, alcune funzionalità non funzioneranno.

```bash
function blastoff(){
    echo "🚀"
}
trap blastoff DEBUG # Trap DEBUG *prima* di eseguire starship
eval $(starship bash)
```

## Cambia il titolo della finestra

Some shell prompts will automatically change the window title for you (e.g. to reflect your working directory). Fish lo fa per impostazione predefinita. Starship non lo fa, ma è abbastanza semplice aggiungere questa funzionalità a `bash` o `zsh`.

Innanzitutto, bisogna definire una funzione per il cambio del titolo della finestra (identica sia per bash che zsh):

```bash
function set_win_title(){
    echo -ne "\033]0; IL_TUO_TITOLO_QUI \007"
}
```

Puoi usare delle variabili per personalizzare questo titolo (`$USER`, `$HOSTNAME`, e `$PWD` sono le scelte più popolari).

In `bash`, impostare questa funzione per essere la precmd Starship function:

```bash
starship_precmd_user_func="set_win_title"
```

In `zsh`, aggiungi questo `precmd_functions` all'array:

```bash
precmd_functions+=(set_win_title)
```

If you like the result, add these lines to your shell configuration file (`~/.bashrc` or `~/.zshrc`) to make it permanent.

Ad esempio, se desideri visualizzare la directory corrente nel titolo della scheda del terminale, aggiungi la seguente snippet al tuo `~/. ashrc` or `~/.zshrc`:

```bash
function set_win_title(){
    echo -ne "\033]0; $(basename "$PWD") \007"
}
starship_precmd_user_func="set_win_title"
```

## Stringhe di stile

Le stringhe di stile sono un elenco di parole, separate da spazi bianchi. Le parole non sono sensibili alle maiuscole (cioè `grassetto` e `BoLd` sono considerate la stessa stringa). Ogni parola può essere una delle seguenti:

  - `bold`
  - `underline`
  - `dimmed`
  - `inverted`
  - `bg:<color>`
  - `fg:<color>`
  - `<color>`
  - `none`

dove `<color>` è un colore specifico (discusso in seguito). `fg:<color>` and `<color>` currently do the same thing, though this may change in the future. `inverted` swaps the background and foreground colors. The order of words in the string does not matter.

Il token `none` sovrascrive tutti gli altri token in una stringa se non fa parte di uno specificatore `bg:`, così ad esempio `fg:red none fg:blue` creerà una stringa senza stile. `bg:none` sets the background to the default color so `fg:red bg:none` is equivalent to `red` or `fg:red` and `bg:green fg:red bg:none` is also equivalent to `fg:red` or `red`. Potrà diventare un errore usare `nessuno` in combinazione con altri token in futuro.

Uno colore specifico può essere uno di questi:

 - Uno dei colori standard del terminale: `nero`, `rosso`, `verde`, `blu`, `giallo`, `viola`, `ciano`, `bianco`. Puoi eventualmente utilizzare il prefisso `bright-` per ottenere la versione luminosa (es. `bright-white`).
 - Un `#` seguito da un valore esadecimale a sei cifre. Questo specifica un [colore esagesimale in RGB](https://www.w3schools.com/colors/colors_hexadecimal.asp).
 - Un numero compreso tra 0-255. Specifica un [codice colore ANSI a 8 bit](https://i.stack.imgur.com/KTSQa.png).

Se sono specificati più colori per il primo piano/sfondo, l'ultimo nella stringa avrà la priorità.
