# Лаб 06 — Category-Partition ба PICT

- Нэр: Б. Отгонсүх 
- Код: B242270021

## Хэсэг A. Category-Partition: POST /transfer

### Давуу эрэмбэ (олон алдаа зэрэг байх үед)

Тодорхойлолтод бичээгүй тул өөрөө тогтоов: **from → to → amount** буюу ERROR_NO_FROM → ERROR_FROZEN → ERROR_NO_TO → ERROR_SAME_ACCOUNT → ERROR_BAD_AMOUNT → ERROR_INSUFFICIENT → ERROR_LIMIT. Үндэслэл: эхлээд оролцогч талууд (данс) байгаа эсэхийг шалгаж, дараа нь дүнтэй холбоотой шалгалтыг хийдэг, учир нь дансгүй бол үлдэгдэл, лимитийг тооцох боломжгүй.

### A1. Сонголт ба төлөөлөх утгууд

| Сонголт | Төлөөлөх утгууд | Тайлбар |
|---|---|---|
| from | Valid / Frozen / Missing | идэвхтэй данс / царцаагдсан данс / байхгүй данс |
| to | Valid / Missing / SameAsFrom | from-оос өөр байгаа данс / байхгүй данс / from-той ижил данс |
| amount | Normal / ExactBalance / Zero / OverBalance / OverLimit | үлдэгдэл ба лимитийн дотор / үлдэгдэлтэй яг тэнцүү / 0 (amount ≤ 0 ангийг төлөөлнө) / үлдэгдлээс их / үлдэгдэл дотор боловч өдрийн лимитээс их |
| currency | Same / Different | данснаасаа ижил валют / өөр валют (ханшийн хөрвүүлэлттэй) |

Далд орчны сонголт (параметр биш, тестийн setup-аар бэлдэнэ):
- from дансны үлдэгдэл: OverBalance, ExactBalance утгуудыг бэлдэхэд шаардлагатай.
- Өдрийн лимит хэр дүүрсэн байсан: OverLimit-ийг үлдэгдэл хүрэлцэх мөртлөө лимит хэтрэхээр бэлдэнэ (лимитийг өмнө нь ашигласан эсвэл том дүн).
- Ханш: Different валютын үед үлдэгдэл, лимитийг хөрвүүлсний дараах дүнгээр шалгадаг.

### A2. Хязгаарлалт

| Сонголт | Утга | Тэмдэглэгээ | Үндэслэл |
|---|---|---|---|
| from | Valid | [property FromExists] | данс байгаа тул to-той ижил болох боломжтой |
| from | Frozen | [ERROR] [property FromExists] | ERROR_FROZEN буцаадаг, бусад утгатай хослуулах шаардлагагүй |
| from | Missing | [ERROR] | ERROR_NO_FROM буцаадаг, дараагийн шалгалтуудыг нууна |
| to | Valid | | энгийн утга |
| to | Missing | [ERROR] | ERROR_NO_TO буцаадаг |
| to | SameAsFrom | [ERROR] [IF FromExists] | ERROR_SAME_ACCOUNT; from байхгүй бол "ижил" гэдэг утгагүй тул зөвхөн from-той данс байхад л хослуулна |
| amount | Normal | | энгийн утга |
| amount | ExactBalance | [SINGLE] | үлдэгдэлтэй яг тэнцүү гэдэг нь хязгаарын онцгой тохиолдол, нэг удаа шалгахад хангалттай (амжилттай шилжинэ) |
| amount | Zero | [ERROR] | ERROR_BAD_AMOUNT (amount ≤ 0 ангийг төлөөлнө) |
| amount | OverBalance | [ERROR] | ERROR_INSUFFICIENT |
| amount | OverLimit | [ERROR] | ERROR_LIMIT |
| currency | Same | | энгийн утга |
| currency | Different | | алдаа биш, хөрвүүлэлтийг шалгана (хоёр утга хоёулаа амжилттай замд орно) |

Тоо бодолт:
- Бүрэн олонлог: 3 × 3 × 5 × 2 = 90.
- Хязгаарлалтын дараа: [ERROR] утга бүр бусад параметр нь энгийн утгатайгаа нэг л удаа орно (from-ийн 2 + to-ийн 2 + amount-ийн 3 = 7); [SINGLE] 1; алдаагүй утгуудын хослол from Valid × to Valid × amount Normal × currency {Same, Different} = 2. Нийт 7 + 1 + 2 = 10.

**90 → 10**

### A3. Спецификацийн хүснэгт

Хүлээгдэх үр дүн: амжилт нь `201 {"result":"OK"}`, оролтын алдаа нь `200 {"result":"ERROR_..."}`. Давхар нөхцөлийн мөрийн үр дүнг дээрх давуу эрэмбээр (from → to → amount) гаргасан.

| ID | from | to | amount | currency | Төрөл | Хүлээгдэх үр дүн |
|---|---|---|---|---|---|---|
| S01 | Valid | Valid | Normal | Same | happy path | 201 OK |
| S02 | Valid | Valid | Normal | Different | happy path (хөрвүүлэлттэй) | 201 OK |
| S03 | Valid | Valid | ExactBalance | Same | [SINGLE] хязгаар | 201 OK |
| S04 | Missing | Valid | Normal | Same | [ERROR] | 200 ERROR_NO_FROM |
| S05 | Frozen | Valid | Normal | Same | [ERROR] | 200 ERROR_FROZEN |
| S06 | Valid | Missing | Normal | Same | [ERROR] | 200 ERROR_NO_TO |
| S07 | Valid | SameAsFrom | Normal | Same | [ERROR] | 200 ERROR_SAME_ACCOUNT |
| S08 | Valid | Valid | Zero | Same | [ERROR] | 200 ERROR_BAD_AMOUNT |
| S09 | Valid | Valid | OverBalance | Same | [ERROR] | 200 ERROR_INSUFFICIENT |
| S10 | Valid | Valid | OverLimit | Same | [ERROR] | 200 ERROR_LIMIT |
| S11 | Missing | Missing | Normal | Same | давхар: from + to | 200 ERROR_NO_FROM (from нь to-оос өмнө) |
| S12 | Frozen | SameAsFrom | Zero | Same | давхар: гурван алдаа | 200 ERROR_FROZEN (from нь to, amount-аас өмнө) |
| S13 | Valid | Missing | OverBalance | Same | давхар: to + amount | 200 ERROR_NO_TO (to нь amount-аас өмнө) |
| S14 | Valid | Valid | OverBalance ба OverLimit хоёуланг хэтэрсэн | Same | давхар: үлдэгдэл + лимит | 200 ERROR_INSUFFICIENT (үлдэгдэл нь лимитээс өмнө) |
| S15 | Valid | Valid | OverBalance | Different | давхар: хөрвүүлэлт + алдаа | 200 ERROR_INSUFFICIENT (үлдэгдлийг хөрвүүлсний дараах дүнгээр шалгана) |

Хамралт: happy path (S01, S02), [ERROR] анги бүр (S04–S10, 7 ангийн 7 мөр), [SINGLE] (S03), давхар нөхцөл (S11–S15). S11–S13 нь from → to → amount эрэмбийг, S14 нь дүнгийн доторх үлдэгдэл → лимит эрэмбийг, S15 нь ханшийн хөрвүүлэлтийг баталгаажуулна.

## Хэсэг B. PICT: pairwise covering array

### B1. Лекцийн browser жишээ

Модель (browser.model):

    Content:   Allow, Restrict, Block
    Cookies:   Allow, Restrict, Block
    Popups:    Yes, No
    Addons:    Yes, No
    Attacks:   Yes, No
    Forgeries: Yes, No

| Хувилбар | Файл | Мөрийн тоо |
|---|---|---|
| 2-way (pairwise) | results/browser-2way.txt | 10 |
| 3-way (/o:3) | results/browser-3way.txt | 20 |
| Бүрэн олонлог | — | 144 (3 × 3 × 2 × 2 × 2 × 2) |

Мөрийн тоо нь толгой мөрийг оруулаагүй (tail -n +2 | wc -l). /r ашиглаагүй тул PICT ажиллуулах бүрт ижил үр дүн өгдөг.

**2-way.** PICT 10 мөр гаргасан бол лекц дээр гараар 9 мөрөнд багтаасан тул PICT нэг мөрөөр илүү өгсөн. Шалтгаан нь PICT greedy алгоритмтай: алхам бүрдээ хамгийн олон хамрагдаагүй хосыг хамардаг мөрийг сонгодог тул орон нутгийн хувьд сайн сонголт хийдэг ч нийт хамгийн цөөн мөрийг баталдаггүй. Доод хязгаар нь 9 (Content ба Cookies-ийн 3 × 3 = 9 хос бүр тусдаа мөр шаарддаг), харин 81 хос бүгд хамрагдсан (pict /s: Combinations 81).

**3-way.** 20 мөр болсон нь бүрэн олонлог 144-ийн ойролцоогоор 14%. 3-way нь 2-way-ээс хоёр дахин илүү мөртэй боловч гурвын бүх хослолыг хамарч байгаа учир бүх 144 хослолыг ажиллуулахаас хамаагүй цөөн тестээр хангана.

### B2. Шилжүүлгийн model (transfer-nc, transfer, transfer-neg)

Хэсэг A-ийн сонголт, утгуудыг PICT хэл рүү буулгав. Гурван хувилбар:
- **transfer-nc.model**: хязгаарлалтгүй, бүх утга энгийн утга шиг.
- **transfer.model**: A2-ын хязгаарлалттай (from Missing үед to нь SameAsFrom биш, from эсвэл to Missing үед amount нь Normal).
- **transfer-neg.model**: [ERROR] утгуудыг ~-ээр тэмдэглэсэн.

Гурван ажиллуулалтын харьцуулалт (results/transfer-nc.txt, transfer.txt, transfer-neg.txt; мөрийн тоо толгойгүй):

| Хувилбар | Мөрийн тоо | Happy path (алдаагүй) | 1 алдаатай | 2 алдаатай | 3 алдаатай |
|---|---|---|---|---|---|
| transfer-nc | 15 | 0 | 5 | 6 | 4 |
| transfer | 14 | 2 | 2 | 10 | 0 |
| transfer-neg | 18 | 4 | 14 | 0 | 0 |

"Алдаа" гэж from-ийн Frozen/Missing, to-ийн Missing/SameAsFrom, amount-ын Zero/OverBalance/OverLimit утгыг тоолсон. Happy path мөр гэдэг нь алдааны утга огт байхгүй мөр (ExactBalance нь алдаа биш).

**Гаралтыг уншихад:**
- Хязгаарлалтгүй (transfer-nc) 15 мөрөнд happy path мөр **байхгүй** (0), тэгэхээр pairwise хос бүрийг хамарсан ч бүх утга хүчинтэй нэг ч мөр гаргах баталгаа байхгүй. Мөн 15 мөрийн 6 нь 2, 4 нь 3 алдаатай тул error masking их: давуу эрэмбийн дагуу эхний алдаа л буцаж, бусад алдаа нуугдана.
- Олон алдаатай мөр юуг шалгаж байгаа нь тодорхойгүй. Жишээ нь Frozen + SameAsFrom + Zero мөрөнд ERROR_FROZEN л буцдаг тул SameAsFrom болон Zero аль нь өөрөө шалгагдсан нь мэдэгдэхгүй, зөвхөн давуу эрэмбийг шалгаж байна.
- transfer (A2-ын хязгаарлалттай) 14 мөр болж, 2 happy path мөртэй, 3 алдаатай мөр байхгүй. Гэхдээ 2 алдаатай мөр 10 хэвээр байна, учир нь хязгаарлалт нь from/to Missing үеийн нуугдлыг л арилгасан, харин Frozen + SameAsFrom гэх мэт хослол үлдсэн.
- transfer-neg 18 мөр: мөр бүрд ихдээ 1 алдаа (14 мөр 1 алдаатай, 2-3 алдаатай мөр 0/0), happy path 4 мөр. Алдаа бүр ганцаараа орох тул A2-ын [ERROR] тэмдэглэгээтэй яг дүйнэ. Үнэ нь мөрийн тоо нэмэгдсэн (18 > 14).

**~ хэрэглэхэд A2-ын хязгаарлалт хэрэгтэй хэвээр үү?** Хэсэгчлэн. Нэг мөрөнд ихдээ 1 сөрөг утга орох тул "from Missing үед to нь SameAsFrom биш" болон "from/to Missing үед amount нь Normal" гэсэн алдаа нуух хоёр төрлийн хязгаарлалт автоматаар хангагдана, тэднийг transfer-neg.model-д бичих шаардлагагүй. Харин нэг хязгаарлалт хэрэгтэй хэвээр: from Missing үед amount нь Normal. Шалтгаан нь ExactBalance алдаа биш (~-гүй) утга тул from Missing-тай нэг мөрөнд орж болно, гэтэл данс байхгүй бол үлдэгдэл гэж байхгүй тул утгагүй мөр болдог (ийм мөр хязгаарлалтгүй үед ~Missing + ExactBalance хэлбэрээр гарч байсан). Хязгаарлалтад ~ тавихгүй, утгыг "Missing" гэж бичнэ.
