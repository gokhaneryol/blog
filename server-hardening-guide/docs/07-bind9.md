# 7. BIND9 DNS Sunucusu

[← Ana Sayfa](../README.md)

BIND9, en yaygın kullanılan authoritative ve recursive DNS sunucusudur. Hem kendi zone'larınız için authoritative, hem de yetkili istemciler için recursive resolver olarak çalışabilir.

## Kurulum

```bash
apt install bind9 bind9utils bind9-doc
```

## Temel Güvenlik Yapılandırması

`/etc/bind/named.conf.options`:

```bind
// ACL tanımı options bloğundan ÖNCE olmalı
acl "izin-verilen-adresler" {
    10.8.0.0/24;           // VPN ağı
    203.0.113.10/32;       // Yetkili harici IP
    localhost;
};

options {
    directory "/var/cache/bind";

    dnssec-validation auto;

    // IPv6 dinlemeyi kapat (gerekli değilse)
    listen-on-v6 { none; };

    // Tüm sorgulara yanıt ver (authoritative için)
    allow-query { any; };

    // Recursive sorgu sadece yetkili adreslerden
    recursion yes;
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

> **Dikkat:** BIND'da yorum karakteri `;` veya `//`'dir. `#` karakteri DNS zone dosyalarında desteklenmez.

## Log Dizini

```bash
mkdir -p /var/log/named
chown bind:bind /var/log/named
```

## Zone Tanımlama

`/etc/bind/named.conf.local`:

```bind
zone "example.com" {
    type master;
    file "/etc/bind/zones/db.example.com";
};
```

### Zone Dosyası Örneği

`/etc/bind/zones/db.example.com`:

```dns
$TTL 86400
@   IN  SOA ns1.example.com. admin.example.com. (
            2024032301 ; Serial (YYYYMMDDNN formatı)
            3600       ; Refresh
            1800       ; Retry
            1209600    ; Expire
            86400 )    ; Minimum TTL

; NS kayıtları
    IN  NS  ns1.example.com.

; A kayıtları
@       IN  A   203.0.113.10
ns1     IN  A   203.0.113.10
www     IN  A   203.0.113.10
mail    IN  A   203.0.113.10

; MX
@       IN  MX  10 mail.example.com.

; SPF
@       IN  TXT "v=spf1 mx a include:_spf.google.com ~all"
```

## Kontrol Komutları

```bash
# Config sözdizimi kontrolü
named-checkconf

# Zone dosyası kontrolü
named-checkzone example.com /etc/bind/zones/db.example.com

# Tüm zone'ları kontrol et
for zone in example.com example.net; do
    echo "=== $zone ==="
    named-checkzone $zone /etc/bind/zones/db.${zone}
done

# Reload (servis durmadan)
systemctl reload named
# veya
rndc reload
```

## rndc — Runtime Yönetimi

`rndc` (Remote Name Daemon Control) BIND'ı yeniden başlatmadan runtime'da yönetmek için kullanılır:

```bash
rndc status              # Genel durum
rndc reload              # Tüm zone'ları yeniden yükle
rndc reload example.com  # Tek zone yenile
rndc flush               # Cache'i temizle
rndc querylog on         # Query log'u aç
rndc querylog off        # Query log'u kapat
```

## Logrotate

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

---

*Sonraki bölüm: [OpenVPN Yapılandırması](08-openvpn.md)*
