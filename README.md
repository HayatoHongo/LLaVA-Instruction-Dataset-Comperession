# LLaVA-Instruction-Dataset-Comperession

# Aim

Compressed images for llava_v1_5_mix665k.json.

https://huggingface.co/datasets/liuhaotian/LLaVA-Instruct-150K 

Used Lambda Cloud SSH A10 GPU instance. 
We don't use GPU, but the large disk, 30 vCPUs and fast download for AI training were ideal for this task.


The first downloading is fast thanks to boosting

```
train2017.zip 19GB
wget http://images.cocodataset.org/zips/train2017.zip
ls -lh train2017.zip
unzip train2017.zip
mkdir -p coco
mv train2017 coco/
```

but it seemed like I ran out of boosting budget.
slow download. it took about an hour.


#suddenly the downloading speed slowed down.
#sudo apt-get update
#sudo apt-get install -y aria2

images.zip 21GB
wget https://downloads.cs.stanford.edu/nlp/data/gqa/images.zip 
ls -lh images.zip
unzip images.zip
mkdir -p gqa
mv images gqa/
mv images.zip qga_images.zip


boost download revived.

wget https://dl.fbaipublicfiles.com/textvqa/images/train_val_images.zip
ls -lh train_val_images.zip
unzip train_val_images.zip
mkdir -p textvqa
mv train_images textvqa/


images.zip 9.1G
wget https://cs.stanford.edu/people/rak248/VG_100K_2/images.zip
ls -lh images.zip
unzip images.zip
mkdir -p vg
mv VG_100K vg/
mv images.zip vg_images.zip


images2.zip 5.1G
wget https://cs.stanford.edu/people/rak248/VG_100K_2/images2.zip
ls -lh images2.zip
unzip images2.zip
mv VG_100K_2 vg/

ocr_vqa_images_llava_v15.zip: 8.4GB
wget https://huggingface.co/datasets/weizhiwang/llava_v15_instruction_images/resolve/main/ocr_vqa_images_llava_v15.zip
ls -lh ocr_vqa_images_llava_v15.zip
unzip ocr_vqa_images_llava_v15.zip
mkdir -p ocr_vqa
mv images ocr_vqa/


tar -I 'zstd -19 -T0' \
  -cf images.tar.zst \
  coco gqa ocr_vqa textvqa vg
