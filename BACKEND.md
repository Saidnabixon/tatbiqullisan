# Backend qo'llanmasi

Bu loyihaning birinchi backend bosqichi: **haqiqiy autentifikatsiya**.
Ma'lumot endi faqat brauzerda emas — server tomonida, haqiqiy bazada
saqlanadi va istalgan qurilmadan bir xil hisob bilan kirish mumkin.

---

## 1. Bu backend nima qiladi (va nima qilmaydi)

### ✅ Qamrab oladi (v1)

- Ro'yxatdan o'tish, kirish, chiqish — **server tomonida**
- Parollar xesh sifatida saqlanadi (PBKDF2-HMAC-SHA256, hech qachon
  ochiq matnda emas)
- Sessiya token orqali — brauzerni yopib qaytsangiz ham tizimga
  kirgan holingizcha qolasiz (30 kun)
- Ikki demo hisob avtomatik yaratiladi (3-bo'limga qarang)
- Rol asosida kirish (admin / student)

### ⚠️ Hali qamrab olmaydi (keyingi bosqich)

| Nima | Hozircha qanday ishlaydi |
|---|---|
| Kurs progressi, XP, seriya, test natijalari | Hali brauzerda (`localStorage`) — qurilmalar orasida sinxronlanmaydi |
| Profilda ism/email o'zgartirish | Faqat lokal saqlanadi; backend hisobida sahifани qayta yuklasangiz eski qiymat qaytadi |
| Hisobni o'chirish | Faqat lokal mirror o'chadi; backenddagi hisob qoladi |
| Admin panel orqali rol o'zgartirish | Faqat lokal, backendga yozilmaydi |
| To'lov, sertifikat, savollar CRUD | O'zgarishsiz — hali lokal demo |

Bu — **atayin tor qamrov**: so'ralgan narsa aniq edi (backend +
ikkita demo hisob), shuning uchun avval shuni to'liq va sinovdan
o'tkazilgan holda qildim, keyingi bosqichlarni (progress, to'lov)
ustiga qo'shish uchun. To'liq reja `ROADMAP.md` da.

**Muhim:** agar backend ishlamayotgan bo'lsa yoki umuman ishga
tushirilmagan bo'lsa, sayt **avvalgidek to'liq ishlayveradi** — faqat
hisoblar shu brauzerga xos bo'lib qoladi (`DEPLOY.md` da tavsiflangan
holat). Bu ataylab shunday qilingan — statik sayt hech qachon
"backend talab qiladi" holatiga tushib qolmasligi kerak edi.

---

## 2. Texnologiya va nega shunday

**Faqat Python standart kutubxonasi** — `http.server`, `sqlite3`,
`hashlib`, `secrets`. **Hech qanday `pip install` kerak emas.**

Sabab: bu loyiha hech qanday tashqi bog'liqliksiz ishlashi kerak edi
(frontend ham xuddi shunday — bog'liqliksiz vanilla JS,
`ARCHITECTURE.md` 2-bo'limga qarang). Backend uchun ham bir xil
tamoyil: `python3 server.py` — va tamom, boshqa hech narsa o'rnatish
shart emas.

| Tanlov | Sabab |
|---|---|
| SQLite (Postgres emas) | Fayl-asosli, sozlash kerak emas, kichik-o'rta loyiha uchun yetarli. Keyinchalik Postgresga ko'chirish `ROADMAP.md` da rejalashtirilgan |
| PBKDF2 (bcrypt/argon2 emas) | Stdlib'da bor, tashqi C-kengaytma kerak emas. Katta miqyosda argon2id afzalroq (`ROADMAP.md` 4-bo'lim) |
| Opaque token (JWT emas) | Oddiyroq, server tomonida bekor qilinadi (JWT buni tabiiy qila olmaydi), imzo kaliti boshqarish kerak emas |
| `http.server` (Flask/FastAPI emas) | Hech narsa o'rnatilmasin degan tamoyil — stdlib yetarli bu hajm uchun |

---

## 3. Demo hisoblar

| Rol | Email | Parol |
|---|---|---|
| Administrator | `admin@gmail.com` | `12345678` |
| Oddiy foydalanuvchi | `user@gmail.com` | `12345678` |

Bu hisoblar server **birinchi marta** ishga tushganda avtomatik
yaratiladi (`seed_demo_accounts()` funksiyasi, `server.py` ichida).

**Muhim xususiyatlar:**
- Qayta ishga tushirish hisoblarni **qayta yaratmaydi yoki
  parolni tiklamaydi** — agar allaqachon mavjud bo'lsa, o'tkazib
  yuboriladi. Demak, parolni o'zgartirsangiz, keyingi restart uni
  qaytarib qo'ymaydi.
- To'liq tozalash uchun: **9-bo'lim** (bazani tiklash).
- Bu — **demo/ishlab chiqish uchun parol**. Real ishga tushirishda
  ikkalasini ham darhol o'zgartiring (10-bo'lim).

---

## 4. Ishga tushirish (lokal)

```bash
# 1. Avval saytni yig'ing (agar hali qilmagan bo'lsangiz)
python3 build.py

# 2. Backendni ishga tushiring
python3 backend/server.py
```

Windows'da:
```
python build.py
python backend\server.py
```

Konsolda shunga o'xshash chiqadi:

```
[seed] yaratildi: admin@gmail.com (rol: admin)
[seed] yaratildi: user@gmail.com (rol: student)
Tatbiqul Lisan Academy backend
  Sayt : http://localhost:8000
  API  : http://localhost:8000/api/health
  Baza : .../backend/data.db
  To'xtatish uchun: Ctrl+C
```

Brauzerda **http://localhost:8000** ni oching — bitta server ham
saytni, ham API'ni beradi (alohida "frontend server" kerak emas).

To'xtatish: `Ctrl+C`.

---

## 5. API — to'liq ma'lumotnoma

Barcha javoblar JSON. Xato holatida: `{"error": "kod", "message": "odam o'qiy oladigan matn"}`.

### `GET /api/health`
Server ishlab turganini tekshirish. Auth kerak emas.

```json
→ 200 {"status": "ok", "time": "2026-08-15T19:03:15+00:00"}
```

### `POST /api/auth/register`
Yangi hisob yaratadi. **Har doim `role: "student"`** — o'z-o'zidan
ro'yxatdan o'tish orqali admin bo'lish mumkin emas (xavfsizlik uchun).

```json
→ body:   {"name": "Ism Familiya", "email": "a@b.com", "password": "kamida8belgi"}
→ 201:    {"token": "...", "user": {"id": "...", "name": "...", "email": "...", "role": "student"}}
→ 400:    {"error": "invalid", "message": "Parol kamida 8 belgidan iborat bo'lishi kerak."}
→ 409:    {"error": "exists", "message": "Bu email allaqachon ro'yxatdan o'tgan."}
```

### `POST /api/auth/login`

```json
→ body:   {"email": "a@b.com", "password": "..."}
→ 200:    {"token": "...", "user": {...}}
→ 401:    {"error": "creds", "message": "Email yoki parol noto'g'ri."}
→ 429:    {"error": "rate_limited", "message": "..."}  (15 daqiqada 10 dan ortiq urinish)
```

### `POST /api/auth/logout`
Header: `Authorization: Bearer <token>`. Tokenni serverda bekor qiladi.
Har doim `200 {"ok": true}` qaytaradi (token noto'g'ri bo'lsa ham).

### `GET /api/auth/me`
Header: `Authorization: Bearer <token>`. Joriy foydalanuvchini qaytaradi.

```json
→ 200: {"user": {"id": "...", "name": "...", "email": "...", "role": "..."}}
→ 401: {"error": "unauthorized", "message": "..."}
```

**Sinash uchun:**
```bash
curl -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@gmail.com","password":"12345678"}'
```

---

## 6. Frontend qanday ulanadi

`src/js/05b-api.js` — `TLA.api` moduli. `site.config.json` dagi
`apiBase` maydoni orqali sozlanadi:

- **Bo'sh qoldirilsa** (standart) — frontend backend bilan **bir xil
  manzilda** ekan deb hisoblanadi (`http://localhost:8000/api/...`).
  Bu — yuqoridagi "bitta server hammasini beradi" sxemasi.
- **To'ldirilsa** (masalan `"https://api.tatbiqul-lisan.uz"`) —
  frontend va backend **alohida joyda** joylashtirilganda ishlatiladi
  (masalan frontend Netlify'da, backend VPS'da). Bu holda backend CORS
  sarlavhalarini avtomatik qo'shadi (`server.py` ichida `_cors()`).

`06-state.js` dagi `register()`/`login()` avval backendga murojaat
qiladi; agar backend **ulanib bo'lmasa** (server ishlamayapti),
avtomatik ravishda eski lokal-demo rejimiga o'tadi — foydalanuvchi
buni sezmaydi, sayt baribir ishlayveradi. Agar backend **ulanadi, lekin
rad etadi** (masalan noto'g'ri parol), bu holat **lokalga o'tkazilmaydi**
— aniq xato ko'rsatiladi. To'liq mantiq: `ARCHITECTURE.md` "Backend
integratsiyasi" bo'limida.

---

## 7. Deploy — backendni serverga chiqarish

Statik frontend (`DEPLOY.md`) Netlify/Vercel kabi joylarda bepul
ishlaydi, lekin ular **Python kodi ishga tushirmaydi**. Backend uchun
alohida joy kerak.

### 7.1 Eng oson: Render.com (bepul tarif bor)

1. GitHub reponi Render'ga ulang
2. **New → Web Service**
3. Sozlamalar:
   - Build command: (bo'sh qoldiring — hech narsa kerak emas)
   - Start command: `python3 build.py && python3 backend/server.py`
   - Environment: `PORT` — Render avtomatik beradi, kod uni o'qiydi
4. Deploy — bir necha daqiqada tayyor

⚠️ Bepul tarifda server 15 daqiqa harakatsizlikdan keyin "uxlaydi" va
keyingi so'rovda 30–60 soniya uyg'onadi. Demo uchun yetarli, doimiy
yuk uchun pullik tarifga o'ting.

### 7.2 O'z VPS'ingizda (systemd bilan doimiy ishlash)

```bash
# Serverga ulaning, loyihani yuklang
git clone https://github.com/FOYDALANUVCHI/tatbiqul-lisan.git
cd tatbiqul-lisan
python3 build.py

# systemd xizmati yarating
sudo tee /etc/systemd/system/tatbiqul-lisan.service <<'EOF'
[Unit]
Description=Tatbiqul Lisan Academy backend
After=network.target

[Service]
Type=simple
WorkingDirectory=/xil/yoli/tatbiqul-lisan
Environment=PORT=8000
ExecStart=/usr/bin/python3 backend/server.py
Restart=always
RestartSec=3
User=www-data

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl enable --now tatbiqul-lisan
sudo systemctl status tatbiqul-lisan
```

Keyin **nginx** orqali 80/443 portga ulang (HTTPS uchun Let's
Encrypt/certbot tavsiya etiladi):

```nginx
server {
    listen 80;
    server_name tatbiqul-lisan.uz;
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### 7.3 Railway / Fly.io

Ikkalasi ham GitHub repo ulash + `Start command: python3 backend/server.py`
bilan ishlaydi, Render'ga o'xshash. Fly.io doimiy ishlaydi (uxlamaydi),
kichik loyihalar uchun bepul kvotasi bor.

---

## 8. Xavfsizlik — nima bor, nima yo'q

### ✅ Bor

- Parollar PBKDF2-HMAC-SHA256 bilan xeshlangan (260 000 iteratsiya),
  har bir foydalanuvchi uchun alohida tuz (salt)
- Sessiya tokenlari kriptografik tasodifiy (`secrets.token_urlsafe`),
  serverda saqlanadi va bekor qilinishi mumkin
- Login uchun oddiy urinish cheklovi (bitta email+IP uchun 15
  daqiqada 10 ta)
- Path traversal himoyasi (statik fayl berishda) — qattiq sinovdan
  o'tkazilgan (URL-kodlangan va backslash variantlar bilan ham)
- SQL in'ektsiyasidan himoya (barcha so'rovlar parametrlashtirilgan)
- Parol xeshi/tuzi hech qachon API javobida qaytarilmaydi

### ⚠️ Real production uchun qo'shilishi kerak (hozircha yo'q)

| Nima | Nega hozircha yo'q |
|---|---|
| HTTPS | Server o'zi HTTP beradi — HTTPS reverse proxy (nginx+certbot) yoki hosting platformasi darajasida qo'shiladi (7-bo'lim) |
| Email tasdiqlash | Ro'yxatdan o'tishda email tekshirilmaydi — istalgan email kiritish mumkin |
| Parolni tiklash ("unutdim") | Hali yo'q — email yuborish infratuzilmasi kerak |
| CORS `*` | Demo uchun barcha originlarga ochiq (`server.py` `_cors()`). Real production'da faqat o'z domeningizga cheklang |
| Kuchli rate limiting | Hozirgi limiter — jarayon xotirasida, oddiy. Ko'p trafik uchun Redis-asosli yechim kerak |
| Audit log | Kim qachon kirgani/o'zgartirgani yozilmaydi |

Bu ro'yxat **yashirilmayapti** — chunki demo/kichik loyiha uchun
qabul qilinadigan tanlovlar bilan real, ko'p foydalanuvchili
production o'rtasidagi farqni bilish kerak.

---

## 9. Bazani boshqarish

### Ma'lumotlarni ko'rish

```bash
python3 -c "
import sqlite3
conn = sqlite3.connect('backend/data.db')
for row in conn.execute('SELECT name, email, role, created_at FROM users'):
    print(row)
"
```

### To'liq tiklash (barcha hisoblarni o'chirib, demo hisoblarni qayta yaratish)

```bash
TLA_RESET_DB=1 python3 backend/server.py
```

Windows (PowerShell):
```powershell
$env:TLA_RESET_DB="1"; python backend\server.py
```

Yoki oddiygina faylni o'chiring — server keyingi ishga tushishda uni
qayta yaratadi:

```bash
rm backend/data.db
python3 backend/server.py
```

### Zaxira nusxa

`backend/data.db` — bitta fayl. Ko'chirib qo'yish yoki nusxalash
yetarli:

```bash
cp backend/data.db backend/data.backup-$(date +%Y%m%d).db
```

---

## 10. Demo parollarni real ishga tushirishdan oldin o'zgartirish

Ikki yo'l bor:

**A) `server.py` dagi `DEMO_ACCOUNTS` ni tahrirlab, bazani tiklash:**

```python
DEMO_ACCOUNTS = [
    {"name": "Administrator", "email": "sizning-emailingiz@...", "password": "KUCHLI-PAROL", "role": "admin"},
]
```

Keyin `TLA_RESET_DB=1 python3 backend/server.py`.

**B) Admin panelda parolni o'zgartirish funksiyasi hozircha yo'q**
(bu — auth-only v1 ning cheklovi, 1-bo'limga qarang). Hozircha (A)
yo'li ishlatiladi.

⚠️ **Real foydalanuvchilar bilan ishga tushirishdan oldin albatta
demo parollarni o'zgartiring** — `admin@gmail.com` / `12345678`
hammaga ma'lum, hujjatlashtirilgan.

---

## 11. Muammolarni hal qilish

| Muammo | Yechim |
|---|---|
| `OGOHLANTIRISH: public/index.html topilmadi` | Avval `python3 build.py` ishga tushiring |
| `Address already in use` | Boshqa dastur shu portni band qilgan. `PORT=8001 python3 backend/server.py` bilan boshqa port tanlang |
| Login "Email yoki parol noto'g'ri" (demo hisob bilan ham) | Baza allaqachon boshqa parol bilan yaratilgan bo'lishi mumkin. `TLA_RESET_DB=1` bilan tiklang |
| Frontend backendга ulanmayapti (boshqa portda ochilgan) | `site.config.json` dagi `apiBase` ni tekshiring, `python3 build.py` qayta ishga tushiring |
| "Failed to fetch" konsolda | Backend ishlamayapti — bu **kutilgan holat**, sayt lokal rejimga o'tadi. Backendni tekshirish uchun: `curl http://localhost:8000/api/health` |
| CORS xatosi (boshqa domendan) | `apiBase` to'g'ri sozlanganini va backend ishlab turganini tekshiring — server CORS sarlavhalarini avtomatik qo'shadi |

---

## 12. Keyingi bosqich

Bu backend — birinchi qadam. Kengaytirish tartibi (`ROADMAP.md` bilan
mos):

1. Progress/XP/test natijalarini backendga ko'chirish (yangi jadval +
   endpointlar)
2. Admin panelni backend bilan bog'lash (foydalanuvchilar ro'yxati,
   rol o'zgartirish — haqiqiy)
3. Parolni tiklash (email infratuzilmasi kerak)
4. PostgreSQL'ga ko'chirish (ko'p foydalanuvchi kutilsa)
5. To'lov, sertifikat, kontent CRUD

Har biri — alohida, kichik, to'liq sinovdan o'tkaziladigan qadam,
xuddi shu auth bosqichi kabi.
