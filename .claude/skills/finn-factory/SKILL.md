---
name: finn-factory
description: Kör en hel fabriksrunda med färska subagenter — en builder-pass, en reviewer-pass och en merge-pass i sekvens. Designad för /loop i en enda orkestratorsession; använd när fabriken ska snurra av sig själv utan separata terminaler för varje loop.
---

# Finn-loop fabriksorkestrator

En pass = en hel runda: build → review → merge, varje steg i en färsk
subagent med rent kontext. Orkestratorn startar subagenter och rapporterar;
den bygger, granskar eller mergar aldrig själv.

## Varför subagenter

Byggaren får inte granska sitt eget arbete i samma konversation. Färska
subagenter ger varje steg rent kontext — samma garanti som separata
sessioner, utan flera terminaler.

## Passet

Kör stegen sekventiellt, aldrig parallellt — de delar arbetsyta och
Linear/GitHub-state. Varje subagent startas synkront (inte i bakgrunden)
så nästa steg ser resultatet av det förra.

1. **Build**: starta en subagent med uppdraget: "Läs
   `.claude/skills/finn-build/SKILL.md` i repots rot och utför exakt en
   builder-pass enligt den. Arbetskatalogen är repots rot. Rapportera kort
   vad som gjordes och vilket ärende/PR som berördes."
2. **Review**: när buildern är klar, starta en NY subagent på samma sätt
   för `.claude/skills/finn-review/SKILL.md`.
3. **Merge**: när reviewern är klar, starta en NY subagent på samma sätt
   för `.claude/skills/finn-merge/SKILL.md`.
4. Sammanfatta rundan med en kort rad per steg: byggde X / granskade
   PR #N → verdict / mergade PR #N — eller "inget att göra".

## Under /loop: självpacing och stopp

Starta med `/loop /finn-factory` (utan intervall — loopen pacar sig själv).
Efter varje pass avgörs nästa steg:

- Något steg utförde arbete → nästa pass om 5–10 minuter.
- Rundan var tom men något väntar på människan — ett merge-ready-meddelande
  utan 🚀, eller en `blocked`-issue utan svar → fortsätt glest (~30 minuter);
  fabriken är i vänteläge, inte klar.
- Rundan var tom och inget väntar på människan i vare sig Slack, GitHub
  eller Linear → dagens arbete är klart: stoppa loopen och sammanfatta vad
  som byggdes, granskades och mergades under dagen.

## Regler

- Orkestratorn ändrar aldrig kod, labels eller state själv — bara
  subagenterna gör det, enligt sina respektive skills.
- Om ett steg misslyckas: rapportera felet i sammanfattningen och låt
  nästa loop-varv försöka igen. Eskalera till användaren först när samma
  fel återkommit tre varv i rad.
- Spec och plan förblir interaktiva människosteg; fabriken startar aldrig
  `/finn-spec` eller `/finn-plan`.
- Tom kö och inget att granska/merga är ett normalt resultat, inte ett fel:
  rapportera lugnt och avsluta passet.
