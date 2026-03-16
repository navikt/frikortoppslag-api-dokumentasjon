# Slå opp frikortstatus

## Endepunkt

```
POST /frikort/oppslag/v1/frikortstatus
```

**Autentisering:** Maskinporten-token med scope `nav:frikort/oppslag`

---

## Request

### Headers

| Header          | Verdi                           |
|-----------------|---------------------------------|
| `Authorization` | `Bearer <maskinporten-token>`   |
| `Content-Type`  | `application/json`              |

### Request body

```json
{
  "fnr": "12345678901",
  "dato": "2025-01-15"
}
```

### Felter i request body

| Felt   | Type   | Påkrevd | Beskrivelse                                                            |
|--------|--------|---------|------------------------------------------------------------------------|
| `fnr`  | String | Ja      | Fødselsnummer (11 siffer) til pasienten det gjøres oppslag for.       |
| `dato` | String | Ja      | Dato det gjøres oppslag for, på formatet `YYYY-MM-DD`.                |

---

## Response

### HTTP-statuskoder

| Statuskode | Beskrivelse                                                     |
|------------|-----------------------------------------------------------------|
| `200 OK`   | Oppslaget var vellykket. Se response body for frikortstatus.   |
| `400 Bad Request` | Ugyldig request, f.eks. ugyldig fødselsnummer eller datoformat. |
| `401 Unauthorized` | Manglende eller ugyldig autentisering.                    |
| `403 Forbidden`    | Tokenet mangler nødvendig scope.                          |
| `500 Internal Server Error` | Feil på serveren.                                |

### Response body (200 OK)

```json
{
  "harFrikort": true,
  "frikortType": "EGENANDELSTAK1",
  "gyldigFra": "2025-01-01",
  "gyldigTil": "2025-12-31"
}
```

### Felter i response body

| Felt          | Type    | Beskrivelse                                                                           |
|---------------|---------|---------------------------------------------------------------------------------------|
| `harFrikort`  | Boolean | Angir om pasienten har gyldig frikort på angitt dato.                                |
| `frikortType` | String  | Type frikort. Se [kodeverk](kodeverk.md) for gyldige verdier.                        |
| `gyldigFra`   | String  | Dato frikortperioden starter, på formatet `YYYY-MM-DD`. Null dersom `harFrikort` er `false`. |
| `gyldigTil`   | String  | Dato frikortperioden slutter, på formatet `YYYY-MM-DD`. Null dersom `harFrikort` er `false`. |

---

## Eksempel

### Request

```http
POST /frikort/oppslag/v1/frikortstatus HTTP/1.1
Host: api.helserefusjon.no
Authorization: Bearer eyJ...
Content-Type: application/json

{
  "fnr": "12345678901",
  "dato": "2025-06-15"
}
```

### Response (pasient har frikort)

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "harFrikort": true,
  "frikortType": "EGENANDELSTAK1",
  "gyldigFra": "2025-05-01",
  "gyldigTil": "2025-12-31"
}
```

### Response (pasient har ikke frikort)

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "harFrikort": false,
  "frikortType": null,
  "gyldigFra": null,
  "gyldigTil": null
}
```
