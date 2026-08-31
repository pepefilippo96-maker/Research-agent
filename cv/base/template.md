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

## Social & Cultural Commitment

<!-- da profile.yaml → volunteering_and_extracurricular; sezione presente nel CV
     originale dell'utente (cv_pf_1.pdf), da NON escludere nei CV generati -->
### {{role}} — {{organization}}
{{location}} · {{start_date}} – {{end_date}}

- {{description}}

## Languages

{{skills.languages_spoken}}

## Data Protection

I authorize the processing of my personal data contained in this CV pursuant to the EU General Data Protection Regulation (GDPR) 2016/679 and, where applicable, Italian Legislative Decree 196/2003.
