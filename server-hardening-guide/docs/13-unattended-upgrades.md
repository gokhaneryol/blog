# 13. Unattended Upgrades

[← Ana Sayfa](../README.md)

Güvenlik güncellemelerinin otomatik uygulanması, bilinen açıkların sistemde uzun süre açık kalmasını engeller.

## /etc/apt/apt.conf.d/50unattended-upgrades

```bash
cat > /etc/apt/apt.conf.d/50unattended-upgrades << 'EOF'
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}";
    "${distro_id}:${distro_codename}-security";
    "${distro_id}ESMApps:${distro_codename}-apps-security";
    "${distro_id}ESM:${distro_codename}-infra-security";
};

Unattended-Upgrade::Package-Blacklist {
    // Güncellenmesini istemediğin paketleri buraya ekle
    // "linux-image-*";
};

Unattended-Upgrade::AutoFixInterruptedDpkg "true";
Unattended-Upgrade::MinimalSteps "true";
Unattended-Upgrade::Remove-Unused-Dependencies "true";
Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";

// Sunucu otomatik yeniden başlatılmasın
Unattended-Upgrade::Automatic-Reboot "false";
Unattended-Upgrade::Automatic-Reboot-WithUsers "false";

// Mail bildirimi (opsiyonel, mail servisi kuruluysa)
// Unattended-Upgrade::Mail "admin@example.com";
// Unattended-Upgrade::MailReport "on-change";
EOF
```

## /etc/apt/apt.conf.d/20auto-upgrades

```bash
cat > /etc/apt/apt.conf.d/20auto-upgrades << 'EOF'
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::Unattended-Upgrade "1";
APT::Periodic::AutocleanInterval "7";
EOF
```

## Test ve İzleme

```bash
# Dry-run (gerçekten güncelleme yapmaz)
unattended-upgrade --dry-run --debug 2>&1 | tail -20

# Manuel çalıştır
unattended-upgrade -v

# Son güncelleme logları
cat /var/log/unattended-upgrades/unattended-upgrades.log

# Güncellenebilir paketler
apt list --upgradable 2>/dev/null
```

---

*Sonraki bölüm: [Faydalı Araçlar](14-tools.md)*
