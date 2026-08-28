# Research-agent — Job Search & Adaptive CV Assistant

## Obiettivo

Aiutare l'utente (pepefilippo96@gmail.com) a trovare una nuova posizione lavorativa
nell'ambito **Data/AI**, in **Italia o all'estero**, producendo per ogni annuncio
selezionato un **CV su misura in inglese**, senza mai inventare esperienze,
competenze o risultati non presenti nella fonte di verità dell'utente.// e mostrando
tutto in una **dashboard HTML locale**.

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

## Struttura del repository

```
data/
  profile/
    source/            # file originali caricati dall'utente (CV, export LinkedIn, ecc.)
    profile.yaml        # profilo strutturato: esperienze, skill, formazione, risultati
    goals.yaml           # obiettivi economici (RAL minima/target, benefit) e personali
                          # (valori, cultura aziendale, temi da evitare)
  jobs/
    jobs.json            # elenco annunci selezionati + metadati + stato + link CV
                          # incl. punteggio match tecnico + esito verifica economica/mission
cv/
  base/
    template.md          # template CV master in inglese
  generated/
    <job-slug>/
      cv.md               # CV su misura in markdown
      cv.pdf              # CV esportato (se richiesto)
web/
  index.html               # dashboard locale (apribile via browser)
  data.js / jobs.json      # dati letti dalla dashboard
scripts/
  (eventuali script di supporto: export PDF, rigenerazione dashboard)
CLAUDE.md
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
`goals.yaml`.

**Fase 3 — Ricerca annunci**
Ricerca (on-demand, su richiesta dell'utente) di annunci compatibili su bacheche
e siti aziendali (scelti di volta in volta in base a pertinenza), valutazione
di compatibilità tecnica col profilo, salvataggio in `jobs.json` di: titolo,
azienda, link annuncio, data, requisiti chiave, punteggio di match tecnico.

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
pertinenti alla job description. Output in `cv/generated/<job-slug>/cv.md`
(+ PDF se richiesto).

**Fase 6 — Dashboard locale**
Generazione/aggiornamento di `web/index.html`: pagina statica consultabile in
locale (apertura diretta nel browser) che elenca i job selezionati con link
all'annuncio originale, link al CV corrispondente, esito match tecnico ed
economico/mission, filtrabile per stato.

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
- [ ] Fase 1 — Attesa CV/profilo e obiettivi economici/personali dall'utente
- [ ] Fase 2 — Criteri di ricerca
- [ ] Fase 3 — Prima ricerca annunci
- [ ] Fase 4 — Prima verifica compatibilità economica/mission
- [ ] Fase 5 — Primi CV generati
- [ ] Fase 6 — Prima versione dashboard
