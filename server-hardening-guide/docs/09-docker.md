# 9. Docker ve Konteyner Servisleri

[← Ana Sayfa](../README.md)

## Temel Komutlar

```bash
# Tüm konteynerleri listele
docker ps -a

# Belirli bir konteynerin logları
docker logs <konteyner-adı> --tail 50 -f

# Konteyner içinde komut çalıştır
docker exec -it <konteyner-adı> bash

# Konteyner kaynak kullanımı
docker stats
```

## docker-compose

```bash
# Servisleri başlat
docker compose up -d

# Servisleri durdur ve kaldır
docker compose down

# Logları takip et
docker compose logs -f

# Tek servisi yeniden başlat
docker compose restart <servis-adı>
```

## Sık Karşılaşılan Sorunlar

### Veritabanı Şifre Uyuşmazlığı

`docker-compose.yml`'deki `POSTGRES_PASSWORD` (veya `MYSQL_ROOT_PASSWORD`) değeri yalnızca volume ilk oluşturulduğunda geçerlidir. Volume zaten mevcutsa şifreyi veritabanı içinden güncellemek gerekir:

```bash
# PostgreSQL
docker exec -it <db-konteyner> psql -U <kullanici> -c \
  "ALTER USER <kullanici> WITH PASSWORD 'yeni_sifre';"

# MySQL/MariaDB
docker exec -it <db-konteyner> mysql -u root -p -e \
  "ALTER USER 'kullanici'@'%' IDENTIFIED BY 'yeni_sifre';"
```

### Konteyner Sürekli Restart Ediyorsa

```bash
# Logları kontrol et
docker logs <konteyner-adı> --tail 100

# Exit koduna bak
docker inspect <konteyner-adı> | grep -A5 "State"
```

### IPv6 sysctl Uyarısı

Docker her konteyner başlatışında container arayüzünde IPv6 ayarını değiştirebilir. Bu `syslog`'da uyarı üretir ancak fonksiyonel sorun yaratmaz. Bkz. [IPv6 Yönetimi](06-ipv6.md).

## Güvenlik Önerileri

### Hassas Bilgileri .env ile Yönet

`docker-compose.yml`'de şifre ve token'ları doğrudan yazmak yerine `.env` dosyası kullan:

```bash
# .env
DB_PASSWORD=guclu_sifre_buraya
APP_SECRET=gizli_token_buraya
```

```yaml
# docker-compose.yml
environment:
  POSTGRES_PASSWORD: ${DB_PASSWORD}
  APP_SECRET: ${APP_SECRET}
```

```bash
# .env dosyasını git'ten dışla
echo ".env" >> .gitignore
```

### Port Binding

Konteynerlerin portlarını sadece gerektiğinde dışa aç. Mümkünse sadece localhost'a bağla:

```yaml
ports:
  - "127.0.0.1:3000:3000"   # Sadece localhost (önerilir)
  # - "3000:3000"            # Tüm arayüzler (dikkatli kullan)
```

---

*Sonraki bölüm: [Apache Reverse Proxy ve Sanal Sunucular](10-apache.md)*
