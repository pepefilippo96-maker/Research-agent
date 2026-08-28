<!--
Template CV master (inglese). Definisce struttura e ordine delle sezioni usate
in Fase 5 per generare ogni CV su misura in cv/generated/<job-slug>/cv.md.
Ogni sezione è popolata ESCLUSIVAMENTE da data/profile/profile.yaml — vedi
CLAUDE.md, regole 1 e 2. Per ogni CV generato si può riordinare/enfatizzare il
contenuto del profilo in base alla job description, mai aggiungerne di nuovo.
-->

# {{full_name}}

{{headline}}
{{location}} · {{contact.email}} · {{contact.phone}} · {{contact.linkedin}}

## Summary

{{summary}}

## Experience

<!-- per ogni voce di profile.yaml → experience, selezionata/riordinata in base alla JD -->
### {{role}} — {{company}}
{{location}} · {{start_date}} – {{end_date}}

- {{responsibility/achievement}}

## Skills

{{skills.technical}} · {{skills.tools}}

## Education

### {{degree}} — {{institution}}
{{location}} · {{start_date}} – {{end_date}}

## Certifications

- {{certification}}

## Languages

{{skills.languages_spoken}}
