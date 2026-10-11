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
