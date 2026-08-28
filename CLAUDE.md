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
5. **Nessuna candidatura automatica**: l'agente prepara annunci e CV, ma non invia
   candidature senza conferma esplicita dell'utente.

## Struttura del repository

```
data/
  profile/
    source/            # file originali caricati dall'utente (CV, export LinkedIn, ecc.)
    profile.yaml        # profilo strutturato: esperienze, skill, formazione, risultati
  jobs/
    jobs.json            # elenco annunci selezionati + metadati + stato + link CV
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

**Fase 0 — Setup (fatto)**
Creazione struttura repo e di questo file.

**Fase 1 — Profilo strutturato**
L'utente carica il CV/LinkedIn esistente. Viene estratto in
`data/profile/profile.yaml`: esperienze (ruolo, azienda, periodo, responsabilità,
risultati misurabili), competenze tecniche, formazione, certificazioni, lingue.
Questo file diventa l'unica fonte ammessa per generare i CV.

**Fase 2 — Criteri di ricerca**
Definizione con l'utente di: ruoli target, seniority, località/remote,
settori/aziende da includere o escludere, eventuali requisiti non negoziabili
(es. visto, contratto).

**Fase 3 — Ricerca annunci**
Ricerca (on-demand, su richiesta dell'utente) di annunci compatibili su bacheche
e siti aziendali (scelti di volta in volta in base a pertinenza), valutazione
di compatibilità col profilo, salvataggio in `jobs.json` di: titolo, azienda,
link annuncio, data, requisiti chiave, punteggio di match, stato
(da valutare / CV pronto / candidato / scartato).

**Fase 4 — Generazione CV su misura**
Per ogni job approvato dall'utente, generazione di un CV in inglese a partire
esclusivamente da `profile.yaml`, con enfasi sulle esperienze/skill pertinenti
alla job description. Output in `cv/generated/<job-slug>/cv.md` (+ PDF se richiesto).

**Fase 5 — Dashboard locale**
Generazione/aggiornamento di `web/index.html`: pagina statica consultabile in
locale (apertura diretta nel browser) che elenca i job selezionati con link
all'annuncio originale e link al CV corrispondente, filtrabile per stato.

**Fase 6 — Iterazione**
Ogni nuova ricerca o aggiornamento del profilo aggiorna `jobs.json` e la
dashboard in modo coerente. Automazione della ricerca periodica valutabile
in futuro su richiesta esplicita dell'utente.

## Stato attuale del progetto

- [x] Fase 0 — Setup e CLAUDE.md
- [ ] Fase 1 — Attesa CV/profilo dall'utente
- [ ] Fase 2 — Criteri di ricerca
- [ ] Fase 3 — Prima ricerca annunci
- [ ] Fase 4 — Primi CV generati
- [ ] Fase 5 — Prima versione dashboard
