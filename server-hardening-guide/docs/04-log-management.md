# 4. Log Yönetimi

[← Ana Sayfa](../README.md)

`syslog` tüm sistem mesajlarını tek dosyada toplar. Yoğun sunucularda bu dosya hızla büyür ve ilgili olayları bulmak zorlaşır. Servis bazlı log ayrıştırması hem okunabilirliği artırır hem de log analiz araçlarının işini kolaylaştırır.

## UFW Loglarını Ayırmak

```bash
# rsyslog kuralı oluştur
cat > /etc/rsyslog.d/20-ufw.conf << 'EOF'
:msg,contains,"[UFW " /var/log/ufw/ufw.log
& stop
EOF

# Log dizini oluştur
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

## BIND9 Loglarını Ayırmak

BIND9 için ayrı log yapılandırması `named.conf.options` içinde yapılır. Bkz. [BIND9](07-bind9.md).

## Genel Logrotate İpuçları

```bash
# Mevcut logrotate konfigürasyonlarını listele
ls /etc/logrotate.d/

# Belirli bir konfigürasyonu test et (dry-run)
logrotate -d /etc/logrotate.d/ufw

# Manuel olarak çalıştır
logrotate -f /etc/logrotate.d/ufw
```

## journald Kalıcı Log

Ubuntu'da `journald` logları varsayılan olarak RAM'de tutulabilir (reboot sonrası kaybolur). Kalıcı hale getirmek için:

```bash
mkdir -p /var/log/journal
systemd-tmpfiles --create --prefix /var/log/journal
systemctl restart systemd-journald
```

Disk kullanımını sınırlamak için `/etc/systemd/journald.conf`:
```ini
[Journal]
SystemMaxUse=500M
SystemKeepFree=1G
MaxRetentionSec=1month
```

---

*Sonraki bölüm: [SSH Güvenliği](05-ssh.md)*
