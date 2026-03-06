# MobEleView Extended
**Element & Race Anzeige für Monster – rAthena Server Emulator**

Zeigt Element-Icon, Element-Name, Level und Race über dem HP-Balken von Monstern.
Jeder Spieler kann seinen eigenen Anzeigemodus per `@mobeleview` setzen.

---

## Voraussetzungen

- rAthena (aktueller Stand)
- Client mit PACKETVER ≥ 20180207

---

## Anzeigemodi

| Modus | Anzeige              | Beispiel              |
|-------|----------------------|-----------------------|
| `0`   | Aus (Server-Default) | –                     |
| `1`   | Nur Icon             | 🔥 *(Sprite-Icon)*    |
| `2`   | Element-Name         | `Fire`                |
| `3`   | Element + Level      | `Fire Lv.2`           |
| `4`   | Nur Race             | `Plant`               |
| `5`   | Element + Race       | `Fire | Plant`        |
| `6`   | Alles                | `Fire Lv.2 | Plant`   |

**Server-Standard:** Modus `3` (in `monster.conf` änderbar)

---

## ZIP-Inhalt & Anwendung

```
MobEleView_Extended/
├── [1] SERVER - rAthena/
│   ├── conf/battle/
│   │   └── monster_mob_ele_view.conf       → ans Ende von monster.conf kopieren
│   └── src/
│       ├── custom/
│       │   ├── battle_config_struct.inc    → 1 Zeile einfügen
│       │   ├── battle_config_init.inc      → 1 Zeile einfügen
│       │   └── atcommand.inc               → Code einfügen + 2 manuelle Schritte
│       └── map/
│           └── clif_mob_ele_view_patch.cpp → Block in clif.cpp ersetzen
├── [2] CLIENT - data/
│   └── texture/유저인터페이스/group/
│       └── group_51..60.bmp                → in RO-Client-Ordner kopieren
└── [3] GUIDE/
    └── APPLY_GUIDE.html                    → vollständige Anleitung im Browser öffnen
```

---

## Server-Änderungen (8 Schritte)

### Schritt 1 – `conf/battle/monster.conf`
Öffne `conf/battle/monster.conf` und füge am **Ende der Datei** ein:

```
// Mob Element/Race View [Hyroshima/Extended]
// 0=Aus  1=Icon  2=Name  3=Name+Lv  4=Race  5=Ele+Race  6=Alles
mob_ele_view: 3
```

→ Inhalt der Datei `monster_mob_ele_view.conf` kopieren.

---

### Schritt 2 – `src/custom/battle_config_struct.inc`
Öffne die Datei und füge **innerhalb** des bestehenden Blocks ein:

```cpp
    // Mob Element/Race View [Hyroshima/Extended]
    int mob_ele_view;
```

---

### Schritt 3 – `src/custom/battle_config_init.inc`
Öffne die Datei und füge **innerhalb** des bestehenden Config-Blocks ein:

```cpp
    // Mob Element/Race View [Hyroshima/Extended]
    { "mob_ele_view",   &battle_config.mob_ele_view,   3,   0,   6,   },
```

---

### Schritt 4 – `src/map/pc.hpp` ⚠️ MANUELL
Öffne `src/map/pc.hpp`.  
Suche in der `map_session_data`-Struktur nach:

```cpp
char fakename[NAME_LENGTH];
```

Füge **direkt danach** ein:

```cpp
uint8 mob_ele_view_mode; ///< 0=Server-Standard, 1-6=Spieler-Override (@mobeleview)
```

> Falls `fakename` nicht gefunden wird: Die Zeile bei einem anderen `uint8`-Feld
> innerhalb von `map_session_data` einfügen (vor der schließenden Klammer `}`).

---

### Schritt 5 – `src/map/clif.cpp` ⚠️ MANUELL
> ⚠️ Falls **MobEleView_A.diff** bereits angewendet wurde:
> Entferne zuerst den alten Block (Variable-Deklarationen `ele_group`, `ele_name`, `output`
> und den `if(battle_config.mob_ele_view)` Block). Dann weiter mit Schritt 5b.

**Schritt 5a – Stelle finden:**
Suche in `clif_name()` (BL_MOB-Case) nach folgendem Abschnitt:

```cpp
#if PACKETVER_MAIN_NUM >= 20180207 || PACKETVER_RE_NUM >= 20171129 || PACKETVER_ZERO_NUM >= 20171130
        unit_data *ud = unit_bl2ud(bl);

        if (ud != nullptr) {
            memcpy(packet.title, ud->title, NAME_LENGTH);
            packet.groupId = ud->group_id;
        }
#endif
```

**Schritt 5b – Block ersetzen:**  
Ersetze **nur** den `if (ud != nullptr) { ... }` Block durch den Inhalt aus `clif_mob_ele_view_patch.cpp`.  
Die `#if` und `#endif` Zeilen bleiben unverändert!

---

### Schritt 6 – `src/custom/atcommand.inc`
Öffne `src/custom/atcommand.inc` und füge den **gesamten Inhalt** der mitgelieferten
`atcommand.inc` Datei ein.

---

### Schritt 7 – `src/map/atcommand.cpp` ⚠️ MANUELL
Suche nach dem `atcommand_basecommand_info[]` Array.  
Füge alphabetisch (z.B. nach `"mail"`) ein:

```cpp
ACMD_DEF(mobeleview),
```

---

### Schritt 8 – `conf/groups.conf` ⚠️ MANUELL
Füge bei der gewünschten Spielergruppe (oder allen Gruppen) ein:

```
        mobeleview: true
```

---

## Client-Seite

Kopiere den mitgelieferten `data/`-Ordner in dein **RO-Client-Verzeichnis**.

Die Icons werden gespeichert unter:
```
data\texture\유저인터페이스\group\group_51.bmp   (Neutral)
data\texture\유저인터페이스\group\group_52.bmp   (Water)
data\texture\유저인터페이스\group\group_53.bmp   (Earth)
data\texture\유저인터페이스\group\group_54.bmp   (Fire)
data\texture\유저인터페이스\group\group_55.bmp   (Wind)
data\texture\유저인터페이스\group\group_56.bmp   (Poison – extra)
data\texture\유저인터페이스\group\group_57.bmp   (Holy)
data\texture\유저인터페이스\group\group_58.bmp   (Shadow)
data\texture\유저인터페이스\group\group_59.bmp   (Ghost)
data\texture\유저인터페이스\group\group_60.bmp   (Undead)
```

**GRF-Setup:** Wenn du ein GRF-basiertes Setup verwendest, packe den `data/`-Ordner
als GRF (z.B. mit GRF Builder) und priorisiere ihn in deiner `DATA.INI`.

---

## Element → groupId Mapping

| Element | `def_ele` | `groupId` |
|---------|-----------|-----------|
| Neutral | 0         | 51        |
| Water   | 1         | 52        |
| Earth   | 2         | 53        |
| Fire    | 3         | 54        |
| Wind    | 4         | 55        |
| Poison  | 5         | 59        |
| Holy    | 6         | 57        |
| Shadow  | 7         | 58        |
| Ghost   | 8         | 59        |
| Undead  | 9         | 60        |

---

## Behobene Fehler vs. MobEleView_A.diff

| Fehler in _A.diff                                    | Fix in Extended                                      |
|------------------------------------------------------|------------------------------------------------------|
| Schreibt in `md->ud.group_id` (permanent, global)   | Schreibt nur in `packet.groupId` (lokal, flüchtig)  |
| Schreibt in `ud->title` (bleibt bis Mob-Respawn)     | Schreibt nur in `packet.title`                       |
| Kein Per-Spieler-Control                             | `@mobeleview` mit 7 Modi (0–6)                       |
| Keine Race-Anzeige                                   | Modi 4, 5, 6 zeigen Race                             |
| `ACMD()` statt `ACMD_FUNC()` – falsches Macro        | `ACMD_FUNC(mobeleview)` – korrekt für rAthena        |
| Title-Buffer 16 Byte (Overflow möglich)              | 64-Byte Zwischenpuffer + `safestrncpy` auf NAME_LENGTH|

---

## In-Game Befehle

```
@mobeleview          – Zeigt Hilfe und aktuellen Modus
@mobeleview 0        – Aus (Server-Standard aktiv)
@mobeleview 1        – Nur Element-Icon
@mobeleview 2        – Element-Name
@mobeleview 3        – Element-Name + Level
@mobeleview 4        – Nur Race
@mobeleview 5        – Element + Race
@mobeleview 6        – Alles
```

---

*MobEleView Extended · Basis: Hyroshima's Original · Fehler behoben & erweitert*
