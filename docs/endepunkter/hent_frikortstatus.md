# Hent egenandelsfritakstatus

Endepunkt for å sjekke om en borger er fritatt fra å betale egenandel for en gitt tjenestetype på en gitt dato.

Fritak gjelder hvis borgeren har innvilget frikort for det kalenderåret tjenestedatoen faller i. For tjenester levert av apotek eller bandasjist gjelder i tillegg at minstepensjonister er fritatt.

**Reservasjon:** Borgere kan reservere seg mot den automatiske frikortordningen. For borgere med reservasjon vil tjenesten alltid svare negativt (`harEgenandelsfritak: false`). Borgere med reservasjon må selv fremvise frikortbevis.


---

## Endepunkt

| Felt         | Verdi                                   |
|--------------|-----------------------------------------|
| **Path**     | `/api/frikortsporring/helseid/v1`       |
| **Metode**   | `POST`                                  |
| **Auth**     | HelseID DPoP-token (scope: `hdir:frikortsporring/read`) |

---

## Request

### Headers

| Header          | Påkrevd | Verdi                                                      |
|-----------------|---------|------------------------------------------------------------|
| `Authorization` | Ja      | `DPoP <helseid-token>`                                     |
| `DPoP`          | Ja      | DPoP-bevis. Må være bundet til request-URI og HTTP-metode. |
| `Content-Type`  | Ja      | `application/jose`                                         |
| `Correlation-Id`| Nei     | Valgfri UUID for sporing. Hvis ikke angitt, genererer API-et en ny ID. Returneres alltid i responsen. |

### Request body

Request-body sendes som en JWE-kryptert streng. Kryptering er påkrevd fordi request-innholdet avslører at en borger har
mottatt en bestemt type helsetjeneste på en gitt dato, noe som utgjør sensitive personopplysninger. Helsedirektoratets
policy krever ende-til-ende-kryptering av sensitive data på offentlig sky. TLS alene er ikke tilstrekkelig da data
ellers ender i klartekst ved TLS-terminering. Se [JWE-kryptering](../kryptering_av_request.md) for detaljer om kryptering. 

Det dekrypterte innholdet i JWE-payloaden skal være følgende JSON:

```json
{
  "borgerIdent": "12345678901",
  "tjenestetypeKode": "LE",
  "tjenestedato": "2026-06-15"
}
```

### Felter i request

| Felt              | Type   | Påkrevd | Beskrivelse                                                                            |
|-------------------|--------|---------|----------------------------------------------------------------------------------------|
| `borgerIdent`     | String | Ja      | Fødselsnummer (11 siffer) eller D-nummer (11 siffer) for borgeren det gjøres oppslag for. |
| `tjenestetypeKode`| String | Ja      | Kode for tjenestetype. Se [kodeverk](../generelt/kodeverk.md) for gyldige verdier.     |
| `tjenestedato`    | String | Ja      | Dato for behandlingen eller utleveringen (ISO 8601, `YYYY-MM-DD`). Fritaksstatus vurderes for denne datoen. |

---

## Response

### HTTP-statuskoder

| Statuskode                  | Beskrivelse                                                                       |
|-----------------------------|-----------------------------------------------------------------------------------|
| `200 OK`                    | Oppslaget var vellykket. Se response body.                                        |
| `400 Bad Request`           | Ugyldig request (ugyldig JWE, ugyldig JSON, valideringsfeil, ugyldig DPoP-bevis). |
| `401 Unauthorized`          | Autentisering feilet — token er ugyldig eller utløpt.                             |
| `403 Forbidden`             | Konsumenten har ikke en aktiv avtale med Helfo som gir tilgang. Se [Kontroll av avtaleforhold](../index.md#kontroll-av-avtaleforhold). |
| `429 Too Many Requests`     | For mange forespørsler sendt på kort tid. Vent i henhold til `Retry-After`.      |
| `500 Internal Server Error` | Uventet feil på serversiden.                                                      |
| `501 Not Implemented`       | Ikke ferdig implementert, brukes under testing.                                   |

### Response body (200 OK)

Responsen returneres som ukryptert JSON med `Content-Type: application/json`.

```json
{
  "harEgenandelsfritak": true
}
```

| Felt                  | Type    | Beskrivelse                                                                                                                                       |
|-----------------------|---------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| `harEgenandelsfritak` | Boolean | `true` hvis borgeren er fritatt fra egenandel for den angitte tjenesten på den angitte datoen. `false` hvis borger ikke er fritatt fra egenandel. | 


### Response headers

| Header           | Beskrivelse                                                        |
|------------------|--------------------------------------------------------------------|
| `Correlation-Id` | Unik ID for kallet. Oppgi denne ved support-henvendelse.           |
| `Retry-After`    | Kun ved 429 — antall sekunder å vente før neste forsøk.            |

### Feilresponser

Ved feil (4xx/5xx) returneres en JSON-body med følgende struktur:

```json
{
  "correlationId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "feilkode": "UGYLDIG_REQUEST",
  "melding": "Request-body er ikke gyldig JWE Compact Serialization",
  "timestamp": "1716283200000"
}
```

| Felt            | Type   | Beskrivelse                                                           |
|-----------------|--------|-----------------------------------------------------------------------|
| `correlationId` | String | Unik ID for kallet (UUID). Oppgi denne ved support-henvendelse.       |
| `feilkode`      | String | Maskinlesbar feilkode — se tabell under.                              |
| `melding`       | String | Menneskelig lesbar feilmelding (ikke egnet for maskinell tolkning).    |
| `timestamp`     | String | Tidspunkt for feilen (Unix millisekunder siden epoch).                |

### Feilkoder

| Feilkode                  | HTTP-status | Beskrivelse                                                                                |
|---------------------------|-------------|--------------------------------------------------------------------------------------------|
| `UGYLDIG_REQUEST`         | 400         | Ugyldig JWE, ugyldig JSON i dekryptert payload, valideringsfeil, eller ugyldig DPoP-bevis. |
| `AUTENTISERING_FEILET`    | 401         | JWT-token er ugyldig, utløpt eller ikke tillitskontrollert.                                |
| `INGEN_TILGANG`           | 403         | Konsumenten mangler aktiv avtale eller nødvendige rettigheter.                             |
| `FOR_MANGE_FORESPORSLER`  | 429         | Konsumenten har sendt for mange forespørsler i et gitt tidsrom.                            |
| `INTERN_FEIL`             | 500         | Uventet feil på serversiden. Oppgi `correlationId` ved support-henvendelse.                |

---

## Eksempler

### Request

```http
POST /api/frikortsporring/helseid/v1 HTTP/1.1
Host: frikortbifrost.nav.no
Authorization: DPoP eyJhbGciOiJSUzI1NiIsInR5cCI6ImF0K2p3dCJ9...
DPoP: eyJ0eXAiOiJkcG9wK2p3dCIsImFsZyI6IkVTMjU2Iiw...
Content-Type: application/jose
Correlation-Id: 3fa85f64-5717-4562-b3fc-2c963f66afa6

eyJhbGciOiJSU0EtT0FFUC0yNTYiLCJlbmMiOiJBMjU2R0NNIiwia2lkIjoiZnJpa29ydGJpZnJvc3QtZW5jLTIwMjYwMzEyLTEifQ...
```

### Response — borger er fritatt

```http
HTTP/1.1 200 OK
Content-Type: application/json
Correlation-Id: 3fa85f64-5717-4562-b3fc-2c963f66afa6

{
  "harEgenandelsfritak": true
}
```

### Response — borger er ikke fritatt

```http
HTTP/1.1 200 OK
Content-Type: application/json
Correlation-Id: 3fa85f64-5717-4562-b3fc-2c963f66afa6

{
  "harEgenandelsfritak": false
}
```

### Response — ugyldig request

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json
Correlation-Id: 3fa85f64-5717-4562-b3fc-2c963f66afa6

{
  "correlationId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "feilkode": "UGYLDIG_REQUEST",
  "melding": "Valideringsfeil: 'tjenestetypeKode' er ikke en gyldig kode",
  "timestamp": "1716283200000"
}
```

### Response — autentisering feilet

```http
HTTP/1.1 401 Unauthorized
Content-Type: application/json
Correlation-Id: 3fa85f64-5717-4562-b3fc-2c963f66afa6

{
  "correlationId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "feilkode": "AUTENTISERING_FEILET",
  "melding": "JWT-token er ugyldig eller utløpt",
  "timestamp": "1716283200000"
}
```

## Testpersoner

Vi har et sett med testpersoner som kan brukes i testmiljøet. Disse personene har ulike kombinasjoner av frikortstatus og reservasjon for å teste forskjellige scenarier.

| Fødselsnummer | Egenandelsfritak | Beskrivelse                                                                                        |
|---------------|------------------|----------------------------------------------------------------------------------------------------|
| Kommer snart  | Ja               | Har frikort for inneværende år.                                                                    |
| Kommer snart  | Ja               | Har frikort for inneværende år.                                                                    |
| Kommer snart  | Nei              | Har ikke frikort.                                                                                  |
| Kommer snart  | Nei              | Har ikke frikort.                                                                                  |
| Kommer snart  | Ja (Blåresept)   | Har ikke frikort, men har status som minstepensjonist. Vil returnere true på tjenestetype A, B, S. |