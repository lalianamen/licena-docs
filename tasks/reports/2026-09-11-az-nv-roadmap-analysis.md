# Arizona и Nevada — анализ лицензионных требований для License Roadmap

Последняя сверка: 2026-09-11
Назначение: фактическая база для рулсетов `ROADMAP_RULES.az` и `ROADMAP_RULES.nv` (`js/roadmap/roadmap-config.js`). Каждая строка — из названного источника с датой чтения; чего не удалось прочитать — `UNKNOWN`.

## 0. Доступность источников (2026-09-11, из песочницы)

| Источник | Доступ | Как читалось |
|---|---|---|
| roc.az.gov, azroc.gov (ROC, Arizona) | HTTP 403 (curl и WebFetch) | не читалось; факты только со страниц ROC — `UNKNOWN` |
| apps.azsos.gov (Arizona Administrative Code, Title 4 Ch. 9) | HTTP 403 | текст AAC R4-9 читался через зеркало law.cornell.edu (LII); номера разделов и history notes совпадают с официальными |
| azleg.gov (A.R.S. Title 32, Ch. 10) | 200 | напрямую |
| PSI — Candidate Information Bulletin, Arizona ROC (`test-takers.psiexams.com/api/content/bulletin/2477`, PDF 15 стр.) | 200 | напрямую; PSI — официальный вендор trade-экзаменов ROC |
| nvcontractorsboard.com (NSCB, Nevada; www.nscb.nv.gov → 301) | 200 | напрямую, страницы + PDF-формы |
| leg.state.nv.us — NRS 624, NAC 624 | 200 | напрямую |
| PSI — Candidate Information Bulletin, Nevada «C» classifications (`…/bulletin/270`, PDF 45 стр.) | 200 | напрямую |

## 1. ARIZONA — Registrar of Contractors (ROC)

### 1.1 Порядок процесса (главное отличие от Калифорнии)

PSI-бюллетень ROC («Guidelines for Licensure»): «Upon completion of all licensing requirements, including passing the necessary examination(s), submit a completed license application and your original score report to AZ ROC for processing within two years from the date of passing the examination. Licensing applications cannot be accepted until all examination requirements have been completed.» Экзамены сдаёт будущий qualifying party (A.R.S. § 32-1127).

Следствие для roadmap: **экзамены — до заявления** (классификация → опыт → SRE → trade-экзамен → заявление со score report → рассмотрение ≤ 60 дней → бонд/сборы → выдача). Текущий `ROADMAP_AZ_STEPS` (2026-09-05) повторяет калифорнийский порядок «заявление → принятие → экзамены» — это ошибка, исправляется в этом раунде.

### 1.2 Экзамены

| Факт | Значение | Источник |
|---|---|---|
| Statutes and Rules Exam (SRE) | онлайн-курс с тестированием, ведёт сам AZ ROC; стоимость **$61**; обязателен для новых заявителей, не бывших qualifying party на другой лицензии AZ за последние 5 лет | PSI AZ bulletin, «Statutes and Rules Exam (SRE)», «SRE Examination Scheduling Procedures» |
| Trade-экзамен | по классификации, у PSI (test center или remote proctored); нужен ли — по форме ROC **RC-L-206B «License Classification Requirements»** (сама форма — `UNKNOWN`, 403); отдельный «solar» экзамен для solar-классификаций | PSI AZ bulletin, «Trade-Specific Examinations by PSI» |
| Сборы PSI | one examination **$66**, two examinations **$116**, solar portion only **$40**; сбор действует 1 год, невозвратный | PSI AZ bulletin, «Trade-Specific Examination Scheduling Procedures» |
| Пересдача | ждать **30 дней**; **3 попытки**, с 4-й — **90 дней** между попытками | PSI AZ bulletin, там же; AAC R4-9-106 (LII) |
| Проходной балл | **70 %** | AAC R4-9-106 (LII) |
| Срок действия сдачи | заявление подаётся не позднее **2 лет** после сдачи («not more than two years prior to application») | PSI AZ bulletin; AAC R4-9-106; A.R.S. § 32-1122(E)(2) |
| Освобождение | SRE и trade: qualifying party той же классификации в AZ за последние 5 лет (записи ROC); trade: qualifying party в другом штате той же классификации в течение 5 лет — Registrar «may waive»; опыт может быть зачтён по сданному trade-экзамену или национально признанной сертификации | A.R.S. § 32-1122(F); AAC R4-9-106 (LII) |
| Взаимность (список штатов) | `UNKNOWN` — на roc.az.gov | — |

### 1.3 Опыт и qualifying party

| Факт | Значение | Источник |
|---|---|---|
| Требование | 4 года practical/management опыта, из них ≥ 2 за последние 10 лет; техническое обучение — до 2 лет зачёта; Registrar «may reduce»; для действующего/бывшего qualifying party той же классификации документирование опыта «shall waive» | A.R.S. § 32-1122(E)(1) |
| Что принимается как доказательство (обязательно) | военная служба/обучение; дипломы/транскрипты аккредитованных программ; сертификаты apprenticeship (DOL или штат); опыт засчитывается независимо от наличия лицензии и возраста на момент работы; Registrar «may accept any evidence it deems appropriate» | AAC R4-9-119 (LII), eff. 1/1/2019 |
| Форма подтверждения опыта ROC, кто подписывает | `UNKNOWN` (roc.az.gov) | — |
| Qualifying party: уход | уведомить в течение 15 дней, переквалифицироваться в 60 дней, иначе приостановка by operation of law | A.R.S. § 32-1127.01 |

### 1.4 Заявление, сроки, отпечатки, сущность

| Факт | Значение | Источник |
|---|---|---|
| Содержание заявления | классификация, лица компании, qualifying party, good standing в Arizona Corporation Commission, attestation по workers' comp | A.R.S. § 32-1122(B) |
| Отпечатки | Registrar **«may require»** fingerprints + сборы по § 41-1750; практика ROC — `UNKNOWN` | A.R.S. § 32-1122(H) |
| Сроки рассмотрения | overall **60 календарных дней**: administrative completeness **40 дней** (лицензия или письмо о недостатках), substantive **20 дней**; на ответ по недостаткам **30 дней**, иначе возврат с потерей application fee; продление по соглашению +15 дней | AAC R4-9-113 (LII); A.R.S. § 32-1124(A) (60 дней) |
| Отказ | сбор не возвращается | A.R.S. § 32-1124(D) |
| Сущности | Arizona Corporation Commission — good standing (azcc.gov) | A.R.S. § 32-1122(B); azcc.gov |

### 1.5 Сборы, бонд, recovery fund (только суммы из AAC/ARS; практика оплаты — по форме ROC, `UNKNOWN`)

| Тип лицензии | Application | License (biennial) | Renewal | Источник |
|---|---|---|---|---|
| General commercial | $200 | $580 | $580 | AAC R4-9-130 (LII, eff. 7/1/2014) |
| Specialty commercial | $100 | $480 | $480 | — » — |
| General residential | $180 | $320 | $320 | — » — |
| Specialty residential | $80 | $270 | $270 | — » — |
| General dual (KB) | $200 | $480 | $480 | — » — |
| Specialty dual (CR) | $100 | $380 | $380 | — » — |
| Recovery Fund (residential/dual) | $370 initial | — | $270 biennial | AAC R4-9-130; A.R.S. § 32-1126(G) (не более $600 за биеннум) |
| Change of qualifying party $100; name change $30 | | | | AAC R4-9-130 |
| Верхние пределы сборов | general residential ≤ $500, general commercial ≤ $1 500, dual ≤ $2 000; specialty res. ≤ $350, com. ≤ $1 000, dual ≤ $1 350 | | | A.R.S. § 32-1126(A) |

Бонд (surety bond или cash deposit, до выдачи; «not effective until filed at a ROC office»; уменьшение только при продлении):

| Классификация | Сумма (по estimated annual volume) | Источник |
|---|---|---|
| General commercial | < $150k → $5 000; $150k–$500k → $15 000; $500k–$1M → $25 000; $1M–$5M → $50 000; $5M–$10M → $75 000; ≥ $10M → $100 000 | AAC R4-9-112 (LII); A.R.S. § 32-1152(B)(1) |
| Specialty commercial | < $150k → $2 500; $150k–$500k → $7 000; $500k–$1M → $17 500; $1M–$5M → $25 000; $5M–$10M → $37 500; ≥ $10M → $50 000 | AAC R4-9-112; § 32-1152(B)(2) |
| General residential | < $750k → $9 000; ≥ $750k → $15 000 (статут: $5 000–$15 000) | AAC R4-9-112; § 32-1152(B)(5) |
| Specialty residential | < $375k → $4 250; ≥ $375k → $7 500 (статут: $1 000–$7 500) | AAC R4-9-112; § 32-1152(B)(7) |
| Residential/dual дополнительно | либо recovery fund (assessment), либо доп. бонд $200 000 | A.R.S. § 32-1152(C) |

### 1.6 Классификации (AAC, через LII)

- Residential (R4-9-103): B (General Residential), B-3, B-4, B-5, B-6, B-10; specialty R-1 … R-70 (в т.ч. R-11 Electrical, R-37 Plumbing incl. solar, R-39 Air Conditioning and Refrigeration incl. solar, R-42 Roofing, R-61, R-62 Minor Home Improvements).
- Commercial (R4-9-102): A (General Engineering) + A-4…A-19; B-1, B-2; C-1 … C-79 (C-11 Electrical, C-37 Plumbing, C-39 A/C and Refrigeration, C-42 Roofing …).
- Dual (R4-9-104): KA, KA-5, KA-6, KE; KB-1, KB-2, KO; CR-1 … CR-80.
- Для каких классификаций нужен trade-экзамен — форма RC-L-206B (`UNKNOWN`). PSI-бюллетень перечисляет content outlines для B-1, B-2, B-3, B-4, B-5, B-6, B-10, KB-1, KB-2, всех A-, C-, CR-, R-классификаций (список в бюллетене, стр. 9–12).

### 1.7 После выдачи
- Номер ROC на объектах, сметах, рекламе — A.R.S. § 32-1124(B). Местные лицензии/DBA — `UNKNOWN`.

### 1.8 Что остаётся UNKNOWN по Arizona (только roc.az.gov)
Форма RC-L-206B (какие классы требуют trade-экзамен), форма подтверждения опыта и подписант, практика отпечатков, порядок оплаты license fee (с заявлением или при выдаче), список штатов взаимности, онлайн-портал подачи, страницы про истёкшую/приостановленную лицензию, workers' comp — только attestation в заявлении (§ 32-1122(B)), правил Industrial Commission не читали.

## 2. NEVADA — State Contractors Board (NSCB)

### 2.1 Порядок процесса
1. Nevada Business ID у Secretary of State (SilverFlume) — **до** заявления (New License Application, rev. 03/2026, стр. 1).
2. Заявление + application fee **$300** + Resume of Experience + 4 Certification of Work Experience + Background Disclosure Statement and Fingerprint Waiver для **каждого** лица в заявлении и qualified individual + копии ID + Financial Statement (по запрошенному monetary limit) + Bank Verification Form (NSCB License Requirements; New License Application checklist).
3. Отпечатки: после подачи форм — «you may proceed with obtaining the required fingerprints»; отпечатки идут в Nevada Department of Public Safety / FBI; сбор = суммы Central Repository + FBI, cashier's check на Nevada Highway Patrol (NAC 624.681; NRS 624.265(2),(4)).
4. Board проверяет опыт → **Examination Eligibility** (candidate ID) → экзамены у PSI: **CMS (Business and Law)** + trade (если требуется классификацией) (NSCB License Examinations).
5. Одобрение → bond (сумму назначает Board при одобрении), license fee **$600** за 2 года, recovery fund assessment (residential), proof of industrial insurance (или affidavit) → выдача; номер появляется на сайте, wall card/pocket card по почте (FAQ Central; NRS 624.256; NRS 624.283).

Порядок совпадает с калифорнийским скелетом roadmap: заявление → рассмотрение (= проверка опыта и допуск) → экзамены → финальные требования → выдача. Отпечатки — сразу после подачи.

### 2.2 Опыт, qualified individual
| Факт | Значение | Источник |
|---|---|---|
| Требование | **4 года** за последние **15 лет** journeyman / foreman / supervising employee / contractor в запрашиваемой классификации; обучение в аккредитованном колледже — до **3 лет** зачёта; для ранее квалифицированного в той же классификации 15-летнее окно не применяется | NRS 624.260(6)–(7) (текущая редакция; FAQ Central ещё говорит «10 years» — устаревшая формулировка) |
| Journeyman | «fully qualified to perform, without supervision» или завершённый apprenticeship (State Apprenticeship Council) | NRS 624.260(8) |
| Подтверждение | 4 Certification of Work Experience per qualifier, либо master's certification госоргана, либо военный опыт/обучение; self-employed — список клиентов с адресами/телефонами | NSCB License Requirements; Resume of Experience |
| Освобождение от подтверждения опыта | qualified employee той же классификации на другой лицензии NV за последние 5 лет; licensure by endorsement | NSCB License Requirements |
| Qualified individual | owner / partner / member / manager / officer / employee, bona fide и активно занят; один человек — не более одной активной лицензии, кроме владения ≥ 25 % | NRS 624.260(2),(5); NSCB FAQ qualifying party |
| Уход qualifier | уведомить в 10 дней; заменить в 30 дней; иначе приостановка | NRS 624.285; NSCB FAQ |
| Образование как требование | нет («You do not have to meet any education requirements») | NSCB License Requirements |

### 2.3 Экзамены (PSI)
| Факт | Значение | Источник |
|---|---|---|
| Состав | CMS (Business and Law) — всегда; trade — по классификации («not all classifications require trade exams») | NSCB FAQ general Q18; License Examinations |
| Допуск | после подачи заявления и проверки опыта Board — eligibility letter, затем запись в PSI | NSCB License Examinations |
| CMS формат | **60** scored items + 3 unscored, **120 мин**, проходной **45 (75 %)**, **open book** (Construction Business and Law Manual for Nevada); разделы: Licensing 10, Estimating/Bidding 7, Lien Law 3, Financial Mgmt 12, Tax 5, Labor 5, PM 3, Contracts 6, Risk 4, Env/Safety 5 | PSI NV bulletin (content outline CMS) |
| Trade | closed book, кроме указанных кодовых ссылок | NSCB License Examinations |
| Сборы PSI | one portion **$95**, two portions **$140** (второй — CMS); сбор действует до даты истечения eligibility | PSI NV bulletin |
| Пересдача | **3 попытки**; между попытками **2 недели**; после 3-й неудачи заявление void, новое заявление через **30 дней** | NSCB License Examinations; FAQ Q33; PSI |
| Освобождение | actively served as qualified employee той же классификации на другой лицензии в последние **4 года** — обычно без экзамена; endorsement: trade-экзамен может быть отменён, **CMS не отменяется никогда**; NASCLA Commercial General Building — trade waiver для general building | NSCB License Examinations; Licensure by Endorsement |
| Центры | Las Vegas, Reno, Elko; вне штата — PSI | NSCB License Requirements |

### 2.4 Financial responsibility и monetary limit
| Monetary limit | Financial statement | Источник |
|---|---|---|
| ≤ $25 000 | self-prepared (бухгалтерское ПО + affidavit) / форма Board / CPA; current within 6 months | New License Application §11; License Requirements |
| > $25 000 – < $500 000 | compiled by CPA (6 мес.) или reviewed/audited (1 год) | — » — |
| ≥ $500 000 – < $1M | compiled with full disclosures (6 мес.) или reviewed/audited (1 год) | — » — |
| ≥ $1M | reviewed or audited by CPA (1 год) | — » — |
| Прочее | financial statement обязателен при любом лимите; Bank Verification Form; indemnification — опция; B и B-2 — минимальный лимит **$200 000**; B-7 restricted — максимум **$7 000**; повышение лимита — заявление, сбор **$250** | New License Application §10–11; Raise Your License Limit |

### 2.5 Бонд, сборы, страховка, срок
| Факт | Значение | Источник |
|---|---|---|
| Бонд | от **$1 000** до **$500 000**, назначает Board при одобрении (тип, лимит, финансовая ответственность, опыт, характер); surety «A» или лучше, либо cashier's check + admin fee **$200**/biennium; отмена surety — 60 дней уведомления | NSCB Bonds; FAQ Q52–54; NRS 624.270 (заголовок) |
| Residential improvement | предоплата ≤ $1 000 или 10 % без consumer protection bond **$100 000** (AB39) | NSCB Bonds |
| Pool/spa | consumer protection bond $10 000–$400 000; performance/payment bonds ≥ 50 % контракта | NSCB Bonds; NRS 624.276 |
| Сборы | application **$300**; license **$600** / 2 года; статутные пределы: application ≤ $550, license ≤ $900, exam ≤ $300 | FAQ Central; FAQ Q49; NRS 624.280 |
| Residential Recovery Fund | assessment по monetary limit: ≤ $1M — $200; > $1M limited — $500; unlimited — $1 000 за биеннум | NRS 624.470; FAQ Q49 |
| Industrial insurance (workers' comp) | до выдачи: proof of coverage / self-insured certificate / affidavit «no employees, not a subcontractor, no bids for a principal contractor» | NRS 624.256; FAQ Q50 |
| Срок лицензии | **2 года**; не продлена в срок — автоматическая приостановка; Board может запросить financial statement в любой момент | NRS 624.283; FAQ Q51 |
| Сроки рассмотрения | «vary» — по данным License Analyst; endorsement — решение ≤ 60 дней | FAQ Q14; NRS (endorsement, 60 days) |
| Отпечатки | по запросу Board: fingerprint cards + authorization для всех officers/directors/partners/associates и qualifier; в заявлении — «ALL persons listed» | NRS 624.265(2); New License Application checklist |

### 2.6 Endorsement (взаимность)
Активная лицензия в endorsing state с тем же qualified individual **4 года**, без дисциплины; trade-экзамен отменяется при «substantially equivalent requirements» — Alabama, Arizona, California, Connecticut, Florida, Hawaii, Louisiana, New Mexico, North Carolina, South Carolina, Tennessee, West Virginia; CMS обязателен всегда; форма Request for Verification of License (NSCB Licensure by Endorsement).

### 2.7 Классификации
Class A General Engineering (A-1…A-25), Class B General Building (B-1…B-7; B-2 Residential and Small Commercial), Class C — 42 специальности (NAC 624): в т.ч. C-1 Plumbing and Heating (C-1D Plumbing), C-2 Electrical (C-2A…C-2G), C-3 Carpentry, C-4 Painting, C-5 Concrete, C-10 Landscape, C-13 Sheet Metal, C-15 Roofing/Siding, C-18 Masonry, C-20 Tiling, C-21 Refrigeration and A/C (C-21A…C-21G), C-37 Solar, C-41 Fire Protection. Экзамены PSI есть на английском и испанском для CMS, A, B-2, B-7, C-10, C-15a, C-1d, C-2, C-20, C-21b, C-2d, C-3b, C-4a, C-5 (PSI NV bulletin). B-7 restricted license (с 1 октября 2025): ремонт частных домов до $7 000, 2 года опыта, бонд ≥ $2 000, курс по бизнесу (New License Application, B-7 checklist; NRS 624.244?) — в roadmap не моделируется.

### 2.8 После выдачи
Номер и имя на коммерческих автомобилях (NRS 624.288); state business license (SilverFlume) — уже до заявления; local business licenses — `UNKNOWN`.

### 2.9 Что остаётся UNKNOWN по Nevada
Точная сумма fingerprint-сбора (равна тарифам Repository + FBI, цифра не опубликована на прочитанных страницах); сроки рассмотрения; правила для истёкшей > срока лицензии (NRS 624.283 читалось частично); практика online portal (app.nvcontractorsboard.com).

## 3. Следствия для продукта

| Элемент | Arizona | Nevada |
|---|---|---|
| Порядок шагов | экзамены → заявление (score report) → рассмотрение (60 дн.) → бонд/сборы → выдача | заявление (+ фин. отчёт, отпечатки) → проверка опыта / допуск → CMS + trade → бонд, $600, страховка → выдача |
| Law-курс LICENA | `az-sre` (SRE — курс ROC, $61) | `nv-cms` (CMS, open book, 75 %) |
| Опыт | 4 года, ≥ 2 в последние 10; practical/management | 4 года за 15 лет; journeyman/foreman/supervising/contractor; до 3 лет обучения |
| Отпечатки | «may require» | все лица в заявлении, сразу после подачи |
| Финальные требования | бонд (таблица по объёму), recovery fund $370 (residential/dual), license fee по типу | бонд (назначает Board), license fee $600, recovery fund $200/$500/$1 000 (residential), industrial insurance / affidavit |
| Application Assistant | нет (формы ROC — 403) | нет (формы NSCB — PDF; ссылки даём) |

## Source References
A.R.S. §§ 32-1121, 32-1122, 32-1123, 32-1124, 32-1125, 32-1126, 32-1127, 32-1127.01, 32-1131, 32-1132, 32-1152 (azleg.gov, 2026-09-11); AAC R4-9-101…131 index, R4-9-102, R4-9-103, R4-9-104, R4-9-106, R4-9-112, R4-9-113, R4-9-119, R4-9-130 (law.cornell.edu mirror, 2026-09-11); PSI Arizona ROC Candidate Information Bulletin (bulletin/2477, 2026-09-11); NSCB: License Requirements, License Examinations, Licensure by Endorsement, License Classifications, Bonds, FAQ Central, FAQ General Requirements, FAQ Qualifying Party, Raise Your License Limit, Pre-Submittal Application Review, Contractor's License Application, New License Application PDF (rev. 03/2026), Background Disclosure Statement and Fingerprint Waiver PDF (rev. 05/2021) (nvcontractorsboard.com, 2026-09-11); NRS 624.256, 624.260, 624.265, 624.270 (заголовок), 624.280, 624.283, 624.285, 624.288, 624.470; NAC 624.681 (leg.state.nv.us, 2026-09-11); PSI Nevada «C» Classification Candidate Information Bulletin (bulletin/270, 2026-09-11).

## Verification Status
**Partially Verified** — статуты и страницы NSCB прочитаны напрямую; AAC Arizona — через зеркало LII (официальный хост 403); страницы ROC не читались (UNKNOWN названы явно); суммы PSI — из PDF-бюллетеней вендора.
