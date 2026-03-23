# 15. Genel Kontrol Listesi

[← Ana Sayfa](../README.md)

Aşağıdaki kontroller hem ilk kurulum sonrasında hem de periyodik olarak (haftalık/aylık) uygulanmalıdır.

## Sistem Sağlığı

```bash
# Başarısız servis var mı?
systemctl list-units --state=failed

# Disk kullanımı
df -h

# Bellek kullanımı
free -h

# Sistem yükü
uptime

# En çok kaynak tüketen prosesler
htop
# veya
ps aux --sort=-%cpu | head -10
ps aux --sort=-%mem | head -10
```

## Ağ ve Güvenlik Duvarı

```bash
# Dinleyen portlar
ss -tlnp

# UFW durumu
ufw status verbose

# Aktif bağlantılar
ss -tnp state established

# Dışarıdan port taraması (kendi sunucuna)
nmap -sV sunucu.example.com
```

## Erişim ve Kimlik Doğrulama

```bash
# Son başarısız SSH girişleri
journalctl -u sshd | grep "Failed\|Invalid" | tail -20

# Son başarılı girişler
last | head -20

# Şu an kim bağlı
who

# Fail2ban durumu
fail2ban-client status

# Banlı IP sayısı (jail bazında)
for jail in $(fail2ban-client status | grep "Jail list" | sed 's/.*://;s/,//g'); do
    echo "$jail: $(fail2ban-client status $jail | grep 'Currently banned' | awk '{print $NF}')"
done
```

## Güncellemeler

```bash
# Güncellenebilir paket var mı?
apt list --upgradable 2>/dev/null

# Son unattended-upgrades çalışması
tail -20 /var/log/unattended-upgrades/unattended-upgrades.log
```

## SSL Sertifikaları

```bash
# Tüm sertifika son kullanma tarihleri (certbot)
certbot certificates

# Manuel kontrol
for domain in example.com app.example.com; do
    echo -n "$domain: "
    echo | openssl s_client -connect $domain:443 2>/dev/null \
      | openssl x509 -noout -enddate 2>/dev/null
done
```

## Auditd

```bash
# Bugünün özet raporu
aureport --summary --start today

# Başarısız olaylar
aureport --failed --start today

# Privilege escalation denemeleri
ausearch -k privilege_escalation -ts today
```

## Log Boyutları

```bash
# Log dizinlerinin boyutu
du -sh /var/log/*/  2>/dev/null | sort -h

# journald disk kullanımı
journalctl --disk-usage
```

---

## Kurulum Sonrası Hızlı Kontrol

Yeni bir sunucu kurulumunun ardından şu adımları sırayla uygula:

| Adım | Kontrol | Bölüm |
|------|---------|-------|
| 1 | Logları incele, crash loop var mı? | [01](01-system-logs.md) |
| 2 | Gereksiz servisleri kapat | [02](02-disable-services.md) |
| 3 | UFW kurallarını yapılandır | [03](03-ufw.md) |
| 4 | SSH anahtar tabanlı girişe geç, root girişini kapat | [05](05-ssh.md) |
| 5 | IPv6 kullanılıyor mu karar ver | [06](06-ipv6.md) |
| 6 | fail2ban'ı kur ve yapılandır | [11](11-fail2ban.md) |
| 7 | unattended-upgrades'i etkinleştir | [13](13-unattended-upgrades.md) |
| 8 | auditd'yi kur | [12](12-auditd.md) |
| 9 | Faydalı araçları kur | [14](14-tools.md) |
| 10 | Dışarıdan port taraması yap | [14](14-tools.md) |
