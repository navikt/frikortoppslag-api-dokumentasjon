# Frikortoppslag API

!!! warning "Kun tilgjengelig i testmiljø"
    API-et er foreløpig kun tilgjengelig i testmiljøet. Kall mot produksjonsmiljøet vil feile.

Frikortløsningen tilbyr API-baserte tjenester for samhandling med behandlere, apotek og andre aktører i helsesektoren som har avtale med Helfo om direkte oppgjør.

## Tjenesten

API-et brukes av helseaktører til oppslag av egenandelsfritakstatus for en borger. Oppslaget gir svar på om borgeren er fritatt fra å betale egenandel for en gitt tjeneste på en gitt dato, basert på frikort eller minstepensjonist-status.

Forespørselen sendes som en JWE-kryptert HTTP POST-request, og responsen returneres som ukryptert JSON.

## Forutsetninger

For å kunne bruke API-et må følgende være på plass:

1. Helseaktøren må være registrert hos Helfo med gyldig avtale om direkte oppgjør. Se [helfo.no](https://www.helfo.no/) for mer informasjon.
2. Helseaktøren må ha en klient registrert hos HelseID med tilgang til API-et (se [Autentisering](#autentisering-og-autorisasjon)).
3. Request-body må krypteres med JWE (se [JWE-kryptering](kryptering_av_request.md)).

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
| [Klientstatus](endepunkter/klientstatus.md) | `/api/frikortsporring/helseid/v1/klientstatus` | POST | Verifiserer at integrasjonen er korrekt satt opp (autentisering, kryptering og avtaleforhold). |
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

Mer informasjon om HelseID og oppsett:
[HelseID – NHN utviklerportal](https://utviklerportal.nhn.no/informasjonstjenester/helseid/)

### Kontroll av avtaleforhold

!!! warning "Kommende funksjonalitet"
    Avtalevalidering er under utvikling og er ikke implementert ennå. Dokumentasjonen beskriver planlagt oppførsel.

I tillegg til autentisering via HelseID, vil API-et kontrollere at den som gjør oppslaget har en aktiv avtale med Helsedirektoratet. Denne kontrollen gjøres mot Helsedirektoratets register over avtaleforhold, og utføres **før** noe svar returneres.

Registeret sjekkes i følgende rekkefølge, basert på informasjonen i claims fra HelseID-tokenet:

1. **Helsepersonellets fødselsnummer (PID-claim):** Dersom tokenet inneholder et PID-claim (innlogget helsepersonell), brukes fødselsnummeret til den innloggede brukeren for å slå opp i avtaleregisteret.
2. **Underenhetens organisasjonsnummer (`orgnr_child`):** Dersom det ikke finnes et PID-claim i tokenet, brukes organisasjonsnummeret til underenheten (child) for oppslag.
3. **Hovedenhetens organisasjonsnummer (`orgnr_parent`):** Dersom hverken PID-claim eller `orgnr_child` er tilgjengelig, brukes organisasjonsnummeret til hovedenheten (parent) for oppslag.

Dersom ingen av identifikatorene gir treff i avtaleregisteret, vil API-et returnere **`403 Forbidden`** med feilkode `INGEN_TILGANG`.

---

## OpenAPI-spesifikasjon

Swagger for OpenAPI: [SWAGGER - FRIKORTSPORRING-API](https://frikortbifrost.ekstern.dev.nav.no/swagger-ui/index.html)

En fullstendig OpenAPI 3.1-spesifikasjon for API-et er tilgjengelig:
[frikortsporring-api.yaml](frikortsporring-api.yaml)

---

## Endringslogg

Det føres en endringslogg for dokumentasjonen:
[Endringslogg](generelt/endringslogg.md)


## Kontakt
Ved spørsmål, ta kontakt på e-post:
**frikort.teknisk@nav.no**