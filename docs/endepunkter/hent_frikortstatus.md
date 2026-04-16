# Hent status på frikort

Endepunkt på status på frikort for oppgitt borger.
Svaret viser status på om borger har frikort eller ikke for oppgitt år.

| Felt                   | Verdi                                                   |
|------------------------|---------------------------------------------------------|
| path                   | /api/frikortsporring/helseid/v1                         |
| method                 | POST                                                    |
| content type           | application/json                                        |


## Response

| Status | Response | Forklaring        |
|--------|----------|-------------------|
| 200    | JSON     |                   |
| 401    |          | Uautorisert       |
| 501    |          | Ikke implementert |

Eksempel på request (url, eller json body?):

```
```
Eksempel på response:

```json
[
  {
    "response": "true"
  }
]
```