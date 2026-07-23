# Changelog

Alla nämnvärda ändringar i det här repot dokumenteras i den här filen.
Formatet följer [Keep a Changelog](https://keepachangelog.com/en/1.1.0/);
sektioner dateras istället för att versionsnumreras.

## Ej släppt

### Tillagt

- Loop-stuck-spärr (NEX-803): GitHub-labeln `loop-stuck`; `finn-review`
  stoppar en PR när ett nytt must-fix-verdikt skulle bli det tredje i raden
  (två misslyckade fixrundor), sätter `loop-stuck` + `needs-human-review`
  och larmar med en `Fastnad:`-rad i `#notifs-factory`; `finn-build` och
  `finn-review` hoppar över stoppade PR:ar tills en människa tar bort
  labeln (räkningen nollställs då), och `finn-factory` räknar dem som
  vänteläge istället för "dagen klar".

## 2026-07-23

### Tillagt

- Finn-loop-skills installerade för team NEX i `.claude/skills/`:
  `finn-spec` (spec-intervju med agent-ready-fråga i chatten), `finn-build`,
  `finn-review` (utökad med Slack merge-ready-notis), `finn-merge`
  (🚀-reaktion i `#notifs-merge-ready` → verifierad squash-merge → ✅) och
  `finn-plan` (dagsplanering utifrån `RIKTNING.md`).
- `CLAUDE.md` med svenska som arbetsspråk och fabrikens konfiguration.
- `RIKTNING.md` som mål- och prioriteringskälla för `finn-plan`.
- `.linear`-koppling till Linear-projektet "Nexar-Loops tests".

### Ändrat

- Repot bytte hem till `NexarPersonal/Nexar-Loops` (publikt), med
  `finna/Finn-loop` kvar som `upstream`.

## 2026-07-22

### Tillagt

- Starter-kitet från upstream ([finna/Finn-loop](https://github.com/finna/Finn-loop)):
  skillsen spec/build/review, install-prompt i README, härdning för publikt
  bruk samt valideringsscript (`scripts/validate.mjs`) med GitHub
  Actions-workflow.
