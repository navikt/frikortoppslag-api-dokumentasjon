# Frikort Oppslag API

Frikortløsningen tilbyr API-baserte tjenester for samhandling med behandlere, apotek og andre aktører i helsesektoren som har avtale med Helfo om direkte oppgjør.

## Tjenesten

API-et brukes av helseaktører til oppslag av egenandelsfritakstatus for en borger. Oppslaget gir svar på om borgeren er fritatt fra å betale egenandel for en gitt tjeneste på en gitt dato, basert på frikort eller minstepensjonist-status.

Forespørselen sendes som en JWE-kryptert HTTP POST-request, og responsen returneres som ukryptert JSON.

## Forutsetninger

For å kunne bruke API-et må følgende være på plass:

1. Helseaktøren må være registrert hos Helfo med gyldig avtale om direkte oppgjør. Se [helfo.no](https://www.helfo.no/) for mer informasjon.
2. Helseaktøren må ha en klient registrert hos HelseID med tilgang til API-et (se [Autentisering](#autentisering-og-autorisasjon)).
3. Request-body må krypteres med JWE (se [JWE-kryptering](jwe_ende_til_ende_kryptering.md)).

## Miljøer

| Miljø      | Base-URL                                          |
|------------|---------------------------------------------------|
| Produksjon | `https://frikortbifrost.nav.no`                   |
| Test       | `https://frikortbifrost.ekstern.dev.nav.no`       |

---

## API-endepunkter

| Navn | Path | Metode | Beskrivelse |
|------|------|--------|-------------|
| [Hent egenandelsfritakstatus](endepunkter/hent_frikortstatus.md) | `/api/frikortsporring/helseid/v1` | POST | Sjekker om en borger er fritatt fra egenandel for en gitt tjenestetype på en gitt dato. |
| [Hent JWK-nøkler](endepunkter/jwk.md) | `/api/frikortsporring/jwks` | GET | Henter offentlige nøkler for JWE-kryptering av request. |

### Typer endepunkt

| Type | Content-Type | Beskrivelse |
|------|-------------|-------------|
| **POST/JWE** | `application/jose` | HTTP POST med JWE-kryptert JSON i request-body. Responsen returneres som ukryptert JSON (`application/json`). |
| **GET/JSON** | `application/json` | HTTP GET som returnerer JSON. |

---

## Autentisering og autorisasjon

### HelseID

API-et bruker **HelseID** for autentisering og autorisasjon. HelseID-tokenet **må** bruke **DPoP** ([RFC 9449](https://datatracker.ietf.org/doc/html/rfc9449)) — vanlige Bearer-tokens er ikke støttet.

**Oppsett:**

1. Opprett en klient i NHNs selvbetjeningsportal med tilgang til API-et **«Helsedirektoratets API for frikortspørring»**.
2. Konfigurer klienten med scope **`hdir:frikortsporring/read`**.
3. Organisasjonen må være registrert hos Helfo. Det er et krav at organisasjonsnummer sendes som claim i tokenet.

Mer informasjon om HelseID og oppsett:
[HelseID – NHN utviklerportal](https://utviklerportal.nhn.no/informasjonstjenester/helseid/)

---

## OpenAPI-spesifikasjon

En fullstendig OpenAPI 3.1-spesifikasjon for API-et er tilgjengelig:
[frikortsporring-api.yaml](https://github.com/navikt/frikort-bifrost/blob/main/frikortsporring-api.yaml)

---

## Endringslogg

Det føres en endringslogg for dokumentasjonen:
[Endringslogg](generelt/endringslogg.md)
