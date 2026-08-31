---
name: clone-software
description: Clona un software esistente orchestrando quattro strumenti in successione — /grilling per l'intervista, /deep-research per falsificare le assunzioni, /goal per la costruzione, /verify-agent per la verifica adversariale cross-model. Usala quando l'utente vuole clonare, rifare o costruirsi la propria versione di un'applicazione esistente ("voglio clonare Loom", "rifammi Superhuman in locale", "una versione mia di Raycast"), anche quando descrive il risultato senza usare la parola clone.
---

Costruire una **replica** di un software esistente. Questa è una skill
**orchestratrice**: il lavoro di ogni fase lo fa uno strumento dedicato, in
successione obbligata. Il tuo compito è preparare l'ingresso di ogni strumento,
raccoglierne l'uscita su disco, e far avanzare la pipeline un cancello alla
volta.

```
🎤 /grilling ──▶ 🧪 /deep-research ──▶ 🔨 /goal ──▶ ✅ /verify-agent
   intervista      ricerca che          costruzione    verifica
                   falsifica            fino ai        adversariale
                   + riconciliazione    criteri        (opzionale)
```

Ogni fase si chiude con un **cancello**: produci, mostri, chiedi, aspetti che
l'utente risponda. Se chiede di tornare indietro, torna indietro e rifai la
fase precedente.

**Due strumenti li invochi tu, due può lanciarli solo l'utente.** `grilling` e
`verify-agent` sono skill: le invochi direttamente. `/deep-research` e `/goal`
sono comandi built-in di Claude Code che una skill non può eseguire: quando la
pipeline li raggiunge, componi la riga **esatta e completa**, mostrala in un
blocco di codice pronto da incollare, e fermati finché l'utente non l'ha
lanciata. La riga va incollata **in questa stessa sessione**, come messaggio
successivo — niente finestre nuove: conversazione e contratto restano in
contesto, e al ritorno del comando la pipeline riparte da dove si era fermata. Se uno strumento manca, ogni fase dichiara il proprio ripiego — la
pipeline non si ferma mai per un'assenza.

Tre parole guidano tutto il percorso:

- **Fedeltà** — l'asse su cui una replica si giudica. Non "funziona", ma
  "somiglia all'originale nei modi che contano per chi la usa".
- **Divergenza** — un punto in cui la replica si allontana dall'originale *di
  proposito*. Una divergenza non scritta diventa, più tardi, un difetto agli
  occhi di chiunque revisioni il lavoro.
- **Provenienza** — per ogni fatto sull'originale: verificato su fonte primaria,
  oppure ricostruito. Tenerli distinti è ciò che separa una replica da
  un'imitazione a memoria.

## Il contratto su disco

Due file in `./clone/`, creati nella fase 1 e aggiornati lungo il percorso:

- `DECISIONI.md` — requisiti, decisioni numerate, divergenze, fuori scopo.
  È **il contratto**. La fase 4 lo riusa così com'è: scriverlo bene ora rende
  gratuita la verifica dopo.
- `RICERCA.md` — cosa si è scoperto sull'originale, con la provenienza di ogni
  valore.

Alla creazione di `./clone/` assicurati che sia ignorata da git (aggiungi
`clone/` a `.gitignore` se il progetto è un repo): sono artefatti di processo,
non codice.

All'inizio di ogni invocazione, se `./clone/DECISIONI.md` esiste, leggilo e
riprendi dalla fase non completata invece di ricominciare.

---

## Fase 1 · Intervista — `/grilling`

Conduci l'intervista con la skill `grilling`: se è installata, invocala; se
non lo è, applica la sua **copia integrata** —
[`references/grilling.md`](references/grilling.md), verbatim dall'originale di
mattpocock — così il comportamento è identico su qualunque macchina.

Il metodo è il suo: mappa il clone come **albero di design** e lavora a **round
di frontiera** — tutte le domande i cui prerequisiti sono già decisi, numerate
`Q1…Qn` col formato della skill, ognuna con la tua raccomandazione `➡️`; poi
aspetti le risposte, ricalcoli la frontiera e fai il round successivo. I fatti
li cerchi tu (versione di Node, browser presenti, contenuto della cartella); le
decisioni le chiedi.

A quel metodo, il clone aggiunge le sue regole:

- **La radice dell'albero è la piattaforma** — la decisione che vincola tutte
  le altre — e **la calibrazione demo/prodotto sta nel primo round**: cambia il
  default di ogni decisione successiva.
- **Dichiara cosa ogni scelta preclude**, mentre la porta si chiude — non a
  metà costruzione.
- **Ogni risposta apre rami nuovi: percorrili.** Scelta la superficie da
  clonare, ogni sua funzione va interrogata — cosa fa esattamente, cosa succede
  ai bordi, quali stati esistono, cosa persiste e dove, cosa vede l'utente
  quando qualcosa fallisce. Sono le domande che scoprono il lavoro vero.
- Quando l'utente restringe o corregge un requisito, **ritira la domanda mal
  posta** e riformulala nel round successivo.
- Non fermarti quando hai abbastanza per iniziare a costruire — quello è il
  punto in cui l'intervista di solito muore troppo presto. Fermati quando la
  frontiera è vuota e **l'utente conferma** che avete finito.

Scrivi `DECISIONI.md`: richiesta originale (con le parole dell'utente),
requisiti di accettazione, decisioni numerate `D1…Dn`, e fuori scopo.

**Cancello.** Riepiloga le decisioni e chiedi se procedere alla ricerca. Prima
di chiedere, nomina i rami che hai lasciato aperti: se ne resta anche uno, la
frontiera non è vuota — continua a chiedere invece di riepilogare.

**Completo quando** l'utente conferma la comprensione condivisa, ogni requisito
ha un criterio di accettazione osservabile e nessuna decisione presa resta
implicita nel riepilogo.

---

## Fase 2 · Ricerca — `/deep-research`

Lo scopo non è confermare il piano: è **falsificarlo**. Le decisioni della fase
1 poggiano su memoria e intuito, e su un software reale sbagliano spesso.

Componi la domanda di ricerca. Deve essere **autosufficiente** — chi la esegue
non vede la conversazione — e contenere:

1. **Le tue assunzioni, numerate, da abbattere.** Chiedi letteralmente quali
   sono sbagliate. Senza questo la ricerca restituisce conferme.
2. **Lo stile visivo** dell'originale, con provenienza: valori esatti di
   colore, tipografia, geometria, distinguendo fonte primaria da aggregatori.
3. **Il comportamento reale dell'originale**, funzione per funzione, sulla
   documentazione ufficiale.
4. **I vincoli tecnici della piattaforma scelta** e lo stato attuale delle
   librerie previste — quelle deprecate e i loro successori.
5. **Repliche open source esistenti** e come hanno risolto il problema centrale.

Poi mostra la riga pronta da incollare:

```
/deep-research <domanda completa>
```

e dichiara il costo prima che l'utente la lanci: una `/deep-research` vera
consuma milioni di token e diversi minuti. Se il clone è piccolo o l'utente
rifiuta il costo — o il comando non esiste nella sua versione di Claude Code —
il ripiego è la **ricerca web mirata** condotta da te sulla stessa domanda, su
fonti primarie: più rapida, ma le affermazioni restano non verificate e vanno
annotate come tali. Aspetta la scelta dell'utente, e il report se lancia il
comando.

Col report in mano, scrivi `RICERCA.md` separando i fatti verificati dalle
ricostruzioni. Poi la **riconciliazione**, che è ciò che rende utile la
ricerca:

1. **Di' apertamente cosa avevi sbagliato**, in cima e senza attenuazioni. Se
   l'originale non ha una funzione che davi per scontata, o la libreria scelta
   è deprecata, questa è la notizia — non un dettaglio a fondo pagina.
2. **Riapri solo le decisioni che la ricerca ha invalidato**, una alla volta
   con `AskUserQuestion` e la tua raccomandazione. Le altre restano chiuse.
3. **Registra le divergenze** in `DECISIONI.md`: ogni punto in cui la replica
   si allontanerà dall'originale, e perché — quelle imposte dalla piattaforma e
   quelle scelte dall'utente.

**Cancello.** Mostra le correzioni e chiedi conferma prima di costruire.

**Completo quando** ogni assunzione numerata ha ricevuto una risposta —
confermata, smentita, o dichiarata non verificabile — `DECISIONI.md` contiene
una sezione divergenze e nessuna decisione contraddice `RICERCA.md`.

---

## Fase 3 · Costruzione — `/goal`

Deriva dai requisiti di accettazione di `DECISIONI.md` una **condizione di
completamento osservabile** — qualcosa che un valutatore esterno può dichiarare
vera o falsa guardando l'artefatto — e mostra la riga pronta da incollare:

```
/goal <condizione>
```

Spiega in una riga cosa fa: la costruzione prosegue finché la condizione non è
soddisfatta, turno dopo turno. Aspetta che l'utente la lanci (o dica di
procedere senza), poi costruisci.

Costruisci in **ordine di rischio**: per prima la parte che non sai ancora se
funzionerà, perché è l'unica che può invalidare il piano. Verifica le firme
reali delle API che usi leggendo i tipi installati, non andando a memoria.

Poi la regola che decide la riuscita del progetto:

> **La replica si giudica sull'artefatto, non sull'anteprima.**

Un'applicazione che sembra perfetta mentre la usi può produrre un file rotto.
Esegui il software sul suo caso d'uso reale e ispeziona **ciò che produce** con
uno strumento che non condivide le assunzioni del codice, campionando lungo
tutta l'estensione dell'artefatto, non solo all'inizio. Ricette per tipo di
artefatto: [`references/verificare-l-artefatto.md`](references/verificare-l-artefatto.md).

Documenta nel repository la **provenienza** dei valori di stile: quali sono
verificati e quali ricostruiti. Chi leggerà il codice non può distinguerli da
solo.

**Completo quando** ogni requisito di accettazione di `DECISIONI.md` è stato
esercitato sull'artefatto prodotto, e sai dire con quale comando l'hai
constatato.

---

## Fase 4 · Verifica — `/verify-agent` (opzionale)

Fase **opzionale**: proponila con `AskUserQuestion` — verifica adversariale
cross-model sì o no — dichiarando cosa comporta (Codex CLI con login ChatGPT,
qualche minuto). Se l'utente declina, la pipeline è conclusa: chiudi con il
riepilogo di cosa è stato costruito e constatato.

Se accetta, invoca la skill `verify-agent` usando `DECISIONI.md` come brief. **Passa anche le divergenze**: un reviewer che non le conosce segnala
come infedeltà proprio le scelte volute dall'utente, e la verifica affoga in
falsi positivi. Se il progetto non è un repo git, il backend Codex richiede
`--skip-git-repo-check`.

Se la skill non è presente, chiedi all'utente il consenso e installala:

```bash
git clone https://github.com/DarioFontanel/verify-agent /tmp/verify-agent \
  && cp -r /tmp/verify-agent/skills/* ~/.claude/skills/ \
  && rm -rf /tmp/verify-agent
```

Riporta il verdetto, i finding per severità, e le riserve rimaste aperte.

**Completo quando** l'utente ha ricevuto il verdetto, oppure ha declinato la
verifica.
