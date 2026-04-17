# JSON-konvensjoner

## Navngivning av felter

Det skal brukes camelCase i navngivning av felter. For å sikre god lesbarhet kan det i noen tilfeller brukes mer orddeling (stor bokstav i camelCase) enn hva som følger direkte av norsk rettskrivning. Et mulig eksempel: behandlerkravid kan få feltnavn `behandlerkravId` eller `behandlerKravId`.

Det skal ikke brukes norske spesialbokstaver (æ, ø, å, Æ, Ø, Å) i navn på felter. Det er tillatt å bruke både en-bokstavs og to-bokstavs erstatning, f.eks. `a` og `aa` for å, og `o` og `oe` for ø.

## Datoer og klokkeslett

Det brukes et subsett av ISO 8601 for spesifikasjon av datoer og klokkeslett.

*   Datoer skal være på format `YYYY-MM-DD`.
    *   Eksempel på gyldig verdi: `"2026-03-25"`.
    *   Eksempler på ugyldige verdier: `"20260325"` (mangler skilletegn) og `"2026-03"` (mangler dag).

## Boolske verdier

Boolske verdier skal alltid angis med JSON-type boolean.

*   Eksempel på gyldig verdi:

    ```json
    { "harEgenandelsfritak": true }
    ```

*   Eksempler på ugyldige verdier:

    ```json
    { "harEgenandelsfritak": "true" }
    ```
    (bruker streng)

    ```json
    { "harEgenandelsfritak": 1 }
    ```
    (bruker tall)



## Håndtering av ukjente felt

For å sikre god endringsevne på APIene skal konsumenter av APIene godta at mottatte JSON-meldinger inneholder felt som konsumenten ikke kjenner til.

Eksempel: Konsument har implementert støtte for feltet `harEgenandelsfritak`, og mottar følgende JSON:

```json
{ "harEgenandelsfritak": true, "merknad": "eksempelMerknad" }
```

Konsument skal her ignorere feltet `merknad`, og behandle resten av meldingen som normalt.

For å raskt avdekke feil når f.eks. nye konsumenter kobler seg til APIene, skal APIene (server-siden) ikke godta at JSON-meldinger mottatt fra API-konsumenter inneholder ukjente felt. Det skal da i stedet returneres en feilmelding som konkret beskriver problemet med feltet tilbake til konsument.

## Enkoding

JSON skal være UTF-8-enkodet.