# Research-agent — Job Search & Adaptive CV Assistant

## Obiettivo

Aiutare l'utente (pepefilippo96@gmail.com) a trovare una nuova posizione lavorativa
nell'ambito **Data/AI**, con sede di lavoro preferibilmente **nel milanese e
dintorni** (dettaglio completo dei criteri in `data/profile/search_criteria.yaml`,
locale), producendo per ogni annuncio
selezionato un **CV su misura in inglese**, senza mai inventare esperienze,
competenze o risultati non presenti nella fonte di verità dell'utente.// e mostrando
tutto in una **dashboard web privata**, pubblicata su un link stabile (lo stesso
ad ogni esecuzione) e visibile solo all'utente.

## Regole non negoziabili

1. **Fonte di verità unica**: `data/profile/profile.yaml` (derivato dal CV/LinkedIn
   che l'utente carica). Ogni CV generato deve poter essere ricondotto a una voce
   di questo file. Non aggiungere mai skill, ruoli, tool, certificazioni o metriche
   non presenti nella fonte.
2. **Adattamento, non invenzione**: per ogni job description si può riordinare,
   riformulare, enfatizzare o sintetizzare il contenuto esistente — mai crearne di
   nuovo. Se un requisito della JD non è coperto dal profilo, non va forzato nel CV.
3. **Tracciabilità**: ogni CV generato è collegato a un annuncio specifico tramite
   `data/jobs/jobs.json`, con link all'annuncio originale e percorso del CV.
4. **Lingua**: CV sempre in inglese. Le note di lavoro/interazione con l'utente in
   italiano.
5. **Nessuna candidatura automatica**: l'agente prepara annunci e CV, ma **non invia
   mai, in nessun caso e per nessun motivo, una candidatura, un'email o il CV per
   conto dell'utente**. L'invio è sempre e solo un'azione manuale dell'utente,
   fatta fuori dall'agente. Il ruolo dell'agente si ferma alla preparazione:
   annuncio + CV pronti in dashboard, la decisione e l'azione di candidarsi
   restano interamente all'utente, che le compie dalla dashboard dopo aver
   controllato CV e job description.
6. **Compatibilità obiettivi personali/economici**: nessun annuncio passa alla
   generazione del CV senza prima essere valutato rispetto a `data/profile/goals.yaml`
   (obiettivi economici e valori personali dell'utente) e alla mission/cultura
   dell'azienda. Annunci sotto i requisiti economici non negoziabili vengono
   segnalati o scartati, non forzati a "CV pronto".

## Privacy e distribuzione dei dati

Il repository GitHub di questo progetto è **pubblico**. Tutti i file che
contengono dati personali dell'utente restano quindi **solo in locale**,
esclusi dal repository via `.gitignore` (mai committati né pushati):
`data/profile/profile.yaml`, `data/profile/goals.yaml`,
`data/profile/search_criteria.yaml`, `data/profile/source/*`,
`data/jobs/jobs.json`, `cv/generated/*`. Su GitHub
resta solo la struttura/il codice del progetto, senza dati personali.

Essendo questa un'esecuzione in un ambiente remoto effimero, i file "solo
locali" **non sono persistenti tra una sessione e l'altra**: se il container
viene rilasciato, vanno ricaricati/rigenerati. Se in futuro l'utente vorrà
persistenza tra sessioni dovrà valutare di rendere il repository privato.

**Distribuzione dei risultati**: poiché i dati non vivono in un percorso
locale raggiungibile dall'utente, la dashboard (Fase 6) viene pubblicata come
pagina web privata (link stabile, aggiornato ad ogni esecuzione — vedi
`workflow.md`), e ogni CV generato (Fase 5) viene anche inviato direttamente
in chat all'utente, oltre a essere mostrato per intero all'interno della
dashboard stessa (non come link a un file locale, che non sarebbe
raggiungibile dalla pagina pubblicata).

## Struttura del repository

```
data/
  profile/
    source/            # file originali caricati dall'utente (CV, export LinkedIn, ecc.)
    profile.yaml        # profilo strutturato: esperienze, skill, formazione, risultati
    goals.yaml           # obiettivi economici (RAL minima/target, benefit) e personali
                          # (valori, cultura aziendale, temi da evitare)
    search_criteria.yaml  # criteri di ricerca: ruoli, seniority, aziende, location, RAL min
  jobs/
    jobs.json            # elenco annunci selezionati + metadati + stato + link CV
                          # incl. punteggio match tecnico + esito verifica economica/mission
cv/
  base/
    template.md          # template CV master in inglese
  generated/
    <job-slug>/
      cv.md               # CV su misura in markdown (locale, non committato)
      cv.pdf              # CV esportato (se richiesto)
web/
  index.html               # template della dashboard, dati embedded come JSON
                              # inline (array vuoto nel repo); i dati reali
                              # vivono solo in data/jobs/jobs.json (locale) e
                              # vengono incorporati nella pagina pubblicata come
                              # Artifact. La pagina ha una capacità `artifact`
                              # che le permette di ripubblicare da sola una
                              # nuova versione di sé stessa quando l'utente
                              # scarta/ripristina un annuncio dalla dashboard.
scripts/
  (eventuali script di supporto: export PDF, rigenerazione dashboard)
CLAUDE.md
workflow.md
.gitignore                   # esclude tutti i file con dati personali (vedi sopra)
```

## Fasi operative

> Per **quando e come** si attiva concretamente il processo di ricerca (Fasi 3-7),
> incluse le domande preliminari obbligatorie e la presentazione del piano prima
> di eseguire, vedi `workflow.md`.

**Fase 0 — Setup (fatto)**
Creazione struttura repo e di questo file.

**Fase 1 — Profilo strutturato**
L'utente carica il CV/LinkedIn esistente. Viene estratto in
`data/profile/profile.yaml`: esperienze (ruolo, azienda, periodo, responsabilità,
risultati misurabili), competenze tecniche, formazione, certificazioni, lingue.
Questo file diventa l'unica fonte ammessa per generare i CV.

Nella stessa fase si raccolgono anche in `data/profile/goals.yaml`:
- **obiettivi economici**: RAL minima accettabile, RAL target, eventuali benefit
  irrinunciabili (es. equity, welfare, ticket, smart working);
- **obiettivi personali/valori**: tipo di cultura aziendale ricercata, temi/mission
  a cui l'utente tiene (es. sostenibilità, impatto sociale, ricerca vs prodotto),
  settori o pratiche aziendali da evitare.

**Fase 2 — Criteri di ricerca**
Definizione con l'utente di: ruoli target, seniority, località/remote,
settori/aziende da includere o escludere, eventuali requisiti non negoziabili
(es. visto, contratto), oltre ai vincoli economici/valoriali già raccolti in
`goals.yaml`. Salvati in `data/profile/search_criteria.yaml`.

**Fase 3 — Ricerca annunci**
Ricerca (on-demand, su richiesta dell'utente) di annunci compatibili su bacheche
e siti aziendali (scelti di volta in volta in base a pertinenza), salvataggio in
`jobs.json` di: titolo, azienda, link annuncio, data, requisiti chiave, punteggio
di match tecnico. WebFetch nativo è bloccato in questo ambiente, ma il
connettore **Tavily** (collegato dall'utente) permette ricerca con filtro data
e lettura del testo integrale delle pagine: ogni lead viene verificato (non
scaduto/chiuso) prima di entrare in `jobs.json`, ed è su quel testo reale che
si calcola subito il match tecnico. Solo se Tavily non riesce a leggere una
pagina si ricade sul metodo precedente (l'utente incolla il testo). Dettagli
in `workflow.md`.

**Fase 4 — Verifica compatibilità economica e mission**
Per ogni annuncio individuato in Fase 3, verifica rispetto a `goals.yaml`:
- **economica**: retribuzione indicata (o stimata/richiesta all'utente se non
  pubblicata) confrontata con la RAL minima/target;
- **mission/valori**: mission, cultura e settore dell'azienda (da annuncio e sito
  aziendale) confrontati con gli obiettivi personali dell'utente.

Esito salvato in `jobs.json` (es. campi `economic_fit`, `mission_fit`, `notes`).
Un annuncio che non supera i requisiti non negoziabili viene marcato come
"scartato" con motivazione, non passa alla generazione del CV senza conferma
esplicita dell'utente.

**Fase 5 — Generazione CV su misura**
Per ogni job approvato dall'utente (che ha superato la Fase 4, o per cui
l'utente conferma comunque di voler procedere), generazione di un CV in inglese
a partire esclusivamente da `profile.yaml`, con enfasi sulle esperienze/skill
pertinenti alla job description. Salvato in locale in
`cv/generated/<job-slug>/cv.md` e **inviato anche direttamente in chat**
all'utente non appena generato.

**Fase 6 — Dashboard**
Pubblicazione/aggiornamento della dashboard come **pagina web privata con link
stabile** (stesso link ad ogni esecuzione, visibile solo all'utente — vedi
`workflow.md` per quando viene fornito/aggiornato). Elenca i job selezionati
con: link all'annuncio originale, CV su misura mostrato per intero all'interno
della pagina stessa (non come link a un file locale), esito match tecnico ed
economico/mission, filtrabile per stato. Dalla dashboard l'utente può anche
**scartare o ripristinare un annuncio con un click**: la pagina salva la
modifica pubblicando da sola una nuova versione di sé stessa (capacità
`artifact` degli Artifact), senza bisogno di un'esecuzione dell'agente. Di
conseguenza la dashboard può diventare più aggiornata di `jobs.json`: prima
di ogni nuova ricerca l'agente rilegge lo stato pubblicato e lo riporta in
`jobs.json` (vedi `workflow.md`, Step 0), così uno scarto fatto dall'utente
non viene mai perso o ripristinato per errore da una ricerca successiva.

La dashboard è il punto in cui **l'utente**, e solo l'utente, confronta CV e
job description e decide se candidarsi. L'agente non invia mai la
candidatura: si limita a presentare annuncio e CV affiancati per la verifica
manuale. Lo stato del job in `jobs.json` riflette questa decisione umana, ad
es.: `da valutare` → `cv pronto` → `approvato dall'utente` → `candidato`
(quest'ultimo aggiornato dall'utente a candidatura avvenuta, mai dall'agente)
oppure `scartato`.

**Fase 7 — Iterazione**
Ogni nuova ricerca o aggiornamento del profilo/obiettivi aggiorna `jobs.json` e
la dashboard in modo coerente. Automazione della ricerca periodica valutabile
in futuro su richiesta esplicita dell'utente.

## Stato attuale del progetto

- [x] Fase 0 — Setup e CLAUDE.md
- [x] Struttura repository applicata (cartelle/file placeholder, dati vuoti)
- [x] Fase 1 — Profilo e obiettivi economici/personali caricati e strutturati
- [x] Fase 2 — Criteri di ricerca
- [x] Fase 3 — Prima ricerca annunci (shortlist trovata via web search; WebFetch bloccato in
      questo ambiente, quindi gli annunci sono marcati "da valutare"/non verificati finché
      l'utente non fornisce il testo delle JD — vedi jobs.json)
- [ ] Fase 4 — Prima verifica compatibilità economica/mission (in attesa dei testi JD)
- [ ] Fase 5 — Primi CV generati
- [x] Fase 6 — Prima versione dashboard pubblicata (link stabile)
