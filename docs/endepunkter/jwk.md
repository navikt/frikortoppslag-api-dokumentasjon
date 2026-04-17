# Hent JWK-nøkler for kryptering

Endepunkt for å hente de offentlige nøklene som brukes til å kryptere request-payloaden med JWE.

---

## Endepunkt

| Felt         | Verdi                              |
|--------------|------------------------------------|
| **Path**     | `/api/frikortsporring/jwks`        |
| **Metode**   | `GET`                              |
| **Auth**     | Ingen — endepunktet er åpent       |

---

## Response

### HTTP-statuskoder

| Statuskode | Beskrivelse                               |
|------------|-------------------------------------------|
| `200 OK`   | JWK-sett returnert.                       |

### Response body (200 OK)

Responsen er et **JSON Web Key Set (JWKS)** med én eller flere offentlige RSA-nøkler:

```json
{
  "keys": [
    {
      "kty": "RSA",
      "use": "enc",
      "alg": "RSA-OAEP-256",
      "kid": "frikortbifrost-enc-20260312-1",
      "n": "sEe2Z3Xv...",
      "e": "AQAB",
      "exp": 1749724800
    }
  ]
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

---

## Nøkkelrotasjon og caching

Nøklene roteres jevnlig. Hver nøkkel har et `exp`-felt (Unix timestamp i sekunder) som angir utløpsdato.

**Anbefalt praksis for konsumenter:**

- Det er tillatt å cache nøkler lokalt for å unngå å hente dem ved hvert kall.
- Når en cachet nøkkel har utløpt (sjekk `exp`-feltet), må konsumenten hente oppdaterte nøkler fra dette endepunktet.
- Bruk alltid `kid`-feltet fra nøkkelen i JWE-headeren, slik at mottaker kan finne riktig nøkkel for dekryptering.
- JWKS-endepunktet kan returnere flere nøkler samtidig. Bruk den nøkkelen som har lengst tid til utløp.
