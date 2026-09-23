# Arbeidsflyt for CodeFusion

Dette kodelageret kjører WHOs kommandolinjeverktøy CodeFusion i Docker.
Verktøyet sammenholder medisinske koder og oversatt medisinsk terminologi med
ICD og andre WHOFIC-klassifikasjoner.

Arbeidsflyten startes manuelt. Den leser filer fra `input/`, bruker innstillingene
i `parameters.json`, skriver genererte filer til `output/` og oppretter et
endringsforslag (pull request) for manuell kvalitetssikring.

## Struktur i kodelageret

- `input/` inneholder kildefilene i formatet `.txt` eller `.xlsx`.
- `output/` inneholder de genererte CodeFusion-resultatene etter at
	endringsforslaget med resultatene er slått sammen.
- `parameters.json` inneholder de felles innstillingene for CodeFusion på
	kommandolinjen.
- `.github/workflows/run-CodeFusion.yaml` kjører CodeFusion og oppretter
	endringsforslaget med resultatene.

Kodelageret inneholder ikke en lokal kjørbar CodeFusion-fil. Arbeidsflyten
bruker det versjonerte Docker-avbildet `whoicd/codefusion:1.1.1`.

## Klargjøre inndata

1. Legg til én eller flere `.txt`- eller `.xlsx`-filer under `input/`.
2. Registrer og send filene til kodelagerets standardgren.
3. Oppdater `parameters.json` dersom det er behov for andre innstillinger for
	 samsvarssøk.

Underkataloger i `input/` støttes. Arbeidsflyten bevarer den samme
katalogstrukturen under `output/`.

CodeFusion forventer tabulatordelte data i `.txt`-filer. Angi `columnNo` og
`fileContainsHeader` i `parameters.json` i samsvar med strukturen i inndataene.

En `.xlsx`-fil gir én Excel-fil som resultat. En `.txt`-fil gir både en
tabulatordelt tekstfil og en Excel-fil som resultat. CodeFusion føyer `-checked`
til standardnavnene på resultatfilene. Koblingsmodus genererer i tillegg filer
med `-mapping` i filnavnet.

## Konfigurere CodeFusion

Rediger `parameters.json` for å styre klassifikasjonsversjon, kilde, språk,
koblingsmodus, terskelverdi og øvrige innstillinger i CodeFusion.

Når `mappingMode` er aktivert, skal `idColumn` angi kolonnen som inneholder
identifikatoren fra den eksterne terminologien. `termTypeColumnNo` er valgfri.
WHO anbefaler å bruke en linearisering som `MMS` ved kobling, slik at reglene for
postkoordinering kan anvendes. Resultater fra `foundation` inneholder ikke
postkoordinering.

Arbeidsflyten overstyrer bare `inputFile` og angir denne som den aktuelle filen
i katalogen `/input` i beholderen. Behold følgende innstillinger for automatisert
kjøring:

```jsonc
"ui": false,
"exitWhenFinished": true
```

`ui: false` velger kommandolinjeverktøyet. `exitWhenFinished: true` gjør at
beholderen og GitHub Actions-jobben kan avsluttes uten interaktiv inntasting.

Se [dokumentasjonen for CodeFusion](https://icd.who.int/docs/codefusion/en/) for
en oversikt over alle tilgjengelige parametere og krav til inndata.

## Kjøre arbeidsflyten

1. Åpne fanen **Actions** for kodelageret på GitHub.
2. Velg **Run CodeFusion**.
3. Velg **Run workflow**.
4. Vent til jobben `codefusion` er fullført.

Arbeidsflyten behandler alle støttede filer i `input/`. Resultatene lastes også
opp som arbeidsflytartefaktet `codefusion-results`.

## Kontrollere resultatene

Etter en vellykket kjøring oppretter og publiserer arbeidsflyten en gren med et
navn som `output-20260923-143012-123456789`. Deretter opprettes et
endringsforslag mot standardgrenen. Endringsforslaget inneholder en sjekkliste
for manuell kvalitetssikring.

Før endringsforslaget slås sammen:

1. Sammenlign hver genererte fil under `output/` med den tilhørende kildefilen
	under `input/`.
2. Kontroller resultater som ikke er merket `GoodMatch`.
3. Bekreft at de valgte kodene og den oversatte terminologien er korrekte.
4. Slå sammen endringsforslaget først når den manuelle kvalitetssikringen er
	fullført.

Hver kjøring erstatter innholdet i arbeidskatalogen `output/` med nye resultater.
Når endringsforslaget slås sammen, blir resultatene fra denne kjøringen dermed
det gjeldende, kvalitetssikrede resultatsettet.

## Innstillinger for kodelageret

Arbeidsflyten må ha tillatelse til å publisere den genererte grenen og opprette
et endringsforslag. Åpne **Settings > Actions > General** på GitHub, velg **Read
and write permissions**, og aktiver **Allow GitHub Actions to create and approve
pull requests**.

## Avgrensning av data

Kodelageret skal bare brukes til medisinske koder og oversettelser. Ikke legg
personopplysninger om helse eller andre pasientidentifiserende opplysninger i
inndatafiler, resultatfiler, logger fra arbeidsflyten eller artefakter.
