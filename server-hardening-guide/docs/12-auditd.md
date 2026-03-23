# 12. Auditd — Sistem Denetim Kaydı

[← Ana Sayfa](../README.md)

`auditd`, Linux çekirdeğinin güvenlik olaylarını (dosya erişimleri, sistem çağrıları, kullanıcı işlemleri) kayıt altına almasını sağlar. Özellikle uyumluluk gereksinimleri (PCI-DSS, ISO 27001, SOC 2) olan ortamlarda zorunludur.

## Kurulum

```bash
apt install auditd audispd-plugins
systemctl enable auditd
systemctl start auditd
```

## Temel Kurallar

`/etc/audit/rules.d/audit.rules`:

```bash
cat > /etc/audit/rules.d/audit.rules << 'EOF'
# Mevcut kuralları temizle
-D

# Buffer boyutu (yoğun sistemlerde artırılabilir)
-b 8192

# Kural yükleme hatalarında davranış: 0=sessiz, 1=printk, 2=panic
# Üretim için 1, test için 0 önerilir
-f 1

# Kimlik değişiklikleri (su, sudo)
-w /bin/su -p x -k privilege_escalation
-w /usr/bin/sudo -p x -k privilege_escalation

# Kullanıcı/grup yönetimi
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/group -p wa -k identity
-w /etc/gshadow -p wa -k identity
-w /etc/sudoers -p wa -k sudoers
-w /etc/sudoers.d/ -p wa -k sudoers

# SSH yapılandırması
-w /etc/ssh/sshd_config -p wa -k sshd_config

# Ağ yapılandırması
-w /etc/hosts -p wa -k network_config
-w /etc/network/ -p wa -k network_config
-w /etc/sysctl.conf -p wa -k network_config
-w /etc/sysctl.d/ -p wa -k network_config

# Cron
-w /etc/cron.d/ -p wa -k cron
-w /etc/crontab -p wa -k cron
-w /var/spool/cron/ -p wa -k cron

# Başarısız dosya erişimleri
-a always,exit -F arch=b64 -S open,openat -F exit=-EACCES -k access_denied
-a always,exit -F arch=b64 -S open,openat -F exit=-EPERM -k access_denied

# Sistem zamanı değişikliği
-a always,exit -F arch=b64 -S adjtimex,settimeofday -k time_change
-w /etc/localtime -p wa -k time_change

# Kernel modülleri
-w /sbin/insmod -p x -k modules
-w /sbin/rmmod -p x -k modules
-w /sbin/modprobe -p x -k modules

# Kuralları değiştirilemez yap — aktif edilince reboot gerekir
# -e 2
EOF

# Kuralları uygula
auditctl -R /etc/audit/rules.d/audit.rules

# Mevcut kuralları listele
auditctl -l
```

## Log Sorgulama

```bash
# Son olaylar (son 10 dakika)
ausearch -ts recent

# Belirli anahtar kelimeye göre
ausearch -k privilege_escalation
ausearch -k identity
ausearch -k sshd_config

# Belirli kullanıcı
ausearch -ua kullanici

# Belirli zaman aralığı
ausearch -ts "03/23/2026 08:00:00" -te "03/23/2026 10:00:00"

# Belirli dosya
ausearch -f /etc/passwd
```

## Raporlama

```bash
# Genel özet
aureport --summary

# Kimlik doğrulama olayları
aureport --auth

# Giriş olayları
aureport --login

# Başarısız olaylar
aureport --failed

# Bugünün özeti
aureport --summary --start today

# Komut çalıştırma olayları
aureport --comm
```

## Log Dosyası

```bash
# Canlı takip
tail -f /var/log/audit/audit.log

# İnsan okunabilir format
ausearch -ts recent | aureport -i
```

## Logrotate

`auditd` kendi log rotasyonunu `/etc/audit/auditd.conf` ile yönetir:

```ini
max_log_file = 50          # MB cinsinden max dosya boyutu
max_log_file_action = ROTATE
num_logs = 7               # Tutulacak dosya sayısı
```

---

*Sonraki bölüm: [Unattended Upgrades](13-unattended-upgrades.md)*
