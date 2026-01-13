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

HFAPI and so on does not have meaning!

---

# ✅ Hugging Face Dataset に 70GB 画像を 5GB shard でアップロードする完全手順（確定版）

---

## 0️⃣ 前提ディレクトリ

```bash
~/LLaVA-Instruction-Dataset-Comperession/
└── images.tar.zst          # 元の 70GB ファイル
```

---

## 1️⃣ 70GB ファイルを 5GB shard に分割

```bash
split -b 5G images.tar.zst images.tar.zst.part_
```

生成例：

```
images.tar.zst.part_aa
images.tar.zst.part_ab
...
images.tar.zst.part_an
```

---

## 2️⃣ 必要ツールをインストール

```bash
sudo apt update
sudo apt install -y git-lfs
```

---

## 3️⃣ git-lfs を初期化（最初に1回）

```bash
git lfs install
```

確認：

```bash
git lfs version
```

---

## 4️⃣ Hugging Face Dataset リポジトリを clone

```bash
git clone https://huggingface.co/datasets/HayatoHongoEveryonesAI/LLaVA-Instruction-665k-Images-Dataset
```

---

## 5️⃣ Dataset リポジトリに移動

```bash
cd LLaVA-Instruction-665k-Images-Dataset
```

---

## 6️⃣ LFS トラッキング設定（最重要）

```bash
git lfs track "*.part_*"
git lfs track "*.zst"
```

---

## 7️⃣ .gitattributes を commit

```bash
git add .gitattributes
git commit -m "Track large dataset shards with git-lfs"
```

※ 初回のみ git identity が必要：

```bash
git config --global user.email "hayato.hongo@everyonesai.org"
git config --global user.name "HayatoHongoEveryonesAI"
```

---

## 8️⃣ shard 用ディレクトリを作成

```bash
mkdir -p data/images
```

---

## 9️⃣ shard を repo 内にコピー（安全・高速）

```bash
rsync -av --progress \
  ../images.tar.zst.part_* \
  data/images/
```

---

## 🔟 shard を add → commit

```bash
git add data/images/images.tar.zst.part_*
git commit -m "Add LLaVA image shards (5GB parts)"
```

---

## 1️⃣1️⃣ 5GB 超 LFS を明示的に許可（HF 固有・必須）

```bash
hf lfs-enable-largefiles .
```

※ **これを忘れると必ず push 失敗**

---
---

## 1️⃣2️⃣  push（切れてもOK、完走するまで繰り返す）

```bash
git push
```

* `broken pipe` が出ても **正常**
* 再度：

```bash
git push
```

---

## ✅ 成功ログ（これが出たら終了）

```text
Uploading LFS objects: 100% (14/14), 72 GB | XXX MB/s, done.
To https://huggingface.co/datasets/HayatoHongoEveryonesAI/LLaVA-Instruction-665k-Images-Dataset
   xxxx..yyyy  main -> main
```

---

## 🔍 最終確認（任意）

```bash
git lfs ls-files
```

---

# 🧠 重要ポイント（再発防止）

* **HFAPI は使わない**
* **5GB shard が安定上限**
* `hf lfs-enable-largefiles .` は **必須**
* `git push` は **再実行前提**
* Web UI は信用しない

---

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
