# Kodeverk

## Tjenestetypekoder

Feltet `tjenestetypeKode` i request angir hvilken type tjeneste oppslaget gjelder for.

### Utlevering på blå resept

Gjelder apotek og bandasjister. For disse tjenestetypene gjelder fritak også for minstepensjonister etter blåreseptregelverket.

| Kode | Beskrivelse    |
|------|----------------|
| `A`  | Apotek         |
| `S`  | Sykehusapotek  |
| `B`  | Bandasjist     |

### Behandling

Gjelder behandlere og helseinstitusjoner. Fritak gjelder borgere som er innvilget frikort.

| Kode | Beskrivelse      |
|------|------------------|
| `LE` | Lege             |
| `HS` | Helsestasjon     |
| `PO` | Poliklinikk      |
| `LR` | Lab/røntgen      |
| `PS` | Psykolog         |
| `PR` | Pasientreiser    |
| `BR` | Behandlingsreiser|
| `TB` | Tannbehandling   |
| `RH` | Rehabilitering   |
| `FT` | Fysioterapi      |
