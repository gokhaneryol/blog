# 8. OpenVPN Yapılandırması

[← Ana Sayfa](../README.md)

## Temel server.conf

```conf
port 1194
proto udp
dev tun

# Topoloji
topology subnet
server 10.8.0.0 255.255.255.0

# Tüm trafiği tünelden geçir
push "redirect-gateway def1"

# DNS push: istemcilere VPN sunucusunun kendi DNS'ini ver
push "dhcp-option DNS 10.8.0.1"

# İç ağa ek route (gerekirse)
# push "route 192.168.1.0 255.255.255.0"

# Bağlantı kararlılığı
keepalive 10 120

# Güvenlik
tls-auth ta.key 0
cipher AES-256-GCM
auth SHA256

# Ayrıcalık düşürme
user nobody
group nogroup
persist-key
persist-tun

# Loglama
status /var/log/openvpn/openvpn-status.log
log /var/log/openvpn/openvpn.log
verb 3
```

## redirect-gateway ve DNS Push İlişkisi

`push "redirect-gateway def1"` tüm IPv4 trafiğini tünelden geçirir. Ancak DNS sorgularının da tünelden gitmesi için DNS push **ayrıca** gereklidir. İkisi birlikte kullanılmazsa:

| Durum | Web trafiği | DNS sorguları |
|-------|-------------|---------------|
| Sadece redirect-gateway | Tünelden ✓ | ISP DNS'e gider ✗ (DNS leak) |
| redirect-gateway + DNS push | Tünelden ✓ | VPN DNS'e gider ✓ |

### DNS Leak Testi

```bash
# İstemcide, VPN bağlıyken çalıştır
dig +short myip.opendns.com @resolver1.opendns.com
# VPN sunucusunun genel IP'sini dönmeli

# Alternatif
curl https://ifconfig.me
# VPN sunucusunun IP'si görünmeli
```

## Route Yönetimi

### Tüm Trafik (Full Tunnel)

```conf
push "redirect-gateway def1"
```

`def1` yöntemi, `0.0.0.0/0` yerine iki özgül route ekler (`0.0.0.0/1` ve `128.0.0.0/1`). Bu sayede VPN sunucusuna giden route korunur.

### Split Tunnel (Sadece Belirli Ağlar)

Tüm trafiği değil, yalnızca belirli ağları tünelden geçirmek için:

```conf
# redirect-gateway push KALDIR veya yorum yap
# push "redirect-gateway def1"

# Sadece iç ağa route ekle
push "route 10.0.0.0 255.255.0.0"
push "route 192.168.1.0 255.255.255.0"
```

### İstemciye Statik IP Atama (ccd)

```bash
# ccd dizini oluştur
mkdir -p /etc/openvpn/ccd
```

`server.conf`'a ekle:
```conf
client-config-dir /etc/openvpn/ccd
```

`/etc/openvpn/ccd/kullanici-adi` dosyası oluştur:
```conf
ifconfig-push 10.8.0.10 255.255.255.0
```

## İstemci Tarafında Kontrol

```bash
# macOS: Atanan DNS'i kontrol et
scutil --dns | grep nameserver

# Linux: Routing tablosu
ip route show
# 0.0.0.0/1 ve 128.0.0.0/1 via <tun_ip> görünmeli

# Bağlı istemciler (sunucuda)
cat /var/log/openvpn/openvpn-status.log
```

## Reload

```bash
systemctl restart openvpn@server
systemctl status openvpn@server
```

---

*Sonraki bölüm: [Docker ve Konteyner Servisleri](09-docker.md)*
