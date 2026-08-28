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
definite in `CLAUDE.md`:

1. **Fase 3 — Ricerca annunci**: raccolta annunci compatibili, calcolo match
   tecnico, salvataggio in `data/jobs/jobs.json`.
2. **Fase 4 — Verifica compatibilità economica e mission**: confronto con
   `goals.yaml`, esito salvato per annuncio (`economic_fit`, `mission_fit`,
   `notes`).
3. **Fase 5 — Generazione CV su misura**: solo per annunci approvati
   dall'utente (o che superano la Fase 4), a partire esclusivamente da
   `profile.yaml`.
4. **Fase 6 — Aggiornamento dashboard locale**: rigenerazione di
   `web/index.html` con i nuovi annunci/CV.
5. **Fase 7 — Riepilogo**: sintesi finale dei risultati mostrata all'utente
   (numero annunci trovati, quanti hanno superato Fase 4, quanti CV generati),
   invitando l'utente ad aprire la dashboard per il controllo finale.

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
