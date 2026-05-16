# Pelnora Blocks

Open library of Polish legal document boilerplates, curated by a Polish radca prawny.

## License
Apache 2.0 — use commercially, modify, redistribute. Attribution appreciated.

## Categories
- `B2B/` — umowy między podmiotami gospodarczymi (zlecenia, dzieło, NDA, ramowa)
- `RODO/` — klauzule informacyjne, zgody, RoPa, naruszenia
- `KPC/` — pisma procesowe (pozwy, sprzeciwy, wnioski)
- `wezwania/` — przedprocesowe wezwania do zapłaty/działania
- `KIO/` — zamówienia publiczne (odwołania, przystąpienia)
- `korporacyjne/` — uchwały, sprawozdania, wnioski KRS
- `praca/` — wypowiedzenia, ugody, oświadczenia
- `konsumenckie/` — reklamacje, odstąpienia

## Format
Each boilerplate has:
- `<slug>.docx` — document with `{{PLACEHOLDER}}` markers
- `<slug>.meta.yaml` — metadata, placeholder definitions, usage notes

## Usage
Designed for [Pelnora](https://pelnora.app) but format is open. Any docx editor + YAML parser works.

## Contributing
PRs welcome. New boilerplate? Include both .docx and meta.yaml. Polish only for now (English variants planned 2027).

## Disclaimer
Templates, not legal advice. Validate with a licensed practitioner before use.

