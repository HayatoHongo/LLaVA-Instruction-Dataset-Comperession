# LLaVA-Instruction-Dataset-Compression

## Aim

This repository provides **compressed image datasets** used by
`llava_v1_5_mix665k.json`.

Original instruction dataset:
[https://huggingface.co/datasets/liuhaotian/LLaVA-Instruct-150K](https://huggingface.co/datasets/liuhaotian/LLaVA-Instruct-150K)

The goal is to download, organize, and compress all related image datasets
into a single high-compression archive for efficient storage and reuse.

---

## Environment

* **Cloud provider**: Lambda Cloud
* **Instance**: A10 GPU instance, 0.75$/hour
* **Note**: GPU was *not* used

  * Large disk capacity
  * 30 vCPUs
  * High network throughput
    made this instance ideal for large-scale dataset downloading and compression.
  * Attatch filesystem

---

## Dataset Downloads

### COCO (train2017)

Initial downloads were very fast thanks to network boosting.

```bash
wget http://images.cocodataset.org/zips/train2017.zip
ls -lh train2017.zip   # ~19GB
unzip train2017.zip
mkdir -p coco
mv train2017 coco/
```

After this step, the download speed dropped significantly (likely due to
boosting quota exhaustion), and subsequent downloads took much longer
(around 1 hour each).

---

### GQA

```bash
wget https://downloads.cs.stanford.edu/nlp/data/gqa/images.zip
ls -lh images.zip      # ~21GB
unzip images.zip
mkdir -p gqa
mv images gqa/
mv images.zip gqa_images.zip
```

---

### TextVQA

Download speed recovered.

```bash
wget https://dl.fbaipublicfiles.com/textvqa/images/train_val_images.zip
ls -lh train_val_images.zip
unzip train_val_images.zip
mkdir -p textvqa
mv train_images textvqa/
```

---

### Visual Genome (VG)

#### VG_100K

```bash
wget https://cs.stanford.edu/people/rak248/VG_100K_2/images.zip
ls -lh images.zip      # ~9.1GB
unzip images.zip
mkdir -p vg
mv VG_100K vg/
mv images.zip vg_images.zip
```

#### VG_100K_2

```bash
wget https://cs.stanford.edu/people/rak248/VG_100K_2/images2.zip
ls -lh images2.zip     # ~5.1GB
unzip images2.zip
mv VG_100K_2 vg/
```

---

### OCR-VQA (LLaVA v1.5)

```bash
wget https://huggingface.co/datasets/weizhiwang/llava_v15_instruction_images/resolve/main/ocr_vqa_images_llava_v15.zip
ls -lh ocr_vqa_images_llava_v15.zip   # ~8.4GB
unzip ocr_vqa_images_llava_v15.zip
mkdir -p ocr_vqa
mv images ocr_vqa/
```

---

## Final Directory Structure

After all downloads and extraction:

```
coco/
gqa/
textvqa/
vg/
ocr_vqa/
```

---

## Compression

All image datasets are compressed into a single archive using **zstd** with
maximum compression and multithreading:

```bash
tar -I 'zstd -19 -T0' \
  -cf images.tar.zst \
  coco gqa ocr_vqa textvqa vg
```

* `-19` : maximum compression level
* `-T0` : use all available CPU cores

---

## Upload to HuggingFace
```bash
pip install -U huggingface_hub
```

```bash
export HF_TOKEN="hf...."
echo $HF_TOKEN
```

```bash
python - << 'EOF'
from huggingface_hub import HfApi

api = HfApi()

api.upload_file(
    path_or_fileobj="images.tar.zst",
    path_in_repo="images.tar.zst",
    repo_id="HayatoHongo/LLaVA-Instruct-150K",
    repo_type="dataset",
)
EOF
```

# Option: Copy to filesystem(if you attached) 

Please take it into account you will be charged based on your disk size in the filesystem. (maybe 20USD per month for 70GB storage)
```bash
rsync -ah --progress \
  /home/ubuntu/LLaVA-Instruction-Dataset-Comperession/images.tar.zst \
  /home/ubuntu/virginia-filesystem/
```


## Result

* A single highly compressed archive: `images.tar.zst`
* Suitable for:

  * Dataset sharing
  * Fast rehydration on new machines
  * Reduced storage and transfer costs
