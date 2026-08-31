# software-clonify

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude_Code-skill-d97757)](https://docs.anthropic.com/en/docs/claude-code)

**software-clonify** è una skill orchestratrice per Claude Code che ti permette di costruire la replica di un software esistente coordinando quattro strumenti in sequenza: `/grilling` per l'intervista iniziale, `/deep-research` per la ricerca, `/goal` per la costruzione e `/verify-agent` per la verifica finale. La skill non esegue il lavoro direttamente: prepara l'input di ogni strumento, ne registra l'output su disco e fa avanzare il processo una fase alla volta, chiedendo conferma a ogni passaggio.

---

## Prerequisiti

Claude Code installato. Gli strumenti orchestrati vengono rilevati a runtime e ognuno dispone di un'alternativa definita: l'assenza di uno di essi non interrompe il processo.

**[grilling](https://github.com/mattpocock/skills)** — utilizzata nella fase 1. Se non è installata, la skill applica la copia integrata ([`references/grilling.md`](skills/clone-software/references/grilling.md)), riprodotta verbatim dall'originale di Matt Pocock (licenza MIT): il comportamento dell'intervista è identico in entrambi i casi.

**`/deep-research` e `/goal`** — comandi nativi di Claude Code. Se `/deep-research` non è disponibile nella versione in uso, la fase 2 procede con una ricerca web mirata; `/goal` viene proposto, ma la costruzione può procedere anche senza.

**[verify-agent](https://github.com/DarioFontanel/verify-agent)** — utilizzata nella fase 4, facoltativa. Viene installata al momento, previo consenso, se accetti la verifica. Richiede Codex CLI con login ChatGPT.

---

## Installazione

1. Clona la repository — `git clone https://github.com/DarioFontanel/software-clonify.git`
2. Copia la cartella della skill tra le skill utente di Claude Code:

```bash
# macOS / Linux
cp -r software-clonify/skills/clone-software ~/.claude/skills/

# Windows (PowerShell)
Copy-Item -Recurse software-clonify\skills\clone-software $env:USERPROFILE\.claude\skills\
```

3. Riavvia Claude Code: le skill utente vengono caricate all'avvio della sessione.

Verifica: digita `/clone-software` — la skill compare tra quelle disponibili. Si attiva anche in linguaggio naturale, quando descrivi l'obiettivo senza nominarla: "voglio clonare Loom", "rifammi una versione locale di Superhuman".

Per installarla su un singolo progetto, copia la stessa cartella in `.claude/skills/` all'interno del progetto. Per aggiornarla, esegui `git pull` nella repository clonata e ripeti il passo 2. Per rimuoverla, elimina `~/.claude/skills/clone-software`.

---

## La pipeline

```
🎤 /grilling ──▶ 🧪 /deep-research ──▶ 🔨 /goal ──▶ ✅ /verify-agent
   intervista      ricerca che          costruzione    verifica
                   falsifica            fino ai        adversariale
                   + riconciliazione    criteri        (opzionale)
```

Ogni fase si conclude con un punto di controllo: la skill produce il risultato, lo presenta, chiede conferma e attende la risposta prima di procedere. In caso di risposta negativa, torna alla fase precedente e la ripete.

Due strumenti vengono invocati direttamente dalla skill, due richiedono un'azione dell'utente. `grilling` e `verify-agent` sono skill, e Claude le esegue in autonomia; `/deep-research` e `/goal` sono comandi nativi che una skill non può lanciare. Quando il processo li raggiunge, la skill compone il comando completo, lo presenta in un blocco pronto da copiare e attende che tu lo esegua — nella stessa sessione, come messaggio successivo: il contesto della conversazione rimane disponibile e il processo riprende dal punto in cui si era fermato.

Lo stato del progetto risiede in due file nella cartella `./clone/`, esclusa da git: `DECISIONI.md` — il contratto, con requisiti, decisioni numerate, divergenze e ambito escluso — e `RICERCA.md` — i fatti raccolti sull'originale, con la provenienza di ciascuno. Se `DECISIONI.md` esiste già, alla successiva invocazione la skill lo rilegge e riprende dalla prima fase non completata.

### 🎤 Fase 1 — Intervista con `/grilling`

La skill invoca `grilling` — o la copia integrata, identica, se non è installata — e modella il progetto come un albero di decisioni, procedendo per round: a ogni round pone tutte le domande i cui prerequisiti sono già stati risolti, numerate e accompagnate da una raccomandazione motivata. Le risposte sbloccano il round successivo. La prima decisione è la piattaforma, che vincola tutte le altre; al primo round appartiene anche la calibrazione tra demo e prodotto, che determina i valori predefiniti di ogni scelta successiva.

I fatti tecnici — versione di Node, browser disponibili, contenuto della cartella — vengono ricavati direttamente dall'ambiente; a te restano soltanto le decisioni che dipendono da preferenze o vincoli personali. Quando una scelta ne preclude un'altra, la skill lo segnala nel momento in cui la decisione viene presa. L'intervista termina soltanto quando non restano domande aperte e tu confermi.

Il risultato è `clone/DECISIONI.md`, il contratto del progetto, riutilizzato senza modifiche nella fase 4.

### 🧪 Fase 2 — Ricerca con `/deep-research`

Le decisioni della fase 1 si basano su assunzioni, e su un software reale le assunzioni si rivelano spesso errate. La skill compone una domanda di ricerca autosufficiente che elenca le proprie assunzioni numerate e chiede esplicitamente quali siano false: una ricerca impostata per confermare restituirebbe soltanto conferme. La domanda copre lo stile visivo con la provenienza di ogni valore, il comportamento reale dell'originale funzione per funzione, i vincoli della piattaforma, lo stato delle librerie previste e le repliche open source esistenti.

La skill presenta quindi il comando `/deep-research` pronto da copiare, indicando in anticipo il costo: l'esecuzione richiede diversi minuti e un consumo di token considerevole. Se il progetto è di dimensioni ridotte o preferisci evitare il costo, l'alternativa è una ricerca web mirata condotta da Claude su fonti primarie: più rapida, ma con affermazioni non verificate, annotate come tali.

Ottenuto il report, segue la riconciliazione: la skill dichiara apertamente quali assunzioni erano errate, riapre soltanto le decisioni invalidate — una alla volta, con una nuova raccomandazione — e registra in `DECISIONI.md` le divergenze, ovvero i punti in cui la replica si discosterà deliberatamente dall'originale.

### 🔨 Fase 3 — Costruzione con `/goal`

La skill deriva dai criteri di accettazione una condizione di completamento verificabile e presenta il comando `/goal` pronto da copiare: la costruzione prosegue, turno dopo turno, finché la condizione non risulta soddisfatta.

Il lavoro procede in ordine di rischio, a partire dai componenti il cui funzionamento non è ancora certo, e le firme delle API vengono lette dai tipi installati anziché ricostruite a memoria. Il principio che decide la riuscita del progetto: **la replica si giudica sull'artefatto prodotto, non sull'anteprima**. Un'applicazione può apparire corretta durante l'uso e generare comunque un file difettoso; per questo la skill esegue il software sul suo caso d'uso reale e ne ispeziona l'output con uno strumento esterno, campionando l'intera estensione dell'artefatto. Le procedure per ciascun tipo di artefatto — video, interfacce web, CLI, dati, API — sono raccolte in [`verificare-l-artefatto.md`](skills/clone-software/references/verificare-l-artefatto.md).

### ✅ Fase 4 — Verifica con `/verify-agent` (opzionale)

Fase facoltativa: la skill propone una verifica adversariale cross-model e, in caso di consenso, installa [verify-agent](https://github.com/DarioFontanel/verify-agent) e la esegue utilizzando `DECISIONI.md` come brief, divergenze incluse — un reviewer che non le conoscesse segnalerebbe come difetti proprio le scelte deliberate, riempiendo la verifica di falsi positivi.

Il risultato è il verdetto, l'elenco dei finding ordinati per severità e le riserve rimaste aperte.

---

Designed by **[Dario Fontanel, PhD](https://dariofontanel.com/)**

*Aiuto PMI italiane ad integrare l'intelligenza artificiale per automatizzare i lavori ripetitivi, abbattere i costi e guadagnare tempo per crescere.*

[![Sito](https://img.shields.io/badge/Sito-dariofontanel.com-4285F4?style=flat&logo=googlechrome&logoColor=white)](https://dariofontanel.com/)
[![YouTube](https://img.shields.io/badge/YouTube-FF0000?style=flat&logo=youtube&logoColor=white)](https://www.youtube.com/@dariofontanel)
[![Instagram](https://img.shields.io/badge/Instagram-E4405F?style=flat&logo=instagram&logoColor=white)](https://www.instagram.com/dariofontanel.ai/)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=flat&logo=data%3Aimage%2Fsvg%2Bxml%3Bbase64%2CPHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI%2BPHBhdGggZmlsbD0id2hpdGUiIGQ9Ik0yMC40NDcgMjAuNDUyaC0zLjU1NHYtNS41NjljMC0xLjMyOC0uMDI3LTMuMDM3LTEuODUyLTMuMDM3LTEuODUzIDAtMi4xMzYgMS40NDUtMi4xMzYgMi45Mzl2NS42NjdIOS4zNTFWOWgzLjQxNHYxLjU2MWguMDQ2Yy40NzctLjkgMS42MzctMS44NSAzLjM3LTEuODUgMy42MDEgMCA0LjI2NyAyLjM3IDQuMjY3IDUuNDU1djYuMjg2ek01LjMzNyA3LjQzM2MtMS4xNDQgMC0yLjA2My0uOTI2LTIuMDYzLTIuMDY1IDAtMS4xMzguOTItMi4wNjMgMi4wNjMtMi4wNjMgMS4xNCAwIDIuMDY0LjkyNSAyLjA2NCAyLjA2MyAwIDEuMTM5LS45MjUgMi4wNjUtMi4wNjQgMi4wNjV6bTEuNzgyIDEzLjAxOUgzLjU1NVY5aDMuNTY0djExLjQ1MnpNMjIuMjI1IDBIMS43NzFDLjc5MiAwIDAgLjc3NCAwIDEuNzI5djIwLjU0MkMwIDIzLjIyNy43OTIgMjQgMS43NzEgMjRoMjAuNDUxQzIzLjIgMjQgMjQgMjMuMjI3IDI0IDIyLjI3MVYxLjcyOUMyNCAuNzc0IDIzLjIgMCAyMi4yMjUgMHoiLz48L3N2Zz4%3D)](https://www.linkedin.com/in/dario-fontanel/)
[![TikTok](https://img.shields.io/badge/TikTok-000000?style=flat&logo=tiktok&logoColor=white)](https://www.tiktok.com/@dario.fontanel)
[![AI Academy](https://img.shields.io/badge/AI_Academy-E7514F?style=flat&logo=data%3Aimage%2Fsvg%2Bxml%3Bbase64%2CPHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI%2BPHBhdGggZmlsbD0id2hpdGUiIGQ9Ik0xMiAzIDEgOWwxMSA2IDktNC45MVYxN2gyVjlMMTIgM3pNNSAxMy4xOFYxN2MwIDEuNjYgMy4xMyAzIDcgM3M3LTEuMzQgNy0zdi0zLjgybC03IDMuODItNy0zLjgyeiIvPjwvc3ZnPg%3D%3D)](https://www.skool.com/ai-academy-2306)

Licenza [MIT](./LICENSE).
