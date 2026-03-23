# Linux Sunucu Kurulum Sonrası Güvenlik ve Servis Yapılandırma Rehberi

Bu rehber, Ubuntu tabanlı bir Linux sunucusunun kurulum sonrasında güvenli ve sağlıklı şekilde çalışabilmesi için uygulanması gereken yapılandırmaları kapsamaktadır. Adımlar gerçek dünya deneyiminden elde edilmiş olup bölüm bölüm açıklanmıştır.

> **Platform:** Ubuntu 24.04 LTS  
> **Kapsam:** Kurulum sonrası temel sıkılaştırma, servis yapılandırması, izleme

---

## Bölümler

| # | Konu | Dosya |
|---|------|-------|
| 1 | Sistem Loglarının Denetimi | [01-system-logs.md](docs/01-system-logs.md) |
| 2 | Gereksiz Servislerin Devre Dışı Bırakılması | [02-disable-services.md](docs/02-disable-services.md) |
| 3 | UFW Güvenlik Duvarı | [03-ufw.md](docs/03-ufw.md) |
| 4 | Log Yönetimi | [04-log-management.md](docs/04-log-management.md) |
| 5 | SSH Güvenliği | [05-ssh.md](docs/05-ssh.md) |
| 6 | IPv6 Yönetimi | [06-ipv6.md](docs/06-ipv6.md) |
| 7 | BIND9 DNS Sunucusu | [07-bind9.md](docs/07-bind9.md) |
| 8 | OpenVPN Yapılandırması | [08-openvpn.md](docs/08-openvpn.md) |
| 9 | Docker ve Konteyner Servisleri | [09-docker.md](docs/09-docker.md) |
| 10 | Apache Reverse Proxy ve Sanal Sunucular | [10-apache.md](docs/10-apache.md) |
| 11 | Fail2Ban | [11-fail2ban.md](docs/11-fail2ban.md) |
| 12 | Auditd — Sistem Denetim Kaydı | [12-auditd.md](docs/12-auditd.md) |
| 13 | Unattended Upgrades | [13-unattended-upgrades.md](docs/13-unattended-upgrades.md) |
| 14 | Faydalı Araçlar | [14-tools.md](docs/14-tools.md) |
| 15 | Genel Kontrol Listesi | [15-checklist.md](docs/15-checklist.md) |

---

## Hızlı Başlangıç

Yeni bir sunucuya bağlandıktan sonra önerilen sıra:

1. **Logları incele** → başarısız servis veya crash loop var mı?
2. **SSH'ı sıkılaştır** → anahtar tabanlı giriş, root girişini kapat
3. **UFW'yi yapılandır** → sadece gerekli portları aç
4. **Gereksiz servisleri kapat** → saldırı yüzeyini küçült
5. **IPv6 kararını ver** → kullanılmıyorsa kapat
6. **fail2ban'ı kur** → brute force koruması
7. **unattended-upgrades** → güvenlik güncellemeleri otomatik gelsin
8. **auditd** → kritik sistem olaylarını kayıt altına al

---

## Katkı

Düzeltme veya ekleme önerileriniz için PR açabilirsiniz.
