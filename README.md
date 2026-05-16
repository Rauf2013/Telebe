# EduGate — Tələbə qəbulu platforması

3 rollü (Moderator / Universitet təmsilçisi / Tələbə) çoxdilli (Az/Tr/Kk/Uz/Tg/Tm) student-admission platforması.

## Struktur

```
Education/
├── frontend/   # React + Vite + TypeScript + Tailwind + i18next
└── backend/    # Express + JWT + SQLite (better-sqlite3) + multer
```

## Çalıştırma

İki ayrı terminalde:

**Backend (port 4000):**
```bash
cd backend
npm install
npm run dev      # geliştirme — kod değişince oto-restart
# veya
npm start        # production — node server.js
```

**Frontend (port 5173):**
```bash
cd frontend
npm install
npm run dev
```

Tarayıcı: http://localhost:5173

## npm scriptleri nedir?

`backend/package.json` içinde:
```json
"scripts": {
  "start": "node server.js",         ← npm start ile çalışır
  "dev":   "node --watch server.js"  ← npm run dev ile çalışır (oto-restart)
}
```
`npm start` = `node server.js`. İkisi de aynı şeyi yapar. `dev` ekstra olarak dosya değişikliklerini izler.

## Database — SQLite

**Şu an SQLite kullanıyoruz.** Kurulum yok — sadece `npm install` ile gelir.

Dosya: `backend/data/edugate.db` (tek bir dosya, tüm veritabanı orada).

### Tablolar

```sql
CREATE TABLE users (
  id            TEXT PRIMARY KEY,
  email         TEXT UNIQUE NOT NULL,
  password      TEXT NOT NULL,           -- bcrypt hash
  full_name     TEXT NOT NULL,
  role          TEXT NOT NULL,           -- student | university | moderator
  phone         TEXT,
  whatsapp      TEXT,
  university_id TEXT,                    -- sadece üniversite rolü için
  created_at    TEXT NOT NULL
);

CREATE TABLE applications (
  id                   TEXT PRIMARY KEY,
  student_id           TEXT UNIQUE NOT NULL REFERENCES users(id),
  status               TEXT NOT NULL DEFAULT 'draft',
  choices              TEXT NOT NULL DEFAULT '[]',  -- JSON
  documents            TEXT NOT NULL DEFAULT '[]',  -- JSON
  first_payment_paid   INTEGER NOT NULL DEFAULT 0,
  second_payment_paid  INTEGER NOT NULL DEFAULT 0,
  created_at           TEXT NOT NULL
);
```

### DB'yi incelemek

İki yol var:

**1. DB Browser for SQLite** (GUI, en kolay)
- İndir: https://sqlitebrowser.org/
- `backend/data/edugate.db` aç → tüm tabloları görürsün

**2. Komut satırı**
```bash
sqlite3 backend/data/edugate.db
> .tables
> SELECT * FROM users;
> SELECT * FROM applications;
> .quit
```

### DB'yi sıfırlamak

```bash
rm backend/data/edugate.db*    # macOS/Linux/Git Bash
del backend\data\edugate.db*   # Windows CMD
```
Sonraki `npm start` ile boş DB otomatik oluşturulur.

## Production'a hazırlamak

### 1) JWT secret değiştir
`.env` dosyası oluştur (`backend/.env`):
```
JWT_SECRET=cok-uzun-rastgele-string-buraya
PORT=4000
```
Sonra `server.js` zaten `process.env.JWT_SECRET` okuyor.

### 2) Database — gerçek production için

**Seçenek A: SQLite (yine kalsın)**
- Küçük/orta sitelerde sorunsuz (binlerce kullanıcı)
- Sunucuda `backend/data/edugate.db` dosyasını backup'la
- VPS yeterli, ek bir DB sunucusu kurmana gerek yok

**Seçenek B: PostgreSQL'e geçiş**
- Eğer yüksek trafik / çoklu sunucu / managed hosting (Heroku, Railway, Supabase) istiyorsan
- Adımlar:
  1. `npm install pg`
  2. `db.js` içinde `better-sqlite3` yerine `pg` kullan — sorgular neredeyse aynı (PostgreSQL SQL syntax'ı destekliyor)
  3. Env: `DATABASE_URL=postgres://user:pass@host/db`
  4. `data` kolonlarını `JSONB` yap (SQLite TEXT yerine)
- ⚠️ Şu anki kod 90% taşınabilir, sadece `db.js` dosyasını değiştirmen yeter

### 3) Dosya yüklemeleri
- Şu an: `backend/uploads/` klasörü (lokal disk)
- Production: **S3 / DigitalOcean Spaces** kullan — kod değişikliği gerekir (multer-s3)

### 4) HTTPS + Domain
- Reverse proxy: **nginx** veya **Caddy**
- SSL: **Let's Encrypt** (ücretsiz)
- Process manager: **pm2** (`pm2 start server.js`)

### 5) WhatsApp gerçek bildirim
- Şu an: `wa.me/...` linki açar (manuel)
- Production: **WhatsApp Business API** veya **Twilio** ile otomatik mesaj

## Admin (Moderator) hesabı

Güvenlik için **moderator rolü kayıt formundan kaldırıldı**. Backend ilk açılışta otomatik bir moderator yaratıyor:

**Varsayılan:**
- Email: `admin@edugate.local`
- Şifrə: `admin123`

## Yeni moderator nasıl eklenir?

Sadece **mevcut moderatorlar** yeni moderator ekleyebilir — **davet linki** sistemi ile:

1. Mevcut moderator → kabinet → **"Moderatorlar"** sekmesi → sağdaki form
2. (Opsiyonel) Kişiyi tanımlayan kısa not yaz (örn. "Yeni HR meneceri")
3. **"✦ Davet linki yarat"** butonu → benzersiz URL oluşur (örn. `http://localhost:5173/invite/abc123...`)
4. **"Kopyala"** butonu ile linki al, WhatsApp/email/Telegram'la güvendiğin kişiye gönder
5. Karşı taraf linke tıklar → "Moderator daveti — EduGate komandasına xoş gəldiniz" sayfası
6. Ad-soyad, email, telefon ve **kendi şifresini** belirler
7. Form gönderildiğinde otomatik moderator hesabı oluşturulur ve otomatik giriş yapılır

**Güvenlik özellikleri:**
- Davet linki **7 gün** geçerli
- **Tek kullanımlık** — kullanıldıktan sonra otomatik geçersiz olur
- Admin panelinden istenen anda **iptal** edilebilir
- Kayıt formunda moderator seçeneği yok (frontend) + backend de moderator rolüyle public register'ı reddediyor
- Sadece moderator rolündeki kullanıcılar `/api/admin/invites` endpoint'ini çağırabilir

Üretimde mutlaka değiştir. `.env` dosyası (backend klasöründe):
```
ADMIN_EMAIL=senin@email.com
ADMIN_PASSWORD=guvenli-sifren
ADMIN_NAME=Senin Adın
JWT_SECRET=cok-uzun-rastgele-string

# SMTP — production için
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=senin@gmail.com
SMTP_PASS=uygulama-sifren
MAIL_FROM="EduGate <noreply@yourdomain.com>"
FRONTEND_URL=https://senin-domain.com
```

> `.env` sadece **DB boş** iken (ilk başlangıçta) admin değerlerini okur. Sonradan değiştirmek için ya DB'yi sil ya da admin paneli üzerinden yeni moderator ekle.

## 📧 Gmail SMTP — Gerçek mail göndərmək

Sistem **şifrə bərpa kodları** üçün email göndərir. Gerçek mail göndərmək (kod **direkt Gmail'ə gəlsin**) üçün:

### 1. Backend `.env` dosyası yaradın

`backend/.env.example` faylını kopyalayıb `backend/.env` adı verin:
```bash
cd backend
cp .env.example .env   # Linux/macOS/Git Bash
# və ya Windows: copy .env.example .env
```

### 2. Gmail App Password yaradın

⚠️ Adi Gmail şifrəsi **işləməz**. Gmail SMTP üçün **App Password** lazımdır:

1. Gmail hesabınızda **2-Step Verification** açın:
   👉 https://myaccount.google.com/security

2. **App Password** yaradın:
   👉 https://myaccount.google.com/apppasswords
   - "Select app" → **Mail**
   - "Select device" → **Other** → "EduGate" yazın
   - **"Generate"** basın
   - 16 karakterli kod alacaqsınız (məsələn: `abcd efgh ijkl mnop`)

### 3. `.env` dosyasını doldurun

```env
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_SECURE=false
SMTP_USER=sizin.email@gmail.com
SMTP_PASS=abcd efgh ijkl mnop
MAIL_FROM="EduGate <sizin.email@gmail.com>"
```

### 4. Backendi yenidən başladın

```bash
cd backend
npm run dev
```

Konsolda göstərəcək: `✉  Mailer: smtp.gmail.com`

✅ Artıq "Şifrəni unutmusunuz?" formundan kod istəyəndə **gerçek mail** istənilən email ünvanına gələcək.

> **Qeyd:** `.env` dosyası boş qalsa, sistem avtomatik olaraq Ethereal (test SMTP) işlədir — bu vəziyyətdə mail real göndərilmir, sadəcə preview link konsolda yazılır.

## Başvuru zaman tüneli (Yeni özellik)

Her başvurunun **tüm tarihçesi** kabinet üzerinde görüntülenir — kim, ne zaman, ne yaptı.

Olay türleri:
- ✦ Müraciət yaradıldı / fakultə seçimi yeniləndi
- 📄 Sənəd yüklədi
- 💳 İlk/ikinci ödəniş tamamlandı
- 🌐 Tərcümə yükləndi
- 📨 Universitetə göndərildi
- ✓ Qəbul təsdiqləndi / ✗ Geri qaytarıldı

3 rolün de detail panelinde otomatik görünür (rol bazlı erişim kontrolü: öğrenci sadece kendi başvurusunu, üniversite sadece kendi üniversitesine gelen başvuruları görür).

## Test akışı

1. `/register`'da 2 hesap aç:
   - **Tələbə** — whatsapp girilmeli
   - **Universitet təmsilçisi** — bir universitet seç
2. **Moderator** hesabı: `admin@edugate.local` / `admin123` ilə daxil ol
3. Tələbə girişi → 5 fakultə seç → sənəd yüklə → ödəniş ($150)
3. Moderator girişi → bu başvuruyu gör → tərcüməni yüklə → "Universitetə göndər"
4. Universitet temsilcisi girişi → kendi başvurusunu gör → təsdiqlə + təhsil haqqı gir
5. Moderator → onayı görür → "WhatsApp bildiriş" (wa.me açılır)
6. Tələbə → izləmədə təsdiq + təhsil haqqı görünür → ikinci ödəniş ($350)

## API endpoints

| Method | Path | Auth |
|--------|------|------|
| POST   | `/api/auth/register`                                       | public |
| POST   | `/api/auth/login`                                          | public |
| GET    | `/api/auth/me`                                             | any |
| GET    | `/api/users/:id`                                           | any |
| GET    | `/api/users`                                               | mod / uni |
| GET    | `/api/applications/mine`                                   | student |
| POST   | `/api/applications/choices`                                | student |
| POST   | `/api/applications/documents` (multipart `file` + `type`)  | student |
| DELETE | `/api/applications/documents/:docId`                       | student |
| POST   | `/api/applications/payment/first`                          | student |
| POST   | `/api/applications/payment/second`                         | student |
| GET    | `/api/applications`                                        | moderator |
| POST   | `/api/applications/:id/documents/:docId/translation`       | moderator |
| POST   | `/api/applications/:id/choices/:facultyId/status`          | mod / uni |
| GET    | `/api/stats`                                               | public |
| GET    | `/api/health`                                              | public |
