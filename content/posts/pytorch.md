---
title: "Pytorch"
date: 2025-03-04T01:12:12+09:00
draft: true
---

# PyTorch

## Install

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
docker compose exec linux bash
```

```
python
>>> import torch
>>> print(torch.cuda.is_available())
```

In case it prints True, the GPU is available.  
