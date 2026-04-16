# JWE-kryptering av POST-requests – `POST/JWE`

API-et bruker **JWE (JSON Web Encryption)** for å kryptere enkelte POST-forespørsler.
Det betyr at klienten krypterer forespørselen med vår offentlige nøkkel fra et publisert sertifikat.

**Merk:** JWE-signering benyttes **ikke**. Kun kryptering er i bruk. Det betyr at klienten ikke trenger å ha et eget sertifikat for signering.

---

## Hva er JWE?

**JSON Web Encryption (JWE)** er et URL-sikkert og kompakt format for kryptering av strukturert data ved hjelp av JSON-baserte mekanismer.
Formatet brukes når man trenger å sende sensitive data kryptert, for eksempel i en POST-request til et API.

Et JWE består av fem deler, separert med punktum:

```
<protected_header>.<encrypted_key>.<initialization_vector>.<ciphertext>.<authentication_tag>
```

---

## Komponenter i en JWE

### 1. Protected Header

Inneholder metadata som beskriver hvordan JWE er kryptert, og hvilken type innhold det bærer.

Eksempel:

```json
{
  "alg": "RSA-OAEP-256",
  "enc": "A256GCM",
  "kid": "12345",
  "typ": "JWT",
  "cty": "JWT",
  "zip": "DEF"
}
```

Forklaring på feltene:

* `alg` – Asymmetrisk algoritme brukt for å kryptere nøkkelen (f.eks. `RSA-OAEP-256`)
* `enc` – Symmetrisk algoritme brukt på innholdet (f.eks. `A256GCM`)
* `kid` – ID til nøkkelen som brukes (matcher en JWK fra vårt nøkkelsett)
* `typ` – Typen token, ofte `JWT`
* `cty` – Content type (innholdstype), ofte også `JWT`
* `zip` – Valgfri kompresjonsalgoritme brukt før kryptering (f.eks. `DEF`)

---

### 2. Encrypted Key (CEK)

En tilfeldig generert symmetrisk nøkkel (Content Encryption Key) brukes til å kryptere selve innholdet.
Denne nøkkelen krypteres med mottakerens offentlige nøkkel (asymmetrisk).

---

### 3. Initialization Vector (IV)

En tilfeldig verdi som brukes av den symmetriske algoritmen for å sikre unik kryptering for hver melding.
Må være forskjellig for hver forespørsel.

---

### 4. Ciphertext (Payload)

Selve dataene som krypteres. I APIet er dette en **nested JWT**.

Struktur på JWT:

```
<header>.<payload>.<signature>
```

Eksempel på dekryptert payload:

```json
{
  "praksisId": "1000005649",
  "behandlerkrav": { }
}
```

Payload signeres **ikke** av klienten i dette oppsettet. Kun kryptering brukes. (Noen implementasjoner av JWE støtter både signering og kryptering, men det er ikke i bruk her.)

---

### 5. Authentication Tag

Brukes ved autentisert kryptering (f.eks. `AES-GCM`) for å sikre at innholdet ikke er endret under transport.
Dette er en integritetsbeskyttelse, ikke en signatur.

---

## 🔗 Nyttige lenker

* [RFC 7516 – JWE-spesifikasjon](https://datatracker.ietf.org/doc/html/rfc7516)
* [Scott Brady – JWE forklaring](https://www.scottbrady.io/jose/json-web-encryption)
* [Nimbus JOSE + JWT eksempler](https://connect2id.com/products/nimbus-jose-jwt/examples/signed-and-encrypted-jwt)