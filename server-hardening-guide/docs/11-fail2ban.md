# 11. Fail2Ban

[← Ana Sayfa](../README.md)

Fail2Ban, log dosyalarını izleyerek belirli sayıda başarısız girişimden sonra kaynak IP'yi geçici olarak engeller. `iptables` veya `nftables` üzerinden çalışır.

## Kurulum

```bash
apt install fail2ban
systemctl enable fail2ban
systemctl start fail2ban
```

## Yapılandırma

`/etc/fail2ban/jail.conf` dosyası doğrudan düzenlenmez; bunun yerine `/etc/fail2ban/jail.local` oluşturulur. Güncelleme sonrası orijinal dosyanın üzerine yazılsa bile özel ayarlar korunur.

```bash
cat > /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
# Ban süresi (1h, 1d, -1 = kalıcı)
bantime = 1h
# Kaç dakika içinde maxretry kadar deneme yapılırsa ban uygulanır
findtime = 10m
# Maksimum başarısız deneme sayısı
maxretry = 5
# Tekrarlayan ihlallerde ban süresini artır (1h → 2h → 4h ...)
bantime.increment = true

[sshd]
enabled = true
port = 22
maxretry = 3

[apache-auth]
enabled = true
port = http,https
maxretry = 5

[apache-badbots]
enabled = true
port = http,https

[apache-noscript]
enabled = true
port = http,https

[apache-overflows]
enabled = true
port = http,https

[named-refused]
enabled = true
port = 53
protocol = udp
logpath = /var/log/named/queries.log
EOF
```

```bash
systemctl restart fail2ban
fail2ban-client status
```

## Yönetim Komutları

```bash
# Aktif jail'leri listele
fail2ban-client status

# Belirli bir jail'in durumu
fail2ban-client status sshd

# Banlı IP'leri listele
fail2ban-client status sshd | grep "Banned IP"

# IP ban kaldır
fail2ban-client set sshd unbanip 203.0.113.99

# Kendi IP'ni test için ban et / kaldır
fail2ban-client set sshd banip 203.0.113.99
fail2ban-client set sshd unbanip 203.0.113.99
```

## Kendi IP'ni Whitelist'e Alma

Yanlışlıkla kendi IP'nin banlanmasını önlemek için `jail.local`'a ekle:

```ini
[DEFAULT]
ignoreip = 127.0.0.1/8 ::1 10.8.0.0/24 203.0.113.10
```

## Log Takibi

```bash
tail -f /var/log/fail2ban.log
grep "Ban\|Unban" /var/log/fail2ban.log | tail -20
```

---

*Sonraki bölüm: [Auditd](12-auditd.md)*
