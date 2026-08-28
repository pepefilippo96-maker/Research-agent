# Workflow di esecuzione — Ricerca lavoro & generazione CV

Questo file descrive **come e quando** si attiva il processo operativo di ricerca
(Fasi 3-7 di `CLAUDE.md`). Le regole non negoziabili e la struttura dei dati restano
definite in `CLAUDE.md`; qui è documentato il comportamento da seguire ad ogni
esecuzione.

## Regola di attivazione

Il processo di ricerca **non parte mai in autonomia**. Si avvia solo quando
l'utente lo richiede esplicitamente (es. "avvia la ricerca", "cerca nuovi
annunci", "esegui la ricerca"). Un aggiornamento del profilo, degli obiettivi
o della dashboard, da solo, non deve far scattare una nuova ricerca.

## Step 0 — Domande preliminari (obbligatorie ad ogni esecuzione)

**Prima di tutto, sincronizzazione dashboard → jobs.json.** Dalla dashboard
l'utente può scartare o ripristinare un annuncio con un click in qualsiasi
momento, anche fuori da un'esecuzione dell'agente (la pagina pubblica da sola
la modifica — vedi Fase 6 in `CLAUDE.md`). Prima di procedere, rileggere la
dashboard pubblicata (stesso link stabile) e riportare eventuali cambi di
stato (`scartato` / ripristinato a `da valutare`) in `data/jobs/jobs.json`,
così una ricerca successiva non ripropone né sovrascrive per errore una
decisione già presa dall'utente sulla pagina.

Poi, prima di iniziare qualunque ricerca, porre sempre alcune domande di verifica —
anche se già risposte in sessioni precedenti — per intercettare eventuali
cambiamenti negli obiettivi:

1. I ruoli/seniority target definiti in Fase 2 sono ancora validi, o sono
   cambiati?
2. La location/modalità di lavoro (remoto, paese, città) è cambiata?
3. Gli obiettivi economici in `goals.yaml` (RAL minima/target, benefit
   irrinunciabili) sono ancora validi?
4. Gli obiettivi personali/valori (mission, cultura aziendale, temi/settori
   da evitare) sono cambiati?
5. Ci sono aziende, bacheche o siti specifici da includere/escludere per
   questa ricerca?
6. Ambito della ricerca: quante posizioni indicativamente o quanto tempo/
   sforzo dedicare a questa sessione?

Se l'utente conferma che nulla è cambiato, si procede rapidamente riepilogando
i criteri correnti. Se qualcosa è cambiato, si aggiornano prima
`profile.yaml` / `goals.yaml` / i criteri in `jobs.json`, e solo dopo si
procede.

## Nota tecnica — accesso al testo degli annunci

In questo ambiente l'accesso diretto a pagine web esterne tramite gli
strumenti nativi (WebFetch) è bloccato da una policy di rete a livello di
ambiente, non aggirabile dall'agente. Dal 29/08/2026 è però collegato il
connettore **Tavily** (`tavily_search` con filtro data reale, `tavily_extract`
per leggere il testo integrale di una pagina), che passa per l'infrastruttura
connettori e non è soggetto a quel blocco. Di conseguenza la Fase 3 funziona
così:

1. **Ricerca con filtro data** (`tavily_search`, `start_date`/`time_range`)
   per proporre lead il più possibile recenti — non solo snippet, ma con
   possibilità di aprirli.
2. **Verifica obbligatoria prima di aggiungere un lead alla shortlist**:
   l'agente usa `tavily_extract` per leggere il testo reale della pagina e
   controllare segnali espliciti di chiusura ("this job has expired", "no
   longer available", "position has been filled", 404, ecc. — anche in
   italiano). Un lead con questi segnali viene marcato `scartato` con il
   motivo verificato, mai aggiunto come "da valutare".
3. Sul testo reale così ottenuto l'agente calcola direttamente il match
   tecnico e la verifica economica/mission (Fase 3b/Fase 4 eseguite
   dall'agente, non più solo dall'utente).
4. **Fallback**: se Tavily non riesce a leggere una pagina (aggregatore che
   blocca l'estrazione, sito non supportato) resta valido il metodo
   precedente — l'utente apre il link e incolla il testo in chat.

Esperienza diretta (29/08/2026): su 10 annunci trovati tramite semplice
ricerca web (senza verifica) e poi controllati con `tavily_extract`, **8
risultavano scaduti, chiusi o rimossi** — la verifica prima di aggiungere un
lead alla shortlist non è opzionale, è la parte che rende la ricerca
affidabile.

Un lead per cui l'utente non fornisce il testo resta in dashboard con stato
"da valutare" e un punteggio di match non calcolato (`null`), mai stimato o
inventato dai soli snippet di ricerca — anche perché i titoli restituiti
dalla ricerca web spesso non coincidono con quelli reali della pagina.

## Step 1 — Presentazione del piano

Prima di eseguire ricerche effettive (web search, fetch di annunci/siti
aziendali), mostrare sempre all'utente un piano sintetico con:

- i criteri di ricerca che verranno usati (ruolo, seniority, location,
  vincoli economici/valoriali);
- le fonti/bacheche/siti aziendali che si intende consultare;
- il numero indicativo di annunci target per questa sessione;
- come verranno valutati gli annunci (match tecnico in Fase 3, compatibilità
  economica/mission in Fase 4).

Si procede solo dopo conferma esplicita dell'utente sul piano (a meno che
l'utente non l'abbia già data nel messaggio di attivazione, es. "cerca e
vai pure").

## Step 2 — Esecuzione

Una volta confermato il piano, si eseguono in sequenza le fasi operative
definite in `CLAUDE.md` (vedi la Nota tecnica sopra per il perché della
Fase 3 in due passaggi):

1. **Fase 3a — Shortlist**: ricerca (Tavily, con filtro data) per proporre
   lead plausibili e recenti (azienda, ruolo, link).
2. **Fase 3b — Verifica e match**: prima di salvare un lead come `da
   valutare` in `data/jobs/jobs.json`, l'agente ne legge il testo reale
   (`tavily_extract`) per escludere annunci scaduti/chiusi/rimossi, e su
   quel testo calcola direttamente il match tecnico. Solo se Tavily non
   riesce a leggere la pagina si ricade sul metodo precedente: l'utente
   apre il link e incolla in chat il testo dell'annuncio.
3. **Fase 4 — Verifica compatibilità economica e mission**: solo per i lead
   con JD confermata in 3b, confronto con `goals.yaml`, esito salvato per
   annuncio (`economic_fit`, `mission_fit`, `notes`).
4. **Fase 5 — Generazione CV su misura**: solo per annunci approvati
   dall'utente (o che superano la Fase 4), a partire esclusivamente da
   `profile.yaml`. Ogni CV viene anche inviato direttamente in chat non
   appena generato.
5. **Fase 6 — Aggiornamento dashboard**: la pagina viene ripubblicata sullo
   **stesso link stabile** di sempre (non ne viene creato uno nuovo ad ogni
   esecuzione), con i nuovi annunci (anche quelli ancora in Fase 3a, come
   shortlist da confermare) e i CV correnti incorporati nella pagina stessa.
6. **Fase 7 — Riepilogo**: sintesi finale dei risultati mostrata all'utente
   (lead trovati in 3a, quanti confermati con JD in 3b, quanti hanno
   superato Fase 4, quanti CV generati) **più il link alla dashboard
   aggiornata**, fornito esplicitamente al termine di ogni esecuzione.

Il workflow **termina sempre qui**: la preparazione di annuncio + CV in
dashboard è l'ultimo passo automatico. Nessuna fase invia candidature, email o
il CV a nome dell'utente — la scelta di candidarsi, dopo aver confrontato CV e
job description nella dashboard, è sempre e solo dell'utente.

## Note operative

- Ogni esecuzione è **incrementale**: non sovrascrive gli annunci/CV già
  presenti in `jobs.json`, aggiunge solo i nuovi o aggiorna quelli esplicitamente
  ricontrollati.
- Le regole non negoziabili definite in `CLAUDE.md` (fonte di verità unica,
  niente invenzioni, tracciabilità, **nessuna candidatura automatica in nessun
  caso**) restano valide in ogni esecuzione di questo workflow.
