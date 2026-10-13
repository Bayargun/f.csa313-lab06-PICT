# Лаборатори №6 — Category-Partition ба PICT

**Оюутны нэр:** Б. Баяргүн  
**Оюутны код:** B222270085

## Хэсэг A. Category-Partition

### A1. Сонголт ба төлөөлөх утгууд

Тестлэх функц:

POST /transfer

{
"from": "ACC001",
"to": "ACC002",
"amount": 100,
"currency": "MNT"
}

| Параметр | Сонголт     | Төлөөлөх утга | Тайлбар                     |
| -------- | ----------- | ------------- | --------------------------- |
| From     | Valid       | ACC001        | Идэвхтэй данс               |
| From     | Frozen      | ACC003        | Царцаагдсан данс            |
| From     | Missing     | ACC999        | Байхгүй данс                |
| To       | Valid       | ACC002        | Хүлээн авагчийн данс        |
| To       | Missing     | ACC998        | Байхгүй данс                |
| To       | SameAsFrom  | ACC001        | Өөрийн данс руу шилжүүлэх   |
| Amount   | Normal      | 100           | Зөв дүн                     |
| Amount   | Zero        | 0             | Тэг дүн                     |
| Amount   | Negative    | -100          | Сөрөг дүн                   |
| Amount   | OverBalance | 2000000       | Үлдэгдлээс хэтэрсэн         |
| Amount   | OverLimit   | 1500000       | Өдрийн лимитээс хэтэрсэн    |
| Currency | Same        | MNT           | Дансны валюттай ижил        |
| Currency | Different   | USD           | Ханш хөрвүүлэх шаардлагатай |

### Далд орчны нөхцөл

- ACC001: MNT данс, үлдэгдэл 1,000,000 MNT.
- ACC002: MNT данс, идэвхтэй.
- ACC003: царцаагдсан данс.
- ACC999, ACC998: системд байхгүй данс.
- Өдрийн үлдсэн лимит: 500,000 MNT.
- Валютын ханш: 1 USD = 3,500 MNT.
- Өдрийн лимитийн үлдэгдэл болон дансны үлдэгдлийг
  валют хөрвүүлсний дараа шалгана.

### Алдааны давуу эрэмбэ

ERROR_NO_FROM → ERROR_FROZEN → ERROR_NO_TO →
ERROR_SAME_ACCOUNT → ERROR_BAD_AMOUNT →
ERROR_INSUFFICIENT → ERROR_LIMIT

Олон алдаа зэрэг илэрвэл эхлээд дансны хүчинтэй байдал,
дараа нь шилжүүлгийн дүнг шалгах нь логикийн хувьд
тохиромжтой тул энэ эрэмбийг сонгов.

### A2. Хязгаарлалтууд

| Параметр | Сонголт     | Тэмдэглэгээ | Үндэслэл                                 |
| -------- | ----------- | ----------- | ---------------------------------------- |
| From     | Valid       | -           | Хэвийн утга                              |
| From     | Frozen      | [ERROR]     | Царцаагдсан данснаас шилжүүлэх боломжгүй |
| From     | Missing     | [ERROR]     | Байхгүй данс                             |
| To       | Valid       | -           | Хэвийн утга                              |
| To       | Missing     | [ERROR]     | Байхгүй данс                             |
| To       | SameAsFrom  | [ERROR]     | Өөрийн данс руу шилжүүлэх боломжгүй      |
| Amount   | Normal      | -           | Үлдэгдэл, лимитэд багтана                |
| Amount   | Zero        | [ERROR]     | amount > 0 нөхцөлийг зөрчинө             |
| Amount   | Negative    | [ERROR]     | amount > 0 нөхцөлийг зөрчинө             |
| Amount   | OverBalance | [ERROR]     | Дансны үлдэгдэл хүрэлцэхгүй              |
| Amount   | OverLimit   | [ERROR]     | Өдрийн лимит хэтэрсэн                    |
| Currency | Same        | -           | Хөрвүүлэлт шаардлагагүй                  |
| Currency | Different   | -           | Ханш хөрвүүлэлт хийгдэнэ                 |

#### [SINGLE] нөхцөл

Amount = Negative утгыг [SINGLE] гэж тэмдэглэнэ.

Сөрөг дүн нь тэгээс бага утгын тусгай алдааны
ангилал тул бусад параметрийн бүх хослолтой
давтан шалгах шаардлагагүй.

#### [IF] нөхцөл

1. From = Missing бол Amount = Normal.
2. To = Missing бол Amount = Normal.
3. From = Frozen бол To = Valid.
4. To = SameAsFrom бол From = Valid.

Эдгээр нөхцөл нь тодорхой алдааг тусгаарлаж,
хүлээгдэх үр дүнг хоёрдмол утгагүй болгоход
зориулагдсан.

#### Бүрэн олонлог

From = 3
To = 3
Amount = 5
Currency = 2

N = 3 × 3 × 5 × 2 = 90

Хязгаарлалтын дараах тоог PICT загварт
ижил нөхцөлүүдийг хэрэгжүүлсний дараа тооцно.

### A3. Спецификацийн хүснэгт

| ID   | From    | To         | Amount      | Currency  | Хүлээгдэх үр дүн       |
| ---- | ------- | ---------- | ----------- | --------- | ---------------------- |
| TC01 | Valid   | Valid      | Normal      | Same      | 201 OK                 |
| TC02 | Valid   | Valid      | Normal      | Different | 201 OK                 |
| TC03 | Missing | Valid      | Normal      | Same      | 200 ERROR_NO_FROM      |
| TC04 | Frozen  | Valid      | Normal      | Same      | 200 ERROR_FROZEN       |
| TC05 | Valid   | Missing    | Normal      | Same      | 200 ERROR_NO_TO        |
| TC06 | Valid   | SameAsFrom | Normal      | Same      | 200 ERROR_SAME_ACCOUNT |
| TC07 | Valid   | Valid      | Zero        | Same      | 200 ERROR_BAD_AMOUNT   |
| TC08 | Valid   | Valid      | Negative    | Same      | 200 ERROR_BAD_AMOUNT   |
| TC09 | Valid   | Valid      | OverBalance | Same      | 200 ERROR_INSUFFICIENT |
| TC10 | Valid   | Valid      | OverLimit   | Same      | 200 ERROR_LIMIT        |
| TC11 | Missing | Missing    | Normal      | Same      | 200 ERROR_NO_FROM      |
| TC12 | Valid   | Valid      | OverBalance | Different | 200 ERROR_INSUFFICIENT |
| TC13 | Valid   | Valid      | OverLimit   | Different | 200 ERROR_LIMIT        |

## Хэсэг B. PICT Pairwise Testing

#### B1. Browser Model

| Үзүүлэлт                    | Тестийн тоо |
| --------------------------- | ----------: |
| Бүрэн олонлог               |         144 |
| Лекцийн гараар хийсэн 2-way |           9 |
| PICT 2-way                  |          10 |
| PICT 3-way                  |          20 |

PICT-ийн 2-way тестээр 10 мөр үүссэн бөгөөд
лекцийн гараар боловсруулсан 9 мөрөөс нэгээр олон байна.
Учир нь PICT greedy алгоритм ашигладаг тул бүх хослолыг
хамруулах боловч хамгийн цөөн тестийн тоог баталгаажуулдаггүй.

3-way тестээр 20 мөр үүссэн нь 2-way-ээс хоёр дахин олон
боловч бүрэн олонлог болох 144 мөрөөс 124 мөрөөр цөөн байна.
