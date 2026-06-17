# Hent JWK for kryptering

Endepunkt for å hente den offentlige nøkkelen som brukes til å kryptere request-payloaden med JWE.

---

## Endepunkt

| Felt         | Verdi                              |
|--------------|------------------------------------|
| **Path**     | `/api/frikortsporring/jwk`         |
| **Metode**   | `GET`                              |
| **Auth**     | Ingen — endepunktet er åpent       |

---

## Response

### HTTP-statuskoder

| Statuskode | Beskrivelse                               |
|------------|-------------------------------------------|
| `200 OK`   | JWK returnert.                            |
| `404 Not Found` | Ukjent API-path.                      |
| `405 Method Not Allowed` | HTTP-metode er ikke tillatt for dette endepunktet. |

### Response body (200 OK)

Responsen er én **JSON Web Key (JWK)** med offentlig RSA-nøkkel — nøkkelen med lengst gjenværende levetid:

```json
{
  "kty": "RSA",
  "use": "enc",
  "alg": "RSA-OAEP-256",
  "kid": "frikortbifrost-enc-20260312-1",
  "n": "sEe2Z3Xv...",
  "e": "AQAB",
  "exp": 1749724800
}
```

### Felter i JWK

| Felt  | Type   | Beskrivelse                                           |
|-------|--------|-------------------------------------------------------|
| `kty` | String | Nøkkeltype — alltid `RSA`.                            |
| `use` | String | Tiltenkt bruk — alltid `enc` (kryptering).            |
| `alg` | String | Algoritme — alltid `RSA-OAEP-256`.                    |
| `kid` | String | Nøkkel-ID. Må inkluderes i JWE-headeren ved kryptering. |
| `n`   | String | RSA modulus (Base64url-kodet).                         |
| `e`   | String | RSA eksponent (Base64url-kodet).                       |
| `exp` | Number | Utløpstidspunkt for nøkkelen (Unix timestamp i sekunder). |

### Response headers

| Header  | Beskrivelse                                                        |
|---------|--------------------------------------------------------------------|
| `Allow` | Kun ved 405 — HTTP-metodene som er tillatt for dette endepunktet.  |

---

## Nøkkelrotasjon og caching

Nøklene roteres jevnlig. Hver nøkkel har et `exp`-felt (Unix timestamp i sekunder) som angir utløpsdato.

**Anbefalt praksis for konsumenter:**

- Det er tillatt å cache nøkkelen lokalt for å unngå å hente den ved hvert kall.
- Når en cachet nøkkel har utløpt (sjekk `exp`-feltet), må konsumenten hente oppdatert JWK fra dette endepunktet.
- Bruk alltid `kid`-feltet fra nøkkelen i JWE-headeren, slik at mottaker kan finne riktig nøkkel for dekryptering.
