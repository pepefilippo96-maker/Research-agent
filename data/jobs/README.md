# jobs.json — schema

Ogni elemento dell'array rappresenta un annuncio individuato in Fase 3.
Popolato/aggiornato solo durante l'esecuzione del workflow di ricerca (vedi
`workflow.md`), mai automaticamente.

```jsonc
{
  "slug": "company-role-2026-01",   // usato anche come nome cartella in cv/generated/
  "title": null,
  "company": null,
  "location": null,
  "remote": null,                    // true/false/"hybrid"
  "url": null,                       // link all'annuncio originale
  "source": null,                    // bacheca/sito da cui è stato trovato
  "date_found": null,
  "date_posted": null,
  "key_requirements": [],
  "technical_match_score": null,     // Fase 3

  "economic_fit": null,              // Fase 4: "ok" | "below_target" | "unknown"
  "mission_fit": null,               // Fase 4: "ok" | "mismatch" | "unknown"
  "fit_notes": null,

  "status": "da valutare",           // da valutare | scartato | cv pronto | approvato dall'utente | candidato
  "discard_reason": null,

  "cv_path": null                    // cv/generated/<slug>/cv.md, valorizzato in Fase 5
}
```
