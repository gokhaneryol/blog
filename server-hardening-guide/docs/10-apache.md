# 10. Apache Reverse Proxy ve Sanal Sunucular

[← Ana Sayfa](../README.md)

## Kurulum ve Gerekli Modüller

```bash
apt install apache2

# Gerekli modülleri etkinleştir
a2enmod rewrite proxy proxy_http headers ssl
systemctl restart apache2
```

## Yeni Bir Sanal Sunucu (VirtualHost) Ekleme

### 1. Site Dizinini Oluştur

```bash
mkdir -p /var/www/example.com
chown -R www-data:www-data /var/www/example.com
chmod -R 755 /var/www/example.com
```

### 2. Config Dosyasını Oluştur

`/etc/apache2/sites-available/example.com.conf`:

```apache
# HTTP → HTTPS redirect
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com

    RewriteEngine on
    RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>

# HTTPS vhost
<IfModule mod_ssl.c>
<VirtualHost *:443>
    ServerName example.com
    ServerAlias www.example.com
    ServerAdmin webmaster@example.com

    DocumentRoot /var/www/example.com

    <Directory /var/www/example.com>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    # Güvenlik başlıkları
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
    Header set X-Content-Type-Options "nosniff"
    Header set X-Frame-Options "SAMEORIGIN"
    Header set Referrer-Policy "strict-origin-when-cross-origin"

    ErrorLog ${APACHE_LOG_DIR}/example.com-error.log
    CustomLog ${APACHE_LOG_DIR}/example.com-access.log combined

    SSLCertificateFile /etc/letsencrypt/live/example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem
    Include /etc/letsencrypt/options-ssl-apache.conf
</VirtualHost>
</IfModule>
```

### 3. Siteyi Etkinleştir

```bash
a2ensite example.com.conf
apache2ctl configtest
systemctl reload apache2
```

### Siteyi Devre Dışı Bırakmak

```bash
a2dissite example.com.conf
systemctl reload apache2
```

---

## Let's Encrypt ile SSL Sertifikası

### Certbot Kurulumu

```bash
apt install certbot python3-certbot-apache
```

### Sertifika Alma

```bash
# Apache eklentisi ile (önerilir — config'i otomatik günceller)
certbot --apache -d example.com -d www.example.com

# Standalone (Apache'yi geçici olarak durdurur)
certbot certonly --standalone -d example.com

# Webroot yöntemi (Apache çalışırken, manuel config gerekir)
certbot certonly --webroot -w /var/www/example.com -d example.com
```

Certbot sertifikayı `/etc/letsencrypt/live/example.com/` altına kaydeder ve Apache config'ini otomatik günceller.

### Sertifika Yenileme

Let's Encrypt sertifikaları 90 gün geçerlidir. Certbot kurulumu sırasında otomatik yenileme için bir systemd timer veya cron job oluşturulur.

```bash
# Otomatik yenilemenin aktif olduğunu kontrol et
systemctl status certbot.timer
# veya
crontab -l | grep certbot

# Manuel yenileme testi (dry-run)
certbot renew --dry-run

# Tüm sertifikaları yenile
certbot renew

# Sadece belirli bir sertifikayı yenile
certbot renew --cert-name example.com
```

### Sertifika Durumu

```bash
# Tüm sertifikaları listele
certbot certificates

# Belirli bir domain için son kullanma tarihi
echo | openssl s_client -connect example.com:443 2>/dev/null \
  | openssl x509 -noout -enddate

# Tüm sertifikaları toplu kontrol
for domain in example.com app.example.com; do
    echo -n "$domain: "
    echo | openssl s_client -connect $domain:443 2>/dev/null \
      | openssl x509 -noout -enddate 2>/dev/null
done
```

### Yenileme Hook'ları

Sertifika yenilendiğinde Apache'nin otomatik reload edilmesi için Certbot hook mekanizması kullanılabilir:

```bash
# Post-renew hook
cat > /etc/letsencrypt/renewal-hooks/post/reload-apache.sh << 'EOF'
#!/bin/bash
systemctl reload apache2
EOF

chmod +x /etc/letsencrypt/renewal-hooks/post/reload-apache.sh
```

---

## Reverse Proxy Yapılandırması

Arka planda çalışan bir uygulama (örn. port 3000'de bir Node.js veya Docker servisi) Apache üzerinden dışarıya açılacaksa:

```apache
<IfModule mod_ssl.c>
<VirtualHost *:443>
    ServerName app.example.com

    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:3000/
    ProxyPassReverse / http://127.0.0.1:3000/

    # HTTPS arkasında çalışıyorsa mutlaka "https" olmalı
    RequestHeader set X-Forwarded-Proto "https"

    # Opsiyonel: IP kısıtlaması
    <Location />
        Require ip 10.8.0.0/24
        Require ip 203.0.113.10
    </Location>

    SSLCertificateFile /etc/letsencrypt/live/app.example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/app.example.com/privkey.pem
    Include /etc/letsencrypt/options-ssl-apache.conf
</VirtualHost>
</IfModule>

<VirtualHost *:80>
    ServerName app.example.com
    RewriteEngine on
    RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>
```

---

## Güvenlik Kontrol Listesi

```bash
# Directory listing kapalı mı? (tüm sonuçlar -Indexes içermeli)
grep -rn "Options Indexes" /etc/apache2/sites-enabled/

# X-Forwarded-Proto doğru mu? (proxy vhost'larında "https" olmalı)
grep -rn "X-Forwarded-Proto" /etc/apache2/sites-enabled/

# Aynı ServerName için çift vhost var mı?
grep -rn "ServerName" /etc/apache2/sites-enabled/ | sort -k2

# Config sözdizimi testi
apache2ctl configtest

# Yüklü modüller
apache2ctl -M | sort
```

## ServerName Uyarısını Gidermek

```bash
echo "ServerName sunucu.example.com" >> /etc/apache2/apache2.conf
apache2ctl configtest && systemctl reload apache2
```

## Güvenlik Başlıkları

```apache
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
Header set X-Content-Type-Options "nosniff"
Header set X-Frame-Options "SAMEORIGIN"
Header set Referrer-Policy "strict-origin-when-cross-origin"
Header set Permissions-Policy "geolocation=(), microphone=(), camera=()"
```

---

*Sonraki bölüm: [Fail2Ban](11-fail2ban.md)*
