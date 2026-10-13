# RSL-logboek

Een persoonlijk logboek voor Seppe's Raid: Shadow Legends-account. Eén pagina (`index.html`) met hash-routing:

| Route | Inhoud |
|---|---|
| `#/overzicht` | Account (clan, league, campagnesterren, Great Hall), actieve opdracht met afvinkbare taken, Kaels stats tegenover doelen, optimizer-voorstel, kerncampions, vaste regels |
| `#/champions` | Volledig roster (filter, zoeken, sorteren) en links naar de detailpagina's |
| `#/champions/<slug>` | Detailpagina per champion met een `profile` (kael, dorothy-gale, sun-wukong, warpriest, diabolist, soulbond-bowyer) |
| `#/teams/<id>` | Teams per spelonderdeel met drops, affiniteit, stages, mechaniek, doelen en progressieplan |
| `#/gear` | Kaels gear per slot, ontbrekende stukken, ongebruikte stukken, sets op het account |
| `#/gids` | Begrippen (buffs, debuffs, mechanieken), affiniteitsdriehoek, rank-up, ascension, masteries, food |
| `#/bronnen` | Alle bronnen met link en wat ze staven (`#/bronnen/<id>` springt naar één bron) |

Geen build-stap, geen frameworks, geen externe scripts. Alleen Google Fonts.
Vinkjes, gekozen thema en laatst gekozen team bewaart de browser lokaal (`localStorage`: `rsl-done`, `rsl-theme`, `rsl-dsel`). Ze gaan niet terug naar de repo.

## Lokaal bekijken
`fetch` werkt niet vanuit een los bestand, dus start een kleine webserver:

```bash
python3 -m http.server 8000
```

Open daarna http://localhost:8000.

## Schema van `data.json`
`data.json` is de enige databron. Claude werkt dit bestand bij na elke sync met de HellHades Optimizer.

### Regels voor bijwerken
- Speltekst uit de export gaat voor op websites. Claimt een website meer, zet het dan in `extra` met de zin "Niet bevestigd voor dit account".
- Elke niet-triviale claim krijgt een `src`: een lijst van id's uit `sources`. Een nieuwe bron voeg je toe aan `sources`; de volgorde bepaalt het nummer `[n]` op de site.
- Gear uit de export heet "zoals gerapporteerd": slotnamen en hoofdstats zijn vertaalde interne codes.
- Een set-id krijgt alleen een naam in `gear.sets` met `confirmed: true` als die bevestigd is; anders toont de site "Set #id".
- Velden die ontbreken, laat de site weg of toont ze als "–". Gebruik `null` voor "niet in de export".

### `meta`
| Veld | Type | Betekenis |
|---|---|---|
| `name` | string | Accountnaam |
| `synced` | string | Datum van de laatste sync, vrije tekst |
| `arena` | number | League-code uit de export |
| `arenaNote` | string | Toelichting bij de league-code |
| `campaign` | `{normal, hard, brutal, nightmare}` | Campagnesterren (max 252 per moeilijkheid) |
| `account` | object | `clan`, `createdOn`, `factionWarsStars`, `greatHall: [{affinity, stat, level}]`, `src` |
| `missionTitle` | string | Titel van de actieve opdracht |
| `report` | string | Wat Seppe moet terugrapporteren |
| `kael.current` / `kael.proposed` | `{HP, ATK, DEF, SPD, CRATE, CDMG, RES, ACC}` | Totalen uit de optimizer-UI; `proposed` is niet geëquipt |
| `kael.proposedNote` | string | Uitleg bij het voorstel |
| `kael.targets` | `[{stat, label, value, unit, why, src}]` | Meters op het overzicht; `stat` is een sleutel uit `current` |
| `kael.src` | string[] | Bron-id's |
| `gearFixes` | string[] | Correcties op Kaels gear |
| `rules` | `[{t, d}]` | Vaste regels (titel, uitleg) |

### `tasks[]`
`{id, order, text, done}`. `done` is de standaardwaarde; de browser overschrijft hem met de lokale vinkjes.

### `champions[]`
| Veld | Type | Betekenis |
|---|---|---|
| `id` | string | `c01` … `c47`, stabiel tussen syncs |
| `name`, `faction`, `affinity` | string | `affinity`: Magic, Spirit, Force of Void |
| `rarity` | 1-5 | Common … Legendary |
| `grade`, `level` | number | Rank (sterren) en level |
| `role` | `kern` \| `bewaren` \| `food` | Rol op het account |
| `prio` | number | Alleen bij kern: volgorde op het overzicht |
| `note`, `detail`, `next` | string | Korte notitie (roster), langere tekst en volgende stap (kernkaarten) |
| `slug` | string | Optioneel. Maakt `#/champions/<slug>` aan; vereist `profile` |
| `profile` | object | Optioneel, zie hieronder |

`profile`:
| Veld | Type | Betekenis |
|---|---|---|
| `roleText` | string | Rol op dit account |
| `aura` | `{text, area, src}` of `null` | Aura met gebied ("Alle gevechten", "Alleen Arena") |
| `base` | `{HP, ATK, DEF, SPD, RES, ACC?, CRATE?, CDMG?}` | Basisstats uit de export |
| `ascension`, `masteries` | number of `null` | `null` = niet in de export |
| `books` | `{used, max, levels}` | Gebruikte en maximale boeken, skillniveaus als tekst |
| `skills[]` | `{key, name, cd, textEn, nl, books?, extra?: [{text, src}], src}` | `key`: A1/A2/A3/P; `cd`: cooldown (0 = geen, `null` = passief); `nl`: eigen Nederlandse uitleg; `extra`: waarschuwingen of niet-bevestigde claims |
| `gearAsReported[]` | `{slot, set, stars, lvl, main, note?}` | `set`: set-id of `null`; `lvl`: upgradeniveau of `null` |
| `ascendNext` | `{raw: {"<potion>": n}, note, src}` | Potions voor de volgende ascension zoals de export ze geeft |
| `recMasteries` | `{text, src}` | Aanbevolen masteries |
| `recGear` | `{sets, slots, priority, src}` | Aanbevolen sets, hoofdstats per slot en stat-prioriteit |
| `next`, `nextSrc` | string, string[] | Volgende stap |

### `content[]` (teams per spelonderdeel)
| Veld | Type | Betekenis |
|---|---|---|
| `id`, `name`, `short` | string | `id` wordt de route `#/teams/<id>` |
| `group` | string | `Dungeons` (standaard), `Clan bosses`, `Overig` |
| `order` | number | Volgorde |
| `status` | `farm` \| `proberen` \| `later` \| `niet` | Statuslabel |
| `purpose` | string | Waarom dit onderdeel |
| `drops`, `affinity`, `stages` | string | Wat het oplevert, affiniteit (per stage), stages of moeilijkheden |
| `mechanics`, `needs`, `targets`, `plan` | string[] | Mechaniek, wat het team nodig heeft, stat-doelen, progressie voor dit account |
| `note` | string | Advies |
| `team[]` | `{cid, role}` | `cid` verwijst naar `champions[].id`. Alleen champions uit het roster |
| `src` | string[] | Bron-id's |

### `gear`
| Veld | Type | Betekenis |
|---|---|---|
| `kaelNote` | string | Waarschuwing bij Kaels gear (Kaels slots staan in zijn `profile.gearAsReported`) |
| `unequipped[]` | `{text, set}` | Opvallende ongebruikte stukken |
| `unequippedTotal` | string | Totaal ongebruikte stukken |
| `missing`, `missingSrc` | string[] | Wat ontbreekt |
| `sets[]` | `{id, name, internal, bonus, confirmed, src}` | Sets op het account; `name` alleen als `confirmed` |
| `setsNote` | string | Hoe de set-id's gekoppeld zijn |
| `mainStats[]`, `mainStatsSrc` | `{slot, stats}` | Mogelijke hoofdstats per slot |

### `guide`
| Veld | Type | Betekenis |
|---|---|---|
| `terms[]` | `{term, kind, nl, src}` | `kind`: `debuff`, `buff`, `effect` of `mechaniek` |
| `affinity` | `{triangle: [{a, beats}], void, effects[], src}` | Affiniteitsdriehoek |
| `topics[]` | `{id, title, body[], src}` | Uitlegblokken (rank-up, ascension, masteries, food) |

### `sources[]`
`{id, title, url, supports}`. `supports` beschrijft wat de bron staaft.

## Sync-procedure (verplicht bij elke sync)
Na elke sync met de HellHades Optimizer:
1. Werk de accountstand bij: `meta` (campagne, `kael.current`), `tasks`, en de rank, level en gear van de `champions`.
2. Zet **bovenaan** `syncs[]` een nieuw item. De site toont het nieuwste item als "Laatste sync" op Overzicht, en alle items op `#/syncs`.

```json
{
  "id": "2026-10-13-1540",
  "date": "13 okt 2026, 15:40",
  "title": "Kael is Rank 6",
  "highlights": ["belangrijkste punten, 1-3"],
  "changes": ["wat veranderde sinds de vorige sync"],
  "notes": ["kanttekeningen en kritiek"],
  "next": ["volgende stappen"],
  "kael": {"grade": 6, "level": 60, "HP": 0, "ATK": 0, "DEF": 0, "SPD": 0, "CRATE": 0, "CDMG": 0, "RES": 0, "ACC": 0},
  "campaignHard": 103
}
```

De tabel "Kaels evolutie" op `#/syncs` wordt automatisch opgebouwd uit `syncs[].kael`.
