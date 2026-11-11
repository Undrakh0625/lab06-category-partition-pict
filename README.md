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
