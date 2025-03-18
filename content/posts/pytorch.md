---
title: "Pytorch"
date: 2025-03-04T01:12:12+09:00
draft: true
---

# PyTorch

## Install on WSL with GeForce RTX 3060

In case you use GeForce RTX 3060, you can use pytorch/pytorch:2.6.0-cuda12.6-cudnn9-devel  

[WSL2+Ubuntu24.04+Docker＋GPUでつくる機械学習環境](https://zenn.dev/yumizz/articles/627d4e4821c636)  
[GPUの型番にあったCUDAバージョンの選び方](https://zenn.dev/yumizz/articles/73d6c7d1085d2f)  
[Compose で GPU アクセスの有効化](https://docs.docker.jp/compose/gpu-support.html)  

compose.yaml  

```
services:
  linux:
    image: pytorch/pytorch:2.6.0-cuda12.6-cudnn9-devel
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
    tty: true
```

```
docker compose up -d
docker compose exec linux bash
```

```
python -c "import torch; print(torch.cuda.is_available())"
```

In case it prints True, the GPU is available.  

## Hugging Face Transformers

[Transformers](https://huggingface.co/docs/transformers)  
[Models](https://huggingface.co/models)  

[AutoClass](https://huggingface.co/docs/transformers/model_doc/auto)  
　[AutoTokenizer](https://huggingface.co/docs/transformers/model_doc/auto#transformers.AutoTokenizer)  
　[AutoModel](https://huggingface.co/docs/transformers/model_doc/auto#transformers.AutoModel)  
　　[AutoModelForSequenceClassification](https://huggingface.co/docs/transformers/model_doc/auto#transformers.AutoModelForSequenceClassification)  
　[AutoConfig](https://huggingface.co/docs/transformers/model_doc/auto#transformers.AutoConfig)  

[Trainer](https://huggingface.co/docs/transformers/main_classes/trainer#transformers.Trainer)  
　[TrainingArguments](https://huggingface.co/docs/transformers/main_classes/trainer#transformers.TrainingArguments)  
[Seq2SeqTrainer](https://huggingface.co/docs/transformers/main_classes/trainer#transformers.Seq2SeqTrainer)  
　[Seq2SeqTrainingArguments](https://huggingface.co/docs/transformers/main_classes/trainer#transformers.Seq2SeqTrainingArguments)  

pytorch/Dockerfile  

```
FROM pytorch/pytorch:2.6.0-cuda12.6-cudnn9-devel

RUN apt update && \
  apt full-upgrade -y && \
  pip install -U pip && \
  pip install transformers datasets evaluate accelerate && \
  pip install sentencepiece

# sentencepiece is for Mizuiro-sakura/luke-japanese-large-sentiment-analysis-wrime
```

Don't do pip install pip-review; pip-review --auto or you'll destroy the environment.  

compose.yaml

```
services:
  linux:
    build: ./pytorch
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
    tty: true
```

```
docker compose up --build -d
docker compose exec linux bash
```

```
python -c "import torch; print(torch.cuda.is_available())"
```

In case it prints True, the GPU is available.  

```
python -c "from transformers import pipeline; print(pipeline('sentiment-analysis')('we love you'))"
```

In case it prints [{'label': 'POSITIVE', 'score': 0.9998704195022583}], Hugging Face Transformers is installed properly.  

### livedoor News Corpus
