# Лаборатори №6 — Category-Partition ба PICT

## Оюутан ба орчин

- Оюутны нэр: **Эрдэнэцогт Ундрах** (**Undrakh Erdenetsogt**)
- Оюутны код: **B252270135**
- OS: macOS (Apple Silicon)
- CMake: `4.4.4`, AppleClang: `17.0.0`
- PICT: `3.7.4`, source commit `ab76c25` (`2026-09-08`)

## Алхам 1 — PICT бүтээх ба ажиллуулах

```sh
export PATH="/Users/undrakherdenetsogt/Documents/Qualityassurance/tools/pict/build/cli:$PATH"
pict browser.model > results/browser-2way.txt
pict browser.model /o:3 > results/browser-3way.txt
Хэсэг A — Гар Category-Partition (A1, A2, A3)
A1–A2. Сонголт, тэмдэглэгээ ба [ERROR] / [SINGLE] / [IF]-ийн үндэслэл
Бүх гурван шилжүүлгийн загварыг (transfer-nc.model, transfer.model, transfer-neg.model) ижил суурь утгын олонлогоос (N = 4 × 4 × 6 × 3 × 3 × 2 = 1,728 хослол) бүтээв.

Category	Сонголтууд ба тэмдэглэгээ	[ERROR] / [SINGLE] / [IF]-ийн нэг өгүүлбэртэй үндэслэл
From	Active, Frozen [ERROR], Missing [ERROR], Malformed [ERROR]	Frozen, Missing, Malformed утгууд нь эхлэх данс шилжүүлэг хийх эрхгүй эсвэл олдохгүй тул бусад оролтоос үл хамааран шууд алдаа буцаах [ERROR] ангилалд орно.
To	Other, SameAsFrom [ERROR], Missing [ERROR], Malformed [ERROR]	SameAsFrom нь From данстай давхцахгүй байх [IF] нөхцөлийг зөрчих бөгөөд Missing, Malformed-ийн хамт хүлээн авагч буруу тул дангаар алдаа үүсгэх [ERROR] утга болно.
Amount	BelowBalance, ExactBalance, AboveBalance [ERROR], OverLimit [ERROR], Zero [ERROR], Negative [ERROR]	BelowBalance ба ExactBalance нь Balance-ийн хэмжээнээс хамаарах [IF] утгууд бол Zero, Negative, AboveBalance, OverLimit нь гүйлгээний дүнгийн дүрэм зөрчих [ERROR] утгууд юм.
Currency	MNT, USD, Unsupported [ERROR]	Систем зөвхөн MNT болон USD валютыг дэмжих [IF] нөхцөлтэй тул Unsupported нь бусад оролт зөв байсан ч шууд татгалзах [ERROR] утга болно.
Balance	Sufficient, Exact, Insufficient	Balance нь Amount-ийн утгатай логик нийцэлтэй байх ёстой [IF] параметр бөгөөд Insufficient нь AboveBalance-тэй хосолж үлдэгдэл хүрэлцэхгүй нөхцөлийг бүрдүүлнэ.
DailyLimit	Available, Reached [ERROR][SINGLE]	Reached нь өдрийн гүйлгээний дээд хязгаар тулсан [ERROR] утга бөгөөд бусад бүх оролт хүчинтэй үед дангаар тусгаарлан шалгах [SINGLE] шинжтэй.
Нэг хүсэлтэд олон алдаа давхацвал шалгах эрэмбэ: From → To → Amount → Currency → Balance → DailyLimit.

A3. Спецификацийн кейсүүд (Бүх алдаа HTTP 200 + ERROR_*)
Тодорхойлолтын дагуу амжилттай гүйлгээ болон алдааны хариу бүгд HTTP 200 төлөвтэйгөөр JSON result талбарт дараах утгуудыг буцаана:

#	From	To	Amount	Currency	Setup / Property	HTTP	Result
1	Active	Other	BelowBalance	MNT	Balance Sufficient, DailyLimit Available	200	OK
2	Active	Other	ExactBalance	USD	Balance Exact (Amount == Balance), DailyLimit Available	200	OK
3	Missing	Other	BelowBalance	MNT	Эх данс бүртгэлгүй (From эхэлж шалгагдана)	200	ERROR_NO_FROM
4	Malformed	Other	BelowBalance	MNT	Эх дансны дугаар буруу форматтай	200	ERROR_NO_FROM
5	Frozen	Other	BelowBalance	MNT	Эх данс царцаагдсан (From идэвхгүй)	200	ERROR_FROZEN
6	Active	Missing	BelowBalance	MNT	Хүлээн авагч данс бүртгэлгүй	200	ERROR_NO_TO
7	Active	Malformed	BelowBalance	MNT	Хүлээн авагч дансны дугаар буруу форматтай	200	ERROR_NO_TO
8	Active	SameAsFrom	BelowBalance	MNT	Эх ба хүлээн авагч данс ижил	200	ERROR_SAME_ACCOUNT
9	Active	Other	Zero	MNT	Шилжүүлэх дүн 0	200	ERROR_BAD_AMOUNT
10	Active	Other	Negative	USD	Шилжүүлэх дүн сөрөг (< 0)	200	ERROR_BAD_AMOUNT
11	Active	Other	BelowBalance	Unsupported	Дэмжигдээгүй валют (буруу дүн/валют)	200	ERROR_BAD_AMOUNT
12	Active	Other	AboveBalance	MNT	Balance Insufficient (Amount > Balance)	200	ERROR_INSUFFICIENT
13	Active	Other	OverLimit	MNT	Нэг удаагийн/өдрийн лимитээс хэтэрсэн дүн	200	ERROR_LIMIT
14	Active	Other	BelowBalance	MNT	DailyLimit Reached ([SINGLE], бусад нь хүчинтэй)	200	ERROR_LIMIT
15	Missing	SameAsFrom	Negative	MNT	Давхар алдаа: From хамгийн түрүүнд шалгагдана	200	ERROR_NO_FROM
Хэсэг B — PICT pairwise ба 3-way covering arrays
B1. Зааврын browser.model (6 параметр) ба лекцийн 9 мөрийн харьцуулалт
browser.model нь зааврын 6 параметрээр бүтээгдсэн: Content (3) × Cookies (3) × Popups (2) × Addons (2) × Attacks (2) × Forgeries (2) = 144 бүрэн хослол.

Арга / Coverage	Файл дахь нийт мөр	Тестийн мөр (толгойг хассан)	Бүрэн 144 хослолоос цөөрсөн хувь
Лекцийн гар арга (2-way)	—	9	93.8%
PICT 2-way (results/browser-2way.txt)	11	10	93.1%
PICT 3-way (results/browser-3way.txt)	21	20	86.1%
Харьцуулалт: PICT 2-way гаралт 10 тестийн мөр өгсөн нь лекцийн гар аргаар гаргасан 9 мөрөөс ердөө 1 мөрөөр илүү байна. Учир нь PICT нь greedy heuristic алгоритм ашиглан одоогийн байдлаар хамгийн олон шинэ хосыг хаах мөрийг дараалан сонгодог тул бүх боломжит багц дундаас математикийн хувьд хамгийн бага (global minimum = 9) шийдлийг үргэлж баталгаажуулдаггүй ч олон параметртэй үед хэдхэн миллисекундэд алдаагүй 100% pairwise бүрхэлт өгдөг давуу талтай. Харин 3-way (/o:3) горимд гурвалсан харилцан үйлчлэлийг бүгдийг хамрахын тулд тестийн мөр 20 болж 2 дахин нэмэгдсэн.

B2. Ижил утгын олонлогоос бүтээсэн 3 transfer загварын харьцуулалт
Гурван загвар гурвуулаа ижил 6 параметр, ижил утгын олонлогоос (N = 1,728, үүнд Missing, Zero, OverLimit бүгд орсон) үүссэн:

Загвар	Файл дахь мөр	Тестийн мөр (толгойг хассан)	Happy path мөр	Нэг мөр дэх алдааны тоо	Онцлог
transfer-nc.model (constraints-гүй)	28	27	0	1–5	Ямар ч IF дүрэмгүй тул алдаанууд нэг мөрөнд олноор давхардана
transfer.model (domain IF constraints)	27	26	0	1–4	Amount ба Balance/DailyLimit-ийн логик нийцлийг хангасан
transfer-neg.model (~ сөрөг + [SINGLE] IF)	26	25	4	0–1	Сөрөг утгуудыг ~-ээр тэмдэглэж, алдаа бүрийг хүчинтэй орчинтой дангаар шалгана
PICT мөрөөс хөрвүүлсэн 3 тест кейс
Happy path (transfer-neg.txt мөр 4): Active / Other / BelowBalance / MNT / Sufficient / Available → Input: {"from":"ACC-1001","to":"ACC-2002","amount":5000,"currency":"MNT"} → Oracle: HTTP 200, result: "OK".

Дан алдаа (transfer-neg.txt мөр 6): ~Missing / Other / BelowBalance / MNT / Sufficient / Available → Input: {"from":"ACC-9999","to":"ACC-2002","amount":5000,"currency":"MNT"} → Oracle: HTTP 200, result: "ERROR_NO_FROM".

Лимит хэтрэх (transfer-neg.txt мөр 20): Active / Other / BelowBalance / MNT / Sufficient / ~Reached → Input: {"from":"ACC-1001","to":"ACC-2002","amount":50000,"currency":"MNT"} (өдрийн лимит дууссан) → Oracle: HTTP 200, result: "ERROR_LIMIT".
