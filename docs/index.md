# Frikortoppslag API

Frikortløsningen tilbyr API-baserte tjenester for samhandling med behandlere, apotek og andre aktører i helsesektoren som har avtale med Helfo om direkte oppgjør.

## Tjenesten

API-et brukes av helseaktører til oppslag av egenandelsfritakstatus for en borger. Oppslaget gir svar på om borgeren er fritatt fra å betale egenandel for en gitt tjeneste på en gitt dato, basert på frikort eller minstepensjonist-status.

Forespørselen sendes som en JWE-kryptert HTTP POST-request, og responsen returneres som ukryptert JSON.

## Forutsetninger

For å kunne bruke API-et må følgende være på plass:

1. Helseaktøren må være registrert hos Helfo med gyldig avtale om direkte oppgjør. Se neste avsnitt og [helfo.no](https://www.helfo.no/) for mer informasjon.
2. Helseaktøren/leverandør må ha en klient registrert hos HelseID med tilgang til API-et (se [Autentisering](#autentisering-og-autorisasjon)).
3. Request-body må krypteres med JWE (se [JWE-kryptering](kryptering_av_request.md)).

### Avtaler for helseaktører

Helseaktører må ha en gyldig avtale om direkte oppgjør med Helfo. Avtalen kan være knyttet enten til virksomheten eller til en person.

- **Avtale på virksomhet**: typisk for aktører der oppslaget gjøres på organisasjonsnummer (f.eks apotek).
- **Personlig avtale**: typisk for behandlere der oppslaget gjøres på innlogget bruker (f.eks tannlege eller fysioterapeut).

Hvilken avtaletype som gjelder, må helseaktøren selv kjenne til og konfigurere integrasjonen etter.

I integrasjonen mot HelseID betyr dette i praksis:

- **Personlig avtale**: bruk `authorization_code`-flyt med innlogget bruker.
- **Avtale på virksomhet**: bruk `client_credentials`-flyt uten innlogging.

Se også [HelseIDs token-endepunkt](https://utviklerportal.nhn.no/informasjonstjenester/helseid/bruksmoenstre-og-eksempelkode/bruk-av-helseid/docs/teknisk-referanse/endepunkt/token-endepunktet_no_nbmd) for detaljene om de to flytene.

Ved personlig avtale begrenses videre bruk tokenet til levetiden på refresh-tokenet som HelseID utsteder. Levetiden oppgis i token-responsen fra NHN/HelseID og vil typisk vare en drøy arbeidsdag. Det betyr at batchjobber og andre prosesser som krever innlogget bruker må fullføres mens refresh-tokenet fortsatt er gyldig. Dersom utløpt må helseaktøren logge inn på nytt for nytt token. Se også [NHNs dokumentasjon om refresh-token](https://selvbetjening.nhn.no/docs#refresh-token).

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
| [Hent JWK](endepunkter/jwk.md) | `/api/frikortsporring/jwk` | GET | Henter offentlig JWK for JWE-kryptering av request. |

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

I tillegg til autentisering via HelseID kontrollerer API-et at den som gjør oppslaget har en aktiv avtale med Helfo. Denne kontrollen gjøres mot Helfos register over avtaleforhold, og utføres **før** noe svar returneres.

Registeret sjekkes i følgende rekkefølge, basert på informasjonen i claims fra HelseID-tokenet:

1. **Helsepersonellets fødselsnummer (PID-claim):** Dersom tokenet inneholder et PID-claim (innlogget helsepersonell), brukes fødselsnummeret til den innloggede brukeren for å slå opp i avtaleregisteret.
2. **Underenhetens organisasjonsnummer (`orgnr_child`):** Dersom det ikke finnes et PID-claim i tokenet, brukes organisasjonsnummeret til underenheten (child) for oppslag.
3. **Hovedenhetens organisasjonsnummer (`orgnr_parent`):** Dersom hverken PID-claim eller `orgnr_child` er tilgjengelig, brukes organisasjonsnummeret til hovedenheten (parent) for oppslag.

Dersom ingen av identifikatorene gir treff i avtaleregisteret, returnerer API-et **`403 Forbidden`** med feilkode `INGEN_TILGANG`.

#### Testdata for avtalekontroll

For å komme gjennom avtalekontrollen i testmiljøet må aktøren finnes i Helfos avtaleregister. Hvordan dette settes opp avhenger av avtaletypen:

- **Avtale på virksomhet (organisasjonsnummer):** Organisasjonsnummeret må legges til i avtaleregisteret manuelt. Kontakt oss.
- **Personlig avtale (typisk tannlege og lege):** Du kan selv finne en test-helseaktør som eksisterer eller opprette en i Syntpop (fødselsnummeret som sendes i PID-claimet).
    - Gå inn på https://syntpop.nhn.no/. Finn eller opprett en helseaktør. Den må eksistere med FNR og i HPR. Legg til gyldig rekvisisjonsrett og gyldig periode. Helst ikke velg en som er markert "Annen eier".
    - Deretter må du inn på https://praksisinformasjon.test.helsedirektoratet.no/. Logg inn med TEST-IDP og FNR til helseaktøren.
        - Helseaktør - Legg inn nødvendig informasjon (Bl.a. kreves e-post og telefonnummer for å registrere praksis)
        - Praksiser - Registrer en gyldig praksis for helseaktøren
        - Avtaler og samtykker - Registrer avtale om direkte oppgjør.

Etter dette er gjort må du forvente noe synk-tid før avtalen er registrert hos oss. Ta kontakt dersom den ikke er registrert innen 24 timer (du vil få 403 - Ingen gyldig HELFO-avtale).


!!! tip "Sett opp personlig avtale også i test"
    Behandlere med personlig avtale (typisk tannlege og lege) må sende med et HelseID-token med `pid` (behandlerens personlige ident/FNR). Avtalekontrollen er ikke like streng i testmiljøet, men dersom dere skal ha personlig avtale i produksjon anbefaler vi å sette den opp i testmiljøet også, slik at integrasjonen testes med riktig oppsett. Det inkluderer også å sende med orgnr_parent.

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