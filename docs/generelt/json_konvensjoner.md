# JSON-konvensjoner

## Datoer og klokkeslett

Det brukes et subsett av ISO 8601 for spesifikasjon av datoer og klokkeslett.

*   Datoer skal være på format `YYYY-MM-DD`.
    *   Eksempel på gyldig verdi: `"2026-03-25"`.
    *   Eksempler på ugyldige verdier: `"20260325"` (mangler skilletegn) og `"2026-03"` (mangler dag).

## Boolske verdier

Boolske verdier angis med JSON-type boolean.

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