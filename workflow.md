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

Prima di iniziare qualunque ricerca, porre sempre alcune domande di verifica —
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

In questo ambiente l'accesso diretto a pagine web esterne (apertura autonoma
di un link per leggerne il contenuto) è bloccato da una policy di rete a
livello di ambiente, non aggirabile dall'agente. Di conseguenza la Fase 3 è
strutturata in due passaggi (vedi Step 2):

1. l'agente usa la ricerca web solo per **proporre una shortlist di lead**
   (azienda, ruolo plausibile, link), senza poter leggere il testo integrale
   dell'annuncio;
2. per ogni lead che l'utente vuole approfondire, **l'utente apre il link e
   incolla in chat il testo della job description**. Solo su questo testo
   reale l'agente calcola il match tecnico (Fase 3b), la verifica economica/
   mission (Fase 4) e genera il CV (Fase 5).

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

1. **Fase 3a — Shortlist**: ricerca web per proporre lead plausibili
   (azienda, ruolo, link), salvati in `data/jobs/jobs.json` con
   `technical_match_score: null` e stato `da valutare`.
2. **Fase 3b — Conferma JD**: per ogni lead che l'utente vuole approfondire,
   l'utente incolla in chat il testo dell'annuncio; l'agente aggiorna la
   voce in `jobs.json` (titolo/requisiti reali) e calcola il match tecnico.
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
