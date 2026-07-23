---
name: finn-plan
description: Fråga vad som ska byggas, läs riktningen och live-läget i Linear/GitHub, och omvandla svaren till spec-intervjuer som fyller byggkön. Använd vid "vad ska vi bygga", morgonplanering, dagsplanering eller när byggkön är tom. Interaktiv — kräver användaren närvarande; loopas aldrig.
---

# Finn-loop planerare

Fyller fabrikens kö med rätt arbete. Planerar; startar aldrig byggare.
En session = ett dagspaket.

## 1. Läs läget innan du frågar

- `RIKTNING.md` i repots rot — mål och prioriteringar. Saknas den, säg det
  och erbjud att skapa den tillsammans först.
- Linear (team NEX): antal `agent-ready` (oassignade), `blocked`-issues med
  obesvarade frågor, backlog utan label.
- GitHub: öppna PR:ar och deras Finn-loop-labels, särskilt
  `needs-human-review`.

Visa en kort lägesbild (3–6 rader): vad som väntar på människan, vad som
ligger i kö, vad som är blockerat. Ställ aldrig frågor vars svar redan står
i RIKTNING.md, kodbasen eller Linear.

## 2. Fråga vad som ska byggas

Ställ en öppen fråga med konkreta förslag:

- 2–4 kandidater härledda ur RIKTNING.md, backloggen och senaste merges,
  med en rads motivering per kandidat och din rekommendation först.
- Alltid ett fritt alternativ: "något annat — beskriv".

Användaren väljer en eller flera idéer. Föreslå inte mer än vad som ryms i
kön på en dag — varje issue ska vara högst en dags agentarbete.

## 3. Speca varje vald idé

Kör finn-spec-processen för varje vald idé i tur och ordning: research i
kodbasen, intervjufrågor i rundor tills kontraktet är entydigt, fila issuen
i Linear (projektet i `.linear` om inget annat sägs), och ställ
agent-ready-frågan per issue.

Beroende idéer kedjas med blocked-by-relationer så en nedströmsissue inte
kan plockas före sin blockerare.

## 4. Avsluta med dagspaketet

Rapportera: filade issues (identifierare och länk), vilka som fick
`agent-ready`, och vad som väntar på svar från användaren.

Fråga sedan om fabriken ska startas: erbjud att köra
`/loop 10m /finn-factory` direkt i den här sessionen (en orkestratorloop
som kör build → review → merge med färska subagenter varje varv). Starta
bara efter ett explicit ja. Kör fabriksloopen redan, säg det istället för
att fråga.

## Hårda regler

- Planera; starta aldrig finn-build, finn-review eller finn-merge härifrån.
- Fila aldrig en issue användaren inte uttryckligen valt.
- `agent-ready` sätts bara efter ett explicit ja per issue — samma regel
  som i finn-spec.
