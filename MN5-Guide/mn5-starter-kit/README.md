# MN5 Starter Kit

MareNostrum 5 (BSC) üzerinde AI / HPC iş yüklerini hızlıca başlatmak için hazır bir şablon dizini.

Kapsamlı kullanım rehberi: [MN5 Kullanıcı Rehberi](https://github.com/gokhaneryol/blog/blob/main/MN5-Guide/README.md)

---

## Dizin Yapısı

```
mn5-starter-kit/
├── .claude/
│   └── CLAUDE.md          ← AI asistanı instruction seti (kritik kurallar, MN5 bağlamı)
├── jobs/
│   └── templates/
│       ├── single-gpu.slurm        ← Tek GPU, genel amaçlı
│       ├── multi-gpu-4task.slurm   ← 4 GPU, 4 bağımsız task (data-parallel)
│       ├── vllm-single.slurm       ← vLLM tek model sunucusu (tensor parallelism)
│       └── vllm-4model.slurm       ← 4 model, 4 GPU, farklı portlar
├── scripts/               ← Python / bash scriptler
├── logs/                  ← SLURM .out / .err dosyaları (sync'e dahil değil)
├── results/               ← Analiz ve benchmark sonuçları (sync'e dahil değil)
├── docs/                  ← Notlar, raporlar
├── models/                ← LLM model ağırlıkları (sync'e dahil değil, büyük dosyalar)
├── containers/            ← .sif container dosyaları (sync'e dahil değil)
├── sync.sh                ← Yerel makine → MN5 rsync scripti
└── .gitignore
```

---

## Hızlı Başlangıç

### 1. Bu repoyu kendi projenize uyarlayın

```bash
# Proje kodunuzu değiştirin (varsayılan: ehpc648)
grep -r "ehpc648" . --include="*.slurm" --include="*.md" --include="*.sh" -l
# Her dosyada ehpc648 → kendi proje kodunuzla değiştirin
```

### 2. MN5'e senkronize edin

```bash
chmod +x sync.sh
./sync.sh <mn5-kullanici-adi> <proje-dizin-adi>
# Örnek:
./sync.sh gokhan my-project
```

### 3. MN5'e bağlanın

```bash
ssh <kullanici>@alogin1.bsc.es
cd /gpfs/projects/ehpc648/my-project
```

### 4. İlk job'ı gönderin

```bash
# Tek GPU test
sbatch jobs/templates/single-gpu.slurm

# Job durumunu izle
squeue -u $USER
tail -f logs/my-single-gpu-<JOBID>.out
```

---

## Job Şablonları

| Dosya | GPU | Senaryo |
|-------|-----|---------|
| `single-gpu.slurm` | 1 | Genel amaçlı, Python/ML iş yükü |
| `multi-gpu-4task.slurm` | 4 | 4 bağımsız paralel görev |
| `vllm-single.slurm` | 1–4 | vLLM tek model sunucusu (tensor parallelism) |
| `vllm-4model.slurm` | 4 | 4 farklı model, 4 farklı port |

---

## Kritik Kurallar (Özet)

**1. `module purge` zorunlu** — `module load python` PYTHONHOME'u container'a sızdırır.

**2. Multi-GPU Singularity binding** — `--gpu-bind=single:1` container'a yansımaz:
```bash
export CUDA_VISIBLE_DEVICES=0
export SINGULARITYENV_CUDA_VISIBLE_DEVICES=0
```

**3. Zorunlu SBATCH direktifleri:**
```bash
#SBATCH --account=ehpc648
#SBATCH --qos=acc_debug
```

Tüm kurallar için: `.claude/CLAUDE.md`

---

## BSC AI Hub Container'ları

BSC, hazır vLLM container'ları sağlar — kendi `.sif` dosyanızı oluşturmanıza gerek yok:

```
/gpfs/scratch/shared/ai-hub/singularity_images/vllm/vllm-v0.18.0-official.sif
/gpfs/scratch/shared/ai-hub/singularity_images/vllm/pytorch_25.12-py3.sif
/gpfs/scratch/shared/ai-hub/singularity_images/vllm/nemo_25.02.sif
```

---

## Lisans

MIT — Serbestçe kullanabilir, uyarlayabilir, paylaşabilirsiniz.
