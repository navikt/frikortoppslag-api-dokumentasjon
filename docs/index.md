# Frikort-APIer
Frikortløsningen tilbyr API-baserte tjenester for samhandling med behandlere, apotek og andre aktører i helsesektoren som har avtale med HELFO om direkte oppgjør.

## Tjenesten
API-et brukes av samhandlere til oppslag av frikortstatus for en person. Det gir informasjon om gyldigheten av frikortet, som gir pasienten/kunden fritak for betaling av egenandel.
Innsendingen skjer via en HTTP-forespørsel, og responsen returneres som JSON.


## Driftsstatus tjenestene
Kan sees på ....drift.nav.no?


## Forutsetninger

For å kunne bruke API-et må helseaktøren være registrert hos Helfo med gyldig avtale.

Se [helfo.no](https://www.helfo.no/) for mer informasjon om avtaler og registrering.

Helseaktøren autentiseres via **HelseID**.

# Tilgangskontroll
Tilgangskontrollen for tjenesten er basert på bruk av tokens utstedt av HelseID.
Det er mange forskjellige typer aktører som skal ha tilgang til tjenestene, og det er derfor etablert støtte for flere forskjellige scenarioer for tilgangskontroll.
Viktige skiller her er:
- Er avtalen med HELFO om direkte oppgjør knyttet til en enkelt behandler (personlig tilgang), eller til en organisasjon?
- Har aktøren selv en egen installasjon av EPJ-system, eller tilsvarende, som utfører HTTP-kallene til tjenestene, eller er det en leverandør/et felles system som gjør kall på vegne av aktøren?


---

## API-endepunkter
NB: Under testing returnerer vi 501 Not Implemented.

| Navn                 | Path                            | Scope                     | Type      | Beskrivelse                                                                                 |
|----------------------|---------------------------------|---------------------------|-----------|---------------------------------------------------------------------------------------------|
| [Hent frikortstatus] | /api/frikortsporring/helseid/v1 | hdir:frikortsporring/read | POST/JSON | Sjekker om en person har gyldig frikort på et gitt tidspunkt. Innhold i JSON er true/false. |

---

### Typer endepunkt

| Type          | Beskrivelse                                                                          |
|---------------|--------------------------------------------------------------------------------------|
| **POST/JSON** | HTTP POST med kryptert JSON i request-body. Responsen returneres som ukryptert JSON. |

*(Ukryptert betyr at dataene kun er sikret med HTTPS, ikke innholdskryptert.)*

---

## Autentisering og autorisasjon

### Helseid

Autentiseringsmekanismen vi bruker er HelseID. for autentisering og autorisasjon.

Mer informasjon:
[HelseID – NHN utviklerportal](https://utviklerportal.nhn.no/informasjonstjenester/helseid/)

For APIet brukes HelseID scope **hdir:frikortsporring/read**

Organisasjonen må være registrert hos Helfo.
Det er et krav at vi får tilsendt organisasjonr som claim for at vi skal kunne gi tilgang til APIet.
APIet heter 'Helsedirektoratets API for frikortspørring' på NHNs selvbetjeningssider.

---

## Endringslogg

Det føres en endringslogg for dokumentasjonen til API-et for å kunne følge oppdateringer:
[Endringslogg](endringslogg.md)
