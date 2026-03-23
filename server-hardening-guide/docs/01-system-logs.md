# 1. Sistem Loglarının Denetimi

[← Ana Sayfa](../README.md)

Kurulum sonrasında ilk yapılacak iş sistem loglarını incelemektir. Crash loop, servis hataları ve beklenmedik ağ bağlantıları bu aşamada tespit edilebilir.

```bash
# Canlı log takibi
tail -f /var/log/syslog

# Başarısız servisleri listele
systemctl list-units --state=failed

# Belirli bir servisin logları
journalctl -u <servis-adı> -n 50

# Tüm boot'taki hataları göster
journalctl -b --priority=err
```

## Dikkat Edilecekler

- **Crash loop:** `restart counter` değeri yüksek olan servisler — sistemde sürekli çöküp yeniden başlayan bir servis gereksiz CPU/IO tüketir ve asıl sorunun üzerini örter.
- **AppArmor/SELinux engelleri:** `apparmor="DENIED"` içeren kernel log satırları — bir servis beklenmedik bir dosyaya veya syscall'a erişmeye çalışıyor olabilir.
- **UFW block:** `[UFW BLOCK]` satırları — beklenmedik kaynak IP veya hedef portlar saldırı girişimini işaret edebilir.

## Sık Karşılaşılan Durumlar

**Servis crash loop tespiti:**
```bash
# Kaç kez restart ettiğini görmek için
systemctl status <servis-adı> | grep "restart counter"

# Son 10 dakikanın logları
journalctl -u <servis-adı> --since "10 minutes ago"
```

**AppArmor engeli:**
```bash
grep "apparmor=\"DENIED\"" /var/log/kern.log | tail -20
```

---

*Sonraki bölüm: [Gereksiz Servislerin Devre Dışı Bırakılması](02-disable-services.md)*
