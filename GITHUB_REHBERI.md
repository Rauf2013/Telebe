# 🚀 GitHub'a Yükleme Rehberi (Sıfırdan)

Hiç GitHub kullanmamış birine adım adım anlatım. Birebir takip et.

---

## ✅ Şu ana kadar yapılanlar (zaten hazır)

- ✓ Git kurulu
- ✓ `git init` yapıldı (repo başlatıldı)
- ✓ `.gitignore` ayarlandı (`.env`, `node_modules`, db dosyaları git'e GİTMEZ)
- ✓ `git config user.name "Rauf"` ve email ayarlandı
- ✓ İlk commit yapıldı: "Initial commit"

---

## 📋 1) GitHub hesabı oluştur (yoksa)

1. https://github.com/signup adresine git
2. Email, şifre, kullanıcı adı seç (örn: `qaracayevrauf` veya istediğin)
3. Doğrulama yap, hesabı aktive et

---

## 📦 2) GitHub'da yeni repo oluştur

1. https://github.com/new adresine git
2. Formu doldur:
   - **Repository name:** `Telebe`
   - **Description:** (opsiyonel) `Tələbə qəbulu platforması — EduGate`
   - **Public/Private:** İstediğini seç. *Public* = herkes görür, *Private* = sadece sen + davet ettiğin kişiler
   - ⚠️ **HİÇBİR kutuyu işaretleme!** README, .gitignore, license — hepsini boş bırak
3. **"Create repository"** bas

Sonraki sayfada **"...or push an existing repository from the command line"** bölümünü göreceksin. Sana 2-3 komut verecek — onları kullanmak yerine **aşağıdakileri** kullan (zaten hazır).

---

## 🔑 3) GitHub Personal Access Token al (şifre yerine)

GitHub artık şifre ile push'a izin vermiyor. **Personal Access Token (PAT)** lazım.

1. https://github.com/settings/tokens/new
2. **Note:** `EduGate push`
3. **Expiration:** 90 days (veya istediğin)
4. **Scopes:** sadece `repo` kutusunu işaretle (tüm alt kutular otomatik seçilir)
5. En altta **"Generate token"**
6. ⚠️ Çıkan token'ı **HEMEN kopyala** (örn: `ghp_xxxxxxxxxxxxxxxx`) — sayfayı kapatınca bir daha göstermez!

Bu token **şifreniz gibi** — kimseyle paylaşma.

---

## 🚀 4) Projeyi GitHub'a gönder

Terminal aç (Education klasöründe), bu komutları **sırayla** yaz:

### 4.1 Remote (GitHub linki) ekle
```bash
git remote add origin https://github.com/SENIN-KULLANICI-ADIN/Telebe.git
```
> `SENIN-KULLANICI-ADIN` yerine kendi GitHub kullanıcı adını yaz.
> Örneğin: `https://github.com/qaracayevrauf/Telebe.git`

### 4.2 Kontrol
```bash
git remote -v
```
Görmen gereken:
```
origin  https://github.com/SENIN-KULLANICI-ADIN/Telebe.git (fetch)
origin  https://github.com/SENIN-KULLANICI-ADIN/Telebe.git (push)
```

### 4.3 İlk push
```bash
git push -u origin main
```

**Sorulacak:**
- Username: `SENIN-KULLANICI-ADIN`
- Password: **3. adımdaki Personal Access Token** (`ghp_...`) — Gmail şifresi DEĞİL

Başarılı olursa `https://github.com/.../Telebe` sayfasına git, dosyaları görürsün. 🎉

---

## 👥 5) Birini repo'ya davet etmek (özel proje ise)

**Private** repo açtıysan:

1. Repo sayfasında **Settings** sekmesi
2. Sol menüde **Collaborators**
3. **"Add people"** butonu
4. Davet etmek istediğin kişinin GitHub kullanıcı adı veya email'ini yaz
5. Davet maili gider, kabul edince repo'ya erişebilir

**Public** repo ise zaten herkes görür, üye eklemen gerekmez (sadece yazma izni için eklersin).

---

## 🔁 6) Sonradan değişiklik gönderme (günlük kullanım)

Kodda her değişiklik yaptığında:

```bash
git add .                         # değişiklikleri ekle
git commit -m "Açıklayıcı mesaj"  # commit yap
git push                          # GitHub'a gönder
```

**Örnek mesajlar:**
- `"Add card payment form"`
- `"Fix forgot password bug"`
- `"Update university list"`
- `"Translate faculty names to Turkish"`

---

## ⚠️ Önemli güvenlik notları

- ❌ **ASLA** `.env` dosyasını commit etme — Gmail şifren oradadır!
- ✓ `.gitignore` zaten engelliyor, kontrol için: `git status` çalıştır, `.env` görünmemeli
- ❌ Token'ı kodda yazma
- ✓ Token kaybolursa yenisini al, eskini revoke et

---

## 🆘 Sorun çıkarsa

**"remote already exists"** → `git remote remove origin` sonra tekrar add

**"authentication failed"** → Token doğru mu? Username GitHub username mu?

**"refusing to merge unrelated histories"** → `git pull origin main --allow-unrelated-histories` sonra tekrar push

**Push işe yaramazsa** → ekran görüntüsü gönder, beraber çözeriz.

---

## 🎓 Mini Git Sözlüğü

| Komut | Ne yapar |
|---|---|
| `git status` | Hangi dosyalar değişti |
| `git add .` | Tüm değişiklikleri "sahneye" al |
| `git commit -m "mesaj"` | Sahneyi kalıcı kayıt yap |
| `git push` | Kayıtları GitHub'a gönder |
| `git pull` | GitHub'daki son halini indir |
| `git log --oneline` | Geçmiş commit'leri gör |
| `git remote -v` | Hangi GitHub repo'suna bağlı |
