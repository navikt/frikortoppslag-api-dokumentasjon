# JSON-konvensjoner

## Navngivning av felter

Det skal brukes camelCase i navngivning av felter. For å sikre god lesbarhet kan det i noen tilfeller brukes mer orddeling (stor bokstav i camelCase) enn hva som følger direkte av norsk rettskrivning. Et mulig eksempel: behandlerkravid kan få feltnavn `behandlerkravId` eller `behandlerKravId`.

Det skal ikke brukes norske spesialbokstaver (æ, ø, å, Æ, Ø, Å) i navn på felter. Det er tillatt å bruke både en-bokstavs og to-bokstavs erstatning, f.eks. `a` og `aa` for å, og `o` og `oe` for ø.

## Datoer og klokkeslett

Det brukes et subsett av ISO 8601 for spesifikasjon av datoer og klokkeslett.

*   Datoer skal være på format `YYYY-MM-DD`.
    *   Eksempel på gyldig verdi: `"2026-03-25"`.
    *   Eksempler på ugyldige verdier: `"20260325"` (mangler skilletegn) og `"2026-03"` (mangler dag).
*   Tidspunkt uten dato skal være på format `hh:mm:ss±hh:mm` eller `hh:mm:ssZ`. Det skal inkluderes tidssone, og presisjon skal være sekunder.

    *   Eksempler på gyldige verdier: `"12:45:01+01:00"` og `"11:45:01Z"`. Sistnevnte har tidssone UTC.
    *   Eksempler på ugyldige verdier: `"12:45:01"` (mangler tidssone), `"12:45:01+0100"` (feil format på tidssone), `"12:45+01:00"` (mangler sekunder) og `"12:45:01.723+01:00"` (for høy presisjon).

*   Datoer med tidspunkt skal være på format `YYYY-MM-DDThh:mm:ss±hh:mm` eller `YYYY-MM-DDThh:mm:ssZ`. Det skal inkluderes tidssone, og presisjon skal være sekunder.

    *   Eksempler på gyldige verdier: `"2026-03-25T12:45:01+01:00"` og `"2026-03-25T11:45:01Z"`.
    *   Eksempler på ugyldige verdier: `"2026-03-25T12:45:01"` (mangler tidssone), `"2026-03-25T12:45:01.240+01:00"` (for høy presisjon) og `"2026-03-25T12:45+01:00"` (mangler sekund).

## Boolske verdier

Boolske verdier skal alltid angis med JSON-type boolean.

*   Eksempel på gyldig verdi:

    ```json
    { "betalt": true }
    ```

*   Eksempler på ugyldige verdier:

    ```json
    { "betalt": "true" }
    ```
    (bruker streng)

    ```json
    { "betalt": 1 }
    ```
    (bruker tall)

## Manglende eller tomme felt og arrays

Felt som mangler verdi kan representeres på flere måter. Det er tillatt både å utelate feltet helt, og å sette det til `null`. Det er **ikke** tillatt å sette tom streng. At et felt er utelatt skal tolkes som at feltet har verdi `null`.

*   Eksempler på gyldige måter for felt `merknad`:

    *   Feltet utelates helt. Tolkes av mottaker som at feltet har verdi `null`.

        ```json
        { "dato": "2026-03-05", "koder": ["2AD", "2FK"] }
        ```

    *   Feltet settes til `null`:

        ```json
        { "dato": "2026-03-05", "merknad": null, "koder": ["2AD", "2FK"] }
        ```

*   Eksempel på ugyldig måte:

    *   Feltet settes til tom streng:

        ```json
        { "dato": "2026-03-05", "merknad": "", "koder": ["2AD", "2FK"] }
        ```

Tomme arrays kan også representeres på flere måter. Det er tillatt både å utelate feltet helt, å sette feltet til `null` og å sette feltet til en tom array. At et array-felt er utelatt, eller har verdi `null`, skal tolkes som at feltet har en tom array.

*   Eksempler på gyldige måter å representere feltet `koder` som er et array uten data:

    *   Feltet utelates helt. Tolkes av mottaker som at feltet har en tom array.

        ```json
        { "dato": "2026-03-25", "merknad": "forsinket" }
        ```

    *   Feltet settes til `null`. Tolkes av mottaker som at feltet har en tom array.

        ```json
        { "dato": "2026-03-25", "merknad": "forsinket", "koder": null }
        ```

    *   Feltet settes til tomt array:

        ```json
        { "dato": "2026-03-25", "merknad": "forsinket", "koder": [] }
        ```

## Håndtering av ukjente felt

For å sikre god endringsevne på APIene skal konsumenter av APIene godta at mottatte JSON-meldinger inneholder felt som konsumenten ikke kjenner til.

Eksempel: Konsument har implementert støtte for feltene `dato` og `koder`, og mottar følgende JSON:

```json
{ "dato": "2026-03-25", "merknad": "forsinket", "koder": ["2AD", "2FK"] }
```

Konsument skal her ignorere feltet `merknad`, og behandle resten av meldingen som normalt.

For å raskt avdekke feil når f.eks. nye konsumenter kobler seg til APIene, skal APIene (server-siden) ikke godta at JSON-meldinger mottatt fra API-konsumenter inneholder ukjente felt. Det skal da i stedet returneres en feilmelding som konkret beskriver problemet med feltet tilbake til konsument.

## Rekkefølge i arrays

Vi garanterer ikke noen gitt eller stabil sortering i arrays. Mottaker må alltid sikre riktig rekkefølge på sin side, når dette er viktig for korrekt håndtering.

## Enkoding

JSON skal være UTF-8-enkodet.