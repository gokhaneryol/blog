# 14. Faydalı Araçlar

[← Ana Sayfa](../README.md)

Her sunucuda bulunması önerilen temel network ve sistem tanı araçları.

## Kurulum

```bash
apt install -y \
    nmap \
    mtr \
    tcpdump \
    netcat-openbsd \
    dnsutils \
    curl \
    wget \
    htop \
    iotop \
    iftop \
    lsof \
    strace \
    net-tools \
    traceroute \
    whois \
    jq
```

## Araç Başvurusu

| Araç | Kullanım Amacı |
|------|---------------|
| `nmap` | Port tarama, servis ve versiyon keşfi |
| `mtr` | Traceroute + ping kombinasyonu, hat kalitesi ölçümü |
| `tcpdump` | Paket yakalama ve analiz |
| `nc` (netcat) | Port testi, ham TCP/UDP bağlantısı |
| `dig` | DNS sorgusu ve tanı |
| `ss` | Açık soket ve bağlantı listesi (netstat'ın modern hali) |
| `lsof` | Açık dosya ve port-proses eşleştirme |
| `htop` | Etkileşimli süreç izleme |
| `iotop` | Disk I/O izleme |
| `iftop` | Arayüz bazlı bant genişliği izleme |
| `strace` | Sistem çağrısı izleme (servis debug) |
| `jq` | JSON çıktı işleme |

## Örnek Kullanımlar

### nmap

```bash
# Sunucunun dışarıdan görünen açık portlarını tara
nmap -sV sunucu.example.com

# Belirli port aralığı
nmap -p 1-1024 sunucu.example.com

# UDP tarama
nmap -sU -p 53,123,161 sunucu.example.com

# OS ve servis versiyon tespiti
nmap -A sunucu.example.com
```

### mtr

```bash
# Interaktif mod
mtr 8.8.8.8

# Rapor modu (20 paket gönderir, çıktı verir)
mtr --report 8.8.8.8

# TCP ile (ICMP bloklu ağlarda)
mtr --tcp --port 443 example.com
```

### tcpdump

```bash
# 53 portuna gelen/giden DNS paketleri
tcpdump -i eth0 -n port 53

# Belirli IP'den gelen paketler
tcpdump -i eth0 src 203.0.113.10

# HTTP trafiğini yakala ve dosyaya yaz
tcpdump -i eth0 -w /tmp/capture.pcap port 80 or port 443

# SYN paketlerini izle (bağlantı denemeleri)
tcpdump -i eth0 "tcp[tcpflags] & tcp-syn != 0"
```

### dig

```bash
# Temel sorgu
dig example.com

# Belirli DNS sunucusuna sorgu
dig example.com @8.8.8.8

# Tam trace (root'tan itibaren)
dig example.com +trace

# MX kaydı
dig example.com MX

# Tüm kayıtlar
dig example.com ANY

# Tersine DNS
dig -x 203.0.113.10

# Kısa çıktı
dig +short example.com
```

### ss / lsof

```bash
# Dinleyen tüm TCP portları
ss -tlnp

# Belirli porta bağlı proses
ss -tlnp | grep :80
lsof -i :80

# Tüm açık bağlantılar
ss -tnp

# ESTABLISHED bağlantılar
ss -tnp state established
```

### nc (netcat)

```bash
# Port açık mı kontrol et
nc -zv -w 3 example.com 443

# UDP port testi
nc -zuv -w 3 example.com 53

# Basit port dinleyici (test için)
nc -l 8080
```

---

*Sonraki bölüm: [Genel Kontrol Listesi](15-checklist.md)*
