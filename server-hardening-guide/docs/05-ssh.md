# 5. SSH Güvenliği

[← Ana Sayfa](../README.md)

SSH, sunucuya uzaktan erişimin temel kapısıdır. Varsayılan yapılandırma birçok güvenlik açığı barındırır; aşağıdaki adımlar bu riskleri önemli ölçüde azaltır.

## Adım 1: Anahtar Çifti Oluşturma

**İstemci makinede** çalıştır (sunucuda değil):

```bash
# Ed25519 algoritması önerilir — RSA'ya göre daha kısa ve en az aynı derecede güvenli
ssh-keygen -t ed25519 -C "kullanici@sunucu-adi"

# İstenirse RSA (eski sistemler için)
ssh-keygen -t rsa -b 4096 -C "kullanici@sunucu-adi"

# Oluşturulan dosyalar:
# ~/.ssh/id_ed25519        → özel anahtar (kesinlikle paylaşılmaz!)
# ~/.ssh/id_ed25519.pub    → açık anahtar (sunucuya kopyalanacak)
```

> **Passphrase:** Anahtar oluşturulurken passphrase girilmesi önerilir. Anahtar dosyası ele geçirilse bile passphrase olmadan kullanılamaz.

## Adım 2: Açık Anahtarı Sunucuya Kopyalama

```bash
# Otomatik yöntem (önerilir)
ssh-copy-id -i ~/.ssh/id_ed25519.pub kullanici@sunucu-ip

# Manuel yöntem (ssh-copy-id yoksa)
cat ~/.ssh/id_ed25519.pub | ssh kullanici@sunucu-ip \
  "mkdir -p ~/.ssh && chmod 700 ~/.ssh && \
   cat >> ~/.ssh/authorized_keys && \
   chmod 600 ~/.ssh/authorized_keys"
```

## Adım 3: Anahtar ile Bağlantı Testi

> **Kritik:** Parolalı girişi kapatmadan önce bu adımı mutlaka tamamla.

```bash
ssh -i ~/.ssh/id_ed25519 kullanici@sunucu-ip

# Verbose mod ile test (sorun varsa detay görmek için)
ssh -v -i ~/.ssh/id_ed25519 kullanici@sunucu-ip
```

## Adım 4: sshd_config Sıkılaştırma

```bash
# Yedek al
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
```

`/etc/ssh/sshd_config` dosyasında şu değerleri ayarla:

```ini
# Root ile doğrudan giriş yasak
PermitRootLogin no

# Parola ile giriş yasak — sadece anahtar
PasswordAuthentication no
ChallengeResponseAuthentication no
KbdInteractiveAuthentication no

# Boş parola kesinlikle yasak
PermitEmptyPasswords no

# Sadece belirli kullanıcılara izin ver (opsiyonel ama önerilir)
AllowUsers kullanici1 kullanici2

# Kimlik doğrulama için süre sınırı (saniye)
LoginGraceTime 30

# Maksimum başarısız deneme
MaxAuthTries 3

# X11 forwarding — gerekli değilse kapat
X11Forwarding no

# Bağlantı canlılık kontrolü
ClientAliveInterval 300
ClientAliveCountMax 2

# Sadece IPv4 (IPv6 kullanılmıyorsa)
AddressFamily inet
```

```bash
# Config sözdizimi kontrolü
sshd -t

# Reload (mevcut oturumlar kesilmez)
systemctl reload sshd
```

## İstemci Tarafında ~/.ssh/config

Sık bağlanılan sunucular için istemcide konfigürasyon tanımlamak kolaylık sağlar:

```ini
Host sunucu-takma-ad
    HostName sunucu.example.com
    User kullanici
    IdentityFile ~/.ssh/id_ed25519
    Port 22

# Birden fazla sunucu
Host staging
    HostName staging.example.com
    User deploy
    IdentityFile ~/.ssh/id_ed25519_staging
```

Artık `ssh sunucu-takma-ad` ile bağlanılabilir.

## Dosya İzinleri

SSH, anahtar dosyalarının izinleri konusunda katıdır:

```bash
# İstemci tarafında
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
chmod 600 ~/.ssh/config

# Sunucu tarafında
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

## Kontrol

```bash
# Başarısız giriş denemeleri
journalctl -u sshd | grep "Failed\|Invalid" | tail -20

# Aktif SSH oturumları
who
ss -tnp | grep :22
```

---

*Sonraki bölüm: [IPv6 Yönetimi](06-ipv6.md)*
