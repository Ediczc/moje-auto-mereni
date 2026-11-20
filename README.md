# Měření z auta

Testovací stránka pro čtení stavu baterie a tachometru z VW ID.4 přes adaptér vLinker MC+ (Bluetooth LE). Běží v prohlížeči Bluefy na iPhonu. Jen čte data, do auta nic nezapisuje a nic neodesílá na internet.

Od verze 8 vypadá a ovládá se jako appka: spodní lišta **Auto · Nabití · Historie · Více**, stav spojení vpravo nahoře.

- **Auto** – nabití baterie (kruh se stářím údaje), tachometr, SoH; „Načíst z auta“ se samo připojí (nejdřív zkusí už povolený vLinker přes `getDevices()`, jinak výběr zařízení), potom „Uložit měření do iCloudu“.
- **Nabití** ve 3 krocích: Před (stav z auta nebo ručně, místo, nabíječka, začátek) → Nabíjí se (čas od začátku, klidně zavři Bluefy) → Po (stav po, kWh, u Tesla / Jiné cena, poznámka) → soubor `nabiti-*.json` do iCloud složky `MojeAuto`, appka Moje auto na Macu ho načte sama. Rozpracované nabití přežije zavření Bluefy.
- **Historie** – měření i nabití z tohoto telefonu, stav Uloženo / Neuloženo, klepnutí = uložit znovu.
- **Více** – připojení vLinkeru a nápověda, jmenovitá kapacita, záznam komunikace, vlastní příkaz.

Ceny ani ceník stránka neukazuje – cenu doplní appka na Macu. Formát souborů je stejný jako ve verzi 7. Písmo Outfit je ve složce `fonts/` (SIL OFL 1.1, `fonts/OFL-outfit.txt`), stránka nic nenačítá z internetu.

## Ověřeno na VW ID.4 (2021, 77 kWh) – 18. 11. 2026

Adaptér: vLinker MC+ (BLE název `vLinker MC-IOS`, služba `18F0`, notify `2AF0`, write `2AF1`), prohlížeč Bluefy 3.9.3, iOS 18.7.

| Údaj | Protokol | Hlavička / odpověď | Příkaz | Odpověď (příklad) | Výpočet | Výsledek |
|---|---|---|---|---|---|---|
| Využitelná kapacita → SoH | `ATSP6` (11 bit) | `ATSH710` / `ATCRA77A`, předtím `1003` | `222AB2` | `622AB205440400` | 2 B × 50 / 1000 kWh (CellPulse) = 4 B / 1310,77 (evDash) | 67,4 kWh → 87,5 % |
| Tachometr | `ATSP7` (29 bit) | `ATSH17FC0076` / `ATCRA17FE0076` | `22295A` | `62295A025D50` | 3 B = km | 154 960 km (**shodné s budíky**) |
| Nabití (BMS) | `ATSP7` (29 bit) | `ATSH17FC007B` / `ATCRA17FE007B` | `22028C` | `62028C42` | 1 B / 2,5 (displej ≈ raw × 0,4425 − 6,19, evDash) | 26,4 % (displej 23 % – **potvrzeno**) |

Nefunguje: `22295A` na 710 (7F 22 31), `01A6` na 7DF, jednotky 714 a 7E0 na 11 bit (NO DATA).
Zdroje: [CellPulse – SOH na MEB](https://cellpulse.pages.dev/blog/meb-soh-atsp6), [evDash CarVWID3.cpp](https://github.com/nickn17/evDash/blob/master/src/CarVWID3.cpp).
