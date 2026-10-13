# RSL-logboek

Een persoonlijk logboek voor Seppe's Raid: Shadow Legends-account. Het toont:
- de actieve opdracht met afvinkbare taken;
- Kaels stats tegenover zijn doelen;
- teams voor dungeons, clan bosses en andere spelonderdelen;
- de kerncampions en het volledige roster.

## Opbouw
- `index.html`: de site. Ze leest alles uit `data.json`.
- `data.json`: alle accountdata. Claude werkt dit bestand bij na elke sync met de HellHades Optimizer.

Vinkjes bij taken bewaart je browser lokaal (localStorage). Ze gaan niet terug naar de repo.

## Lokaal bekijken
`fetch` werkt niet vanuit een los bestand, dus start een kleine webserver:

```bash
python3 -m http.server 8000
```

Open daarna http://localhost:8000.
