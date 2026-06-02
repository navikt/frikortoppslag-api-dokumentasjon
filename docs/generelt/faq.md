# Ofte stilte spørsmål (FAQ)

## Generelt

??? question "Kan tjenesten testes fra internett?"
    Ja. Testmiljøet er tilgjengelig fra internett på `https://frikortbifrost.ekstern.dev.nav.no`. Du trenger et gyldig HelseID-token fra testmiljøet for å autentisere deg.

??? question "Har dere en Slack-kanal?"
    Ja. Send e-post til [frikort.teknisk@nav.no](mailto:frikort.teknisk@nav.no) dersom du ønsker å bli lagt til.

??? question "Hvor finner jeg OpenAPI-spesifikasjonen?"
    OpenAPI 3.1-spesifikasjonen er tilgjengelig både som [YAML-fil](../frikortsporring-api.yaml) og via [Swagger UI](https://frikortbifrost.ekstern.dev.nav.no/swagger-ui/index.html).

## Kryptering

??? question "Hvorfor må requesten krypteres?"
    Selv om request-payloaden i seg selv kan virke uskyldig, avslører den at en borger har mottatt en bestemt helsetjeneste på en gitt dato. Dette regnes som sensitive personopplysninger. Helsedirektoratets policy krever ende-til-ende-kryptering av slike data i offentlig sky.

??? question "Trenger jeg et eget sertifikat for å signere requesten?"
    Nei. API-et bruker kun JWE-kryptering, ikke signering. Du trenger derfor ikke et eget sertifikat — kun den offentlige nøkkelen fra [JWKS-endepunktet](../endepunkter/jwk.md) for å kryptere requesten.

??? question "Hvor ofte roteres JWK-nøklene?"
    Nøklene kan roteres uten forvarsel. Klienter bør hente oppdaterte nøkler fra [JWKS-endepunktet](../endepunkter/jwk.md) jevnlig, og **ikke** hardkode nøkler lokalt. Nøklene leveres med en `exp`-verdi som angir utløpstidspunkt, og klienter bør sørge for å hente nye nøkler før de nåværende utløper. Nøklene kan gjerne caches.

## Autentisering

??? question "Vil spørringen kreve CPA?"
    Nei, tjenesten krever ingen CPA (samarbeidsavtale). Autentisering skjer utelukkende via HelseID.

??? question "Støttes vanlige Bearer-tokens?"
    Nei. API-et krever DPoP-baserte tokens. Vanlige Bearer-tokens blir avvist.

??? question "Hvordan registrerer jeg en klient i HelseID?"
    Opprett en klient i [NHNs selvbetjeningsportal](https://utviklerportal.nhn.no/informasjonstjenester/helseid/) med tilgang til API-et «Helsedirektoratets API for frikortspørring» og scope `hdir:frikortsporring/read`.

## Funksjonalitet

??? question "Støtter tjenesten mengdespørring (batch-oppslag)?"
    Nei. API-et tilbyr kun enkeltoppslag. Dersom du tidligere har brukt mengdespørring, må hvert oppslag nå gjøres som separate kall mot API-et.

??? question "Kan jeg slå opp historisk frikortstatus?"
    Ja, du kan angi en vilkårlig `tjenestedato` i requesten, men maks 2 år tilbake i tid. 

??? question "Hvilke tjenestetyper støttes?"
    Se [kodeverket](kodeverk.md) for en oversikt over gyldige tjenestetypekoder.
