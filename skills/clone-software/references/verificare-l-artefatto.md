# Verificare l'artefatto

Reference della fase 4 di [`clone-software`](../SKILL.md). Serve a rispondere a una sola
domanda: **ciò che il software produce è davvero quello che deve produrre?**

## Il principio

Un controllo vale quanto la sua capacità di fallire. "Nessun errore in console"
non è una verifica: è l'assenza di una verifica.

Due regole rendono un controllo reale.

**Usa uno strumento che non condivide le assunzioni del codice.** Se il tuo
codice compone un video e la tua anteprima usa la stessa funzione di
composizione, l'anteprima confermerà qualunque cosa faccia il codice — anche
sbagliata. Un decoder esterno non sa nulla delle tue intenzioni: per questo può
smentirti.

**Misura, non guardare.** Un'ispezione a occhio conferma ciò che ti aspetti di
vedere. Un numero estratto dall'artefatto no. Preferisci sempre una grandezza
confrontabile con una soglia a un'impressione.

## Il campionamento che smaschera

L'errore più comune è verificare un solo punto — il primo frame, la prima riga,
la home page — dove quasi tutto funziona.

Campiona invece lungo **tutta l'estensione** dell'artefatto: inizio, mezzo, fine.
I difetti che contano si manifestano quando qualcosa si esaurisce: un buffer, uno
stream, una sessione, una lista. Un video corretto per due secondi e nero per i
successivi otto passa qualsiasi controllo fatto solo sul primo frame.

## Ricette per tipo di artefatto

### Video e audio

`ffprobe` legge il file senza sapere nulla del codice che l'ha scritto.

```bash
# Il file è ciò che dichiari? Contenitore, codec, dimensioni, durata.
ffprobe -v error -show_entries format=duration \
  -show_entries stream=codec_name,codec_type,width,height,nb_frames \
  -of default=noprint_wrappers=1 output.mp4

# Il contenuto c'è davvero, lungo tutta la durata?
for T in 0 2 5 10; do
  ffmpeg -v error -ss $T -i output.mp4 -frames:v 1 -y /tmp/f.png 2>/dev/null
  echo "t=${T}s luma=$(ffprobe -v error -f lavfi \
    -i "movie=/tmp/f.png,signalstats" \
    -show_entries frame_tags=lavfi.signalstats.YAVG -of csv=p=0 | head -1)"
done

# L'audio è suono o silenzio?
ffprobe -v error -f lavfi -i "amovie=output.mp4,astats=metadata=1:reset=0" \
  -show_entries frame_tags=lavfi.astats.Overall.RMS_level -of csv=p=0 | tail -1
```

Una luminanza che crolla a metà, o un conteggio di frame incoerente con la
durata, sono difetti che nessuna anteprima mostra.

### Interfacce web

Guida un browser vero e **interroga la geometria reale**, perché i difetti di
interazione vivono negli spazi fra gli elementi.

```bash
playwright-cli open --config=cfg.json http://localhost:5173
playwright-cli --raw eval "(() => { const r = document.querySelector(SEL).getBoundingClientRect(); return JSON.stringify(r) })()"
```

Cosa misurare, oltre al fatto che la pagina si apra:

- **Distanze fra elementi che devono essere raggiungibili col mouse.** Un
  pannello che compare al passaggio del puntatore e vive oltre il bordo del suo
  contenitore genera una fascia morta: il puntatore esce, l'evento di uscita
  scatta, il pannello sparisce prima di essere raggiunto. Si vede solo misurando
  i due rettangoli e la loro distanza.
- **Ritaglio da `overflow: hidden`.** Un elemento posizionato fuori dal suo
  contenitore viene renderizzato e tagliato: esiste nel DOM, è invisibile
  all'utente. Confronta i suoi limiti con quelli dell'antenato che ritaglia.
- **Che lo stato cambi davvero dopo l'interazione.** Confronta una grandezza
  prima e dopo il click, non la presenza del pulsante.
- **Percorsi alternativi**: ogni modalità, ogni ramo. Un difetto ama la modalità
  che hai provato meno.

Per dispositivi multimediali servono sorgenti simulate:
`--use-fake-device-for-media-stream`, `--use-fake-ui-for-media-stream`.

### Interfacce a riga di comando

Eseguile sul caso d'uso reale del brief, non su `--help`. Controlla codice di
uscita, forma dell'output, e cosa succede con input assente o malformato.

### Dati e migrazioni

Conteggi prima e dopo, invarianti che devono reggere, e una query di controllo
che riesegue il calcolo da una via diversa da quella del codice.

### API e integrazioni

Chiamate reali, non simulate: forma della risposta, codici di errore, e il
comportamento al superamento dei limiti.

## Quando un controllo trova qualcosa

Il difetto vero è quasi sempre più generale del sintomo che l'ha rivelato.
Prima di correggere, chiediti in quali **altre** condizioni la stessa causa si
manifesta: di norma il sintomo osservato è il caso più visibile, non l'unico.

Poi ripeti la misura che aveva fallito e **allegane il risultato**. Una
correzione senza la sua controprova è una speranza.
