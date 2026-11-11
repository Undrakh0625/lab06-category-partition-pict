# Лаборатори №6 — Category-Partition ба PICT

## Оюутан ба орчин

- Оюутны нэр: **Ундрах**
- Оюутны код: **B252270135**
- OS: macOS (Apple Silicon)
- CMake: `4.4.4`
- C++ compiler: AppleClang `17.0.0`
- PICT: `3.7.4`, source commit `ab76c25` (`2026-09-08`)
- PICT source/build: repository-оос гадна `/Users/undrakherdenetsogt/Documents/Codex/tools/pict`.

Энэ бол тест дизайн лаборатори; ажиллуулж турших REST API эсвэл application code агуулаагүй. PICT-ийн эх код repository-оос гадна байрлана, энд зөвхөн model болон гаралтын файлууд бий.

## Алхам 1 — PICT бүтээх

macOS дээр CMake болон C++ compiler шаардлагатай. Энэ компьютерт CMake, AppleClang аль хэдийн байсан тул эх кодыг clone хийж CMake-аар бүтээсэн. PICT binary-г өөрийн `PATH`-д нэмэх жишээ:

```sh
export PATH="/Users/undrakherdenetsogt/Documents/Codex/tools/pict/build/cli:$PATH"
pict browser.model
```

Build болон source commit-ийн нотолгоо `results/pict-build.txt`-д хадгалагдсан.

## Хэсэг A — Гар Category-Partition

### Сонголт, төлөөлөх утга ба тэмдэглэгээ

`[ERROR]` нь дангаараа алдаа өгөх сонголт, `[SINGLE]` нь бусад параметрүүд хүчинтэй үед ганцаар турших утга, `[IF]` нь property/нөхцөлөөс хамаарах утгыг тэмдэглэнэ. PICT model дахь `~` нь сөрөг буюу зөвшөөрөгдөхгүй утгыг заана.

| Category | Сонголтууд ба төлөөлөх утга | Property/тэмдэглэгээ |
|---|---|---|
| `From` данс | Active; Frozen `[ERROR]`; байхгүй `~Missing` `[ERROR]`; буруу хэлбэр `~Malformed` `[ERROR]` | Transfer эхлэх дансны төлөв |
| `To` данс | Other; From-той ижил `SameAsFrom` `[ERROR]`; байхгүй `~Missing` `[ERROR]`; буруу хэлбэр `~Malformed` `[ERROR]` | `To` нь `From`-оос өөр байх ёстой `[IF]` |
| `Amount` | BelowBalance; ExactBalance; AboveBalance `[ERROR]`; `~Zero`, `~Negative`, `~Malformed` `[ERROR]` | Below: balance > amount; Exact: balance = amount; Above: balance < amount `[IF]` |
| `Currency` | MNT; USD; дэмжигдээгүй `~Unsupported` `[ERROR]` | Валют нь MNT эсвэл USD байх ёстой `[IF]` |
| Source `Balance` | Sufficient; Exact; Insufficient | Amount-тай нийцнэ `[IF]` |
| Өдрийн `DailyLimit` | Available; Reached `[ERROR][SINGLE]` | Reached-г бусад бүх оролт хүчинтэй үед шалгана `[IF]` |

Өргөн Category-Partition сонголтуудын бүрэн олонлог `N = 4 × 4 × 6 × 3 × 3 × 2 = 1,728`; PICT model-ийн нөхцөлүүд сөрөг/алдаатай утгыг бусад зөв утгатайгаар дангаар шалгаж, нийт `M = 27` боломжит хослол үлдээнэ. Эерэг утгатай `transfer.model`-ийн хувьд `N = 144`, нөхцөлийн дараа `M = 12` болно. `transfer-nc.model` нь эдгээр 144 эерэг боломжид хязгаарлалт тавихгүй.

Нэг хүсэлтэд хэд хэдэн алдаа давхацвал буцаах алдааны эрэмбэ `From → To → Amount → Currency → Balance → DailyLimit`; иймээс давхар алдааны кейст эхний илэрсэн шалгалтын алдаа хүлээгдэнэ.

### Спецификацийн кейсүүд

Эдгээр хүснэгт нь системийн тодорхойлолт бөгөөд бодитоор ажиллах шилжүүлгийн API даалгаварт өгөөгүй.

| # | From | To | Amount | Currency | Setup/property | HTTP / result |
|---:|---|---|---|---|---|---|
| 1 | Active | Other | BelowBalance | MNT | Balance sufficient, limit available | 201 / `OK` |
| 2 | Active | Other | ExactBalance | MNT | Amount = balance | 201 / `OK` |
| 3 | Frozen | Other | BelowBalance | MNT | Бусад оролт хүчинтэй | 403 / `ERROR_FROM_FROZEN` |
| 4 | `~Missing` | Other | BelowBalance | MNT | From account байхгүй | 404 / `ERROR_FROM_NOT_FOUND` |
| 5 | `~Malformed` | Other | BelowBalance | MNT | From ID буруу форматтай | 400 / `ERROR_FROM_INVALID` |
| 6 | Active | SameAsFrom | BelowBalance | MNT | Нэг данснаас өөрт нь шилжүүлэх | 400 / `ERROR_SAME_ACCOUNT` |
| 7 | Active | `~Missing` | BelowBalance | MNT | To account байхгүй | 404 / `ERROR_TO_NOT_FOUND` |
| 8 | Active | `~Malformed` | BelowBalance | MNT | To ID буруу форматтай | 400 / `ERROR_TO_INVALID` |
| 9 | Active | Other | `~Zero` | MNT | Amount = 0 | 400 / `ERROR_AMOUNT_INVALID` |
| 10 | Active | Other | `~Negative` | USD | Amount < 0 | 400 / `ERROR_AMOUNT_INVALID` |
| 11 | Active | Other | AboveBalance | MNT | Balance хүрэлцэхгүй | 422 / `ERROR_INSUFFICIENT_FUNDS` |
| 12 | Active | Other | BelowBalance | `~Unsupported` | Валют дэмжигдээгүй | 400 / `ERROR_UNSUPPORTED_CURRENCY` |
| 13 | Active | Other | BelowBalance | MNT | DailyLimit reached; бусад нь зөв `[SINGLE]` | 429 / `ERROR_DAILY_LIMIT` |
| 14 | `~Missing` | `~Missing` | BelowBalance | MNT | Давхар алдаа; From түрүүлж шалгана | 404 / `ERROR_FROM_NOT_FOUND` |
| 15 | Active | Other | `~Negative` | `~Unsupported` | Давхар алдаа; Amount Currency-гээс түрүүлнэ | 400 / `ERROR_AMOUNT_INVALID` |

## Хэсэг B — PICT pairwise covering arrays

### B1. Browser model

`browser.model`-ийн бүрэн олонлог нь Browser `4` × OS `3` × Device `2` × Network `3` × Locale `2` = **144** хослол. PICT-ийн гаралтын эхний мөр нь баганын толгой тул өгөгдлийн мөрийг толгойг хасаж тоолов.

| Coverage | Гаралтын файл дахь нийт мөр | Тестийн мөр (толгойг хассан) | Бүрэн 144 хослолоос цөөрсөн |
|---|---:|---:|---:|
| 2-way | 14 | **13** | 131 |
| 3-way | 41 | **40** | 104 |

PICT greedy аргаар одоогоор хамгийн олон шинэ pair/triple хамрах мөрийг сонгодог тул үр дүн нь хүчинтэй covering array байна. Энэ нь бүх боломжит багц дундаас хамгийн цөөн мөртэйг математик аргаар баталдаггүй; параметрийн дараалал болон constraints-ийг өөрчлөхөд гарах мөрийн тоо ч өөрчлөгдөж болно.

### B2. Transfer models-ийн гаралт

Доорх `N → M` нь сонголтуудын бүх cross-product-оос constraints хангаж үлдсэн боломжит хослолын тоо; PICT output дахь мөр нь pairwise covering set. Error тоолохдоо `From`, `To`, funds/amount, currency, daily-limit алдааны ангийг тус тус тооцож, `Amount`–`Balance` зөрчлийг нэг funds алдаа гэж үзсэн.

| Model | Сонголтын бүтэн хэмжээ N → хязгаарын дараах M | PICT тестийн мөр | Happy path мөр | Нэг мөр дэх error-ийн тоо |
|---|---:|---:|---:|---:|
| `transfer-nc.model` (constraints-гүй) | 144 → 144 | 10 | 0 | 1–4 |
| `transfer.model` (эерэг утга, constraints-тэй) | 144 → 12 | 10 | 2 | 0–1 |
| `transfer-neg.model` (`~` сөрөг утгатай) | 1,728 → 27 | 25 | 2 | 0–1 |

Файлын мөрийн тоо нь толгой мөртэйгээ нийлээд `transfer-nc.txt` — **11**, `transfer.txt` — **11**, `transfer-neg.txt` — **26** байна. Constraints-гүй модель pairwise мөрүүдэд хэд хэдэн алдааг давхардуулж чадсан; constrained болон negative model-ийн дүрмүүд алдааны утгыг бусад хүчинтэй утгатай дангаар нь хослуулсан.

### PICT мөрөөс хөрвүүлсэн 3 тест кейс

#### 1. Happy path — `transfer.txt`-ийн `Active / Other / ExactBalance / MNT / Exact / Available` мөр

- **Setup:** `ACC-1001` идэвхтэй, үлдэгдэл `10,000 MNT`; `ACC-2002` идэвхтэй; өдрийн ашиглалт лимитээс бага.
- **Input JSON:** `{"from":"ACC-1001","to":"ACC-2002","amount":10000,"currency":"MNT"}`
- **Oracle:** HTTP `201`, `result: "OK"`.

#### 2. Сөрөг утга — `transfer-neg.txt`-ийн `~Missing / Other / BelowBalance / MNT / Sufficient / Available` мөр

- **Setup:** `ACC-404` бүртгэлгүй; `ACC-2002` идэвхтэй, үлдэгдэл хүрэлцээтэй; лимит боломжтой.
- **Input JSON:** `{"from":"ACC-404","to":"ACC-2002","amount":5000,"currency":"MNT"}`
- **Oracle:** HTTP `404`, `result: "ERROR_FROM_NOT_FOUND"`.

#### 3. Сөрөг amount — `transfer-neg.txt`-ийн `Active / Other / ~Negative / USD / Sufficient / Available` мөр

- **Setup:** `ACC-1001`, `ACC-2002` идэвхтэй; эх дансны үлдэгдэл `100,000`; лимит боломжтой.
- **Input JSON:** `{"from":"ACC-1001","to":"ACC-2002","amount":-500,"currency":"USD"}`
- **Oracle:** HTTP `400`, `result: "ERROR_AMOUNT_INVALID"`.

## Дүгнэлт — 9 өгүүлбэр

1. Category-Partition нь шилжүүлгийн параметрүүд, орчны төлөв, алдааны утгыг ойлгомжтой ангилсан.
2. Алдааны давуу эрэмбэ нь давхар буруу оролтын үед хүлээгдэх oracle-ийг нэг утгатай болгосон.
3. Гар спецификацид happy, boundary, дан алдаа болон давхар алдааг хамарсан 15 кейс тодорхойлсон.
4. Browser 2-way set 144 боломжийг 13 тестийн мөрөөр төлөөлсөн.
5. Browser 3-way set 40 мөр болсон нь гурвалсан харилцан үйлчлэлийг хамрах нэмэлт өртгийг харуулсан.
6. Transfer constraints боломжит эерэг хослолыг 144-өөс 12 болгож, negative model-ийн олонлогийг 1,728-аас 27 болгож хязгаарласан.
7. Constraints-гүй transfer set 10 мөртэй ч happy path оруулаагүй бөгөөд нэг мөрөнд 1–4 алдаа давхардуулсан.
8. Negative model 25 мөр гаргаж, happy болон нэг алдаатай кейсүүдийг хамруулсан.
9. Гар Category-Partition нь oracle болон бизнесийн утгыг тайлбарлахад давуу, PICT нь олон параметрийн хослолыг системтэй цөөлөхөд давуу боловч гарсан хамгийн бага мөрийн тоог баталдаггүй.
