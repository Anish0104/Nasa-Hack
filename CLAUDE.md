# FlameAtlas: instructions for Claude Code

NASA Space Apps Challenge 2026, "Flame in Freefall". FlameAtlas ranks NASA's 24 PSI combustion
investigations for a crew environment (spacecraft, station, Moon, Mars, rover), shows which results
can fairly be compared, and proposes the experiment that would close the gap. Every number links to
its NASA source.

Team: Anish (AI: worker, search, Gemini, validator), Navneet (data: scraper, extraction, releases),
Juvana (science + evaluation: rules engine, Referee, benchmark). Frontend is shared.

## Repo layout

- `packages/engine/` TypeScript rules engine: Zod schema, units, ranking, Referee. Source of truth for types.
- `apps/worker/` Cloudflare Worker: `/api/search`, `/api/ask` (Gemini + citation validator + template fallback).
- `apps/web/` React + Vite + Tailwind v4 frontend.
- `ingest/` Python, offline: scrape PSI, parse PDFs, extract, normalize, release. `schema.py` mirrors the Zod schema.
- `data/` scenarios, tolerances, synonyms, eval questions, fixtures, releases.
- `docs/CONTRACTS.md` the three contracts: data format, `search()`, `POST /api/ask`.

## Commands

- `pnpm install` / `pnpm test` / `pnpm typecheck`
- `pnpm dev:worker` (http://localhost:8787) and `pnpm dev:web` (http://localhost:5173, proxies /api)
- Python: `cd ingest && pytest -q`
- Regenerate fixtures: `pnpm fixtures`

## Rules (always follow)

### Git
- Commits carry only the author's name. Never add `Co-Authored-By`, "Generated with Claude Code",
  or any other AI attribution line to commit messages or PR descriptions.
- Run `pnpm test` (and `pytest` if `ingest/` changed) before committing. Don't commit failing tests.
- Never commit secrets. `apps/worker/.dev.vars` holds `GEMINI_API_KEY` and is gitignored.

### Contracts
- Build against `docs/CONTRACTS.md`. Changing the data format, `SearchRequest`, `RankedResult`, or
  `AskResponse` means changing `packages/engine/src/schema.ts` / `contracts.ts` AND `ingest/schema.py`
  together, and telling the team. Ask before changing a contract.
- Unit math lives in `packages/engine/src/units.ts` and `ingest/normalize.py`; keep them in sync.

### Data integrity (this is the science pitch; get it right)
- `data/fixtures.json` is FAKE data for development. Never cite it, never present it as NASA data.
- Every displayed value needs a source (and page). Missing values are `null` with `missingReason`,
  never 0 and never guessed.
- `ASSUMED` scenario values are never shown as evidence. `o2PartialKPa` is always `COMPUTED` with its formula.
- Never invent numbers, sources, PSI IDs, DOIs, or NTRS IDs. If a value isn't in a source, say "not documented".
- Whoever collects a value never verifies it (`collectedBy` != `verifiedBy`).

### Product claims (keep these in UI text, prompts, and README)
- Never say a material is safe or unsafe. Lights and verdicts rate evidence coverage/strength, not safety.
- Earth (1 g) and orbit (~0 g) do not bound the Moon or Mars; lunar gravity may be more flammable.
- Burner gas is not a cabin atmosphere (CFI-G "85% O2"); liquid droplets are not solid fabric (FLEX-2).
- Say "no sustained, spacecraft-based lunar-gravity flame-spread data in PSI", not "no lunar data".
- FM2 is "targeting launch"; proposals "line up with FM2's design", never "predicted".
- Proposals are "illustrative, not an approved NASA protocol". Say "SI-powered" / "AI agents", never "superintelligence".

### AI layer
- LLM is Gemini (model in `apps/worker/wrangler.toml` `GEMINI_MODEL`). Gemini only proposes; every
  answer passes Zod + `validateSentences` (numbers must match cited records) or falls back to the template.
- Send Gemini only the retrieved, id-tagged records. Don't loosen the validator to make tests pass.

### Code style
- TypeScript strict, no `any` unless unavoidable. Small, typed functions. Vitest tests next to features.
- Prefer editing existing files over adding new abstractions. Keep the frontend on `src/lib/data.ts`
  and `ValueCell` for every value.
