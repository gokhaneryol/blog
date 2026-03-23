# Linux Sunucu Kurulum Sonrası Güvenlik ve Servis Yapılandırma Rehberi

Bu rehber, Ubuntu tabanlı bir Linux sunucusunun kurulum sonrasında güvenli ve sağlıklı şekilde çalışabilmesi için uygulanması gereken yapılandırmaları kapsamaktadır. Adımlar gerçek dünya deneyiminden elde edilmiş olup bölüm bölüm açıklanmıştır.

---

## İçindekiler

1. [Sistem Loglarının Denetimi](#1-sistem-loglarının-denetimi)
2. [Gereksiz Servislerin Devre Dışı Bırakılması](#2-gereksiz-servislerin-devre-dışı-bırakılması)
3. [UFW Güvenlik Duvarı](#3-ufw-güvenlik-duvarı)
4. [Log Yönetimi](#4-log-yönetimi)
5. [BIND9 DNS Sunucusu](#5-bind9-dns-sunucusu)
6. [OpenVPN Yapılandırması](#6-openvpn-yapılandırması)
7. [Docker ve Konteyner Servisleri](#7-docker-ve-konteyner-servisleri)
8. [Apache Reverse Proxy](#8-apache-reverse-proxy)
9. [Fail2Ban](#9-fail2ban)
10. [Unattended Upgrades](#10-unattended-upgrades)

---

## 1. Sistem Loglarının Denetimi

Kurulum sonrasında ilk yapılacak iş sistem loglarını incelemektir. Crash loop, servis hataları ve beklenmedik ağ bağlantıları bu aşamada tespit edilebilir.

```bash
# Canlı log takibi
tail -f /var/log/syslog

# Başarısız servisleri listele
systemctl list-units --state=failed

# Belirli bir servisin logları
journalctl -u <servis-adı> -n 50
```

### Dikkat Edilecekler

- Crash loop içindeki servisler (`restart counter` değeri yüksek olanlar)
- AppArmor veya SELinux tarafından engellenen işlemler
- UFW tarafından bloklanan beklenmedik bağlantılar

---

## 2. Gereksiz Servislerin Devre Dışı Bırakılması

Kullanılmayan servisler hem kaynak tüketir hem de saldırı yüzeyini genişletir.

```bash
# Servisi durdur ve devre dışı bırak
systemctl stop <servis-adı>
systemctl disable <servis-adı>

# Örnek: Postfix SPF policy daemon kullanılmıyorsa
systemctl stop postfix-policyd-spf.service
systemctl disable postfix-policyd-spf.service

# Postfix'i de kapatmak için
systemctl stop postfix
systemctl disable postfix
```

> **Not:** Mail gönderimi yapan uygulamalar varsa (cron bildirimleri, uygulama mailleri vb.) Postfix'i kapatmadan önce bağımlılıkları kontrol edin.

---

## 3. UFW Güvenlik Duvarı

### Temel Kurallar

```bash
# Mevcut durumu gör
ufw status

# Temel kurallar
ufw allow 22/tcp     # SSH
ufw allow 80/tcp     # HTTP
ufw allow 443/tcp    # HTTPS
ufw allow 1194/udp   # OpenVPN (kullanılıyorsa)

# DNS sunucusu çalıştırılıyorsa
ufw allow 53/tcp
ufw allow 53/udp

ufw enable
```

### Önemli Notlar

- UFW varsayılan olarak gelen bağlantıları reddeder, giden bağlantılara izin verir.
- Kural ekledikten sonra `ufw reload` ile yenile.
- IPv6 kuralları da otomatik eklenmez; `ufw status` çıktısında `(v6)` satırlarını kontrol edin.

---

## 4. Log Yönetimi

### UFW Loglarını Ayırmak

`syslog`'u temiz tutmak için UFW loglarını ayrı bir dosyaya yönlendir.

```bash
# rsyslog kuralı
cat > /etc/rsyslog.d/20-ufw.conf << 'EOF'
:msg,contains,"[UFW " /var/log/ufw/ufw.log
& stop
EOF

mkdir -p /var/log/ufw
chown syslog:adm /var/log/ufw
systemctl restart rsyslog
```

### Logrotate

```bash
cat > /etc/logrotate.d/ufw << 'EOF'
/var/log/ufw/ufw.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    postrotate
        invoke-rc.d rsyslog rotate > /dev/null 2>&1 || true
    endscript
}
EOF
```

---

## 5. BIND9 DNS Sunucusu

### Temel Güvenlik Yapılandırması

`/etc/bind/named.conf.options` dosyasına aşağıdaki yapılandırmayı uygulayın:

```bind
// ACL tanımı options bloğundan ÖNCE olmalı
acl "izin-verilen-adresler" {
    10.0.0.0/24;           // İç ağ
    203.0.113.10/32;       // Yetkili harici IP
    localhost;
};

options {
    directory "/var/cache/bind";

    dnssec-validation auto;

    // IPv6 dinlemeyi kapat (gerekli değilse)
    listen-on-v6 { none; };

    recursion yes;
    allow-query { any; };
    allow-recursion { izin-verilen-adresler; };
    allow-query-cache { izin-verilen-adresler; };

    // Query logging aktif
    querylog yes;
};

logging {
    channel query_log {
        file "/var/log/named/queries.log" versions 7 size 50m;
        severity info;
        print-time yes;
        print-severity yes;
        print-category yes;
    };

    category queries { query_log; };
    category resolver { query_log; };
};
```

### Log Dizini ve İzinler

```bash
mkdir -p /var/log/named
chown bind:bind /var/log/named
```

### Logrotate

```bash
cat > /etc/logrotate.d/named << 'EOF'
/var/log/named/queries.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    postrotate
        systemctl reload named > /dev/null 2>&1 || true
    endscript
}
EOF
```

### Zone Dosyası Kontrolü

```bash
# Tüm zone'ları kontrol et
for zone in example.com example.net; do
    echo "=== $zone ==="
    named-checkzone $zone /etc/bind/zones/db.${zone}
done

# Genel config kontrolü
named-checkconf
```

### Dikkat Edilecekler

- BIND'da yorum karakteri `;` veya `//`'dir, `#` değildir.
- ACL tanımları kullanıldıkları yerden önce tanımlanmalıdır.
- Serial numaraları her zone değişikliğinde artırılmalıdır (`YYYYMMDDNN` formatı önerilir).
- `rndc querylog on/off` ile runtime'da query logging açılıp kapatılabilir.

---

## 6. OpenVPN Yapılandırması

### DNS Push

VPN istemcilerine sunucunun kendi DNS'ini vermek için `server.conf` dosyasına ekle:

```conf
push "dhcp-option DNS 10.8.0.1"   # VPN sunucusunun tünel IP'si
push "redirect-gateway def1"       # Tüm trafiği tünelden geçir
```

> **Neden önemli?** `redirect-gateway def1` tüm trafiği tünelden geçirir ancak DNS sorgularının da tünelden gitmesi için DNS push şarttır. Aksi hâlde istemci kendi ISP DNS'ini kullanmaya devam eder.

### Reload

```bash
systemctl restart openvpn@server
```

### İstemci Testi (macOS)

```bash
# DNS sunucusunun doğru atandığını kontrol et
scutil --dns | grep nameserver
```

---

## 7. Docker ve Konteyner Servisleri

### Konteyner Durumu

```bash
# Tüm konteynerleri listele
docker ps -a

# Belirli bir konteynerin logları
docker logs <konteyner-adı> --tail 50
```

### Sık Karşılaşılan Sorunlar

**Veritabanı şifre uyuşmazlığı:**  
`docker-compose.yml`'deki `POSTGRES_PASSWORD` değeri yalnızca ilk kurulumda geçerlidir. Volume zaten oluşturulmuşsa şifreyi veritabanı içinden güncellemek gerekir:

```bash
docker exec -it <db-konteyner> psql -U <kullanici> -c \
  "ALTER USER <kullanici> WITH PASSWORD 'yeni_sifre';"
```

**IPv6 sysctl çakışması:**  
Docker her konteyner başlatılışında container'ın `eth0` arayüzünde IPv6'yı devre dışı bırakır. `systemd-networkd` bunu tersine çevirmeye çalışabilir. Bu kozmetik bir uyarıdır, fonksiyonel soruna yol açmaz.

### docker-compose.yml

Şifre ve hassas bilgileri `.env` dosyasında saklayın, `docker-compose.yml`'de doğrudan yazmayın:

```yaml
environment:
  POSTGRES_PASSWORD: ${DB_PASSWORD}
```

---

## 8. Apache Reverse Proxy

### Temel Güvenlik Yapılandırması

```apache
<VirtualHost *:443>
    ServerName app.example.com

    # Reverse proxy
    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:3000/
    ProxyPassReverse / http://127.0.0.1:3000/

    # HTTPS arkasındaysa mutlaka "https" olmalı
    RequestHeader set X-Forwarded-Proto "https"

    # IP kısıtlaması (örnek: sadece VPN ve ofis IP'leri)
    <Location />
        Require ip 10.8.0.0/24
        Require ip 203.0.113.0/24
    </Location>

    SSLCertificateFile /etc/letsencrypt/live/app.example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/app.example.com/privkey.pem
    Include /etc/letsencrypt/options-ssl-apache.conf
</VirtualHost>

# HTTP → HTTPS redirect
<VirtualHost *:80>
    ServerName app.example.com
    RewriteEngine on
    RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>
```

### Kontrol Listesi

```bash
# Directory listing kapalı mı?
grep -rn "Options Indexes" /etc/apache2/sites-enabled/
# Tüm sonuçlar "-Indexes" içermeli

# X-Forwarded-Proto doğru mu?
grep -rn "X-Forwarded-Proto" /etc/apache2/sites-enabled/
# HTTPS arkasındaki tüm proxy vhost'larında "https" olmalı

# Aynı ServerName için çift vhost var mı?
grep -rn "ServerName" /etc/apache2/sites-enabled/

# Config testi
apache2ctl configtest
```

### Güvenlik Başlıkları

```apache
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
Header set X-Content-Type-Options "nosniff"
Header set X-Frame-Options "SAMEORIGIN"
Header set Referrer-Policy "strict-origin-when-cross-origin"
Header set Permissions-Policy "geolocation=(), microphone=(), camera=()"
```

### ServerName Uyarısını Gidermek

```bash
echo "ServerName sunucu.example.com" >> /etc/apache2/apache2.conf
```

---

## 9. Fail2Ban

### Kurulum

```bash
apt install fail2ban
```

### jail.local Oluşturma

`/etc/fail2ban/jail.conf` dosyası doğrudan düzenlenmez; bunun yerine `/etc/fail2ban/jail.local` oluşturulur:

```bash
cat > /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
bantime = 1h
findtime = 10m
maxretry = 5
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

### Durum Kontrolü

```bash
# Aktif jail'leri listele
fail2ban-client status

# Belirli bir jail'in durumu
fail2ban-client status sshd

# Banlı IP'leri listele
fail2ban-client status sshd | grep "Banned IP"

# IP ban kaldırma
fail2ban-client set sshd unbanip <IP>
```

---

## 10. Unattended Upgrades

Güvenlik güncellemelerinin otomatik uygulanması için:

### /etc/apt/apt.conf.d/50unattended-upgrades

```bash
cat > /etc/apt/apt.conf.d/50unattended-upgrades << 'EOF'
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}";
    "${distro_id}:${distro_codename}-security";
    "${distro_id}ESMApps:${distro_codename}-apps-security";
    "${distro_id}ESM:${distro_codename}-infra-security";
};

Unattended-Upgrade::Package-Blacklist {
};

Unattended-Upgrade::AutoFixInterruptedDpkg "true";
Unattended-Upgrade::MinimalSteps "true";
Unattended-Upgrade::Remove-Unused-Dependencies "true";
Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";

// Sunucu otomatik yeniden başlatılmasın
Unattended-Upgrade::Automatic-Reboot "false";
Unattended-Upgrade::Automatic-Reboot-WithUsers "false";
EOF
```

### /etc/apt/apt.conf.d/20auto-upgrades

```bash
cat > /etc/apt/apt.conf.d/20auto-upgrades << 'EOF'
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::Unattended-Upgrade "1";
APT::Periodic::AutocleanInterval "7";
EOF
```

### Test

```bash
unattended-upgrade --dry-run --debug 2>&1 | tail -20
```

---

## Genel Kontrol Listesi

Aşağıdaki kontroller periyodik olarak yapılmalıdır:

```bash
# Başarısız servis var mı?
systemctl list-units --state=failed

# Disk kullanımı
df -h

# Bellek kullanımı
free -h

# Açık portlar
ss -tlnp

# Son başarısız SSH girişleri
journalctl -u sshd | grep "Failed" | tail -20

# Fail2ban durumu
fail2ban-client status

# Sertifika son kullanma tarihleri
for domain in example.com app.example.com; do
    echo -n "$domain: "
    echo | openssl s_client -connect $domain:443 2>/dev/null \
      | openssl x509 -noout -enddate
done
```

---

## Kaynaklar

- [Ubuntu Server Guide](https://ubuntu.com/server/docs)
- [BIND9 ARM](https://bind9.readthedocs.io/)
- [fail2ban Documentation](https://www.fail2ban.org/wiki/index.php/Main_Page)
- [Apache HTTP Server Documentation](https://httpd.apache.org/docs/)
- [OpenVPN Documentation](https://openvpn.net/community-resources/)

---

*Bu rehber Ubuntu 24.04 LTS üzerinde test edilmiştir.*
