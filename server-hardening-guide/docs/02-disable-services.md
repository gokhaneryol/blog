# 2. Gereksiz Servislerin Devre Dışı Bırakılması

[← Ana Sayfa](../README.md)

Kullanılmayan servisler hem kaynak tüketir hem de saldırı yüzeyini genişletir. Her açık servis potansiyel bir giriş noktasıdır.

## Temel Komutlar

```bash
# Servisi durdur ve devre dışı bırak
systemctl stop <servis-adı>
systemctl disable <servis-adı>

# Çalışan tüm servisleri listele
systemctl list-units --type=service --state=running

# Aktif olan ancak kullanılmayan servisleri bul
systemctl list-units --type=service --state=active
```

## Yaygın Örnekler

**Postfix / Mail servisleri** — Sunucu uygulama maili göndermiyorsa:
```bash
systemctl stop postfix
systemctl disable postfix

# SPF policy daemon da varsa
systemctl stop postfix-policyd-spf.service
systemctl disable postfix-policyd-spf.service
```

> **Not:** Mail gönderimi yapan uygulamalar varsa (cron bildirimleri, uygulama mailleri vb.) Postfix'i kapatmadan önce bağımlılıkları kontrol edin:
> ```bash
> grep -r "sendmail\|/usr/sbin/mail\|smtp" /etc/cron* /var/spool/cron/ 2>/dev/null
> ```

**Avahi / mDNS** — Yerel ağ servisi keşfi, sunucularda genellikle gerekmez:
```bash
systemctl stop avahi-daemon
systemctl disable avahi-daemon
```

**Bluetooth** — Fiziksel sunucularda:
```bash
systemctl stop bluetooth
systemctl disable bluetooth
```

## Servis Bağımlılığı Kontrolü

Bir servisi kapatmadan önce başka servislerin ona bağlı olup olmadığını kontrol et:

```bash
systemctl list-dependencies --reverse <servis-adı>
```

---

*Sonraki bölüm: [UFW Güvenlik Duvarı](03-ufw.md)*
