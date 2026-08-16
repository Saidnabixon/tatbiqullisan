# Serverda ishga tushirish qo'llanmasi

Saytni internetga chiqarish va foydalanuvchilar masofadan ishlata olishi
uchun amaliy qadamlar.

> **Yangilanish:** endi ixtiyoriy backend ham mavjud (haqiqiy
> autentifikatsiya, ikkita demo hisob). Bu qo'llanma **statik**
> (backendsiz) deploy uchun — eng oson va tezkor yo'l, hamon to'liq
> ishlaydigan mahsulot beradi. Backend qo'shishni istasangiz —
> **`BACKEND.md`** ga qarang; ikkalasi bir-birini istisno qilmaydi,
> statik sayt backend bilan ham, backendsiz ham ishlayveradi
> (`ARCHITECTURE.md` 12-bo'lim).

---

## 0. Avval — nima ishlaydi, nima yo'q

Bu muhim. Sayt **statik** (server tomonida baza yo'q), shuning uchun
deploydan keyin holat quyidagicha bo'ladi:

### ✅ Ishlaydi

- Sayt istalgan joydan, istalgan qurilmadan ochiladi
- Barcha 26 sahifa, testlar, darslar, kutubxona, lug'at
- Ro'yxatdan o'tish va kirish
- Test topshirish, natija, tavsiyalar, sertifikat
- Progress, XP, seriya, yutuqlar
- UZ/EN/AR + RTL, dark mode
- Telefon, planshet, kompyuter

### ⚠️ Cheklovlar (statik sayt tabiati)

| Cheklov | Ma'nosi |
|---|---|
| **Ma'lumot brauzerda saqlanadi** | Har bir foydalanuvchining progressi **o'z qurilmasida**. Telefonda boshlagan darsni kompyuterda davom ettira olmaydi. |
| **Hisoblar umumiy emas** | Bir foydalanuvchi ro'yxatdan o'tsa, boshqa qurilmada u hisob mavjud emas. *(Bu cheklov backend bilan hal qilinadi — pastga qarang.)* |
| **Admin paneldagi o'zgarish tarqalmaydi** | Admin panelda savol qo'shsangiz — faqat sizning brauzeringizda. Tashrifchilarga yetib bormaydi. |
| **Brauzer ma'lumotini tozalasa — progress yo'qoladi** | Ogohlantirish ko'rsatilsa yaxshi bo'ladi. |
| **To'lov demo** | Haqiqiy pul qabul qilinmaydi. |
| **SEO cheklangan** | Hash-router (`#/learn`) sababli Google faqat bosh sahifani indekslaydi. |

### Shunda ham nega deploy qilish arziydi

- Odamlarga **ko'rsata olasiz** — havola yuborasiz, ular ishlatib ko'radi
- **Haqiqiy fikr yig'asiz** — qaysi qism tushunarsiz, nima yetishmayapti
- Kontent sifatini sinaysiz — savollar juda oson yoki qiyinmi
- Bu **portfolio** va investor/hamkorga ko'rsatiladigan dalil
- Backend qo'shganda dizayn va oqim tayyor bo'ladi

**Kontaktlar va narxlar esa hammaga ko'rinadi** — ular `site.config.json`
orqali saytga kiritiladi (2-bo'limga qarang). **Hisoblar** endi ham
umumiy bo'lishi mumkin — agar `backend/server.py` ni ham ishga
tushirsangiz (`BACKEND.md`), yuqoridagi "Hisoblar umumiy emas" qatori
bekor bo'ladi: istalgan qurilmadan bir xil email/parol bilan kirish
mumkin bo'ladi. Qolgan cheklovlar (progress, admin kontent, to'lov)
hali ham amal qiladi — backend hozircha faqat auth'ni qamrab oladi.

---

## 1. Loyiha tuzilmasi

```
tatbiqul-lisan/
├── public/                  ← STATIK HOSTINGGA YUKLANADIGAN PAPKA
│   ├── index.html               yig'ilgan sayt (~600 KB, gzip ~140 KB)
│   ├── og-image.png             ijtimoiy tarmoqda ulashish rasmi
│   ├── robots.txt
│   ├── sitemap.xml
│   └── _headers                 kesh va xavfsizlik sarlavhalari
├── src/                     ← manba kod (tahrirlanadigan)
│   ├── index.template.html
│   ├── styles/                  6 ta CSS
│   └── js/                      23 ta JS
├── backend/                 ← IXTIYORIY — auth backend (BACKEND.md)
│   └── server.py                Python stdlib, SQLite, nol tashqi paket
├── tests/                   ← Puppeteer sinov skriptlari
├── .github/workflows/
│   └── deploy.yml               avtomatik deploy (statik qism uchun)
├── site.config.json         ← KONTAKT, IJTIMOIY TARMOQ, DOMEN, apiBase
├── build.py                     yig'uvchi skript
├── netlify.toml                 Netlify sozlamasi
├── vercel.json                  Vercel sozlamasi
├── .gitignore
└── *.md                     ← README / ARCHITECTURE / BACKEND / ROADMAP / ANALYSIS
```

---

## 2. Birinchi qadam: sozlamalarni to'ldirish

`site.config.json` faylini oching va to'ldiring:

```json
{
  "version": 1,
  "siteUrl": "https://tatbiqul-lisan.uz",

  "contact": {
    "phone": "+998 90 123 45 67",
    "email": "info@tatbiqul-lisan.uz",
    "telegram": "https://t.me/kanalingiz",
    "address": "Toshkent sh., ...",
    "workHours": "Du–Sha 09:00–18:00"
  },

  "social": {
    "telegram": "https://t.me/kanalingiz",
    "instagram": "https://instagram.com/hisobingiz",
    "youtube": ""
  },

  "currency": "so'm",
  "certificatePrefix": "TLA",
  "verifyBaseUrl": "https://tatbiqul-lisan.uz/#/verify?id="
}
```

Keyin:

```bash
python build.py
```

⚠️ **Muhim qoida:** kelajakda kontaktni o'zgartirsangiz, **`version`
raqamini oshiring** (1 → 2). Aks holda ilgari saytga kirgan
foydalanuvchilar eski raqamni ko'rishda davom etadi. Versiya oshirilganda
ularning **progressi va hisobi saqlanib qoladi** — faqat sayt sozlamalari
yangilanadi.

**`apiBase` maydoni** (yuqoridagi namunada ko'rsatilmagan, standart
bo'sh) — faqat backend qo'shsangiz kerak bo'ladi. Batafsil: `BACKEND.md`.

Bo'sh qoldirilgan maydonlar saytda «Admin panelda kiritiladi» deb
ko'rsatiladi — soxta raqam hech qachon chiqmaydi.

---

## 3. Hosting tanlash

| Variant | Narx | Qiyinlik | Kimga mos |
|---|---|---|---|
| **Netlify** | Bepul | ⭐ Eng oson | Tavsiya etiladi |
| **Cloudflare Pages** | Bepul | ⭐ Oson | Tezlik muhim bo'lsa |
| **Vercel** | Bepul | ⭐ Oson | Keyin Next.js ga o'tsangiz |
| **GitHub Pages** | Bepul | ⭐⭐ | Kod allaqachon GitHub da bo'lsa |
| **O'zbekiston hostingi** | ~50–150 ming so'm/oy | ⭐⭐⭐ | Mahalliy domen/tezlik kerak bo'lsa |

Uchala bepul variant ham: HTTPS avtomatik, CDN, o'z domeningizni ulash
mumkin, oylik trafik sizning hajmingiz uchun yetarli.

---

## 4. Deploy: Netlify (eng oson yo'l)

### 4.1 Sichqoncha bilan (2 daqiqa, GitHub kerak emas)

1. https://app.netlify.com/drop ochiladi
2. `public` papkasini sahifaga **sudrab tashlang**
3. Tayyor — sayt `random-nom.netlify.app` manzilida ishlaydi

Yangilash: qayta `python build.py` → papkani qayta tashlang.

### 4.2 GitHub orqali (tavsiya etiladi)

```bash
cd tatbiqul-lisan
git init
git add .
git commit -m "feat: initial platform"
git branch -M main
git remote add origin https://github.com/FOYDALANUVCHI/tatbiqul-lisan.git
git push -u origin main
```

Netlify da: **Add new site → Import from Git → GitHub → repo tanlang**

Sozlamalar (`netlify.toml` dan avtomatik o'qiladi):
- Build command: `python3 build.py`
- Publish directory: `public`

Endi har `git push` da sayt **avtomatik yangilanadi**.

---

## 5. Deploy: GitHub Pages

Reponi GitHub ga yuklang (yuqoridagi buyruqlar), keyin:

1. Repo → **Settings → Pages**
2. **Source: GitHub Actions** ni tanlang
3. `.github/workflows/deploy.yml` allaqachon tayyor — `main` ga push
   qilganda avtomatik ishga tushadi

Sayt: `https://FOYDALANUVCHI.github.io/tatbiqul-lisan/`

⚠️ **Eslatma:** GitHub Pages da sayt ildizda emas, papkada bo'ladi.
Bizning sayt nisbiy yo'llar ishlatgani uchun bu muammo qilmaydi, lekin
`og-image.png` uchun `site.config.json` dagi `siteUrl` ni to'liq manzil
bilan yozing.

---

## 6. Deploy: Cloudflare Pages

1. https://dash.cloudflare.com → **Workers & Pages → Create → Pages**
2. GitHub reposini ulang
3. Build command: `python3 build.py`
4. Output directory: `public`

`public/_headers` fayli avtomatik o'qiladi — kesh va xavfsizlik
sarlavhalari ishlaydi.

---

## 7. O'z domeningizni ulash

1. Domen sotib oling (`.uz` uchun: ahost.uz, uzinfocom; xalqaro:
   Namecheap, Cloudflare)
2. Hosting panelida **Add custom domain**
3. DNS yozuvlarini qo'shing (hosting aniq qiymatlarni ko'rsatadi):

```
Turi    Nomi    Qiymati
CNAME   www     sizning-saytingiz.netlify.app
A       @       75.2.60.5              (Netlify ko'rsatgan IP)
```

4. HTTPS avtomatik yoqiladi (Let's Encrypt) — 5–30 daqiqa kutiladi
5. `site.config.json` dagi `siteUrl` ni yangi domenga o'zgartiring,
   `version` ni oshiring, qayta build qiling

---

## 8. Deploydan keyin — birinchi kunlik ro'yxat

- [ ] Sayt telefonda ochilishini tekshiring (o'z telefoningizda)
- [ ] Ro'yxatdan o'tib, testni to'liq topshirib ko'ring
- [ ] **Birinchi ro'yxatdan o'tgan hisob — admin.** Uni o'zingiz oling.
- [ ] Kontaktlar to'g'ri ko'rinayotganini tekshiring
- [ ] Telegramda havolani o'zingizga yuboring — ulashish rasmi
      chiqayotganini ko'ring
- [ ] Huquqiy sahifalarni o'qing (`#/legal/privacy`) — hozir **namuna
      matn**, yurist bilan to'ldirilishi kerak
- [ ] Google Search Console ga saytni qo'shing
- [ ] Analitika ulang (Plausible yoki PostHog — cookie'siz variantlari bor)

---

## 9. Kontentni yangilash

Statik saytda kontent **kodda** — o'zgartirish uchun:

| Nimani o'zgartirmoqchisiz | Qaysi fayl |
|---|---|
| Savollar | `src/js/04-data-questions-a.js`, `-b.js` |
| Kurslar, darslar | `src/js/03-data-courses.js` |
| Lug'at, blog, kutubxona, tariflar | `src/js/05-data-content.js` |
| Kontakt, ijtimoiy tarmoq | `site.config.json` |
| Matnlar/tarjimalar (UZ/EN/AR) | `src/js/02-i18n.js` |
| Ranglar, shriftlar | `src/styles/01-tokens.css` |

Har o'zgartirishdan keyin:

```bash
python build.py       # va git push (avtomatik deploy bo'lsa)
```

**Maslahat:** o'zgartirishdan oldin sintaksisni tekshiring —

```bash
node --check src/js/04-data-questions-a.js
```

Bitta qo'shtirnoq xatosi butun saytni ishlamay qo'yadi. Bu tekshiruv
1 soniya oladi va sizni katta boshog'riqdan saqlaydi.

---

## 10. Foydalanuvchini ogohlantirish (tavsiya)

Agar **backendsiz** ishlatsangiz, foydalanuvchi progressi brauzerda
ekanini bilishi adolatli. `src/js/11-ui.js` ichida ro'yxatdan o'tish
oynasiga bitta qator qo'shishingiz mumkin:

> «Hozircha progressingiz shu qurilmada saqlanadi. Brauzer ma'lumotini
> tozalasangiz yoki boshqa qurilmadan kirsangiz, natijalar ko'chmaydi.»

Bu ishonchni oshiradi va keyinroq «progressim yo'qoldi» degan shikoyatni
oldini oladi. Agar `backend/server.py` ni ham ishlatsangiz (`BACKEND.md`),
bu ogohlantirish endi to'liq to'g'ri emas — **hisob** endi qurilmalar
orasida ko'chadi, faqat **progress/XP/sertifikat** hali ko'chmaydi (bu
farqni ogohlantirish matnida aniq ko'rsating, aks holda chalkashlik
tug'diradi).

---

## 11. Keyingi qadam qachon kerak bo'ladi

**Auth backend allaqachon mavjud** (`backend/server.py`, `BACKEND.md`)
— agar hali ishlatmagan bo'lsangiz, keyingi qadam ko'pincha shu, chunki
eng tez natija beradi (bir kunlik ish, tashqi paket kerak emas).

Undan keyingi bosqich — **progress/XP/test natijalarini** ham backendga
ko'chirish — vaqti kelganini quyidagi belgilar ko'rsatadi:

- Foydalanuvchilar «boshqa telefonda progressim yo'q» deb yozishmoqda
  (auth bor, lekin progress hali lokal — `BACKEND.md` 1-bo'lim)
- Siz savol qo'shmoqchisiz, lekin har safar dasturchi kerak
- Haqiqiy to'lov qabul qilmoqchisiz
- Kim qancha o'qiganini bilmoqchisiz (analitika)
- O'qituvchi yozma ishlarni tekshirishi kerak

Bu qadam uchun naqsh tayyor — `BACKEND.md` 12-bo'limida keyingi
kengaytirish tartibi va `ARCHITECTURE.md` 12-bo'limida "backend avval,
tarmoq xatosida lokal zaxira" qolipi tasvirlangan (auth uchun qanday
qilingan bo'lsa, xuddi shunday).

To'liq, ko'p foydalanuvchili production (Postgres, to'lov, AI) uchun
esa — `ROADMAP.md`.

---

## 12. Muammolarni hal qilish

| Muammo | Yechim |
|---|---|
| `python3 build.py` — «Python was not found» | Windowsda `python build.py` yozing |
| Build da `SyntaxError: unicodeescape` | Endi tuzatildi — yo'l qattiq yozilmagan. Eski `build.py` bo'lsa, yangisini oling |
| Sayt oq ekran | Brauzerda F12 → Console. Odatda JS sintaksis xatosi. `node --check` bilan toping |
| Yangilanish ko'rinmayapti | Ctrl+Shift+R (majburiy yangilash). `_headers` bu muammoni oldini oladi |
| Kontaktlar bo'sh chiqmoqda | `site.config.json` to'ldirilganmi? `version` oshirilganmi? Qayta build qilinganmi? |
| Deploy muvaffaqiyatsiz (Netlify) | Build log ni o'qing — odatda Python versiyasi. `netlify.toml` da 3.11 belgilangan |

---

## Xulosa

Bugun qilinadigan ish: `site.config.json` ni to'ldiring → `python build.py`
→ `public` papkasini Netlify ga tashlang. **Sayt 5 daqiqada internetda
bo'ladi.**

Keyingi 2–4 hafta: haqiqiy foydalanuvchilardan fikr yig'ing, kontentni
to'ldiring. Backend — undan keyin.
