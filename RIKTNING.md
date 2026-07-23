# Riktning

Vad Nexar-Loops ska bli och i vilken ordning. `finn-plan` läser den här
filen för att föreslå dagens arbete — det här är källan till "vad ska vi
bygga". Uppdatera den när målen ändras; bara människor redigerar den.

## Mål

- Ett eget AI-byggsystem för Nexar, vidareutvecklat från Finn-loop-kitet:
  idé → spec → autonomt bygge → autonom granskning → 🚀-merge från Slack.
- Fabriken ska så småningom kunna driva flera repon (t.ex. Nexar
  Projektplattform), inte bara det här.

## Prioriterad utbyggnad

1. Verifiera hela kedjan med testärendet NEX-802 (CHANGELOG.md)
2. (förslag ur README:s utbyggnadsordning — ändra fritt:
   fresh-reviewer-konvergens med loop-stuck-spärr, status-skill som listar
   vad människan måste göra, blocked-frågor till Slack, watchdog)

## Beslutade policyer

Stående beslut — spec- och planintervjuer ställer ALDRIG frågor vars svar
står här; de viker in policyn i utkastet och noterar vilken som tillämpats.
Bara människor lägger till eller ändrar rader här. Genuina produktbeslut
för nya features defaultas aldrig — de frågas alltid.

- Fixrundor: 2 per PR; tredje underkända granskningen ⇒ `loop-stuck`-label,
  PR:en lämnar den automatiska kön, en rad postas i `#notifs-factory`.
- Statuslistor och köfrågor avgränsas till projektet i `.linear`, inte hela
  NEX-teamet.
- Merge sker endast via 🚀-reaktion från godkänt founder-ID; ✅ i tråden
  som kvitto. Agenter mergar aldrig på eget initiativ.
- Allt arbete, alla issues, PR:ar och Slack-meddelanden på svenska.
- Varje issue ryms i högst en dags agentarbete; större saker kedjas med
  blocked-by-relationer.

## Inte nu

- Auto-merge utan 🚀-reaktion — människan förblir merge-gaten.
- (fyll på med sådant som medvetet får vänta)
