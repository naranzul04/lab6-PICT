# Лаборатори №6: Тест тохиолдол боловсруулах — Category-Partition ба PICT

Программ хангамжийн чанарын баталгаа ба тест (F.CSA313)

**B222270823 А. Наранзул**

---

## Хэсэг А: Category-Partition арга (гараар)

Тестлэх функц: `POST /transfer { "from": ID, "to": ID, "amount": NUM, "currency": CODE }`  
Амжилт: `201 {"result":"OK"}` · Оролтын алдаа: `200 {"result":"ERROR_..."}`

**Тестийн орчны тохиргоо (бүх тестэд ижил):**
- `ACC-A` — идэвхтэй данс, валют MNT, үлдэгдэл 10000 MNT, өдрийн лимит 1000 MNT
- `ACC-B` — идэвхтэй данс (хүлээн авагч), валют MNT
- `ACC-F` — царцаагдсан данс
- `ACC-999` — байхгүй данс
- Ханш: 1 USD = 10 MNT (`Different` валют нь USD)

### Даалгавар A1 — Сонголт ба төлөөлөх утгууд

| Параметр | Сонголт | Төлөөлөх утга | Тайлбар |
| :--- | :--- | :--- | :--- |
| **from** | `Valid` | `ACC-A` | Хүчинтэй, идэвхтэй илгээгч данс |
| | `Frozen` | `ACC-F` | Царцаагдсан данс |
| | `Missing` | `ACC-999` | Системд байхгүй илгээгч данс |
| **to** | `Valid` | `ACC-B` | Хүчинтэй хүлээн авагч данс |
| | `Missing` | `ACC-999` | Байхгүй хүлээн авагч данс |
| | `SameAsFrom` | `ACC-A` | Илгээгчтэй ижил данс |
| **amount** | `Normal` | `50` | Хэвийн дүн (0 < дүн ≤ үлдэгдэл, лимитэд багтана) |
| | `Zero` | `0` | Тэг дүн (≤ 0) |
| | `OverBalance` | `15000` | Үлдэгдэл (10000)-ээс их |
| | `OverLimit` | `2000` | Үлдэгдэлд багтах боловч өдрийн лимит (1000)-ээс их |
| **currency** | `Same` | `MNT` | Дансны валюттай ижил |
| | `Different` | `USD` | Ханшийн хөрвүүлэлт хийгдэнэ (1 USD = 10 MNT) |
| **Далд орчин** | `Fresh` | өнөөдөр 0 MNT зарцуулсан | Өдрийн лимит хоосон |
| | `NearFull` | өнөөдөр 980 MNT зарцуулсан | Лимит бараг дүүрсэн (980 + 50 > 1000) |

Далд орчны сонголт байгаа: **өдрийн лимит хэр дүүрсэн байсан бэ** (`DailyLimitStatus`). Энэ нь хүсэлтийн бие дотор байхгүй боловч `ERROR_LIMIT`-ийн үр дүнд нөлөөлнө.

### Даалгавар A2 — Хязгаарлалтууд

| Утга | Тэмдэглэгээ | Үндэслэл |
| :--- | :--- | :--- |
| from = `Frozen` | `[ERROR]` | Тодорхойлолтоор царцаагдсан данснаас шилжүүлэх боломжгүй → `ERROR_FROZEN`. |
| from = `Missing` | `[ERROR]` | Илгээгч данс байх ёстой → `ERROR_NO_FROM`. |
| to = `Missing` | `[ERROR]` | Хүлээн авагч данс байх ёстой → `ERROR_NO_TO`. |
| to = `SameAsFrom` | `[ERROR]` | to нь from-оос өөр байх ёстой → `ERROR_SAME_ACCOUNT`. |
| amount = `Zero` | `[ERROR]` | amount > 0 байх ёстой → `ERROR_BAD_AMOUNT`. |
| amount = `OverBalance` | `[ERROR]` | Үлдэгдлээс хэтэрсэн → `ERROR_INSUFFICIENT`. |
| amount = `OverLimit` | `[ERROR]` | Өдрийн лимитээс хэтэрсэн → `ERROR_LIMIT`. |
| DailyLimitStatus = `NearFull` | `[SINGLE]` | Далд төлөв нэг удаа шалгахад хангалттай; хэвийн дүнтэй ч гэсэн `ERROR_LIMIT` гаргадаг онцгой тохиолдол. |
| amount = `Zero` / `OverBalance` / `OverLimit` | `[IF from ≠ Missing AND to ≠ Missing]` | Данс байхгүй үед дүнг шалгах хүртэл очдоггүй (давуу эрэмбээр from/to алдаа түрүүлж буцна), тиймээс эдгээр хослол утгагүй. |
| `Valid`, `Normal`, `Same`, `Different`, `Fresh` | (тэмдэглэгээгүй) | Хэвийн утга — бусад сонголттой бүрэн хосолно. |

**Бүрэн олонлог ба хязгаарлалтын тооцоо (N → M):**

- Бүрэн олонлог: `from`(3) × `to`(3) × `amount`(4) × `currency`(2) × `DailyLimitStatus`(2) = **N = 144**
- Хязгаарлалтын дараа: хэвийн утгуудын хослол 1 × 1 × 1 × 2 × 1 = **2** (happy path, валют `Same`/`Different`) + `[ERROR]` утга бүр нэг удаа = **7** + `[SINGLE]` = **1** → **M = 10**
- **Үр дүн: 144 → 10**

### Даалгавар A3 — Спецификацийн хүснэгт ба алдааны давуу эрэмбэ

**Давуу эрэмбэ:** `from` → `to` → `amount`. Дотроо: `ERROR_NO_FROM` → `ERROR_FROZEN` → `ERROR_NO_TO` → `ERROR_SAME_ACCOUNT` → `ERROR_BAD_AMOUNT` → `ERROR_INSUFFICIENT` → `ERROR_LIMIT`.

| № | from | to | amount | currency | Далд орчин | Хүлээгдэх үр дүн (Oracle) | Тайлбар / Төрөл |
|---|---|---|---|---|---|---|---|
| 1 | Valid | Valid | Normal | Same | Fresh | `201 OK` | Happy path |
| 2 | Valid | Valid | Normal | Different | Fresh | `201 OK` | Валютын хөрвүүлэлттэй (50 USD = 500 MNT) |
| 3 | Missing | Valid | Normal | Same | Fresh | `200 ERROR_NO_FROM` | Илгээгч байхгүй [ERROR] |
| 4 | Frozen | Valid | Normal | Same | Fresh | `200 ERROR_FROZEN` | Царцаагдсан данс [ERROR] |
| 5 | Valid | Missing | Normal | Same | Fresh | `200 ERROR_NO_TO` | Хүлээн авагч байхгүй [ERROR] |
| 6 | Valid | SameAsFrom | Normal | Same | Fresh | `200 ERROR_SAME_ACCOUNT` | Ижил данс хооронд [ERROR] |
| 7 | Valid | Valid | Zero | Same | Fresh | `200 ERROR_BAD_AMOUNT` | Тэг дүн [ERROR] |
| 8 | Valid | Valid | OverBalance | Same | Fresh | `200 ERROR_INSUFFICIENT` | Үлдэгдлээс их [ERROR] |
| 9 | Valid | Valid | OverLimit | Same | Fresh | `200 ERROR_LIMIT` | Өдрийн лимитээс их [ERROR] |
| 10 | Valid | Valid | Normal | Same | NearFull | `200 ERROR_LIMIT` | Хэвийн дүн ч лимит дүүрсэн (980 + 50 > 1000) [SINGLE] |
| 11 | Valid | Valid | OverBalance | Different | Fresh | `200 ERROR_INSUFFICIENT` | 2000 USD = 20000 MNT > үлдэгдэл; хөрвүүлсний дараа шалгана [ERROR] |
| 12 | Missing | Missing | Normal | Same | Fresh | `200 ERROR_NO_FROM` | Давхар алдаа: from эхэлж шалгагдана |
| 13 | Frozen | Valid | Zero | Same | Fresh | `200 ERROR_FROZEN` | Давхар алдаа: from алдаа дүнгийн алдаанаас түрүүлнэ |
| 14 | Valid | SameAsFrom | Zero | Same | Fresh | `200 ERROR_SAME_ACCOUNT` | Давхар алдаа: to алдаа amount-аас түрүүлнэ |
| 15 | Valid | Valid | OverBalance | Same | NearFull | `200 ERROR_INSUFFICIENT` | Давхар алдаа: үлдэгдлийн алдаа лимитийн алдаанаас түрүүлнэ |

---

## Хэсэг Б: PICT хэрэгсэл

PICT-ийг `microsoft/pict` репозиторийн эх кодоос бүтээсэн (репогоос гадна байрлуулсан). Нотолгоо: `results/pict-build.txt`.

### Даалгавар B1 — Browser жишээ (`browser.model`)

| Горим | Мөрийн тоо (толгойгүй) | Файл |
|---|---|---|
| 2-way | **10** | `results/browser-2way.txt` |
| 3-way | **20** | `results/browser-3way.txt` |
| Бүрэн олонлог | 144 (3×3×2×2×2×2) | — |

**Greedy-гийн тайлбар:** PICT нь greedy алгоритм ашигладаг тул алхам бүрт үлдсэн хосуудыг хамгийн их бүрхэх мөрийг сонгохдоо заримдаа глобал хамгийн цөөн тоог гаргадаггүй (жишээ нь гараар 9 мөрөнд багтааж болох 2-way хослолыг PICT 10 мөрөөр гаргасан). 3-way нь 144 бүрэн олонлогийн хуваарьт ердөө 20 мөр буюу ойролцоогоор 14%-д багтсан.

### Даалгавар B2 — Шилжүүлгийн загваруудын харьцуулалт

| Загвар | Файл | Мөрийн тоо | Happy path мөр | Нэг мөр дэх алдааны онцлог |
|---|---|---|---|---|
| `transfer-nc.model` | `results/transfer-nc.txt` | **13** | 1 мөр | Хязгаарлалтгүй тул нэг мөрөнд олон алдаа орж masking үүснэ |
| `transfer.model` | `results/transfer.txt` | **12** | **0 мөр** | `IF` хязгаарлалттай ч сөрөг утгыг ялгаагүй тул олон алдаа холилдоно |
| `transfer-neg.model` | `results/transfer-neg.txt` | **16** | 2 мөр | `~` ашигласнаар нэг мөрөнд хамгийн ихдээ 1 сөрөг утга орно |

**Бодит тест кейсүүд:**

| № | PICT мөр | Setup | Оролт (`POST /transfer` body) | Oracle |
|---|---|---|---|---|
| TC1 | `transfer-neg` (`Valid Valid Normal Same`) | `ACC-A` идэвхтэй, 10000 MNT | `{"from":"ACC-A","to":"ACC-B","amount":50,"currency":"MNT"}` | `201`, `{"result":"OK"}` |
| TC2 | `transfer-neg` (`Valid Valid ~OverBalance Different`) | `ACC-A` 10000 MNT | `{"from":"ACC-A","to":"ACC-B","amount":2000,"currency":"USD"}` | `200`, `{"result":"ERROR_INSUFFICIENT"}` |
| TC3 | `transfer` (`Frozen SameAsFrom Zero Same`) | `ACC-F` царцсан | `{"from":"ACC-F","to":"ACC-F","amount":0,"currency":"MNT"}` | `200`, `{"result":"ERROR_FROZEN"}` (давуу эрэмбээр) |

---

## Файлууд

`README.md`, `browser.model`, `transfer-nc.model`, `transfer.model`, `transfer-neg.model`, `results/` (`pict-build.txt`, `browser-2way.txt`, `browser-3way.txt`, `transfer-nc.txt`, `transfer.txt`, `transfer-neg.txt`)

---

## Дүгнэлт

Энгийн утгатай (`nc`, хязгаарлалттай) PICT загварууд нэг мөрөнд олон алдаа хольдог бөгөөд хязгаарлалт нь зөвхөн зарим боломжгүй хослолыг хаасан тул мөр бүрийн зорилго тодорхойгүй байх (`error masking`) тохиолдол гардаг. Харин `~` сөрөг утга ашигласан загвар нэг мөрөнд хамгийн ихдээ нэг алдаа оруулж, happy path мөрүүдийг ч тодорхой гаргадаг тул oracle илүү ойлгомжтой болдог. Гэсэн хэдий ч `~` ашиглахад нэг мөрөнд нэгээс олон сөрөг утга орохгүй тул өмнө нь гаргасан зарим `IF` хязгаарлалт практикт идэвхжихгүй байх жишээ ажиглагдсан. Гараар хийсэн category-partition (A3) нь алдааны семантик, давуу эрэмбэ болон `NearFull` зэрэг далд орчны нөхцөлийг нарийн илэрхийлдэг ч хосын хамралтыг математикаар баталгаажуулдаггүй. PICT нь pairwise хамралтыг хурдан, цөөн мөрөөр хангадаг боловч алдааны логикийг өөрөө ойлгохгүй тул хоёр аргыг хослуулах нь хамгийн үр дүнтэй. Хамгийн их бодол шаардсан алхам нь давуу эрэмбийг үндэслэлтэй тогтоож, түүнд нийцүүлэн `[ERROR]`, `[SINGLE]`, `[IF]` болон `~` сөрөг утгуудын уялдааг гаргах явдал байв. Мөн greedy алгоритмаас шалтгаалан PICT-ийн мөрийн тоо гараар тооцсоноос зөрөх нь урьдчилан таамаглах боломжгүй гэдгийг харуулсан.
---

## Хавсралт: Model файлууд

### `browser.model`
```text
# browser.model Ñ„Ð°Ð¹Ð» Ò¯Ò¯ÑÐ³Ñ: 
Content: Allow, Restrict, Block 
Cookies: Allow, Restrict, Block 
Popups: Yes, No 
Addons: Yes, No 
Attacks: Yes, No 
Forgeries: Yes, No ```

### `transfer-nc.model`
```text
From: Valid, Frozen, Missing
To: Valid, Missing, SameAsFrom
Amount: Zero, Normal, OverBalance, OverLimit
Currency: Same, Different```

### `transfer.model`
```text
From: Valid, Frozen, Missing
To: Valid, Missing, SameAsFrom
Amount: Zero, Normal, OverBalance, OverLimit
Currency: Same, Different

IF [From] = "Missing" THEN [Amount] = "Normal";
IF [To] = "Missing" THEN [Amount] = "Normal";```

### `transfer-neg.model`
```text
From: Valid, ~Frozen, ~Missing
To: Valid, ~Missing, ~SameAsFrom
Amount: Normal, ~Zero, ~OverBalance, ~OverLimit
Currency: Same, Different

IF [From] = "Missing" THEN [Amount] = "Normal";
IF [To] = "Missing" THEN [Amount] = "Normal";```

