# MareNostrum 5 Kullanıcı Rehberi
### EuroHPC Süperbilgisayarında HPC ve Yapay Zeka Çalışmaları

**Yazar:** Gökhan Eryol
**Tarih:** Nisan 2026
**Platform:** MareNostrum 5 — Barcelona Supercomputing Center (BSC)
**Kaynak:** EuroHPC AI Factory Playground — `ehpc648`

> **Not:** Bu rehber, BSC'nin resmi dokümantasyonuna ([supportkc.bsc.es](https://www.bsc.es/supportkc/docs/MareNostrum5/overview/)) dayalı olarak ve bizzat gerçekleştirilen çalışmalardan elde edilen deneyimler ışığında hazırlanmıştır. Resmi dokümantasyon her zaman nihai referans kaynağıdır.

---

## İçindekiler

1. [Giriş ve Bağlam](#1-giriş-ve-bağlam)
2. [MareNostrum 5 Sistem Mimarisi](#2-marenostrum-5-sistem-mimarisi)
3. [Erişim ve Bağlantı](#3-erişim-ve-bağlantı)
4. [Depolama Alanları ve Dizin Yapısı](#4-depolama-alanları-ve-dizin-yapısı)
5. [Dosya Transferi](#5-dosya-transferi)
6. [Modül Sistemi](#6-modül-sistemi)
7. [Python Paket Yönetimi](#7-python-paket-yönetimi)
8. [Container Ortamları: Singularity / Apptainer](#8-container-ortamları-singularity--apptainer)
9. [SLURM ile İş Yönetimi](#9-slurm-ile-i̇ş-yönetimi)
10. [Paralel ve Çok Görevli İşler](#10-paralel-ve-çok-görevli-i̇şler)
11. [Yapay Zeka ve LLM Servisleri](#11-yapay-zeka-ve-llm-servisleri)
12. [Güvenlik ve İzin Yönetimi](#12-güvenlik-ve-i̇zin-yönetimi)
13. [Veri, Etik ve Kullanım Politikaları](#13-veri-etik-ve-kullanım-politikaları)
14. [Sorun Giderme](#14-sorun-giderme)
15. [Hızlı Başvuru Kartı](#15-hızlı-başvuru-kartı)
16. [Sıkça Sorulan Sorular (SSS)](#16-sıkça-sorulan-sorular-sss)

---

## 1. Giriş ve Bağlam

### 1.1 Bu Rehber Kimler İçin?

Bu rehber, MareNostrum 5 (MN5) üzerinde HPC ve özellikle yapay zeka (AI/LLM) çalışmaları yapmak isteyen kullanıcılar için hazırlanmıştır. Hedef kitle, temel Linux ve terminal bilgisine sahip, SLURM veya benzeri iş planlayıcılarla aşinalığı olan ya da edinmeye çalışan araştırmacılar ve mühendislerdir.

Kendi geçmişim açısından belirtmek gerekirse: Bu rehberi yazan kişi olarak MN5'ten önce Türkiye'nin ulusal HPC altyapısı olan TRUBA'yı kullandım. TRUBA deneyimi sayesinde GPFS depolama sistemleri, SLURM iş kuyruğu ve tipik HPC iş akışları hakkında temel bir altyapıya sahip olarak MN5'e başladım. Süper kullanıcı (system administrator) değilim; kullanıcı perspektifinden yazan biri olarak bu rehberi, "keşke baştan bilseydim" dediğim şeyleri içerecek şekilde hazırladım. Bir konuda BİLMİYORUM demeyi tercih ediyor ve doğru kaynağa yönlendirmeyi önemsiyorum.

### 1.2 EuroHPC ve MN5'e Erişim

MareNostrum 5, Avrupa'nın pre-exascale süperbilgisayarlarından biridir ve Barcelona Supercomputing Center'da (BSC, İspanya) konuşlandırılmıştır. EuroHPC bünyesinde çeşitli tahsis çağrıları aracılığıyla erişim sağlanabilir:

- **Regular Access:** Büyük araştırma projeleri için (milyonlarca CPU-saati)
- **Benchmark Access:** Ölçekleme testleri için
- **Development Access:** Küçük geliştirme ve test için
- **AI Factory Playground:** Yapay zeka odaklı pilot projeler için

Başvurular EuroHPC portalı üzerinden yapılmaktadır. Tahsis alındıktan sonra BSC hesabı aktive edilir ve bu rehberde anlatılan adımlar uygulanabilir.

> **Önemli:** Tahsis başvurusu yapılmadan önce, proje tanımını ve kaynak gerekçesini dikkatli hazırlayın. Tahsis miktarı proje tanımına göre belirlenir ve sonradan artırılması ek başvuru gerektirir.

---

## 2. MareNostrum 5 Sistem Mimarisi

MN5, iki ana partition'dan (bölüm) oluşur. Bunlar farklı donanım özelliklerine sahiptir ve farklı iş tipleri için uygundur.

### 2.1 GPP — General Purpose Partition

CPU ağırlıklı hesaplama için tasarlanmış klasik HPC partition'ıdır.

| Özellik | Değer |
|---|---|
| Node sayısı | 6,192 (standart) + 216 HighMem + 10 Data + 72 HBM |
| CPU/node | 2x Intel Xeon Platinum 8480+ (56 core × 2 = 112 core) |
| RAM/node | 256 GB (standart), 1 TB (HighMem), 2 TB (Data) |
| Depolama | 960 GB NVMe local SSD |
| Ağ | ConnectX-7 NDR200 InfiniBand (100 Gb/s/node) |
| Kullanılabilir RAM/core | ~2 GB |

GPP büyük NUMA topolojisine sahiptir: her node 2 socket, her socket 56 fiziksel core (Hyper-Threading ile 112 logical thread). Bellek 2 NUMA domaininde bölünmüştür; dolayısıyla cross-NUMA erişim performans kaybına yol açabilir.

### 2.2 ACC — Accelerated Partition (GPU)

AI, derin öğrenme ve GPU tabanlı simülasyonlar için tasarlanmış partition.

| Özellik | Değer |
|---|---|
| Node sayısı | 1,120 |
| CPU/node | 2x Intel Xeon Platinum 8460Y+ (40 core × 2 = 80 core) |
| GPU/node | 4× NVIDIA H100 SXM5 (64 GB HBM2 — bazı kaynaklarda 80 GB SXM5 olarak geçer; pratik deneyimde 80 GB görülmüştür) |
| RAM/node | 512 GB DDR5 (6.25 GB/core kullanılabilir) |
| Depolama | 480 GB NVMe local SSD |
| Ağ | 4× ConnectX-7 NDR200 InfiniBand (800 Gb/s/node) |

> **Bu rehberin odak noktası ACC partition'dır.** Tüm LLM ve AI örnekleri H100 GPU'lu ACC node'ları üzerinde çalıştırılmıştır.

### 2.3 InfiniBand Topolojisi

MN5'in ağ altyapısı 3 katmanlı fat-tree topolojisine sahip 324 QM9790 switch'ten oluşur. Toplam 3 GPP island, 1 storage island ve 7 ACC island bulunmaktadır. Bu yüksek bantgenişliği, özellikle multi-node MPI ve NCCL tabanlı AI eğitim iş yükleri için kritiktir.

---

## 3. Erişim ve Bağlantı

### 3.1 Login Node'ları

MN5'e sadece belirli giriş sunucuları (login node) üzerinden SSH ile bağlanılabilir. Hangi partition'da çalışacağınıza göre uygun login node'u seçin:

**GPP için:**
```
glogin1.bsc.es
glogin2.bsc.es
```

**ACC için:**
```
alogin1.bsc.es
alogin2.bsc.es
```

**Dosya transferi için:**
```
transfer1.bsc.es
transfer2.bsc.es
transfer3.bsc.es
transfer4.bsc.es
```

Yeni MN5 konfigürasyonunda herhangi bir login node'undan tüm partition'lara iş gönderilebilir; bu nedenle `glogin1` veya `alogin1` arasındaki tercih günlük kullanım için çoğunlukla fark yaratmaz. Büyük dosya transferlerinde ise mutlaka `transfer` node'larını kullanın.

### 3.2 İlk Bağlantı ve Şifre Değişikliği

```bash
# İlk bağlantı
ssh {kullanıcıadı}@alogin1.bsc.es
```

> **Önemli:** BSC ilk şifrenizi e-posta ile gönderir. **Güvenlik gerekçesiyle ilk şifreyi hemen değiştirmeniz zorunludur.** Şifre değişikliği **transfer node** üzerinden yapılır:

```bash
ssh {kullanıcıadı}@transfer1.bsc.es
transfer1$ passwd
```

Yeni şifre yaklaşık **5 dakika** içinde tüm sistemde aktif olur.

> **Kritik Kural:** Kullanıcı adınız kişisel ve devredilemezdir. Başkasıyla paylaşmak politika ihlalidir. Projenize başka kullanıcı eklenmesi gerekiyorsa proje yöneticisi BSC'ye ayrı başvuru yapmalıdır.

### 3.3 SSH Anahtar Yönetimi (Key-Based Auth)

Her oturum açışta şifre girmek yerine SSH anahtar çifti kullanabilirsiniz. Bu hem daha güvenli hem de pratik bir çözümdür:

```bash
# Yerel makinenizde anahtar çifti oluşturun (zaten yoksa)
ssh-keygen -t ed25519 -C "mn5-key"

# Public key'i MN5'e kopyalayın
ssh-copy-id -i ~/.ssh/id_ed25519.pub {kullanıcıadı}@alogin1.bsc.es

# Artık şifresiz bağlanabilirsiniz
ssh {kullanıcıadı}@alogin1.bsc.es
```

`~/.ssh/config` dosyasına kısa takma ad tanımlayabilirsiniz:

```
Host mn5
    HostName alogin1.bsc.es
    User {kullanıcıadı}
    IdentityFile ~/.ssh/id_ed25519
```

Bu tanımdan sonra `ssh mn5` komutu yeterli olur.

### 3.4 İnternete Çıkış Kısıtlaması

MN5'in compute ve login node'larından dışarıya çıkış (outbound internet) **mevcut değildir**. Bu çok önemli bir kısıtlamadır:

- Pip install, conda install, wget, curl ile internet erişimi **login node'larında çalışmaz**
- HuggingFace, GitHub gibi kaynaklardan doğrudan indirme yapılamaz
- Tüm dosya transferleri **yerel makinenizden** başlatılmalıdır

**İstisna:** `login4` node'ları (`glogin4.bsc.es`, `alogin4.bsc.es`) internet erişimine sahiptir ancak yalnızca BSC personeli ve VPN erişimine sahip kullanıcılar bu node'lara bağlanabilir. Dış kullanıcılar için bu yol genellikle erişilebilir değildir.

Sonuç olarak, modeller ve Python paketleri gibi büyük dosyalar için şu strateji kullanılır: önce yerel makinede indir, sonra `rsync` ile MN5'e aktar.

### 3.5 SSH Tüneli ile GPU Node'larına Tarayıcı Erişimi

SLURM job'ları çalışırken GPU node'ları üzerinde açılan HTTP servislere (örneğin vLLM API, Jupyter Notebook, Grafana) yerel tarayıcıdan ulaşmak mümkündür. Login node, compute node'lara ulaşabildiğinden SSH port forwarding ile bir tünel kurulabilir:

```bash
# Yerel makineden:
# -L <yerel_port>:<compute_node_hostname>:<uzak_port>
ssh -L 8000:as01r5b08:8000 {kullanıcıadı}@alogin1.bsc.es
```

Bu komutu çalıştırdıktan sonra yerel tarayıcınızdan `http://localhost:8000` adresini açarak compute node'daki servise ulaşabilirsiniz. Node adını SLURM job log'larından (`squeue` veya job çıktı dosyasından) alırsınız.

Birden fazla port yönlendirmek için:
```bash
ssh -L 8000:NODE:8000 -L 9000:NODE:9000 {kullanıcıadı}@alogin1.bsc.es
```

> **Not:** Bu yaklaşım yalnızca job çalışırken geçerlidir. Job bittiğinde bağlantı kesilir.

### 3.6 SSHFS ile Uzak Dizini Yerel Olarak Mount Etme

`sshfs` aracıyla MN5'teki bir dizini sanki yerel diskmiş gibi kullanabilirsiniz:

```bash
# Yerel makinede:
mkdir -p ~/mn5-mount
sshfs {kullanıcıadı}@transfer1.bsc.es:/gpfs/projects/ehpc648 ~/mn5-mount

# İşiniz bitince unmount edin
fusermount -u ~/mn5-mount   # Linux
umount ~/mn5-mount          # macOS
```

Bu yöntem küçük dosyalar üzerinde çalışmak için uygun olsa da büyük dosya transferlerinde (`rsync` yerine) önerilmez; bağlantı koptuğunda I/O operasyonları askıda kalabilir. Kendi deneyimimde büyük model dosyaları için doğrudan `rsync` kullanmayı tercih ettim.

---

## 4. Depolama Alanları ve Dizin Yapısı

MN5'te üç temel GPFS depolama alanı ve bir yerel SSD seçeneği mevcuttur. Her birinin kullanım amacı, performansı ve politikaları birbirinden farklıdır.

### 4.1 Depolama Türleri Özeti

| Alan | Yol | Amaç | Backup | Performans |
|---|---|---|---|---|
| Home | `/gpfs/home/<GRUP>/$USER` | Küçük kalıcı dosyalar | Evet (~1 gün) | Düşük |
| Projects | `/gpfs/projects/<GRUP>` | Proje çalışma alanı | Evet (3-4 gün) | Yüksek |
| Scratch | `/gpfs/scratch/<GRUP>` | Geçici, I/O yoğun | **Hayır** | Çok yüksek |
| Local SSD | `$TMPDIR` | Job süresi boyunca | **Hayır** | En yüksek |
| Tapes | `/gpfs/tapes/hpc/<GRUP>` | Uzun vadeli arşiv | Hayır (kullanıcı sorumlu) | Çok düşük (latency) |

### 4.2 /gpfs/home — Kişisel Home Dizini

Login olduğunuzda otomatik olarak buraya düşersiniz. Küçük, kalıcı dosyalar için uygundur:
- `.bashrc`, `.ssh/`, config dosyaları
- Küçük script'ler, notlar
- `git clone` ile çekilen repo metadata'sı

> **Uyarı:** Job'ları bu dizinden çalıştırmayın. BSC tarafından açıkça önerilmemektedir ve performans sorunlarına yol açar.

### 4.3 /gpfs/projects — Proje Çalışma Alanı

Gruba ait kalıcı depolama alanıdır. En sık kullanacağınız alan budur:

```
/gpfs/projects/<GRUP>/          ← group root
├── my-project/                 ← proje dizini
│   ├── containers/             ← Singularity .sif dosyaları
│   ├── models/                 ← LLM model ağırlıkları
│   ├── jobs/                   ← SLURM job script'leri
│   ├── scripts/                ← Python/bash script'ler
│   ├── logs/                   ← job çıktıları
│   ├── results/                ← analiz sonuçları
│   └── datasets/               ← veri setleri
```

Bu yapı hem düzenli kalmanızı sağlar hem de grup üyeleri arasında paylaşımı kolaylaştırır.

### 4.4 /gpfs/scratch — Yüksek Performanslı Geçici Alan

Job çalışırken üretilen büyük geçici veriler için idealdir. Performansı projeden çok daha yüksektir.

```bash
# Scratch dizinine symlink oluşturmak erişimi kolaylaştırır
ln -s /gpfs/scratch/ehpc648 ~/scratch
```

> **Kritik Uyarı:** Scratch'in backup'ı yoktur ve BSC zaman zaman temizleme yapabilir. Job bitiminde önemli sonuçları mutlaka `/gpfs/projects`'e kopyalayın.

Job script'lerinde scratch kullanımı:
```bash
SCRATCH=/gpfs/scratch/ehpc648/$USER
mkdir -p $SCRATCH/my_job_output
# ... iş yap ...
cp -r $SCRATCH/my_job_output /gpfs/projects/ehpc648/results/
```

### 4.5 $TMPDIR — Node'a Özel Yerel SSD

Her compute node'un 480-960 GB NVMe SSD'si vardır ve bu alan job süresi boyunca kullanılabilir:

```bash
echo $TMPDIR     # → /scratch/tmp/<JOBID>
ls $TMPDIR       # job başlangıcında boştur
```

Bu alan, özellikle çok fazla küçük dosya I/O'su gerektiren işlemler için (örneğin büyük dataset'in parça parça okunması) idealdir. Ancak **login node'larından görülmez** ve **job bitiminde otomatik silinir**.

### 4.6 Kota Kontrolü

```bash
bsc_quota        # tüm alanlar için kota durumu
```

Kota aşıldığında yeni dosya yazılamaz ve job'lar çökebilir. Düzenli kontrol alışkanlığı edinin.

---

## 5. Dosya Transferi

MN5'in outbound internet erişimi olmadığından **tüm dosya transferleri yerel makinenizden başlatılmalıdır**. Transfer node'larını kullanın.

### 5.1 scp ile Basit Transfer

```bash
# Yerel makineden MN5'e dosya göndermek:
scp -r "local_dir/" {user}@transfer1.bsc.es:"/gpfs/projects/ehpc648/my-project/"

# MN5'ten yerel makineye almak:
scp -r {user}@transfer1.bsc.es:"/gpfs/projects/ehpc648/results/" "local_results/"
```

`scp`, küçük/orta boyutlu tek seferlik transferler için yeterlidir. Büyük ve çok dosyalı transferlerde `rsync` daha sağlıklıdır.

### 5.2 rsync ile Verimli Senkronizasyon

`rsync` yalnızca değişen dosyaları aktarır, yarıda kesilirse kaldığı yerden devam eder.

```bash
# MN5'e senkronize et (ilerleme göster, checksum doğrula)
rsync -avz --progress \
  local_project/ \
  {user}@transfer1.bsc.es:/gpfs/projects/ehpc648/my-project/

# Büyük model dosyaları için (partial transfer desteği):
rsync -avz --partial --progress \
  models/ \
  {user}@transfer1.bsc.es:/gpfs/projects/ehpc648/models/
```

`-a` → archive mode (izinleri, zamanı korur)
`-v` → verbose
`-z` → sıkıştırma (LAN üzerinde pek fayda sağlamaz ama WAN için iyi)
`--partial` → yarıda kesilen transferleri tamamla

### 5.3 HuggingFace Modeli İndirme

MN5'in internet erişimi olmadığından, modelleri önce yerel makinenize indirip sonra rsync ile aktarmanız gerekir. Alternatif olarak, login node üzerinde bir Python venv kurup HuggingFace Hub client'ı kullanabilirsiniz — ancak bu yalnızca login node'da internete çıkabiliyorsanız (BSC VPN/login4 erişimi) mümkündür.

**Standart yöntem (yerel → rsync):**

```bash
# Yerel makinede Python kurulu olduğunu varsayarak:
pip install huggingface_hub

python3 - <<'EOF'
from huggingface_hub import snapshot_download

# Modeli yerel bir dizine indir
snapshot_download(
    repo_id="google/gemma-3-4b-it",
    local_dir="./models/gemma-3-4b-it",
    token="hf_XXXX"  # gated model için gerekli
)
EOF

# Sonra MN5'e aktar:
rsync -avz --partial --progress \
  ./models/ \
  {user}@transfer1.bsc.es:/gpfs/projects/ehpc648/models/
```

**Model dizin yapısı önerisi:**
```
/gpfs/projects/ehpc648/models/
├── google/
│   ├── gemma-3-4b-it/
│   └── gemma-2-9b/
├── Mistral-7B-Instruct-v0.3/
└── meta-llama/
    └── Llama-3.1-8B-Instruct/
```

---

## 6. Modül Sistemi

MN5, tüm yazılım bağımlılıklarını "Environment Modules" sistemi ile yönetir. Yüklediğiniz her modül gerekli `PATH`, `LD_LIBRARY_PATH`, `PYTHONPATH` gibi değişkenleri otomatik olarak ayarlar.

### 6.1 Temel Komutlar

```bash
# Mevcut tüm modülleri listele
module avail

# Belirli bir modülü ara
module avail python
module avail singularity

# Modül yükle
module load singularity/3.11.5

# Versiyon belirterek yükle
module load python/3.11.5-gcc

# Yüklü modülleri gör
module list

# Belirli bir modülü kaldır
module unload python

# TÜM yüklü modülleri temizle (en güvenli başlangıç noktası)
module purge

# İki modülü takas et
module switch singularity/3.11.5 singularity/4.1.5
```

### 6.2 En Önemli Kural: module purge

Job script'lerinizin başında her zaman `module purge` kullanın. Login sırasında otomatik yüklenen bazı modüller (örneğin `python`, `gcc`) beklenmedik ortam değişkenlerini set eder. Bu değişkenler özellikle Singularity container'larına sızarak Python ortamını bozabilir.

```bash
#!/bin/bash
#SBATCH ...

# DOĞRU: Her zaman temiz başla
module purge
module load singularity/3.11.5

# YANLIŞ: module purge olmadan devam etmek
# module load singularity/3.11.5  ← önceki session'dan kalan modüller hâlâ yüklü olabilir
```

### 6.3 ACC Partition için Önerilen Modül Stack'i

GPU tabanlı işler için aşağıdaki stack iyi bir başlangıç noktasıdır:

```bash
module purge
module load gcc/11.4.0
module load nvidia-hpc-sdk/23.11-cuda11.8
module load python/3.11.5-gcc
module load singularity/3.11.5
```

PyTorch doğrudan kullanmak istiyorsanız:
```bash
module purge
module load pytorch/2.4.0   # CUDA 11.x ile birlikte gelir
```

### 6.4 Default Modüller

Login olduğunuzda bazı modüller otomatik yüklenir. Bu davranışı devre dışı bırakmak isterseniz:
```bash
touch ~/.avoid_load_def_modules.mn5
```

Bu dosya bulunduğunda login sırasında otomatik modül yüklenmez.

### 6.5 Compiler Environments

Kendi yazılımınızı derlemeniz gerekiyorsa, birbiriyle uyumlu derleyici + kütüphane setlerini içeren "compiler environments" kullanın:

```bash
# ACC partition için:
module load compenv-acc/nvhpx23.11    # NVIDIA HPC SDK tabanlı
module load compenv-acc/gnu13.2.0-ompi   # GCC + OpenMPI

# GPP partition için:
module load compenv-gpp/intel2023-impi
module load compenv-gpp/gnu12.3.0-ompi
```

---

## 7. Python Paket Yönetimi

### 7.1 Durum: İnternet Erişimi Yok

MN5'in standard login node'larından internet erişimi olmadığını hatırlayalım. Bu, `pip install somepackage` komutunun çalışmadığı anlamına gelir. Bunun yerine aşağıdaki stratejiler kullanılır.

### 7.2 pip ile Paket Kurulumu

**A) Offline kurulum (wheel dosyası ile):**

```bash
# Yerel makinede:
pip download somepackage -d ./wheels/

# MN5'e aktar:
rsync -avz wheels/ {user}@transfer1.bsc.es:/gpfs/projects/ehpc648/wheels/

# MN5 login node'unda:
module purge && module load python/3.11.5-gcc
python3 -m pip install --no-index --find-links=/gpfs/projects/ehpc648/wheels/ somepackage
```

**B) Tar.gz kaynak kodu ile:**

```bash
# Yerel makinede:
pip download somepackage --no-binary :all: -d ./src_wheels/

# MN5'te:
tar xzf somepackage-1.0.0.tar.gz
cd somepackage-1.0.0
python setup.py install --user
# veya
pip install . --user
```

**C) --user flag'i ile kullanıcı dizinine kurulum:**

```bash
python3 -m pip install --user SomePackage
```

Bu kurulum `~/.local/lib/python3.x/site-packages/` altına yapılır.

**D) PYTHONUSERBASE ile özel konuma kurulum:**

```bash
export PYTHONUSERBASE=/gpfs/projects/ehpc648/python-env
python3 -m pip install --user SomePackage
```

Bu yaklaşım, paketleri proje dizininde grupla paylaşmak için kullanışlıdır.

### 7.3 Virtual Environment Kullanımı

Proje başına izole Python ortamı için virtual environment önerilir:

```bash
module purge && module load python/3.11.5-gcc

# venv oluştur (projects dizininde)
python3 -m venv /gpfs/projects/ehpc648/my-project/venv

# Aktive et
source /gpfs/projects/ehpc648/my-project/venv/bin/activate

# Paket kur (offline wheel'lardan)
pip install --no-index --find-links=/path/to/wheels/ numpy pandas

# Deactivate
deactivate
```

### 7.4 Conda ile Paket Yönetimi

Conda, MN5'te `miniconda3` modülü aracılığıyla kullanılabilir.

> **Uyarı:** Anaconda, BSC'ye karşı kurumsal düzeyde erişim kısıtlaması uygulamaktadır. `defaults` kanalı çalışmaz. İlk kurulumdan önce bu kanalı kaldırın:

```bash
module load miniconda3

# ZORUNLU: defaults kanalını kaldır
conda config --remove channels defaults

# Alternatif kanallar ekle
conda config --add channels conda-forge
conda config --add channels bioconda
```

Conda environment oluşturma:
```bash
# Özel konumda oluştur (home'da kota sıkıntısı olabilir)
conda create --prefix /gpfs/projects/ehpc648/my-project/conda-env python=3.11

# Aktive et
conda activate /gpfs/projects/ehpc648/my-project/conda-env

# Paket kur
conda install numpy pytorch

# Deactivate
conda deactivate
```

### 7.5 Singularity Kullanımı Önerisi

Karmaşık bağımlılıklar (PyTorch + CUDA + transformers gibi) için en temiz ve tekrar üretilebilir yaklaşım, tüm bağımlılıkları bir Singularity container içine koymaktır. Bu yöntem bir sonraki bölümde anlatılmaktadır.

---

## 8. Container Ortamları: Singularity / Apptainer

MN5'te kullanıcılar kendi özelleştirilmiş yazılım ortamlarını **Singularity (Apptainer)** container'ları aracılığıyla çalıştırabilir. Docker doğrudan desteklenmez çünkü Docker root erişimi gerektirir ve HPC ortamlarında güvenlik riski oluşturur. Singularity ise root'suz çalışır, GPFS üzerinden her node'dan erişilebilir ve SLURM ile sorunsuz entegre olur.

### 8.1 Singularity Modülü

```bash
module purge
module load singularity/3.11.5
# veya
module load singularity/4.1.5

singularity --version
```

### 8.2 Docker Image'den SIF Dosyası Oluşturma

Kendi Docker image'inizi yerel makinenizde SIF formatına dönüştürebilirsiniz:

```bash
# Yerel makinede (Docker kurulu olmalı):
docker pull myimage:latest
docker save myimage:latest | gzip > myimage.tar.gz

# singularity build ile SIF'e çevir
singularity build myimage.sif docker-archive://myimage.tar.gz

# Alternatif: Docker Hub'dan doğrudan build
singularity build myimage.sif docker://ubuntu:22.04

# SIF'i MN5'e aktar
rsync -avz --partial myimage.sif {user}@transfer1.bsc.es:/gpfs/projects/ehpc648/containers/
```

### 8.3 Singularity ile Container Çalıştırma

```bash
# Basit komut çalıştırma
singularity exec myimage.sif python3 --version

# GPU erişimiyle (--nv flag'i CUDA kütüphanelerini bind eder)
singularity exec --nv myimage.sif python3 -c "import torch; print(torch.cuda.is_available())"

# GPFS dizinlerini bind etme
singularity exec --nv \
  --bind /gpfs/projects/ehpc648:/data \
  myimage.sif python3 /data/my_script.py

# Container içinde interaktif shell
singularity shell --nv myimage.sif
```

### 8.4 BSC AI Hub — Hazır Container Image'leri

BSC, en güncel AI framework'lerini içeren container image'leri hazır olarak sağlamaktadır. Bunlar **paylaşımlı bir dizinde symlink** olarak bulunur ve tüm kullanıcılar tarafından kullanılabilir:

```
/gpfs/scratch/shared/ai-hub/singularity_images/vllm/
```

Mevcut image'ler (Nisan 2026 itibarıyla):

| Image | İçerik |
|---|---|
| `vllm-v0.18.0-official.sif` | vLLM 0.18.0, transformers 4.57.6, Python 3.12, CUDA 12.x |
| `pytorch_25.12-py3.sif` | PyTorch 2.x, CUDA 12.x |
| `nemo_25.02.sif` | NVIDIA NeMo framework |
| `tensorrt-llm-v10.1.0.sif` | TensorRT-LLM |
| `tritonserver_25.02-trtllm-python-py3.sif` | NVIDIA Triton Inference Server |
| `sglang-dev-0.5.6.post1.sif` | SGLang LLM serving |

> **Önerimiz:** Kendi image'inizi sıfırdan derlemek yerine BSC'nin sağladığı bu image'leri kullanın. Sürekli güncel tutulurlar ve CUDA sürüm uyumluluğu BSC tarafından garanti edilir. Deneyimimizde DIY image'deki `transformers==4.46.3` Gemma 3'ü desteklemiyorken BSC'nin `vllm-v0.18.0` image'indeki `transformers==4.57.6` ile sorun çözüldü.

### 8.5 Önemli: PYTHONHOME/PYTHONPATH Sızması

`module load python` veya conda aktifleştirmesi yapıldıktan sonra Singularity container çalıştırırsanız, `PYTHONHOME` ve `PYTHONPATH` değişkenleri container içine sızar ve container'ın Python ortamını bozar. Bu çok sık karşılaşılan bir sorundur.

**Çözüm: Her zaman `module purge` ile başlayın:**

```bash
module purge
module load singularity/3.11.5
# module load python ← YAPMAYIN
singularity exec --nv myimage.sif python3 my_script.py
```

Container içine kontrollü biçimde değişken aktarmak için `SINGULARITYENV_*` prefix'ini kullanın:

```bash
export SINGULARITYENV_MY_VAR="my_value"
singularity exec myimage.sif env | grep MY_VAR
```

---

## 9. SLURM ile İş Yönetimi

MN5'te tüm hesaplama işleri SLURM iş planlayıcısı üzerinden gönderilir. Login node'larında doğrudan ağır hesaplama yapmak yasaktır ve sistem yöneticileri tarafından izlenir.

### 9.1 Temel SLURM Komutları

```bash
# Job gönder
sbatch my_job.slurm

# Kuyruktaki job'ları gör
squeue -u $USER

# Belirli proje/account için:
squeue -A ehpc648

# Job iptal et
scancel <JOBID>

# Job detaylarını gör
scontrol show job <JOBID>

# Çalışan job'un çıktısını izle
tail -f logs/my_job-<JOBID>.out

# Erişebildiğin account'ları gör
bsc_project list

# Erişebildiğin queue'ları gör
bsc_queues

# Kota kontrol
bsc_quota

# Partition durumu
sinfo -p acc
```

### 9.2 Zorunlu SBATCH Direktifleri

MN5'in yeni konfigürasyonunda `--account` ve `--qos` direktifleri **zorunludur**. Bunlar olmadan job reddedilir:

```
salloc: error: No account specified, please specify an account
salloc: error: No QoS specified, please specify a QoS
```

### 9.3 ACC Partition Kuyrukları

| Kuyruk | Slurm QoS | Max Node | Wallclock | Açıklama |
|---|---|---|---|---|
| EuroHPC | `acc_ehpc` | 100 node (8,000 core) | 72 saat | EuroHPC tahsisi için |
| Debug | `acc_debug` | 8 node (640 core) | 2 saat | Hızlı test ve geliştirme |
| Interactive | `acc_interactive` | 1 node (40 core) | 2 saat | İnteraktif oturum |
| Training | `acc_training` | 4 node (320 core) | 48 saat | Eğitim amaçlı |
| RES A | `acc_resa` | 100 node | 72 saat | |
| RES B | `acc_resb` | 100 node | 48 saat | |
| RES C | `acc_resc` | 10 node | 24 saat | |

### 9.4 Job Script Anatomisi

Bir SLURM job script'i `#SBATCH` direktifleri ile başlar, ardından çalıştırılacak komutlar gelir:

```bash
#!/bin/bash
#SBATCH --job-name=my_job         # İş adı (squeue'da görünür)
#SBATCH --account=ehpc648         # Proje/account (ZORUNLU)
#SBATCH --qos=acc_debug           # Queue (ZORUNLU)
#SBATCH --partition=acc           # Partition (genellikle otomatik, ama belirtmek iyi pratik)
#SBATCH --nodes=1                 # Node sayısı
#SBATCH --ntasks-per-node=1       # Node başına task sayısı
#SBATCH --cpus-per-task=20        # Task başına CPU
#SBATCH --gres=gpu:1              # GPU sayısı
#SBATCH --time=00:30:00           # HH:MM:SS maksimum süre
#SBATCH --output=logs/%x-%j.out   # Standart çıktı dosyası (%x=job adı, %j=job ID)
#SBATCH --error=logs/%x-%j.err    # Hata çıktısı

# Ortam hazırlığı
module purge
module load singularity/3.11.5

# İş
echo "Running on node: $(hostname)"
echo "Job ID: $SLURM_JOB_ID"
nvidia-smi

singularity exec --nv /path/to/image.sif python3 my_script.py
```

### 9.5 Örnek 1: GPU Doğrulama (Smoke Test)

Sisteme alışmak için başlangıçta kullanışlı bir basit test:

```bash
#!/bin/bash
#SBATCH --job-name=gpu-smoke
#SBATCH --account=ehpc648
#SBATCH --qos=acc_debug
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=20
#SBATCH --gres=gpu:1
#SBATCH --time=00:05:00
#SBATCH --output=logs/gpu-smoke-%j.out

module purge
module load nvidia-hpc-sdk/23.11-cuda11.8

echo "=== Node Info ==="
hostname
nvidia-smi

echo "=== GPU Count ==="
nvidia-smi --query-gpu=name,memory.total --format=csv,noheader
```

### 9.6 Örnek 2: Python Script'i GPU Üzerinde Çalıştırma

```bash
#!/bin/bash
#SBATCH --job-name=pytorch-test
#SBATCH --account=ehpc648
#SBATCH --qos=acc_debug
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=20
#SBATCH --gres=gpu:1
#SBATCH --time=00:15:00
#SBATCH --output=logs/pytorch-%j.out

module purge
module load singularity/3.11.5

# Ortam değişkenlerini ayarla
export CUDA_VISIBLE_DEVICES=0
export SINGULARITYENV_CUDA_VISIBLE_DEVICES=0

singularity exec --nv \
  /gpfs/scratch/shared/ai-hub/singularity_images/vllm/vllm-v0.18.0-official.sif \
  python3 - <<'EOF'
import torch
print(f"CUDA available: {torch.cuda.is_available()}")
print(f"GPU count: {torch.cuda.device_count()}")
print(f"GPU name: {torch.cuda.get_device_name(0)}")
x = torch.randn(4096, 4096).cuda()
y = torch.matmul(x, x.T)
print(f"Matrix multiply (4096x4096): DONE")
EOF
```

### 9.7 İnteraktif Oturum

Geliştirme ve hata ayıklama için compute node'da interaktif terminal açabilirsiniz:

```bash
salloc --account=ehpc648 --qos=acc_interactive \
       --nodes=1 --gres=gpu:1 --cpus-per-task=20 \
       --time=02:00:00

# Tahsis alındığında:
# salloc: Granted job allocation XXXXX
# Şimdi compute node'dasınız

nvidia-smi   # GPU'yu doğrula
module load singularity/3.11.5
singularity shell --nv /path/to/image.sif

# Çıkmak için:
exit
```

### 9.8 SLURM Ortam Değişkenleri

Job içinden erişilebilir faydalı değişkenler:

| Değişken | Açıklama |
|---|---|
| `$SLURM_JOB_ID` | Job ID'si |
| `$SLURM_NODELIST` | Tahsis edilen node listesi |
| `$SLURM_LOCALID` | Task'ın node içindeki sırası (0, 1, 2, 3) |
| `$SLURM_NTASKS` | Toplam task sayısı |
| `$SLURM_CPUS_PER_TASK` | Task başına CPU sayısı |
| `$TMPDIR` | Yerel SSD geçici dizin |

---

## 10. Paralel ve Çok Görevli İşler

### 10.1 srun ile Çoklu Task

```bash
#!/bin/bash
#SBATCH --job-name=multi-task
#SBATCH --account=ehpc648
#SBATCH --qos=acc_debug
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=4        # 4 task
#SBATCH --cpus-per-task=20         # her task 20 CPU
#SBATCH --gres=gpu:4               # 4 GPU
#SBATCH --time=00:30:00
#SBATCH --output=logs/multi-%j.out

module purge
module load singularity/3.11.5

# srun ile 4 paralel task başlat
# Her task kendi GPU'sunu (SLURM_LOCALID) alır
srun --ntasks=4 --gpus-per-task=1 bash -c '
  GPU_ID=$SLURM_LOCALID
  PORT=$((8000 + SLURM_LOCALID))
  echo "Task $SLURM_LOCALID: GPU=$GPU_ID, PORT=$PORT on $(hostname)"

  export CUDA_VISIBLE_DEVICES=0
  export SINGULARITYENV_CUDA_VISIBLE_DEVICES=0

  singularity exec --nv /path/to/image.sif \
    python3 my_worker.py --port $PORT
'
```

> **MN5'e Özgü Kritik Not:** `--gpu-bind=single:1` direktifi Singularity container'larının içine yansımaz. SLURM'un cgroup tabanlı GPU izolasyonu her container'a fiziksel GPU'yu NVML index 0 olarak gösterir. Bu nedenle multi-task Singularity job'larında her zaman şunu kullanın:
> ```bash
> export CUDA_VISIBLE_DEVICES=0
> export SINGULARITYENV_CUDA_VISIBLE_DEVICES=0
> ```
> Fiziksel GPU izolasyonu SLURM'a bırakılır; container her zaman kendi GPU'sunu "GPU 0" olarak görür.

### 10.2 SLURM_CPU_BIND Problemi

MPI tabanlı işlerde (mpirun ile) varsayılan CPU binding NVIDIA HPC SDK ile çakışır:

```bash
# MPI kullanan job'larda ZORUNLU:
export SLURM_CPU_BIND=none
```

`srun` ile OpenMP/multi-thread işlerde ise `--cpus-per-task`'ı her zaman açıkça belirtin:

```bash
srun --cpus-per-task=$SLURM_CPUS_PER_TASK ./my_program
# veya
export SRUN_CPUS_PER_TASK=$SLURM_CPUS_PER_TASK
srun ./my_program
```

### 10.3 Array Job'ları

Benzer parametrelerle çok sayıda bağımsız iş çalıştırmak için array job kullanımı oldukça verimlidir. Örneğin 10 farklı seed ile aynı deneyi çalıştırmak:

```bash
#!/bin/bash
#SBATCH --job-name=array-exp
#SBATCH --account=ehpc648
#SBATCH --qos=acc_ehpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --gres=gpu:1
#SBATCH --cpus-per-task=20
#SBATCH --time=02:00:00
#SBATCH --array=0-9                # 10 job (index 0'dan 9'a)
#SBATCH --output=logs/array-%A-%a.out   # %A=array ID, %a=task index

module purge
module load singularity/3.11.5

SEED=$SLURM_ARRAY_TASK_ID

echo "Running experiment with seed=$SEED"

singularity exec --nv /path/to/image.sif \
  python3 run_experiment.py --seed $SEED --output results/seed_$SEED.json
```

Array job'ları yönetmek için:
```bash
# Gönder
sbatch my_array_job.slurm

# Tüm array task'larını gör
squeue -u $USER

# Sadece belirli task'ı iptal et
scancel <JOBID>_3    # 3. indeksi iptal

# Tüm array'i iptal et
scancel <JOBID>
```

### 10.4 Multi-Node Job

Birden fazla node gerektiğinde:

```bash
#!/bin/bash
#SBATCH --job-name=multi-node
#SBATCH --account=ehpc648
#SBATCH --qos=acc_ehpc
#SBATCH --nodes=2                  # 2 node
#SBATCH --ntasks-per-node=4        # Her node'da 4 task
#SBATCH --gres=gpu:4               # Her node'da 4 GPU
#SBATCH --cpus-per-task=20
#SBATCH --time=01:00:00
#SBATCH --output=logs/multinode-%j.out

module purge
module load singularity/3.11.5

# Node listesini al
echo "Nodes: $SLURM_NODELIST"

# Her node'daki task'ları başlat
srun --ntasks=8 --gpus-per-task=1 bash -c '
  export CUDA_VISIBLE_DEVICES=0
  export SINGULARITYENV_CUDA_VISIBLE_DEVICES=0

  NODE=$(hostname)
  PORT=$((8000 + SLURM_LOCALID))
  echo "Task $SLURM_PROCID: $NODE:$PORT"
  singularity exec --nv /path/to/image.sif python3 worker.py --port $PORT
'
```

---

## 11. Yapay Zeka ve LLM Servisleri

Bu bölüm, MN5 ACC partition'ında LLM inference servisleri kurmak ve bir orchestrator arkasında birden fazla modeli yönetmek isteyen kullanıcılar için hazırlanmıştır. Örnekler bizzat çalıştırılmış ve doğrulanmıştır.

### 11.1 Genel Mimari

MN5 üzerinde LLM servisleri için önerilen mimari şu şekildedir:

```
                    HTTP Client (login node / siz)
                           │
                           ▼
                  ┌─────────────────┐
                  │  Orchestrator   │  port 9000
                  │  (Model Router) │
                  └────────┬────────┘
                           │
           ┌───────────────┼───────────────┐
           ▼               ▼               ▼
     ┌──────────┐    ┌──────────┐    ┌──────────┐
     │  GPU 0   │    │  GPU 1   │    │  GPU 2   │
     │ Model A  │    │ Model B  │    │ Model C  │
     │ :8000    │    │ :8001    │    │ :8002    │
     └──────────┘    └──────────┘    └──────────┘
```

Her GPU'da ayrı bir vLLM instance çalışır ve kendi portunu dinler. Orchestrator, gelen isteği model adına göre doğru backend'e yönlendirir.

### 11.2 vLLM ile Tek GPU Inference

```bash
#!/bin/bash
#SBATCH --job-name=vllm-single
#SBATCH --account=ehpc648
#SBATCH --qos=acc_debug
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --gres=gpu:1
#SBATCH --cpus-per-task=20
#SBATCH --time=01:00:00
#SBATCH --output=logs/vllm-single-%j.out

module purge
module load singularity/3.11.5

CONTAINER=/gpfs/scratch/shared/ai-hub/singularity_images/vllm/vllm-v0.18.0-official.sif
MODEL=/gpfs/projects/ehpc648/models/Mistral-7B-Instruct-v0.3
PORT=8000

export CUDA_VISIBLE_DEVICES=0
export SINGULARITYENV_CUDA_VISIBLE_DEVICES=0
export SINGULARITYENV_HF_HOME=/gpfs/projects/ehpc648/cache

echo "=== Starting vLLM on $(hostname):$PORT ==="

singularity exec --nv \
  --bind /gpfs/projects/ehpc648:/workspace \
  $CONTAINER \
  python3 -m vllm.entrypoints.openai.api_server \
    --model $MODEL \
    --served-model-name "mistral" \
    --host 0.0.0.0 \
    --port $PORT \
    --tensor-parallel-size 1 \
    --gpu-memory-utilization 0.90 \
    --max-model-len 4096 &

# Model yüklenene kadar bekle (polling loop — sabit sleep kullanmayın!)
echo "Waiting for backend to be ready..."
for i in $(seq 1 18); do
  if curl -s http://localhost:$PORT/health | grep -q "ok"; then
    echo "Backend ready after $((i * 10)) seconds."
    break
  fi
  echo "  ... waiting ($((i * 10))s)"
  sleep 10
done

echo "Node: $(hostname)"
echo "API endpoint: http://$(hostname):$PORT/v1"

# Test isteği
curl http://localhost:$PORT/v1/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "mistral",
    "prompt": "What is privacy-preserving AI?",
    "max_tokens": 100
  }'

# Servisi ayakta tut (manuel test için)
wait
```

### 11.3 4 GPU, 4 Farklı Model — Orchestrator ile

Bu senaryo, aynı node üzerindeki 4 GPU'ya 4 farklı LLM yerleştirip bunları model alias'ı bazında yönlendiren bir orchestrator kurulumudur.

```bash
#!/bin/bash
#SBATCH --job-name=vllm-4model
#SBATCH --account=ehpc648
#SBATCH --qos=acc_ehpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=4
#SBATCH --gres=gpu:4
#SBATCH --cpus-per-task=20
#SBATCH --time=02:00:00
#SBATCH --output=logs/vllm-4model-%j.out

module purge
module load singularity/3.11.5

CONTAINER=/gpfs/scratch/shared/ai-hub/singularity_images/vllm/vllm-v0.18.0-official.sif
MODEL_BASE=/gpfs/projects/ehpc648/models
export SLURM_CPU_BIND=none

# GPU-Model eşleşmesi (SLURM_LOCALID → Model)
declare -A MODELS=(
  [0]="google/gemma-3-4b-it gemma3"
  [1]="Mistral-7B-Instruct-v0.3 mistral"
  [2]="Llama-3.1-8B-Instruct llama3"
  [3]="google/gemma-2-9b gemma2"
)

# 4 vLLM instance'ı paralel başlat
srun --ntasks=4 --gpus-per-task=1 bash -c '
  read MODEL_PATH MODEL_ALIAS <<< "${MODELS[$SLURM_LOCALID]}"
  PORT=$((8000 + SLURM_LOCALID))

  export CUDA_VISIBLE_DEVICES=0
  export SINGULARITYENV_CUDA_VISIBLE_DEVICES=0
  export SINGULARITYENV_HF_HOME=/gpfs/projects/ehpc648/cache

  echo "[Task $SLURM_LOCALID] Starting $MODEL_ALIAS on GPU $SLURM_LOCALID → port $PORT"

  singularity exec --nv \
    --bind /gpfs/projects/ehpc648:/workspace \
    '"$CONTAINER"' \
    python3 -m vllm.entrypoints.openai.api_server \
      --model /workspace/models/$MODEL_PATH \
      --served-model-name "$MODEL_ALIAS" \
      --host 0.0.0.0 \
      --port $PORT \
      --tensor-parallel-size 1 \
      --gpu-memory-utilization 0.90 \
      --max-model-len 4096 &
  wait
' &

# Tüm backend'ler hazır olana kadar polling
NODE=$(hostname)
ALL_READY=0
for i in $(seq 1 18); do
  COUNT=0
  for port in 8000 8001 8002 8003; do
    curl -s http://localhost:$port/health | grep -q "ok" && COUNT=$((COUNT+1))
  done
  echo "Ready backends: $COUNT/4 ($((i*10))s)"
  [ $COUNT -eq 4 ] && ALL_READY=1 && break
  sleep 10
done

if [ $ALL_READY -eq 1 ]; then
  echo ""
  echo "=== ALL MODELS READY ==="
  echo "Node: $NODE"
  echo "  gemma3  → http://$NODE:8000/v1"
  echo "  mistral → http://$NODE:8001/v1"
  echo "  llama3  → http://$NODE:8002/v1"
  echo "  gemma2  → http://$NODE:8003/v1"
fi

wait
```

### 11.4 Tensor Parallelism — Tek Büyük Model, 4 GPU

Daha büyük modeller (13B+) için tüm GPU'ları tek bir inference instance'ına tahsis edebilirsiniz:

```bash
#!/bin/bash
#SBATCH --job-name=vllm-tp4
#SBATCH --account=ehpc648
#SBATCH --qos=acc_debug
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --gres=gpu:4
#SBATCH --cpus-per-task=80
#SBATCH --time=01:00:00
#SBATCH --output=logs/vllm-tp4-%j.out

module purge
module load singularity/3.11.5

CONTAINER=/gpfs/scratch/shared/ai-hub/singularity_images/vllm/vllm-v0.18.0-official.sif

export CUDA_VISIBLE_DEVICES=0,1,2,3
export SINGULARITYENV_CUDA_VISIBLE_DEVICES=0,1,2,3

singularity exec --nv \
  --bind /gpfs/projects/ehpc648:/workspace \
  $CONTAINER \
  python3 -m vllm.entrypoints.openai.api_server \
    --model /workspace/models/google/gemma-2-9b \
    --served-model-name "gemma2-tp4" \
    --host 0.0.0.0 \
    --port 8000 \
    --tensor-parallel-size 4 \
    --gpu-memory-utilization 0.90 \
    --max-model-len 8192
```

### 11.5 Inference Testi — curl ile

```bash
NODE=as01r5b08  # squeue veya job log'undan alın

# Doğrudan modele
curl http://${NODE}:8001/v1/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "mistral", "prompt": "Explain differential privacy.", "max_tokens": 150}'

# OpenAI-uyumlu chat formatı
curl http://${NODE}:9000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemma3",
    "messages": [
      {"role": "system", "content": "You are a privacy expert."},
      {"role": "user", "content": "What is GDPR?"}
    ],
    "max_tokens": 200
  }'

# Health check
curl http://${NODE}:9000/health

# Model listesi
curl http://${NODE}:9000/v1/models
```

### 11.6 GPU Kullanımını İzleme

Job çalışırken GPU durumunu izlemek için:

```bash
# Job'unuz çalışan node'da (srun ile)
srun --jobid=<JOBID> nvidia-smi

# Veya interaktif oturumda:
watch -n 2 nvidia-smi
```

---

## 12. Güvenlik ve İzin Yönetimi

### 12.1 Dosya İzinleri

HPC ortamında dosya izinleri hem güvenlik hem de grup içi işbirliği açısından önemlidir. Unix izin modeli şu şekilde çalışır:

```
rwxrwxrwx
│││││││││
│││││││└┘ Others (diğer herkes)
│││└──┘   Group (proje grubu)
└──┘      Owner (siz)
```

**Önerilen izin stratejisi:**

```bash
# Script ve kod dosyaları: sahip rwx, grup rx, diğer 0
chmod 750 scripts/*.py

# Sonuç ve model dosyaları (grup ile paylaşmak için): sahip rwx, grup r-x, diğer 0
chmod -R 750 models/
chmod -R 750 results/

# Paylaşmak istediğiniz hazır modeller:
chmod -R 755 models/google/gemma-2-9b/  # herkese r+x

# Kişisel config ve key dosyaları: sadece sahip
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

**Pratik örnek:**

```bash
# Kendi projenizdeki tüm dosyaları düzenle
find /gpfs/projects/ehpc648/my-project -type f -exec chmod 640 {} \;
find /gpfs/projects/ehpc648/my-project -type d -exec chmod 750 {} \;

# Sadece modelleri herkesle paylaş
chmod -R 755 /gpfs/projects/ehpc648/models/
```

### 12.2 Credentials Yönetimi

HuggingFace token, API anahtarları gibi bilgileri job script'lerine yazmayın. Bunun yerine ortam değişkeni veya gizli dosya kullanın:

```bash
# ~/.hf_token dosyasına yaz (izinleri kısıtla)
echo "hf_XXXX" > ~/.hf_token
chmod 600 ~/.hf_token

# Job script'inde:
export HF_TOKEN=$(cat ~/.hf_token)
export SINGULARITYENV_HF_TOKEN=$HF_TOKEN

singularity exec --nv image.sif python3 download.py
```

### 12.3 SSH Güvenliği

- Public key authentication kullanın (parola auth yerine)
- `~/.ssh/config` dosyasında `ServerAliveInterval 60` ekleyin (idle bağlantı kopmaması için)
- Private key'inizi asla MN5'e kopyalamayın
- Login node'dan başka bir sistemin private key'ine erişmeyin

---

## 13. Veri, Etik ve Kullanım Politikaları

Bu bölüm önemlidir. EuroHPC kaynaklarının kullanımında hem BSC hem de AB/EuroHPC kurallarına uymak zorunludur.

### 13.1 Kullanıcı Hesabı Kuralları

BSC'nin resmi politikasından:

- Her kullanıcıya **kişisel ve devredilemez** bir kullanıcı adı atanır
- Kullanıcı adı başka biriyle **paylaşılamaz**
- Projeye yeni kullanıcı eklenmesi gerekiyorsa proje yöneticisi BSC'ye başvurur
- Tahsis edilen kaynaklar yalnızca **başvuruda belirtilen proje amaçları** için kullanılabilir

### 13.2 Kaynak Kullanım Politikası

EuroHPC tahsis kararı projenin bilimsel/teknik hedeflerine dayanır. Bu çerçevede:

- Kaynaklar ticari amaçlı üçüncü taraf hesaplaması için kullanılamaz
- Tahsis edilen kaynak miktarı ve kullanım süresi sınırlıdır; kullanılmayan saatler devredilmez
- Job'larınız diğer kullanıcıların kuyruğunu etkiler; gereksiz yüksek kaynak talebi yapmaktan kaçının
- Büyük wallclock süresi talepleri yerine ihtiyaç duyduğunuz kadar talep edin

**Kullanılan GPU saatini izlemek:**
```bash
bsc_quota        # depolama kotası
sbalance -a ehpc648   # kalan compute allocation
```

### 13.3 GDPR ve Kişisel Veri

Araştırmanızda kişisel veri işliyorsanız (kullanıcı verileri, sağlık verileri, iletişim kayıtları vb.) GDPR yükümlülükleri aynen geçerlidir:

- Kişisel veriyi MN5'e taşımadan önce hukuki dayanağınızın (rıza, meşru menfaat vb.) mevcut olduğundan emin olun
- Mümkünse veriyi anonimleştirin veya sözde-anonimleştirin
- Veri güvenliğini sağlamak için uygun dizin izinleri kullanın
- Projenizin veri işleme faaliyetlerini kurumunuzun DPO'suna bildirin

### 13.4 LLM Eğitimi ve Fine-Tuning

BSC, büyük dil modellerinin eğitimi ve fine-tuning için açık bir kısıtlama yayınlamamıştır; bu tür çalışmalar akademik ve araştırma amaçlı olarak yapılabilir. Ancak:

- Fine-tuning için kullanılan veri setinin lisansı uyumlu olmalıdır
- Kişisel veri içeren veri setleriyle fine-tuning yukarıdaki GDPR kurallarına tabidir
- Zararlı içerik üretmek üzere optimize edilmiş modeller kesinlikle yasaktır

### 13.5 Yayın ve Teşekkür

EuroHPC kaynaklarını kullanan çalışmalar yayımlanırken aşağıdaki türden bir teşekkür ifadesi eklenmelidir:

> *"This work was supported by EuroHPC Joint Undertaking and BSC (Barcelona Supercomputing Center) through the AI Factory Playground allocation [proje adı/kodu]."*

BSC, zaman zaman çalışmalarınızı vaka çalışması olarak kullanmak için izin isteyebilir.

### 13.6 Tahsis Süresi ve Uzatma

EuroHPC Playground tahsisleri genellikle 6-12 aylık dönemler için verilir. Süre sonunda:
- Kaynaklar devre dışı kalır
- `/gpfs/scratch` içeriği silinir
- `/gpfs/projects` içeriğini yerel makinenize yedeklemeniz önerilir

Süre uzatımı veya yeni tahsis için tekrar başvuru yapılması gerekir.

---

## 14. Sorun Giderme

### 14.1 Sık Karşılaşılan Hatalar

**Hata: Job "No account specified"**
```
salloc: error: No account specified, please specify an account
```
Çözüm: `#SBATCH --account=ehpc648` direktifini ekleyin veya
```bash
bsc_project list    # erişebildiğin account'ları gör
```

---

**Hata: Job "No QoS specified"**
```
salloc: error: No QoS specified, please specify a QoS
```
Çözüm: `#SBATCH --qos=acc_debug` direktifini ekleyin:
```bash
bsc_queues    # kullanabileceğin QoS'ları gör
```

---

**Hata: Singularity container içinde CUDA bulunamıyor**
```
RuntimeError: No CUDA GPUs are available
```
Olası nedenler ve çözümler:
1. `--nv` flag'i eksik → `singularity exec --nv` kullanın
2. CUDA_VISIBLE_DEVICES container'a aktarılmamış → `export SINGULARITYENV_CUDA_VISIBLE_DEVICES=0`
3. GPU tahsis edilmemiş → `#SBATCH --gres=gpu:1` ekleyin

---

**Hata: Multi-GPU job'da tüm task'lar aynı GPU'da (OOM)**
```
torch.cuda.OutOfMemoryError: CUDA out of memory
```
Neden: `--gpu-bind=single:1` Singularity container'larına yansımaz.
Çözüm:
```bash
srun --ntasks=4 --gpus-per-task=1 bash -c '
  export CUDA_VISIBLE_DEVICES=0
  export SINGULARITYENV_CUDA_VISIBLE_DEVICES=0
  singularity exec --nv image.sif python3 worker.py
'
```

---

**Hata: Python paket bulunamıyor (container içinde)**
```
ModuleNotFoundError: No module named 'mypackage'
```
Neden: `PYTHONHOME`/`PYTHONPATH` login ortamından container'a sızıyor.
Çözüm:
```bash
module purge          # Tüm modülleri temizle
module load singularity/3.11.5
# module load python  ← YAPMAYIN
singularity exec --nv image.sif python3 script.py
```

---

**Hata: InfiniBand / UCX "Remote operation error"**
```
ib_mlx5_log.c:179 Remote operation error on mlx5_0:1/IB
```
Çözüm:
```bash
module load ucx
```

---

**Hata: Floating-Point Exception (SIGFPE)**
```
Program received signal SIGFPE: Floating-point exception
```
Çözüm: Yine UCX modülü:
```bash
module load ucx
```

---

**Hata: Job "QOSGrpNodeLimit"**
```
(QOSGrpNodeLimit)
```
Neden: Seçilen QoS'un toplam node limiti dolmuş.
Çözüm: Job kuyruğa girmiştir, node açılınca çalışmaya başlayacaktır. Bekleyin veya farklı bir QoS deneyin.

---

**Hata: Conda "defaults channel" zaman aşımı**
```
Collecting package metadata: / | \ - ... (sonsuza kadar bekler)
```
Çözüm:
```bash
conda config --remove channels defaults
conda config --add channels conda-forge
```

---

**Hata: Disk quota exceeded**
```
OSError: [Errno 122] Disk quota exceeded
```
Çözüm:
```bash
bsc_quota                    # hangi alanda doldu?
du -sh /gpfs/projects/ehpc648/*   # nerede yer var?
```
Scratch kullanın veya gereksiz dosyaları temizleyin.

---

### 14.2 Debug için Yararlı Komutlar

```bash
# Job'un neden pending olduğunu göster
squeue -u $USER --format="%.18i %.9P %.8j %.8u %.2t %.10M %.6D %R"

# Job detaylı bilgi
scontrol show job <JOBID>

# Çalışan job'un log'unu canlı izle
tail -f logs/my_job-<JOBID>.out

# Node'un GPU durumu (job çalışırken)
srun --jobid=<JOBID> nvidia-smi

# Container içindeki ortamı kontrol et
singularity exec --nv image.sif env | grep -E "CUDA|PYTHON|PATH"
```

---

## 15. Hızlı Başvuru Kartı

### Bağlantı

```bash
ssh kullanici@alogin1.bsc.es          # ACC login
ssh kullanici@transfer1.bsc.es        # transfer
ssh -L 8000:NODENAME:8000 kullanici@alogin1.bsc.es  # SSH tunnel
```

### Dosya Transferi

```bash
rsync -avz --partial local/ user@transfer1.bsc.es:/gpfs/projects/ehpc648/dest/
scp file user@transfer1.bsc.es:/gpfs/projects/ehpc648/
```

### SLURM Temel Komutlar

```bash
sbatch job.slurm              # job gönder
squeue -u $USER               # kuyruğu gör
scancel <JOBID>               # job iptal
scontrol show job <JOBID>     # detay
bsc_project list              # account'larım
bsc_queues                    # kullanabildiğim queue'lar
bsc_quota                     # kota durumu
sinfo -p acc                  # partition durumu
```

### Modül Sistemi

```bash
module purge                        # temizle
module load singularity/3.11.5      # yükle
module avail python                 # ara
module list                         # yüklüler
```

### Container

```bash
# GPU ile çalıştır
singularity exec --nv image.sif python3 script.py

# BSC AI Hub vLLM image'i
SIF=/gpfs/scratch/shared/ai-hub/singularity_images/vllm/vllm-v0.18.0-official.sif
```

### Kritik Ortam Değişkenleri

```bash
export CUDA_VISIBLE_DEVICES=0
export SINGULARITYENV_CUDA_VISIBLE_DEVICES=0
export SLURM_CPU_BIND=none          # MPI kullanan job'larda
export SRUN_CPUS_PER_TASK=$SLURM_CPUS_PER_TASK  # OpenMP için
```

### Anahtar Dizinler

```bash
/gpfs/home/<GRUP>/$USER             # kişisel home (küçük dosyalar)
/gpfs/projects/ehpc648/             # proje çalışma alanı
/gpfs/scratch/ehpc648/              # geçici, yüksek I/O
$TMPDIR                             # node'a özel yerel SSD
/gpfs/scratch/shared/ai-hub/singularity_images/  # BSC hazır containerlar
```

---

## 16. Sıkça Sorulan Sorular (SSS)

**S: MN5'e erişmek için özel bir VPN gerekiyor mu?**
C: Hayır. Login node'larına (`alogin1.bsc.es` vb.) doğrudan internet üzerinden SSH ile bağlanabilirsiniz. VPN yalnızca `login4` node'larına erişmek için gereklidir ve bu node'lar BSC personeline özeldir.

---

**S: HuggingFace'den model nasıl indiririm?**
C: MN5'ten internet erişimi olmadığından modeli önce yerel makinenize, ardından `rsync` ile transfer node üzerinden MN5'e aktarmanız gerekir. Detaylar için Bölüm 5.3'e bakınız.

---

**S: Job'um saatlerdir "Pending" durumunda, ne yapmalıyım?**
C: `squeue -u $USER` ile pending nedenini kontrol edin. `QOSGrpNodeLimit` görüyorsanız seçtiğiniz QoS'un toplam node limiti dolmuş demektir; açılmasını bekleyin veya farklı bir QoS deneyin. `Resources` görüyorsanız kaynak bekliyor, birkaç dakika içinde çalışmaya başlar.

---

**S: Docker imajımı MN5'te kullanabilir miyim?**
C: Docker doğrudan çalışmaz. Docker imajınızı önce yerel makinede `singularity build` ile `.sif` formatına çevirip MN5'e aktarabilirsiniz. Ya da BSC'nin hazır container'larını kullanabilirsiniz (Bölüm 8).

---

**S: Conda kullanabilir miyim?**
C: Evet, `module load miniconda3` ile kullanabilirsiniz. Ancak Anaconda `defaults` kanalı BSC'de engelli olduğundan bu kanalı kaldırıp `conda-forge` eklemeniz gerekir (Bölüm 7.4).

---

**S: Aynı node'da birden fazla GPU kullanmak için ne yapmalıyım?**
C: `#SBATCH --gres=gpu:4` ile 4 GPU talep edebilirsiniz. Bu GPU'ları tek bir model için (tensor parallelism) veya 4 ayrı task için kullanabilirsiniz. Multi-task Singularity senaryosunda `CUDA_VISIBLE_DEVICES=0` ve `SINGULARITYENV_CUDA_VISIBLE_DEVICES=0` ayarını unutmayın (Bölüm 10.1).

---

**S: Birden fazla node kullanabilir miyim?**
C: Evet. `#SBATCH --nodes=2` ile birden fazla node talep edebilirsiniz. Multi-node iş için NCCL ve InfiniBand ayarları önemlidir. Büyük model eğitimi için `NCCL_NET=IB`, `NCCL_SOCKET_IFNAME=ib0` değişkenlerini set edin.

---

**S: Job'um çalışırken GPU node'una tarayıcıdan nasıl erişebilirim?**
C: SSH port forwarding ile tünel kurabilirsiniz. `ssh -L 8000:NODENAME:8000 user@alogin1.bsc.es` komutundan sonra `http://localhost:8000` adresini tarayıcınızda açın (Bölüm 3.5).

---

**S: Disk kotama yaklaştığımda ne yapmalıyım?**
C: `bsc_quota` ile durumu kontrol edin. Önce `/gpfs/scratch` içindeki eski job çıktılarını temizleyin. Büyük ara dosyaları `/gpfs/scratch` yerine `$TMPDIR` (node-local SSD) üzerinde üretin ve job bitiminde önemli sonuçları `/gpfs/projects`'e kopyalayın. Ek kota gerekiyorsa proje yöneticisi BSC'ye başvurmalıdır.

---

**S: Job çalışırken GPU kullanımı ne kadar olduğunu görebilir miyim?**
C: `srun --jobid=<JOBID> nvidia-smi` komutu ile anlık durum görülebilir. Sürekli izleme için `watch -n 5 nvidia-smi` kullanın. EAR aracı (`module load ear`) daha kapsamlı enerji ve performans metrikleri sağlar.

---

**S: Kişisel verilerimi (örneğin kullanıcı anketleri) LLM fine-tuning için kullanabilir miyim?**
C: Teknik olarak mümkündür, ancak GDPR uyumluluğu sizin sorumluluğunuzdadır. Veriyi MN5'e taşımadan önce uygun anonimleştirme yapılmalı ve kurumunuzun DPO'su bilgilendirilmelidir. BSC'nin veri işleme sözleşmesini (DPA) incelemenizi öneririz.

---

**S: Sonuçlarımı yayımlarken nasıl teşekkür etmeliyim?**
C: EuroHPC tahsisini aldıysanız yayınlarınızda EuroHPC Joint Undertaking ve BSC'yi belirtmeniz beklenir. Örnek: *"The authors acknowledge the EuroHPC Joint Undertaking for awarding this project access to the MareNostrum5 supercomputer at BSC (Barcelona Supercomputing Center)."*

---

**S: İki farklı model versiyonunu (4B ve 9B) aynı anda çalıştırabilir miyim?**
C: Evet. 4B model tek GPU'ya (~8-10 GB bellek) sığar; 9B model de tek H100'e rahatlıkla yüklenir. Farklı portlarda ayrı vLLM instance'ları çalıştırarak her ikisini aynı job içinde aktif tutabilirsiniz (Bölüm 11.3).

---

*Yazar: Gökhan Eryol | gokhaneryol@gmail.com*
*Son güncelleme: Nisan 2026*
*Platform: MareNostrum 5, BSC Barcelona — EuroHPC AI Factory Playground `ehpc648`*
