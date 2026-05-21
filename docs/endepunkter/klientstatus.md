# Klientstatus

Endepunkt som lar konsumenter verifisere at integrasjonen er korrekt satt opp, uten å gjøre oppslag på ekte borgere.

Endepunktet validerer hele kjeden — autentisering, kryptering og avtaleforhold — og returnerer status for hver enkelt sjekk.

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
| `200 OK`                    | Statussjekken ble utført. Se response body for resultat.  |
| `400 Bad Request`           | Ugyldig request (ugyldig JWE eller ugyldig DPoP-bevis).   |
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
  "hdirAvtale": {
    "status": "IKKE_IMPLEMENTERT",
    "beskrivelse": "Avtalevalidering er ikke implementert ennå"
  }
}
```

### Felter i response

| Felt              | Type   | Beskrivelse                                                                |
|-------------------|--------|----------------------------------------------------------------------------|
| `overordnetStatus`| String | Samlet status for alle sjekker. `OK` hvis alle sjekker er OK eller IKKE_IMPLEMENTERT. `FEIL` hvis én eller flere sjekker feiler. |
| `autentisering`   | Sjekk  | Resultat av DPoP-autentiseringssjekken.                                    |
| `kryptering`      | Sjekk  | Resultat av JWE-dekrypteringssjekken.                                      |
| `hdirAvtale`      | Sjekk  | Resultat av avtalesjekk mot Helsedirektoratets register. Se [Kontroll av avtaleforhold](../index.md#kontroll-av-avtaleforhold). |

### Sjekk-objekt

| Felt         | Type   | Beskrivelse                             |
|--------------|--------|-----------------------------------------|
| `status`     | String | `OK`, `FEIL` eller `IKKE_IMPLEMENTERT`. |
| `beskrivelse`| String | Beskrivelse av resultatet.              |

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
  "hdirAvtale": {
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
  "hdirAvtale": {
    "status": "IKKE_IMPLEMENTERT",
    "beskrivelse": "Avtalevalidering er ikke implementert ennå"
  }
}
```
