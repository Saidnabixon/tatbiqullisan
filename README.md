# Tatbiqul Lisan Academy — texnik hujjat

Arab tili o'quv va imtihon tayyorgarlik platformasi. Frontend — bitta
mustaqil HTML fayl, tashqi bog'liqliksiz; endi ixtiyoriy backend
(autentifikatsiya) bilan ham ishlaydi.

**Qaysi hujjatni o'qish kerak:**

| Vaziyat | Hujjat |
|---|---|
| Saytni shunchaki ochmoqchiman | Shu fayl, 1-bo'lim |
| Statik hostingga joylashtirmoqchiman (Netlify/Vercel/GitHub Pages) | `DEPLOY.md` |
| Backend (auth, demo hisoblar) qo'shmoqchiman | `BACKEND.md` |
| Kodni tushunmoqchiman / AI agentga vazifa bermoqchiman | `ARCHITECTURE.md` |
| To'liq production (to'lov, AI, ko'p foydalanuvchi) rejalashtirmoqchiman | `ROADMAP.md` |
| Prompt tahlili (nima qo'shildi/olib tashlandi) qiziqtiryapti | `ANALYSIS.md` |

---

## 1. Ishga tushirish

```
index.html
```

Faylni brauzerda ochish kifoya. Server, npm, internet — kerak emas
(bu — statik, backendsiz rejim; backend bilan ishga tushirish uchun
**`BACKEND.md`** ga qarang).

Joylashtirish uchun ham shu faylning o'zi yetarli: har qanday statik hosting
(GitHub Pages, Netlify, Cloudflare Pages, nginx) — build qadamisiz.

**Birinchi ishga tushirish:** backendsiz (statik) rejimda birinchi
ro'yxatdan o'tgan foydalanuvchi avtomatik **administrator** bo'ladi.
Backend ishlatilsa (`BACKEND.md`), demo admin hisobi allaqachon
mavjud (`admin@gmail.com` / `12345678`) — o'z-o'zidan ro'yxatdan
o'tish orqali admin bo'lib bo'lmaydi.

---

## 2. Nima qurilgan

| Ko'rsatkich | Miqdor |
|---|---|
| Darajalar (CEFR) | 6 (A1–C2) |
| Kurslar / modullar / darslar | 6 / 36 / **120** |
| Savollar bazasi | **88** (grammatika 28, leksika 24, o'qish 18, tinglash 18) |
| O'qish matnlari / tinglash skriptlari | 6 / 6 (tarjimasi bilan) |
| Grammatika izohlari | 11 |
| Test to'plamlari | **52** (diagnostika, 36 darajaviy, Multilevel/TANAL bo'limlari + mock) |
| Kutubxona / lug'at / blog | 19 / 28 / 6 maqola |
| Yozish / gapirish topshiriqlari | 6 / 6 |
| Yutuqlar / tariflar | 10 / 4 |
| Sahifalar (route) | 26 |
| Tillar | UZ / EN / AR (+ to'liq RTL) |
| Manba kodi | ~12 000 qator (23 JS + 6 CSS moduli) |

---

## 3. Arxitektura

### Nega vanilla?

Ishlab chiqish muhitida tarmoq o'chirilgan edi — npm va CDN mavjud emas.
Shu sababli **bog'liqliksiz SPA** tanlandi. Bu tasodifiy cheklov emas, balki
foyda keltirdi:

- Bitta fayl, nol build, nol `node_modules`
- Sovuq yuklanish tez (bir HTTP so'rov)
- Uzoq muddatli barqarorlik — eskiradigan paket yo'q

### Modul tuzilmasi

```
src/
├── index.template.html      HTML skeleti
├── styles/
│   ├── 01-tokens.css        rang, tipografika, masofa, soya tokenlari
│   ├── 02-base.css          reset, tipografika, RTL, a11y
│   ├── 03-components.css    tugma, karta, forma, modal, jadval, toast
│   ├── 04-layout.css        header, mega-menyu, drawer, footer, bottom nav
│   ├── 05-pages.css         hero 3D, shamsa, test, natija, sertifikat
│   └── 06-responsive.css    320 / 768 / 1024 / 1440 / 1920 + print
└── js/
    ├── 00-utils.js          DOM, formatlash, urug'langan tasodifiy
    ├── 01-storage.js        3 bosqichli saqlash drayveri
    ├── 02-i18n.js           UZ/EN/AR lug'at (~230 kalit) + RTL
    ├── 03-data-*.js         kurslar, matnlar, grammatika
    ├── 04-data-questions-*  savollar bazasi
    ├── 05-data-content.js   testlar, kutubxona, lug'at, blog, tariflar
    ├── 05b-api.js           backend mijozi (fetch, token, BACKEND.md)
    ├── 06-state.js          holat, auth (backend + lokal zaxira), progress
    ├── 07-engine.js         test yig'ish, baholash, CEFR, tavsiyalar
    ├── 08-charts.js         SVG grafiklar (shamsa rozetkasi)
    ├── 09-qr.js             QR generatori (ISO/IEC 18004)
    ├── 10-audio.js          nutq sintezi + demo zaxira
    ├── 11-ui.js             shell, modal, drawer, qidiruv, auth
    ├── 12–19-page-*.js      sahifalar
    └── 20-router.js         hash router + boot

backend/
└── server.py                auth API (Python stdlib, SQLite) — BACKEND.md
```

Har bir faylning to'liq tavsifi, nomlar fazosi va "qachon o'zgartirish
kerak" ko'rsatmasi: **`ARCHITECTURE.md`** — bu hujjat kod bilan birga
yangilanib boriladi va AI kodlash agentlari uchun mo'ljallangan.

**Qayta qurish:**

```bash
python3 build.py
```

Fayllarni nom tartibida o'qib, `index.html` ga inline qiladi. Barcha modullar
`window.TLA` nomlar fazosiga yozadi, shuning uchun bitta `<script>` blokida
to'qnashuv bo'lmaydi.

---

## 4. Dizayn tizimi

**Vizual manba:** arab qo'lyozma illuminatsiyasi (naqsh, oltin, lojuvard).

| Token | Qiymat |
|---|---|
| Lojuvard (asosiy) | `#071A2C` |
| Oltin (aksent) | `#C8A253` |
| Fil suyagi (fon) | `#FBF7EF` |
| Yashma (muvaffaqiyat) | `#1E7D6E` |
| Rubrika qizili | `#B3452F` |

**Tipografika:** display — Palatino/Iowan serif stack; matn — tizim sans;
arabcha — Amiri/Naskh stack. Veb-shrift yuklanmaydi (tarmoq yo'q) — barchasi
tizim shriftlariga tayanadi.

**Imzo elementi — "Shamsa rozetkasi":** ko'nikmalar radar diagrammasi
qo'lyozma medalyoni shaklida. Natijalar, kabinet va bosh sahifada takrorlanadi.

**Harakat:** `prefers-reduced-motion` va `data-motion="off"` hurmat qilinadi —
ikkalasida ham animatsiyalar to'xtaydi, girih naqshi statik chiziladi.

---

## 5. Ma'lumot oqimi

```
UI  →  TLA.state  →  TLA.store  →  window.storage / localStorage / xotira
        ↑
   TLA.engine (baholash, daraja, tavsiya)
```

**Muhim tamoyil:** UI hech qachon saqlashga to'g'ridan-to'g'ri murojaat qilmaydi.
Barcha o'zgarishlar `TLA.state` orqali o'tadi — shuning uchun backendga o'tish
uchun faqat `state.js` ichidagi funksiya tanalarini API chaqiruvlariga
almashtirish kifoya.

### Saqlash drayveri (`01-storage.js`)

Zanjir: `window.storage` (artifact muhiti) → `localStorage` → xotira.
Boot'da bir marta yuklanadi, yozuv debounce bilan.

---

## 6. Backend ulash nuqtalari

**Yangilanish (2026-08-15):** autentifikatsiya uchun backend endi
**mavjud** — `backend/server.py` (Python stdlib, SQLite). To'liq
ma'lumotnoma, API, deploy va xavfsizlik holati: **`BACKEND.md`**.
Qolgan qatorlar hali ham ochiq — quyidagi jadval nima qilinganini
(✅) va nima hali qolganini (—) ko'rsatadi.

| Fayl | Funksiya | Holat |
|---|---|---|
| `06-state.js` | `register`, `login` | ✅ Bajarildi — `backend/server.py`, PBKDF2-HMAC-SHA256, `BACKEND.md` |
| `06-state.js` | `recordAttempt` | — Test javoblari hali mijozda (pastga qarang) |
| `06-state.js` | `recordPayment` | — Demo, haqiqiy to'lov yo'q |
| `06-state.js` | `issueCertificate`, `findCertificate` | — Hali lokal |
| `06-state.js` | `notify` | — Telegram Bot API ulanmagan |
| `06-state.js` | `track` | — Hali lokal (`localStorage`) |
| `01-storage.js` | `load`, `save` | — Auth'dan tashqari hamma narsa hali lokal |
| `10-audio.js` | `createPlayer` | — Hali nutq sintezi, haqiqiy audio fayl yo'q |

### Xavfsizlik eslatmalari

- **Test javoblari hozir mijozda.** Ishlab chiqarishda savol javoblari serverda
  qolishi va baholash `POST /api/attempts` orqali bo'lishi shart.
- **Rol tekshiruvi endi qisman serverda** — auth uchun `backend/server.py`
  rolni bazadan qaytaradi. Admin panelning boshqa amallari (rol
  o'zgartirish va h.k.) hali faqat frontendda tekshiriladi — bu hali
  haqiqiy himoya emas (`BACKEND.md` 8-bo'lim).
- **To'lov kalitlari hech qachon frontendda saqlanmaydi.**
- Parol endi **haqiqiy xeshlanadi** (PBKDF2-HMAC-SHA256, tuz bilan) —
  agar backend ishlamasa, sayt eski demo-digest zaxira rejimiga
  o'tadi (`ARCHITECTURE.md` 12-bo'lim).

---

## 7. Next.js + Prisma ga ko'chirish yo'li

Ko'chirish bosqichma-bosqich mumkin — hammasini birdan qayta yozish shart emas.

### 7.1 Ma'lumotlar bazasi sxemasi (Prisma)

```prisma
model User {
  id        String   @id @default(cuid())
  name      String
  email     String   @unique
  password  String                          // argon2 hash
  role      Role     @default(STUDENT)
  createdAt DateTime @default(now())
  profile   Profile?
  attempts  Attempt[]
  payments  Payment[]
  certificates Certificate[]
}

model Profile {
  userId      String  @id
  user        User    @relation(fields: [userId], references: [id], onDelete: Cascade)
  level       String?
  targetLevel String  @default("b2")
  examDate    DateTime?
  xp          Int     @default(0)
  streak      Int     @default(0)
  plan        String  @default("free")
  skills      Json                          // { grammar: 80, ... }
  vocab       SavedWord[]
  lessons     LessonProgress[]
}

model Question {
  id       String @id @default(cuid())
  level    String
  skill    String
  promptUz String
  ar       String?
  options  Json
  answer   Int                              // serverda qoladi, mijozga yuborilmaydi
  expUz    String
  textId   String?
  scriptId String?
}

model Attempt {
  id          String   @id @default(cuid())
  userId      String
  user        User     @relation(fields: [userId], references: [id])
  testId      String
  score       Int
  correct     Int
  total       Int
  skillScores Json
  level       String?
  seconds     Int
  detail      Json
  createdAt   DateTime @default(now())
}

model Certificate {
  id       String   @id                     // TLA-2026-0001
  userId   String
  user     User     @relation(fields: [userId], references: [id])
  titleUz  String
  level    String?
  score    Int?
  issuedAt DateTime @default(now())
}

model SavedWord     { id String @id @default(cuid()) profileId String word String box Int @default(0) due DateTime }
model LessonProgress{ id String @id @default(cuid()) profileId String lessonId String doneAt DateTime @default(now()) }
model Payment       { id String @id @default(cuid()) userId String planId String amount Int method String status String createdAt DateTime @default(now()) }
model Teacher       { id String @id @default(cuid()) name String title String? bio String? languages String? }
model Config        { key String @id value Json }

enum Role { STUDENT TEACHER ADMIN }
```

### 7.2 Bosqichlar

1. **Kontentni ko'chirish** — `03-data-*.js` va `04-data-questions-*.js`
   fayllaridagi massivlarni seed skriptiga aylantirish (`prisma/seed.ts`).
   Kontent tuzilmasi allaqachon toza JSON — qo'lda qayta yozish kerak emas.

2. **API marshrutlari** — 6-bo'limdagi jadval bo'yicha `app/api/*/route.ts`.
   Baholash logikasi `07-engine.js` dan deyarli o'zgarishsiz ko'chadi
   (u sof funksiyalar to'plami, DOM ga bog'liq emas).

3. **Sahifalar** — har bir `TLA.pages.*` moduli bitta React sahifasiga mos
   keladi. Route jadvali `20-router.js` da tayyor holda mavjud va Next.js
   `app/` tuzilmasiga to'g'ridan-to'g'ri ko'chadi:

   ```
   #/learn/:id       →  app/learn/[id]/page.tsx
   #/lesson/:id      →  app/lesson/[id]/page.tsx
   #/test/:id        →  app/test/[id]/page.tsx
   #/result/:id      →  app/result/[id]/page.tsx
   #/blog/:slug      →  app/blog/[slug]/page.tsx
   #/legal/:doc      →  app/legal/[doc]/page.tsx
   ```

4. **CSS** — `styles/` fayllari o'zgarishsiz ishlaydi (Tailwind shart emas).
   Tokenlar allaqachon CSS custom properties.

5. **i18n** — `02-i18n.js` lug'ati `next-intl` yoki `next-i18next` formatiga
   mexanik ravishda o'tadi.

**Muhim:** `07-engine.js` (baholash), `08-charts.js` (SVG) va `09-qr.js`
(QR) — sof funksiyalar, brauzer API siga bog'liq emas. Ular serverda ham,
mijozda ham o'zgarishsiz ishlaydi.

---

## 8. Sinovdan o'tkazish

Chromium (Puppeteer) orqali avtomatik tekshirildi:

| Sinov | Natija |
|---|---|
| 26 sahifa × 5 ekran o'lchami (360/390/768/1024/1440) | **130/130** — gorizontal overflow yo'q |
| To'liq foydalanuvchi oqimi (ro'yxat → onboarding → 20 savolli test → natija → kabinet → admin 8 tab → sertifikat) | **Nol JS xatosi** |
| QR kodni teskari dekodlash | **To'liq mos** (mode 4, uzunlik 40, payload bir xil) |
| Klaviatura (1–4/A–D, o'q, F) | ishlaydi |
| Mehmon darvozasi (gate) | ball ochiq, tahlil yopiq, progress ro'yxatdan o'tgach ko'chadi |
| Til/tema saqlanishi (reload) | ishlaydi (AR + RTL + dark) |
| `prefers-reduced-motion` | animatsiya to'xtaydi |
| A11y (skip-link, label, aria) | yorliqsiz input 0, nomsiz tugma 0 |

Sinov skriptlari: `tests/` (e2e.js, journey.js, responsive.js, final.js).

### Topilgan va tuzatilgan xatolar

1. Uch joyda qo'shtirnoq mos kelmasligi — sintaksis buzilishi
2. **Grid blowout** — `1fr` da `min-width:auto` sababli dars va admin sahifalari
   mobilda 710px ga cho'zilardi → `minmax(0,1fr)` + `min-width:0`
3. 1024px da desktop navigatsiya sig'masdi (1032px kerak edi) → breakpoint
   1023 → 1119px
4. Shamsa rozetkasidagi yorliqlar SVG chetida kesilardi → viewBox kengaytirildi
5. Yutuq nomlari kartadan chiqib ketardi → 2 qatorli clamp
6. Toast'da `✦` belgisi ikkilanardi
7. Arabcha gliflar (logo, harf plitalari) juda kichik edi

---

## 9. Ma'lum cheklovlar

- **Audio** — qurilma nutq sintezi. Arabcha ovoz bo'lmasa, demo rejim ochiq
  belgilanadi va matn ko'rsatiladi. Haqiqiy audio fayllar bilan almashtirilishi
  kerak.
- **To'lov** — demo. Haqiqiy pul harakati yo'q.
- **Test javoblari mijozda** — prototip uchun; ishlab chiqarishda serverga
  ko'chirilishi shart (6-bo'limga qarang).
- **Kutubxona PDF/video** — soxta havola yaratilmaydi; fayl admin panel orqali
  biriktiriladi.
- **Ma'lumotlar qurilmada** — `localStorage`. Boshqa qurilmada sinxronlanmaydi.

---

## 10. Litsenziya va kontent

Barcha o'quv matnlari, savollar, tinglash skriptlari va blog maqolalari shu
loyiha uchun **maxsus yozilgan**. Hech bir rasmiy imtihon savoli yoki
mualliflik huquqi bilan himoyalangan material ko'chirilmagan.

Platforma hech bir imtihon tashkilotining rasmiy vakili emas.
