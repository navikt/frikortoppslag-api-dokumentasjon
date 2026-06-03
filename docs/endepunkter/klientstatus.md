# Klientstatus

Endepunkt som lar konsumenter verifisere at integrasjonen er korrekt satt opp, uten å gjøre oppslag på ekte borgere.

Endepunktet validerer kryptering og avtaleforhold, og returnerer status for hver sjekk. Autentisering (DPoP) valideres først — ved autentiseringsfeil returneres en vanlig feilrespons (se [feilresponser](hent_frikortstatus.md#feilresponser)) før klientstatus-sjekken utføres.

---

## Endepunkt

| Felt         | Verdi                                            |
|--------------|--------------------------------------------------|
| **Path**     | `/api/frikortsporring/helseid/v1/klientstatus`         |
| **Metode**   | `POST`                                           |
| **Auth**     | HelseID DPoP-token (scope: `hdir:frikortsporring/read`) |

---

## Request

### Headers

| Header          | Påkrevd | Verdi                                                      |
|-----------------|---------|-------------------------------------------------------------|
| `Authorization` | Ja      | `DPoP <helseid-token>`                                      |
| `DPoP`          | Ja      | DPoP-bevis. Må være bundet til request-URI og HTTP-metode.  |
| `Content-Type`  | Ja      | `application/jose`                                          |
| `Correlation-Id`| Nei     | Valgfri UUID for sporing. Hvis ikke angitt, genererer API-et en ny ID. Returneres alltid i responsen. |

### Request body

Request-body sendes som en JWE-kryptert streng, på samme måte som for [hent egenandelsfritakstatus](hent_frikortstatus.md). Se [JWE-kryptering](../kryptering_av_request.md) for detaljer.

Innholdet i den dekrypterte payloaden er ikke relevant for statussjekken — det er selve JWE-strukturen som valideres. Payloaden kan derfor inneholde vilkårlig innhold, så lenge den er gyldig JWE kryptert med riktig nøkkel.

---

## Response

### HTTP-statuskoder

| Statuskode                  | Beskrivelse                                               |
|-----------------------------|-----------------------------------------------------------|
| `200 OK`                    | Autentisering var vellykket. Se response body for resultat av øvrige sjekker. |
| `400 Bad Request`           | Ugyldig request (ugyldig DPoP-bevis).                     |
| `401 Unauthorized`          | Autentisering feilet — token er ugyldig eller utløpt.     |
| `429 Too Many Requests`     | For mange forespørsler sendt på kort tid.                 |
| `500 Internal Server Error` | Uventet feil på serversiden.                              |

### Response body (200 OK)

Responsen returneres som ukryptert JSON med `Content-Type: application/json`.

```json
{
  "overordnetStatus": "OK",
  "autentisering": {
    "status": "OK",
    "beskrivelse": "DPoP-token er gyldig"
  },
  "kryptering": {
    "status": "OK",
    "beskrivelse": "JWE-dekryptering vellykket"
  },
  "helfoAvtale": {
    "status": "IKKE_IMPLEMENTERT",
    "beskrivelse": "Avtalevalidering er ikke implementert ennå"
  }
}
```

### Felter i response

| Felt              | Type   | Beskrivelse                                                                                                                                                   |
|-------------------|--------|---------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `overordnetStatus`| String | Samlet status for alle sjekker. `OK` hvis alle sjekker er OK eller IKKE_IMPLEMENTERT. `FEIL` hvis én eller flere sjekker feiler.                              |
| `autentisering`   | Sjekk  | Resultat av DPoP-autentiseringssjekken. Vil alltid være `OK` i 200-responsen, ettersom autentiseringsfeil gir en feilrespons (401/400) før denne sjekken nås. |
| `kryptering`      | Sjekk  | Resultat av JWE-dekrypteringssjekken.                                                                                                                         |
| `helfoAvtale`      | Sjekk  | Resultat av avtalesjekk mot Helfos register. Se [Kontroll av avtaleforhold](../index.md#kontroll-av-avtaleforhold).                                           |

### Sjekk-objekt

| Felt         | Type   | Beskrivelse                              |
|--------------|--------|------------------------------------------|
| `status`     | String | `OK`, `FEIL` eller `IKKE_IMPLEMENTERT`.  |
| `beskrivelse`| String | Menneskelig lesbar beskrivelse av resultatet. |

### Mulige statusverdier

| Status               | Beskrivelse                                                      |
|----------------------|------------------------------------------------------------------|
| `OK`                 | Sjekken ble utført og bestått.                                   |
| `FEIL`               | Sjekken ble utført, men feilet.                                  |
| `IKKE_IMPLEMENTERT`  | Sjekken er ikke implementert ennå.                               |

### Response headers

| Header           | Beskrivelse                                                        |
|------------------|--------------------------------------------------------------------|
| `Correlation-Id` | Unik ID for kallet. Oppgi denne ved support-henvendelse.           |

### Feilresponser

Ved autentiseringsfeil (400/401) returneres en feilrespons på samme format som for [hent egenandelsfritakstatus](hent_frikortstatus.md#feilresponser).

---

## Eksempler

### Request

```http
POST /api/frikortsporring/helseid/v1/klientstatus HTTP/1.1
Host: frikortbifrost.nav.no
Authorization: DPoP eyJhbGciOiJSUzI1NiIsInR5cCI6ImF0K2p3dCJ9...
DPoP: eyJ0eXAiOiJkcG9wK2p3dCIsImFsZyI6IkVTMjU2Iiw...
Content-Type: application/jose
Correlation-Id: 3fa85f64-5717-4562-b3fc-2c963f66afa6

eyJhbGciOiJSU0EtT0FFUC0yNTYiLCJlbmMiOiJBMjU2R0NNIiwia2lkIjoiZnJpa29ydGJpZnJvc3QtZW5jLTIwMjYwMzEyLTEifQ...
```

### Response — alt OK

```http
HTTP/1.1 200 OK
Content-Type: application/json
Correlation-Id: 3fa85f64-5717-4562-b3fc-2c963f66afa6

{
  "overordnetStatus": "OK",
  "autentisering": {
    "status": "OK",
    "beskrivelse": "DPoP-token er gyldig"
  },
  "kryptering": {
    "status": "OK",
    "beskrivelse": "JWE-dekryptering vellykket"
  },
  "helfoAvtale": {
    "status": "OK",
    "beskrivelse": "Aktiv avtale funnet"
  }
}
```

### Response — kryptering feiler

```http
HTTP/1.1 200 OK
Content-Type: application/json
Correlation-Id: 3fa85f64-5717-4562-b3fc-2c963f66afa6

{
  "overordnetStatus": "FEIL",
  "autentisering": {
    "status": "OK",
    "beskrivelse": "DPoP-token er gyldig"
  },
  "kryptering": {
    "status": "FEIL",
    "beskrivelse": "JWE-dekryptering feilet: ugyldig nøkkel"
  },
  "helfoAvtale": {
    "status": "IKKE_IMPLEMENTERT",
    "beskrivelse": "Avtalevalidering er ikke implementert ennå"
  }
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
