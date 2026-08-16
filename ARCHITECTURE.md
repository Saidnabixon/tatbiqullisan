# ARCHITECTURE.md — Tatbiqul Lisan Academy

> **Bu fayl kim uchun:** AI kodlash agentlari (Claude Code va shunga
> o'xshash) uchun. Maqsad — loyihaga birinchi marta kirgan agent hech
> qanday qo'shimcha tushuntirishsiz to'g'ri faylni topib, to'g'ri joyga,
> mavjud naqshlarga mos o'zgartirish kirita olishi.
>
> **Agent uchun qoida:** Har qanday o'zgartirish kiritishdan oldin shu
> faylning tegishli bo'limini o'qing. O'zgartirish loyiha strukturasini
> o'zgartirsa (yangi fayl, yangi eksport, yangi route, yangi ID naqsh),
> **shu faylni ham yangilang** — bu fayl loyiha bilan birga eskirmasligi
> kerak. Qaysi bo'lim yangilanishi kerakligi har bir bo'lim oxirida
> "Buni qachon yangilash kerak" sifatida ko'rsatilgan.

---

## 1. Loyiha nima

Arab tili o'quv va imtihon-tayyorgarlik platformasi (o'zbek tilli
auditoriya uchun). CEFR A1–C2 kurslari, Multilevel/TANAL/Darajaviy
imtihon tayyorgarligi, testlar, kutubxona, lug'at, gamifikatsiya,
sertifikatlar, admin panel.

**Hozirgi bosqich:** statik frontend prototip (backend yo'q, ma'lumot
brauzerda saqlanadi). Backend qo'shish rejasi `ROADMAP.md` da.

**Muhim mental model:** bu — **bitta buyurtma bilan qurilgan, xatosiz,
sinovdan o'tgan prototip**, "boshlang'ich skelet" emas. Har bir sahifa,
har bir hisoblash to'liq ishlaydi. Vazifangiz uni **kengaytirish**,
qayta qurish emas.

---

## 2. Texnologiya va nega shunday

| Tanlov | Sabab |
|---|---|
| Vanilla JS (framework yo'q) | Ishlab chiqarish muhitida tarmoq yo'q edi — npm/CDN mavjud emas |
| Global nomlar fazosi (`window.TLA`) | Barcha modul bitta `<script>` blokiga birlashtiriladi — build vositasi yo'q |
| Hash-router (`#/learn`) | Server konfiguratsiyasisiz (rewrite qoidalarisiz) statik hostingda ishlash uchun |
| CSS custom properties (Tailwind emas) | Bog'liqliksiz; dizayn tokenlari `01-tokens.css` da markazlashgan |
| `localStorage`/`window.storage` | Backend yo'q — 12-bo'limga qarang |

**Bu tanlovlarni "eskirgan" deb hisoblab qayta yozishga urinmang** —
ular ataylab, muhit cheklovi tufayli tanlangan. Next.js ga o'tish
rejalashtirilgan (`ROADMAP.md` 7-bo'lim), lekin bu **alohida, katta
vazifa**, kichik o'zgartirish emas.

---

## 3. Fayl daraxti — qayerga borish kerak

```
tla/
├── site.config.json          ← Kontakt, ijtimoiy tarmoq, domen (build vaqtida kiritiladi)
├── build.py                   ← src/ ni public/index.html ga birlashtiradi
├── src/
│   ├── index.template.html    ← HTML skeleti (CSS/JS joy egalari bilan)
│   ├── styles/                ← 6 fayl, TARTIB MUHIM (raqam prefiksi)
│   └── js/                    ← 22 fayl, TARTIB MUHIM (raqam prefiksi = yuklash tartibi)
├── public/                    ← build.py chiqishi — QO'LDA TAHRIRLAMANG
│   ├── index.html                 (build.py qayta yozadi)
│   ├── og-image.png
│   ├── robots.txt, sitemap.xml, _headers
├── tests/                     ← Puppeteer sinov skriptlari (qo'lda ishga tushiriladi)
├── .github/workflows/deploy.yml
└── ANALYSIS.md, README.md, ROADMAP.md, DEPLOY.md   ← boshqa hujjatlar (14-bo'limga qarang)
```

### 3.1 `src/js/` — TO'LIQ XARITA

Fayllar **raqam prefiksi bilan tartiblangan va shu tartibda
`<script>` ga birlashtiriladi.** Raqam = bog'liqlik tartibi (masalan,
`06-state.js` `03-05` dagi ma'lumotlarga tayanadi). **Fayl nomini
o'zgartirmang yoki tartibini buzmang** — bu bog'liqlik ketma-ketligini
buzadi.

| Fayl | Nomlar fazosi | Nima bor | Qachon o'zgartirish kerak |
|---|---|---|---|
| `00-utils.js` | `TLA.utils` | DOM qisqartmalar, formatlash (`fmtDate`, `fmtMoney`...), `shuffle`, `seededRandom`, `debounce` | Yangi umumiy yordamchi funksiya kerak bo'lganda |
| `01-storage.js` | `TLA.store` | 3 bosqichli saqlash: `window.storage` → `localStorage` → xotira | Saqlash strategiyasini o'zgartirganda (masalan, backend qo'shilganda) |
| `02-i18n.js` | `TLA.i18n` | UZ/EN/AR lug'at (~230 kalit), RTL boshqaruvi | **Yangi matn qo'shsangiz — HAR DOIM shu yerga.** Hech qachon UI kodida qattiq yozilgan matn ishlatmang |
| `03-data-courses.js` | `TLA.data.*` | Darajalar, kurslar, modullar, darslar (generatsiya), grammatika, matnlar | Yangi kurs/daraja/dars qo'shganda |
| `04-data-questions-a.js` | `TLA.data.questions` (qism 1) | Grammatika + leksika savollari | Yangi grammatika/leksika savoli qo'shganda |
| `04-data-questions-b.js` | `TLA.data.questions` (qism 2), `TLA.data.scripts`, `writingTasks`, `speakingTasks` | O'qish/tinglash savollari, tinglash skriptlari, yozish/gapirish topshiriqlari | Yangi o'qish/tinglash/yozish/gapirish materiali qo'shganda |
| `05-data-content.js` | `TLA.data.tests/library/dictionary/blog/achievements/defaultPlans/defaultConfig/legal` | Test katalogi, kutubxona, lug'at, blog, tariflar, **sayt sozlamalari birlashtiruvchisi** | Yangi test to'plami, kutubxona resursi, blog maqolasi, tarif qo'shganda |
| `05b-api.js` | `TLA.api` | **Backend mijozi.** `fetch` o'ramchisi, token boshqaruvi, tarmoq xatosini validatsiya xatosidan ajratadi (12-bo'limga qarang) | Yangi backend endpoint chaqirilishi kerak bo'lganda |
| `06-state.js` | `TLA.state` | **Yagona holat boshqaruvchisi.** Auth (backend + lokal zaxira), profil, progress, XP, sertifikat, to'lov. `register`/`login` **async** (Promise qaytaradi) | Yangi saqlanadigan foydalanuvchi ma'lumoti yoki mutatsiya funksiyasi kerak bo'lganda |
| `07-engine.js` | `TLA.engine` | Test yig'ish, baholash, CEFR daraja hisoblash, tavsiyalar. **DOM ga bog'liq emas — sof funksiyalar** | Baholash/daraja/tavsiya algoritmini o'zgartirganda |
| `08-charts.js` | `TLA.charts` | SVG grafiklar: shamsa (radar), bars, line, donut, sparkline | Yangi grafik turi kerak bo'lganda |
| `09-qr.js` | `TLA.qr` | QR kod generatori (ISO/IEC 18004, versiya 4, EC-L). **Sof funksiya, DOM yo'q** | QR formatini o'zgartirish kamdan-kam kerak bo'ladi |
| `10-audio.js` | `TLA.audio` | Nutq sintezi pleyeri + demo zaxira rejim | Audio pleyer xatti-harakatini o'zgartirganda |
| `11-ui.js` | `TLA.ui` | Shell (header/footer/drawer), modal, toast, qidiruv, auth oynasi, umumiy fragmentlar (`pageHead`, `levelPill`...) | Yangi umumiy UI komponenti yoki global harakat (`data-action`) qo'shganda |
| `12-page-home.js` | `TLA.pages.home` | Bosh sahifa (hero 3D, girih, bo'limlar) | Bosh sahifa bo'limini o'zgartirganda |
| `13-page-learn.js` | `TLA.pages.learn/course/lesson`, `TLA.widgets.*` | Kurs ro'yxati, kurs sahifasi, dars interfeysi (8 tab), qayta ishlatiladigan vidjetlar (audio pleyer, yozish, gapirish) | Dars/kurs sahifasi yoki vidjet o'zgarganda |
| `14-page-exams.js` | `TLA.pages.exams/examTrack/tests`, `TLA.pages.testCard` | Imtihon yo'nalishlari, test katalogi, `testCard` umumiy komponenti | Imtihon hub yoki katalog filtri o'zgarganda |
| `15-page-test.js` | `TLA.pages.test` | **Test topshirish dvigateli**: intro→running→confirm, taymer, palette, klaviatura | Test topshirish oqimini o'zgartirganda — **ehtiyot bo'ling, murakkab holat mashinasi** |
| `16-page-result.js` | `TLA.pages.result` | Natija sahifasi: ball, shamsa, tavsiyalar, mehmon darvozasi | Natija ko'rsatilishini o'zgartirganda |
| `17-page-dash.js` | `TLA.pages.dashboard/progress/certificates/verify/profile/leaderboard` | Kabinet, progress tahlili, sertifikat+QR, profil sozlamalari | Kabinet/profil/sertifikat sahifalarini o'zgartirganda |
| `18-page-content.js` | `TLA.pages.library/dictionary/blog/blogPost/free/pricing/about/teachers/contact/legal/notFound` | Kontent sahifalari, checkout modali | Kontent sahifalarini o'zgartirganda |
| `19-page-admin.js` | `TLA.pages.admin` | Admin panel — 8 tab (`views` obyekti) | Admin panelga yangi boshqaruv qo'shganda |
| `20-router.js` | `TLA.router` | Hash router (`ROUTES` massivi) + `boot()` ishga tushirish ketma-ketligi | **Yangi sahifa qo'shganda — bu faylda route ro'yxatdan o'tkaziladi** (7-bo'limga qarang) |

### 3.2 `src/styles/` — TO'LIQ XARITA

| Fayl | Nima bor |
|---|---|
| `01-tokens.css` | Rang, tipografika, masofa, soya, motion **CSS custom properties**. Barcha rang shu yerdan olinadi |
| `02-base.css` | Reset, asosiy tipografika, RTL (logical properties), a11y, `.grid`/`.grid-2/3/4` utility'lari |
| `03-components.css` | Qayta ishlatiladigan komponentlar: tugma, karta, badge, forma, modal, jadval, toast |
| `04-layout.css` | Header, mega-menyu, drawer, bottom nav, footer, `.page-head` |
| `05-pages.css` | Sahifaga xos uslublar: hero 3D, shamsa, test runner, natija, sertifikat, admin |
| `06-responsive.css` | Breakpointlar: 380 / 767 / **1119** / 1200 / 1440 / 1920+ + print. `min-width:0` tuzatishlari shu yerda |

**Qoida:** yangi rang kerak bo'lsa — `01-tokens.css` ga qo'shing, boshqa
faylga qattiq yozilgan hex kod yozmang. Yangi komponent — `03-`, sahifaga
xos uslub — `05-`.

---

## 4. Ma'lumot oqimi (bu tuzilma buzilmasligi kerak)

```
UI (sahifalar)  →  TLA.state (yagona holat)  →  TLA.store (saqlash)
       ↑                    ↑
       └──────────  TLA.engine (baholash, sof funksiyalar)
```

**Qat'iy qoida:** sahifalar (`12-19` fayllar) hech qachon `TLA.store`
ga yoki `localStorage` ga to'g'ridan-to'g'ri murojaat qilmaydi. Har doim
`TLA.state.*` orqali. Sabab: backend qo'shilganda faqat `06-state.js`
ichidagi funksiya tanalarini API chaqiruviga almashtirish kifoya —
UI kodiga tegilmaydi (`ROADMAP.md` 6-bo'limida batafsil).

**Buni qachon eslash kerak:** yangi saqlanadigan ma'lumot (masalan,
"foydalanuvchi eslatma yozadi") qo'shsangiz — avval `06-state.js` ga
`profile()` obyektiga maydon va mutatsiya funksiyasi qo'shing, keyin
sahifada shu funksiyani chaqiring.

---

## 5. Global holat shakli (`TLA.state.profile()`)

`06-state.js` dagi `blankProfile()` funksiyasi haqiqiy manba. Muhim
maydonlar (yangi funksiya yozayotganda shu shaklga tayaning):

```js
{
  level, targetLevel, goal, dailyMinutes, examDate, plan,
  xp, streak, bestStreak, lastActiveDay, activeDays: [],
  lessons: {},        // lessonId -> { done, at }
  attempts: [],        // test urinishlari, eng yangisi oxirida
  skills: {},          // skillId -> 0..100 (yumaloq o'rtacha, 06-state.js recordAttempt)
  vocab: {},           // arabcha so'z -> { added, box, due } — SRS
  bookmarks: [], certificates: [], achievements: [], payments: [],
  tasks: {},           // sana -> { taskKey: true }
  studyMinutes, onboarded
}
```

**Buni qachon yangilash kerak:** `blankProfile()` ga yangi maydon
qo'shsangiz, shu jadvalga ham qo'shing — bu keyingi agentga qayta kodni
o'qishga to'g'ri kelmasligi uchun.

---

## 6. ID naqshlari — MUHIM, buzilsa butun sayt sinadi

Ko'p joyda ID **generatsiya qilingan** naqshga tayanadi. Yangi kontent
qo'shganda shu naqshni buzmang:

| Narsa | Naqsh | Misol |
|---|---|---|
| Dars ID | `{levelId}-l{raqam}` | `a1-l1`, `b2-l24` |
| Modul ID | `{levelId}-m{raqam}` | `a1-m1` |
| Kurs ID | `{levelId}` | `a1`, `c2` |
| Darajaviy test ID | `darajaviy-{level}-{turi}` | `darajaviy-a1-grammar`, `darajaviy-b1-full` |
| Imtihon test ID | `{track}-{bo'lim}` | `multilevel-reading`, `tanal-full` |
| Savol ID | `{turi}-{level}-{raqam}` | `g-a1-1` (grammar), `v-b2-3` (vocabulary), `r-c1-2` (reading), `l-a2-1` (listening) |
| Tinglash skripti | `s_{level}` | `s_a1`, `s_c2` |
| O'qish matni | `t_{level}` | `t_a1`, `t_c2` |
| Grammatika izohi | erkin, lekin `g_{level}_{mavzu}` odat | `g_a1_jumla` |
| Sertifikat ID | `{prefiks}-{yil}-{4 raqam}` (avtomatik, `nextCertId()`) | `TLA-2026-0001` |

**Buni qachon eslash kerak:** yangi dars/savol/test qo'shganda ID ni
qo'lda o'ylab topmang — mavjud generatsiya naqshiga (`03-data-courses.js`
dagi `lvl.id + "-l" + lessonNo` kabi) ergashing yoki mavjud naqshni
takrorlang.

---

## 7. Yangi sahifa qo'shish — to'liq retsept

Agent yangi route (masalan, `#/faq`) qo'shishi kerak bo'lsa, **aniq shu
tartibda**:

1. **`20-router.js`** — `ROUTES` massiviga qo'shing:
   ```js
   { re: /^\/faq$/, page: "faq" },
   ```
2. **Tegishli `page-*.js` fayl** (yoki yangisi) — `TLA.pages.faq`
   obyektini yarating:
   ```js
   TLA.pages.faq = {
     render: function (params) { /* HTML string qaytaradi */ },
     mounted: function (params) { /* ixtiyoriy: hodisa bog'lash */ },
     unmount: function () { /* ixtiyoriy: tozalash (taymer, audio) */ },
     meta: { title: "...", desc: "..." }   // yoki function(params){...}
   };
   ```
3. **`11-ui.js`** — agar navigatsiyada ko'rinishi kerak bo'lsa,
   `navMarkup()` yoki `openDrawer()` ichiga havola qo'shing.
3. **`02-i18n.js`** — yangi matnlar uchun kalit qo'shing (qattiq matn
   yozmang).
4. **Sinov:** `node --check src/js/<fayl>.js`, keyin `python3 build.py`,
   keyin brauzerda `#/faq` ni oching.

**`render()` qoidalari:** faqat HTML string qaytaradi (DOM ga
to'g'ridan-to'g'ri yozmaydi). `mounted()` — DOM allaqachon joylashgandan
keyin hodisa bog'lash uchun. Boshqa sahifalarga qarang (`12-page-home.js`
eng sodda misol).

---

## 8. Test topshirish holat mashinasi (`15-page-test.js`)

Bu eng murakkab fayl — o'zgartirishdan oldin diqqat bilan o'qing.

```
intro (introMarkup)
  → foydalanuvchi "Boshlash" bosadi
  → session obyekti yaratiladi (module-level yopiq o'zgaruvchi, DOM emas)
  → running (runningMarkup + paintQuestion/paintPalette)
  → har javob: session.answers[i] yangilanadi, DOM qo'lda yangilanadi (to'liq qayta render emas)
  → "Yakunlash" → confirmFinish() → finish()
  → TLA.engine.scoreAttempt() → TLA.state.recordAttempt()
  → TLA.router.go("#/result/" + attemptId)
```

**Muhim xususiyatlar:**
- `session` — modul darajasidagi o'zgaruvchi, sahifalar orasida saqlanadi
  (foydalanuvchi tasodifan chiqib ketsa yo'qolmasligi uchun)
- `TLA.pages.test.isRunning()` — router bu funksiyani navigatsiya
  paytida tekshiradi va tasdiqlash so'raydi (`20-router.js`
  `onHashChange` ichida)
- Taymer `setInterval` bilan — `unmount()` da **albatta** tozalanishi
  kerak, aks holda xotira sizib chiqadi

**Buni qachon yangilash kerak:** yangi savol turi (masalan, "moslashtirish"
mashqi) qo'shsangiz, `questionMarkup()` ichiga yangi shart-blok qo'shing
va `TLA.engine.scoreAttempt()` bu turni hisoblay olishini tekshiring.

---

## 9. Dizayn tizimi — tez ma'lumotnoma

**Palitra manbai:** arab qo'lyozma illuminatsiyasi. Barcha token
`01-tokens.css` da.

| Token oilasi | Nima uchun |
|---|---|
| `--navy-*` (950→100) | Asosiy (lojuvard) — header, dark tema, tugmalar |
| `--gold-*` (700→100) | Aksent (oltin) — CTA, belgilar, links |
| `--jade-*` | Muvaffaqiyat holati |
| `--ivory`, `--surface*` | Fon ranglari |
| `--sp-*` | Masofa shkalasi (spacing) |
| `--fs-*` | Shrift o'lchami shkalasi |
| `--r-*` | Border-radius shkalasi |
| `--font-display` | Palatino/Iowan serif — sarlavhalar |
| `--font-arabic` | Amiri/Naskh stack — **faqat arabcha matn uchun**, `.ar` klassi bilan |

**Imzo komponenti — "Shamsa rozetkasi"** (`08-charts.js` `shamsa()`
funksiyasi): ko'nikmalar radar diagrammasi qo'lyozma medalyoni shaklida.
Natija, kabinet, bosh sahifada, sertifikatda (`17-page-dash.js`
`rosetteSvg()` — soddalashtirilgan versiya) takrorlanadi. **Yangi joyda
statistika ko'rsatish kerak bo'lsa, avval shamsadan foydalanish
mumkinligini o'ylang** — bu vizual izchillikni saqlaydi.

**Buni qachon yangilash kerak:** yangi rang/o'lcham kerak bo'lsa,
avval `01-tokens.css` da mavjud tokenni tekshiring, faqat yo'q bo'lsa
yangi token qo'shing.

---

## 10. i18n va RTL

- Barcha matn `02-i18n.js` da, kalit-qiymat, uch til: `uz`, `en`, `ar`
- Chaqirish: `TLA.i18n.t("kalit.yoli")` yoki interpolyatsiya bilan
  `TLA.i18n.t("misc.xpEarned", { n: 15 })`
- **Hech qachon** JS yoki HTML da qattiq yozilgan o'zbekcha/inglizcha/
  arabcha matn qoldirmang — har doim i18n kalitiga qo'shing
- RTL avtomatik: `TLA.i18n.set("ar")` `document.documentElement.dir`
  ni o'zgartiradi, CSS logical properties (`margin-inline-start` va
  h.k.) qolgan ishni qiladi
- Arabcha matn parchalari (til emas, kontent — masalan, Qur'on oyati
  emas, dars matni) `.ar` klassi bilan belgilanadi, `--font-arabic`
  ishlatadi, `dir="rtl"` mahalliy

**Buni qachon yangilash kerak:** yangi UI matni qo'shganda — darhol
uchala tilga kalit qo'shing, hech bo'lmasa `uz` ni to'ldiring va
qolganini keyinroq tarjima qilish uchun belgilang.

---

## 11. Sayt sozlamalari va build jarayoni

```
site.config.json  →  build.py o'qiydi  →  window.TLA_SITE_CONFIG
                                        →  05-data-content.js:
                                           deepMerge(baseConfig, siteConfig)
                                        →  TLA.data.defaultConfig
                                        →  06-state.js: agar saqlangan
                                           configVersion < fayl versiyasi,
                                           yangi qiymat ustun turadi
```

**Nega bu muhim:** bu — statik saytda "admin panel o'zgarishi hammaga
yetib borishi" muammosining yechimi. `site.config.json` build vaqtida
saytga **kiritiladi**, shuning uchun har bir tashrifchi ko'radi (admin
panelda kiritilgan narsa esa faqat o'sha brauzerda qoladi).

**Buni qachon yangilash kerak:** yangi global sayt sozlamasi (masalan,
yangi ijtimoiy tarmoq maydoni) qo'shsangiz: `site.config.json` ga
maydon qo'shing → `05-data-content.js` dagi `baseConfig` ga standart
qiymat qo'shing → kerak bo'lsa `19-page-admin.js` dagi `settings` tab
formasiga input qo'shing.

---

## 12. Backend integratsiyasi — nima bor, nima hali yo'q

**2026-08-15 dan boshlab qisman backend bor:** `backend/server.py`
(Python stdlib, SQLite) — **faqat autentifikatsiya**. To'liq
ma'lumotnoma: `BACKEND.md`.

### 12.1 Auth oqimi (`05b-api.js` + `06-state.js`)

```
TLA.state.register()/login()
   → avval TLA.api orqali backendga so'rov
   → tarmoq XATOSI (backend ishlamayapti) → avtomatik ravishda
     registerLocal()/loginLocal() ga o'tadi (asl, backendsiz mantiq —
     o'zgarishsiz saqlangan)
   → backend RAD ETSA (noto'g'ri parol, email band) → aniq xato
     qaytariladi, lokalga O'TKAZILMAYDI
   → backend TASDIQLASA → mirrorBackendUser() foydalanuvchini
     lokal data.users/data.profiles ga ko'chiradi (shu orqali admin
     panel, leaderboard va boshqa hamma narsa o'zgarishsiz ishlayveradi)
```

**Muhim oqibat:** `TLA.state.register()` va `TLA.state.login()` endi
**async** — `Promise` qaytaradi. Chaqiruvchi kod `await` qilishi kerak
(`11-ui.js` `openAuth()` da namuna bor). `TLA.state.logout()` esa
**sinxron** qoladi (backend tokenini bekor qilish fon rejimida,
"fire-and-forget" tarzda amalga oshadi — chaqiruvchi kutishi shart
emas).

Boot ketma-ketligiga qo'shildi (`20-router.js`):
```
TLA.store.load → TLA.state.init() → TLA.state.restoreSession()
   → i18n.set → ui.applySettings → ui.renderShell → ...
```
`restoreSession()` saqlangan token bo'lsa `/api/auth/me` ni tekshiradi
(sahifa qayta yuklanganda sessiya yo'qolmasligi uchun); token bo'lmasa
darhol qaytadi — statik (backendsiz) foydalanuvchi uchun **hech qanday
tarmoq so'rovi yo'q, hech qanday kechikish yo'q**.

### 12.2 Hali lokal (backend qamrab olmaydi)

- `15-page-test.js` da savol javoblari **hali mijozda** — bu keyingi
  backend bosqichi (`ROADMAP.md` 6-bo'lim, 7.3-band; `BACKEND.md` 1-bo'lim)
- Kurs progressi, XP, seriya, test natijalari, sertifikatlar — hali
  faqat `localStorage`
- Admin panel roli o'zgartirish, `updateUser` (profil ism/email) —
  faqat lokal, backendga yozilmaydi (`06-state.js` da izohlangan —
  "Local-only for now")
- To'lov (`recordPayment`) — demo, haqiqiy pul harakati yo'q

**Bu cheklovlarni kengaytirish uchun** — `BACKEND.md` 12-bo'limidagi
navbatni ko'ring va bir xil naqshga ergashing: yangi jadval + yangi
endpoint `backend/server.py` da + `06-state.js` dagi mos funksiyani
"backend avval, tarmoq xatosida lokal zaxira" qoliplariga moslang
(yuqoridagi 12.1 kabi).

---

## 13. Sinov

```bash
# Har bir o'zgartirishdan keyin — MAJBURIY
node --check src/js/<o'zgartirilgan-fayl>.js

# Loyihani qayta yig'ish
python3 build.py

# To'liq regressiya (Puppeteer kerak — Chromium yo'li mashinaga bog'liq)
node tests/e2e.js          # 26 marshrut, JS xatolari
node tests/responsive.js   # 26 sahifa x 5 ekran o'lchami, overflow
node tests/journey.js      # to'liq foydalanuvchi oqimi
node tests/final.js        # a11y, klaviatura, RTL, saqlash

# Backend o'zgartirilgan bo'lsa — qo'shimcha:
python3 -m py_compile backend/server.py
# Server + curl asosidagi to'liq API sinovi tests/backend_e2e.sh da
# (qo'lda ishga tushiriladi: server.py fon rejimida, keyin curl bilan
# tekshirish — bg jarayonlar tool chaqiruvlari orasida saqlanmaydi,
# shuning uchun server ishga tushirish + sinov BITTA buyruqda bo'lishi
# kerak, ARCHITECTURE.md ushbu maslahatni o'zi shunday tekshirgan edi)
```

**Buni qachon ishga tushirish kerak:** har qanday `src/js/` yoki
`src/styles/` o'zgartirishidan keyin, kamida `node --check` va
`python3 build.py`. Layout o'zgartirsangiz — `tests/responsive.js`
ham ishga tushiring (grid/flex o'zgarishi mobil overflow keltirib
chiqarishi mumkin — bu loyihada avval sodir bo'lgan haqiqiy xato).
`backend/server.py` o'zgartirsangiz — path traversal himoyasini
**xom soket orqali** sinang (curl/fetch URL'ni yuborishdan oldin
o'zi normalizatsiya qiladi, shuning uchun oddiy curl sinovi soxta
"xavfsiz" natija berishi mumkin).

---

## 14. Boshqa hujjatlar — qaysi birini qachon o'qish

| Fayl | Qachon o'qish kerak |
|---|---|
| `ANALYSIS.md` | Nima uchun ba'zi funksiyalar yo'q yoki cheklangan (masalan, AI baholash yo'q — atayin) |
| `README.md` | Umumiy statistika, dizayn tizimi tavsifi, backend ulash nuqtalari jadvali |
| `ROADMAP.md` | To'lov/AI/production/Next.js-Prisma ko'chirish haqida vazifa berilganda — **to'liq Prisma sxemasi shu yerda** |
| `DEPLOY.md` | Statik hosting/deploy haqida vazifa berilganda (Netlify/Vercel/GitHub Pages) |
| `BACKEND.md` | Backend (`backend/server.py`) haqida vazifa berilganda — API ma'lumotnomasi, deploy, xavfsizlik, demo hisoblar |
| **`ARCHITECTURE.md` (shu fayl)** | Har qanday kod o'zgartirish vazifasidan oldin, birinchi navbatda |

---

## 15. Halollik cheklovlari — bularni chetlab o'tmang

Loyihaning asosiy tamoyili: **hech qachon o'ylab topilgan ma'lumot
ko'rsatilmaydi.** Bu vazifa berilganda eslab qoling:

- O'qituvchi profillari — faqat `TLA.state.teachers()` orqali admin
  kiritgan. Namuna/placeholder o'qituvchi yozmang.
- Kontaktlar — faqat `site.config.json` / `TLA.state.config()` orqali.
  Bo'sh bo'lsa "admin panelda kiritiladi" ko'rsatiladi, soxta raqam yo'q.
- Statistika ("N talaba", "M% muvaffaqiyat") — faqat haqiqiy hisoblangan
  qiymatlar (`TLA.state.stats()`, `TLA.data.questions.length` va h.k.).
  Ixtiro qilingan raqam yozmang.
- Sertifikat — "rasmiy imtihon sertifikati emas" degan izoh har doim
  yonida bo'lishi kerak (`TLA.ui.complianceNote`).
- Imtihon sahifalari — platforma hech bir rasmiy tashkilotning vakili
  emasligi haqida izoh saqlanishi kerak.

Batafsil: `ANALYSIS.md` 4-bo'lim.

---

## 16. Tezkor boshlash — agent uchun tekshiruv ro'yxati

Har qanday vazifa boshida:

1. Vazifa qaysi qatlamga tegishli? → 3-bo'limdagi jadvaldan faylni toping
2. Yangi ma'lumot saqlash kerakmi? → 5-bo'lim (holat shakli) + `06-state.js`
3. Yangi ID kerakmi? → 6-bo'lim (naqshlar)
4. Yangi sahifa/route kerakmi? → 7-bo'lim (to'liq retsept)
5. Yangi matn kerakmi? → 10-bo'lim (i18n, hech qachon qattiq yozilmagan matn)
6. O'zgartirish tugagach → 13-bo'lim (sinov)
7. Loyiha strukturasi o'zgardimi (yangi fayl, yangi eksport, yangi
   naqsh)? → **shu faylni yangilang**
