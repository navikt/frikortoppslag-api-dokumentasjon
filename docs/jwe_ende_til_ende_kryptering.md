# JWE-kryptering av request

API-et bruker **JWE (JSON Web Encryption)** for å kryptere request-body i POST-forespørsler.
Klienten krypterer JSON-payloaden med vår offentlige nøkkel, hentet fra [JWKS-endepunktet](endepunkter/jwk.md).

**Merk:** JWE-signering benyttes **ikke**. Kun kryptering er i bruk. Klienten trenger ikke et eget sertifikat for signering.

---

## Algoritmer

| Parameter | Verdi           | Beskrivelse                                 |
|-----------|-----------------|---------------------------------------------|
| `alg`     | `RSA-OAEP-256`  | Asymmetrisk algoritme for kryptering av CEK |
| `enc`     | `A256GCM`       | Symmetrisk algoritme for kryptering av innhold |

---

## Steg-for-steg: Kryptere en request

### 1. Hent offentlig nøkkel

Gjør et `GET`-kall til [/api/frikortsporring/jwks](endepunkter/jwk.md) for å hente gjeldende JWKS.
Velg en nøkkel fra `keys`-arrayet. Noter `kid`-verdien — den må inkluderes i JWE-headeren.

### 2. Bygg JSON-payload

Lag JSON-objektet som skal krypteres, f.eks.:

```json
{
  "borgerIdent": "12345678901",
  "tjenestetypeKode": "LE",
  "tjenestedato": "2026-06-15"
}
```

### 3. Krypter med JWE

Krypter JSON-payloaden til en **JWE Compact Serialization**-streng med følgende header-parametre:

```json
{
  "alg": "RSA-OAEP-256",
  "enc": "A256GCM",
  "kid": "<kid fra JWK>"
}
```

Resultatet er en streng med fem Base64url-kodede deler separert med punktum:

```
<protected_header>.<encrypted_key>.<initialization_vector>.<ciphertext>.<authentication_tag>
```

### 4. Send request

Send JWE-strengen som request-body med `Content-Type: application/jose`:

```http
POST /api/frikortsporring/helseid/v1 HTTP/1.1
Host: frikortbifrost.nav.no
Authorization: DPoP eyJ...
DPoP: eyJ...
Content-Type: application/jose

eyJhbGciOiJSU0EtT0FFUC0yNTYiLCJlbmMiOiJBMjU2R0NNIiwia2lkIjoiZnJpa29ydGJpZnJvc3QtZW5jLTIwMjYwMzEyLTEifQ.OKOawDo13gRp2ojaHV7LFpZcgV7T6DVZKTyKOMTYUmKoTCVJRgckCL9kiMT03JGe...
```

Responsen returneres som **ukryptert JSON** (`application/json`).

---

## Hva er JWE?

**JSON Web Encryption (JWE)** er et kompakt format for kryptering av data ved hjelp av JSON-baserte mekanismer, definert i [RFC 7516](https://datatracker.ietf.org/doc/html/rfc7516).

### Komponenter i en JWE Compact Serialization

| Del | Beskrivelse |
|-----|-------------|
| **Protected Header** | Metadata om krypteringen (`alg`, `enc`, `kid`). Base64url-kodet. |
| **Encrypted Key** | Tilfeldig generert symmetrisk nøkkel (CEK), kryptert med mottakerens offentlige nøkkel. |
| **Initialization Vector** | Tilfeldig verdi for unik kryptering per melding. Må være unik for hver request. |
| **Ciphertext** | Den krypterte JSON-payloaden. |
| **Authentication Tag** | Integritetsbeskyttelse (AES-GCM) som sikrer at innholdet ikke er endret under transport. |

---

## Kodeeksempel (Java med Nimbus JOSE + JWT)

```java
import com.nimbusds.jose.*;
import com.nimbusds.jose.crypto.RSAEncrypter;
import com.nimbusds.jose.jwk.*;

// 1. Hent JWKS fra endepunktet
JWKSet jwkSet = JWKSet.load(new URL("https://frikortbifrost.nav.no/api/frikortsporring/jwks"));

// 2. Velg nøkkel
RSAKey rsaKey = (RSAKey) jwkSet.getKeys().get(0);

// 3. Bygg JWE-header
JWEHeader header = new JWEHeader.Builder(JWEAlgorithm.RSA_OAEP_256, EncryptionMethod.A256GCM)
    .keyID(rsaKey.getKeyID())
    .build();

// 4. Opprett JWE-objekt med JSON-payload
String jsonPayload = """
    {
      "borgerIdent": "12345678901",
      "tjenestetypeKode": "LE",
      "tjenestedato": "2026-06-15"
    }
    """;

JWEObject jweObject = new JWEObject(header, new Payload(jsonPayload));

// 5. Krypter
jweObject.encrypt(new RSAEncrypter(rsaKey));

// 6. Serialiser til JWE Compact Serialization
String jweString = jweObject.serialize();

// 7. Send jweString som request-body med Content-Type: application/jose
```

**Maven-avhengighet:**

```xml
<dependency>
    <groupId>com.nimbusds</groupId>
    <artifactId>nimbus-jose-jwt</artifactId>
</dependency>
```

---

## 🔗 Nyttige lenker

* [RFC 7516 – JWE-spesifikasjon](https://datatracker.ietf.org/doc/html/rfc7516)
* [Scott Brady – JWE forklaring](https://www.scottbrady.io/jose/json-web-encryption)
* [Nimbus JOSE + JWT eksempler](https://connect2id.com/products/nimbus-jose-jwt/examples/signed-and-encrypted-jwt)