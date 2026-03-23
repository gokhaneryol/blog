# 3. UFW Güvenlik Duvarı

[← Ana Sayfa](../README.md)

UFW (Uncomplicated Firewall), `iptables` üzerine kurulu basit bir güvenlik duvarı yönetim aracıdır. Ubuntu'da varsayılan olarak kurulu gelir.

## Temel Kurulum

```bash
# Mevcut durumu gör
ufw status verbose

# Varsayılan politikalar (önerilir)
ufw default deny incoming
ufw default allow outgoing

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

## Belirli IP'lerden Erişime İzin Verme

```bash
# Sadece belirli IP'den SSH
ufw allow from 203.0.113.10 to any port 22

# Belirli subnet'ten tüm erişim
ufw allow from 10.8.0.0/24
```

## Kural Yönetimi

```bash
# Kuralları numaralı listele
ufw status numbered

# Belirli kuralı sil (numara ile)
ufw delete 3

# Kural sil (tanım ile)
ufw delete allow 80/tcp

# Reload
ufw reload
```

## IPv6 Farkındalığı

UFW, `/etc/default/ufw` dosyasında `IPV6=yes` ile hem IPv4 hem IPv6 için kural oluşturur.

```bash
# (v6) kurallarının varlığını kontrol et
ufw status | grep v6
```

IPv6 kullanılmıyorsa:
```bash
sed -i 's/IPV6=yes/IPV6=no/' /etc/default/ufw
ufw reload
```

> IPv6 stack'ini sysctl ile tamamen kapatmak daha temiz bir yaklaşımdır. Bkz. [IPv6 Yönetimi](06-ipv6.md).

## Loglama

```bash
# UFW loglama seviyesi
ufw logging medium    # low / medium / high / full

# Log dosyası (rsyslog ayarı yapılmışsa)
tail -f /var/log/ufw/ufw.log
```

> UFW loglarını syslog'dan ayırmak için bkz. [Log Yönetimi](04-log-management.md).

---

*Sonraki bölüm: [Log Yönetimi](04-log-management.md)*
