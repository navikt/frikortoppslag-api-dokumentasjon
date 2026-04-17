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

Responsen er et **JSON Web Key Set (JWKS)** med én eller flere offentlige EC-nøkler:

```json
{
  "keys": [
    {
      "kty": "EC",
      "use": "enc",
      "alg": "ECDH-ES",
      "kid": "frikortbifrost-enc-20260312-1",
      "crv": "P-256",
      "x": "f83OJ3D2xF1Bg8vub9tLe1gHMzV76e8Tus9uPHvRVEU...",
      "y": "x_FEzRu9m36HLN_tue659LNpXW6pCyStikYjKIWI5a0..."
    }
  ]
}
```

### Felter i JWK

| Felt  | Type   | Beskrivelse                                                      |
|-------|--------|------------------------------------------------------------------|
| `kty` | String | Nøkkeltype — alltid `EC`.                                        |
| `use` | String | Tiltenkt bruk — alltid `enc` (kryptering).                       |
| `alg` | String | Algoritme — alltid `ECDH-ES`.                                    |
| `kid` | String | Nøkkel-ID. Må inkluderes i JWE-headeren ved kryptering.          |
| `crv` | String | Elliptisk kurve — alltid `P-256`.                                |
| `x`   | String | X-koordinat for EC-nøkkelen (Base64url-kodet).                   |
| `y`   | String | Y-koordinat for EC-nøkkelen (Base64url-kodet).                   |

---

## Nøkkelrotasjon og caching

Nøklene roteres jevnlig. Hver nøkkel har en utløpsdato.

**Anbefalt praksis for konsumenter:**

- Det er tillatt å cache nøkler lokalt for å unngå å hente dem ved hvert kall.
- Når en cachet nøkkel har utløpt (basert på utløpsdato), må konsumenten hente oppdaterte nøkler fra dette endepunktet.
- Bruk alltid `kid`-feltet fra nøkkelen i JWE-headeren, slik at mottaker kan finne riktig nøkkel for dekryptering.
- JWKS-endepunktet kan returnere flere nøkler samtidig. Bruk den nøkkelen som har lengst tid til utløp.
