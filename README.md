# Frikort Oppslag API

API-et brukes av helseaktører til å slå opp om en pasient har gyldig frikort. Oppslaget skjer via en HTTP-forespørsel og returnerer frikortstatus for angitt person.

---

## Forutsetninger

For å kunne bruke API-et må helseaktøren være registrert hos Helfo med gyldig avtale.

Se [helfo.no](https://www.helfo.no/) for mer informasjon om avtaler og registrering.

Helseaktøren autentiseres via **Maskinporten**.

---

## API-endepunkter

| Navn                                                              | Path                                              | Scope              | Type      | Beskrivelse                                           |
|-------------------------------------------------------------------|---------------------------------------------------|--------------------|-----------|-------------------------------------------------------|
| [Slå opp frikortstatus](endepunkt_oppslag_frikortstatus.md)       | /frikort/oppslag/v1/frikortstatus                 | nav:frikort/oppslag | POST/JSON | Sjekker om en person har gyldig frikort på et gitt tidspunkt. |

---

### Typer endepunkt

| Type          | Beskrivelse                                                                              |
|---------------|------------------------------------------------------------------------------------------|
| **POST/JSON** | HTTP POST med ukryptert JSON i request-body. Responsen returneres som ukryptert JSON.    |

*(Ukryptert betyr at dataene kun er sikret med HTTPS, ikke innholdskryptert.)*

---

## Autentisering og autorisasjon

### Maskinporten (virksomheter)

Helseaktører bruker **Maskinporten** for autentisering og autorisasjon.
Organisasjonen må være registrert både i Maskinporten og hos Helfo, med samme organisasjonsnummer.

Mer informasjon:
[Maskinporten – API-konsumentguide](https://docs.digdir.no/docs/Maskinporten/maskinporten_guide_apikonsument.html)

API-tilgangen kan delegeres videre til en leverandør gjennom Altinn.

**Maskinporten-scopes**

| Scope                | Beskrivelse                                   |
|----------------------|-----------------------------------------------|
| nav:frikort/oppslag  | Tilgang til oppslag av frikortstatus.         |

---

## Miljøer

| Miljø | Frikort Oppslag API URL                                                        | Maskinporten                                                         |
|-------|--------------------------------------------------------------------------------|----------------------------------------------------------------------|
| DEV   | [https://api-preprod.helserefusjon.no](https://api-preprod.helserefusjon.no)  | [https://test.maskinporten.no](https://test.maskinporten.no/jwk)     |
| PROD  | [https://api.helserefusjon.no](https://api.helserefusjon.no)                  | [https://maskinporten.no](https://maskinporten.no/.well-known/oauth-authorization-server) |

---

## Endringslogg

Det føres en endringslogg for dokumentasjonen til API-et for å kunne følge oppdateringer:
[Endringslogg](endringslogg.md)
