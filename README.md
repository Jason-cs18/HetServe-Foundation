# HetServe-Foundation
> [A Overview of Efficiently Serving Foundation Models across Edge Devices [arXiv] (TBD)]()

## What is This Overview About? 
Foundation models (e.g., large language models and diffusion models) have achieved remarkable results across diverse tasks. However, their resource-intensive characteristics present challenges for efficient deployment. In this overview, we explore the approach of serving them on distributed edge devices, aiming to address concerns regarding scalability and latency. Unlike existing surveys, our objective is to offer **a comprehensive overview of foundation model serving rather than merely focusing on LLM serving.**

## Why Efficiently Serving Foundation Models over Distributed Heterogeneous Devices is Needed?
Efficiently serving foundation models over distributed heterogeneous devices is necessary to provide a seamless user experience with low latency. It enables scalability by leveraging resources from multiple devices, optimizing load balancing and resource utilization. This approach helps manage network congestion by distributing the delivery load across different devices. With the diverse range of device types available, efficient serving ensures that each device receives an optimized stream tailored to its capabilities. Overall, efficiently serving LLMs over distributed heterogeneous devices improves user experience, scalability, network performance, and accommodates the growing variety of devices in use.

## Table of Contents
- [LLM Serving](#LLM-Serving)
- - [LLM Serving Survey](#LLM-Serving-Survey)
  - [LLM Background](#Background)
  - [Serving Metrics for LLMs](#Serving-Metrics)
  - [Latency-oriented Optimizations for LLMs](#Latency-oriented-Optimizations)
    - [Efficient Models](#Efficient-Models)
    - [Efficient Operators/Kernels](#Efficient-Operators/Kernels)
    - [Quantization](#Quantization)
  - [Throughput-oriented Optimizations for LLMs](#Throughput-oriented-Optimizations)
    - [Resource Management](#Resource-Management)
    - [Parallelism](#Parallelism)
  - [Open-source LLM Serving Systems](#Open-source-LLM-Serving-Systems)
- [Diffusion Serving](#Diffusion-Serving)
  - [Diffusion Serving Survey]()
  - [Diffusion Background]()
  - [Serving Metrics for Diffusion]()
  - [Latency-oriented Optimizations for Diffusion]()
  - [Throughput-oriented Optimizations for Diffusion]()
  - [Open-source Diffusion Serving Systems]()
- [Serving on Heterogeneous Devices](#Serving-on-Heterogeneous-Devices)
  - [A Single Device](#A-Single-Device)
  - [Scaling to Distributed Devices](#Scaling-to-Distributed-Devices)
- [Other List](#Other-List)

## LLM Serving

![](https://github.com/yinuotxie/Efficient-LLM-Inferencing-on-GPUs/raw/main/media/llm_inferece_dataflow.png)
*Image source: [Efficient Large Language Model (LLM) Inferencing on GPUs](https://github.com/yinuotxie/Efficient-LLM-Inferencing-on-GPUs?tab=readme-ov-file)*

![](https://mmbiz.qpic.cn/mmbiz_jpg/AAQtmjCc74AzI4S2pibIpCrWfDWiaBeFZOibkVAQgVwI2m6IicEdEjHgPHTwR6aGclubTx9MRFVFoYSjObI9QgvQzA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

*Image source: [Efficient Memory Management for Large Language Model Serving with PagedAttention](https://dl.acm.org/doi/abs/10.1145/3600006.3613165)*

### LLM Serving Survey
- [arXiv 2023.02] [Full Stack Optimization of Transformer Inference: a Survey](https://arxiv.org/abs/2302.14017) | UC Berkeley
- [arXiv 2023.12] [Towards Efficient Generative Large Language Model Serving: A Survey from Algorithms to Systems](https://arxiv.org/pdf/2312.15234) | Carnegie Mellon University
- [TMLR 2024] [Efficient Large Language Models: A Survey](https://github.com/AIoT-MLSys-Lab/Efficient-LLMs-Survey), The Ohio State University
- [arXiv 2024.04] [A Survey on Efficient Inference for Large Language Models](https://arxiv.org/abs/2404.14294), Infinigence-AI and Tsinghua University

### Background
- [NeurIPS 2017] [Attention is all you need](https://proceedings.neurips.cc/paper/2017/file/3f5ee243547dee91fbd053c1c4a845aa-Paper.pdf) | Google Brain
- [NeurIPS 2020] [Language Models are Few-Shot Learners](https://papers.nips.cc/paper/2020/file/1457c0d6bfcb4967418bfb8ac142f64a-Paper.pdf) | OpenAI
- [arXiv 2020.01] [Scaling Laws for Neural Language Models](https://arxiv.org/pdf/2001.08361) | Johns Hopkins University and OpenAI
- [arXiv 2022.01] [Scaling Language Models: Methods, Analysis & Insights from Training Gopher](https://arxiv.org/pdf/2112.11446) | DeepMind

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74AzI4S2pibIpCrWfDWiaBeFZO6xKDZNW06WkyAfGByibk2iacC5bJkKbrzskicG84C1dM7d74iaMoRJXoQA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
*Image source: [Large Language Models (in 2023)](https://www.youtube.com/watch?v=dbo3kNKPaUA)*

### Serving Metrics
- [Tech Blog] [LLM Inference Performance Engineering: Best Practices](https://www.databricks.com/blog/llm-inference-performance-engineering-best-practices) | Mosaic AI Research
- [arXiv 2024.04] [Andes: Defining and Enhancing Quality-of-Experience in LLM-Based Text Streaming Services](https://arxiv.org/pdf/2404.16283) | University of Michigan
#### Common Metrics
- **Time To First Token (TTFT)**: How quickly users start seeing the model's output after entering their query. Low waiting times for a response are essential in real-time interactions, but less important in offline workloads. This metric is driven by the time required to process the prompt and then generate the first output token.
- **Time Per Output Token (TPOT)**: Time to generate an output token for each user that is querying our system. This metric corresponds with how each user will perceive the "speed" of the model. For example, a TPOT of 100 milliseconds/tok would be 10 tokens per second per user, or ~450 words per minute, which is faster than a typical person can read.
- **Latency**: The overall time it takes for the model to generate the full response for a user. Overall response latency can be calculated using the previous two metrics: latency = (TTFT) + (TPOT) * (the number of tokens to be generated).
- **Throughput**: The number of output tokens per second an inference server can generate across all users and requests.

### Latency-oriented Optimizations

#### Efficient Models
- [arXiv 2020.04] [Longformer: The Long-Document Transformer](https://arxiv.org/abs/2004.05150v2) | Allen Institute for Artificial Intelligence
- [ICML 2022] [DeepSpeed-MoE: Advancing Mixture-of-Experts Inference and Training to Power Next-Generation AI Scale](https://arxiv.org/abs/2201.05596) | Microsoft
- [EMNLP 2023] [GQA: Training Generalized Multi-Query Transformer Models from Multi-Head Checkpoints](https://arxiv.org/pdf/2305.13245) | Google Research
- [ICML 2023] [Fast Inference from Transformers via Speculative Decoding](https://arxiv.org/abs/2211.17192) | Google Research
- [ACL 2023] [Distilling Step-by-Step! Outperforming Larger Language Models with Less Training Data and Smaller Model Sizes](https://arxiv.org/abs/2305.02301) | University of Washington
- [arXiv 2023.05] [SpecInfer: Accelerating Generative LLM Serving with Speculative Inference and Token Tree Verification](https://www.cs.cmu.edu/~zhihaoj2/papers/specinfer.pdf) | Carnegie Mellon University
- [arXiv 2024.02] [Break the Sequential Dependency of LLM Inference Using Lookahead Decoding](https://arxiv.org/pdf/2402.02057) | University of California, San Diego
- [ACL 2024] [Draft & Verify: Lossless Large Language Model Acceleration via Self-Speculative Decoding](https://arxiv.org/pdf/2309.08168) | Zhejiang University

#### Efficient Operators/Kernels
- [NeurIPS 2022] [FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness](https://proceedings.neurips.cc/paper_files/paper/2022/file/67d57c32e20fd0a7a302cb81d36e40d5-Paper-Conference.pdf) | Stanford University
- [ICLR 2024] [FlashAttention-2: Faster Attention with Better Parallelism and Work Partitioning](https://openreview.net/forum?id=mZn2Xyh9Ec) | Princeton University
- [arXiv 2023.11] [FlashDecoding++: Faster Large Language Model Inference on GPUs](https://arxiv.org/abs/2311.01282) | Tsinghua University & Infinigence-AI

#### Quantization
- [ICLR 2023] [GPTQ: Accurate Post-Training Quantization for Generative Pre-trained Transformers](https://arxiv.org/abs/2210.17323) | IST Austria
- [ICML 2023] [SmoothQuant: Accurate and Efficient Post-Training Quantization for Large Language Models](https://github.com/mit-han-lab/smoothquant) | MIT
- [MLSys 2024] [AWQ: Activation-aware Weight Quantization for LLM Compression and Acceleration](https://hanlab.mit.edu/projects/awq) | MIT

### Throughput-oriented Optimizations

#### Resource Management
- [PPoPP 2021] [TurboTransformers: An Efficient GPU Serving System For Transformer Models](https://dl.acm.org/doi/pdf/10.1145/3437801.3441578) | Tencent
- [OSDI 2022] [Orca: A Distributed Serving System for Transformer-Based Generative Models](https://www.usenix.org/conference/osdi22/presentation/yu) | Seoul National University
- [SOSP 2023] [Efficient Memory Management for Large Language Model Serving with PagedAttention](https://dl.acm.org/doi/abs/10.1145/3600006.3613165) | UC Berkeley
- [ICLR 2024] [Efficient Streaming Language Models with Attention Sinks](https://hanlab.mit.edu/projects/streamingllm) | MIT
- [arXiv 2024.01] [DistServe: Disaggregating Prefill and Decoding for Goodput-optimized Large Language Model Serving](https://arxiv.org/pdf/2401.09670) | Peking University
- [arXiv 2024.02] [FlexLLM: A System for Co-Serving Large Language Model Inference and Parameter-Efficient Finetuning](https://arxiv.org/pdf/2402.18789) | Carnegie Mellon 
- [arXiv 2024.02] [MuxServe: Flexible Multiplexing for Efficient Multiple LLM Serving](https://arxiv.org/abs/2404.02015)
- [arXiv 2024.03] [ALTO: An Efficient Network Orchestrator for Compound AI Systems](https://arxiv.org/pdf/2403.04311)
- [arXiv 2024.03] [AttentionStore: Cost-effective Attention Reuse across Multi-turn Conversations in Large Language Model Serving](https://arxiv.org/pdf/2403.19708) | National University of Singapore
- [arXiv 2024.04] [MuxServe: Flexible Multiplexing for Efficient Multiple LLM Serving](https://arxiv.org/abs/2404.02015) | The Chinese University of Hong Kong
- [arXiv 2024.04] [BlockLLM: Multi-tenant Finer-grained Serving for Large Language Models](https://arxiv.org/pdf/2404.18322) | City University of Hong Kong
- [arXiv 2024.05] [vAttention: Dynamic Memory Management for Serving LLMs without PagedAttention](https://arxiv.org/pdf/2405.04437) | Microsoft Research India

#### Parallelism

|Parallelism|Tensor Parallelism (TP)|Sequence Parallelism (SP)|
|:---:|:---:|:---:|
|Illustration|![](https://github.com/Jason-cs18/HetServe-LLMs/blob/main/tensor_parallelism.png)<br>*Image source: [LoongServe](https://arxiv.org/pdf/2404.09526)*| ![](https://github.com/Jason-cs18/HetServe-LLMs/blob/main/sequence_parallelism.png)<br>*Image source: [LoongServe](https://arxiv.org/pdf/2404.09526)*|

- [NeurIPS 2019] [GPipe: Efficient Training of Giant Neural Networks using Pipeline Parallelism](https://arxiv.org/abs/1811.06965) | Google
- [SC 2021] [Efficient Large-Scale Language Model Training on GPU Clusters Using Megatron-LM](https://arxiv.org/abs/2104.04473) | NVIDIA
- [OSDI 2022] [Orca: A Distributed Serving System for Transformer-Based Generative Models](https://www.usenix.org/conference/osdi22/presentation/yu) | Seoul National University
- [ICML 2023] [FlexGen: high-throughput generative inference of large language models with a single GPU](https://dl.acm.org/doi/10.5555/3618408.3619696) | Stanford Univeristy
- [arXiv 2023.05] [Fast Distributed Inference Serving for Large Language Models](https://arxiv.org/abs/2305.05920) | Peking University
- [arXiv 2023.12] [PowerInfer: Fast Large Language Model Serving with a Consumer-grade GPU](https://ipads.se.sjtu.edu.cn/_media/publications/powerinfer-20231219.pdf) | Shanghai Jiao Tong University
- [arXiv 2024.01] [Infinite-LLM: Efficient LLM Service for Long Context with DistAttention and Distributed KVCache](https://arxiv.org/abs/2401.02669) | Alibaba Group
- [arXiv 2024.03] [FASTDECODE: High-Throughput GPU-Efficient LLM Serving using Heterogeneous Pipelines](https://arxiv.org/pdf/2403.11421) | Tsinghua University
- [arXiv 2024.04] [LoongServe: Efficiently Serving Long-context Large Language Models with Elastic Sequence Parallelism](https://arxiv.org/pdf/2404.09526) | Peking University

### Open-source LLM Serving Systems
> Target: Latency or Throughput

> Optimization: Quantization, Flash Attention, PagedAttention, Speculation, Continuous batching

> Parallelism: Tensor Parallelism (TP), Pipeline Parallelism (PP), Sequence Parallelism (SP), CPU-GPU offloading (offload)

> Supported hardware: NVIDIA GPU, AMD GPU, Intel CPU

|Serving System|Target|Optimization|Parallelism|Hardware|
|:---|:---|:---|:---|:---|
|[vLLM](https://github.com/vllm-project/vllm) ![Github stars](https://img.shields.io/github/stars/vllm-project/vllm.svg) ![Github forks](https://img.shields.io/github/forks/vllm-project/vllm.svg)|Throughput|Quantization, Flash Attention, PagedAttention, Speculation, Continuous batching|TP|NVIDIA GPU, Intel CPU, AMD GPU|
|[TensorRT-LLM](https://github.com/NVIDIA/TensorRT-LLM) ![Github stars](https://img.shields.io/github/stars/NVIDIA/TensorRT-LLM.svg) ![Github forks](https://img.shields.io/github/forks/NVIDIA/TensorRT-LLM.svg)|Latency|Quantization, Flash Attention|TP, PP|NVIDIA GPU|
|[llama.cpp](https://github.com/ggerganov/llama.cpp) ![Github stars](https://img.shields.io/github/stars/ggerganov/llama.cpp.svg) ![Github forks](https://img.shields.io/github/forks/ggerganov/llama.cpp.svg)|Latency|Quantization, Flash Attention, Speculation, Continuous batching|TP, PP|NVIDIA GPU, AMD GPU, Intel GPU/CPU, Mac|
|[text-generation-inference](https://huggingface.co/docs/text-generation-inference/en/index) ![Github stars](https://img.shields.io/github/stars/huggingface/text-generation-inference.svg) ![Github forks](https://img.shields.io/github/forks/huggingface/text-generation-inference.svg)|Latency and Throughput|Quantization, Flash Attention, PagedAttention, Continuous batching|TP|NVIDIA GPU, AMD GPU, Intel CPU|

*Suggestion: If your tasks are latency-sensistive, please use llama.cpp or TensorRT-LLM. If your tasks are latency-insensitive and have enough powerful GPUs, please use vLLM for a better throughput. If you just want to evaluate your research ideas with serving optimizations, text-generation-inference is a good chooice.*

## Serving on Heterogeneous Devices
### A Single Device
- [ICML 2023] [FlexGen: high-throughput generative inference of large language models with a single GPU](https://dl.acm.org/doi/10.5555/3618408.3619696) | Stanford Univeristy
- [arXiv 2023.12] [LLM in a flash: Efficient Large Language Model Inference with Limited Memory](https://arxiv.org/abs/2312.11514) | Apple
- [arXiv 2023.08] [EdgeMoE: Fast On-Device Inference of MoE-based Large Language Models](https://arxiv.org/pdf/2308.14352) | Beijing University of Posts and Telecommunications
- [ICML 2023, Oral] [Deja Vu: Contextual Sparsity for Efficient LLMs at Inference Time](https://openreview.net/pdf?id=wIPIhHd00i)
- [ASPLOS'24] [NeuPIMs: NPU-PIM Heterogeneous Acceleration for Batched LLM Inferencing](https://dl.acm.org/doi/10.1145/3620666.3651380) | KAIST
### Scaling to Distributed Devices
- [FMEC 2023] [PipeEdge: Pipeline Parallelism for Large-Scale Model Inference on Heterogeneous Edge Devices](https://github.com/usc-isi/PipeEdge) | Purdue University
- [ASPLOS 2023] [STI: Turbocharge NLP Inference at the Edge via Elastic Pipelining](https://arxiv.org/abs/2207.05022) | University of Virginia
- [arXiv 2023.11] [Moirai: Towards Optimal Placement for Distributed Inference on Heterogeneous Devices](https://arxiv.org/abs/2312.04025) | Zhejiang University
- [ACL 2023 Demo] [PETALS: Collaborative Inference and Fine-tuning of Large Models](https://aclanthology.org/2023.acl-demo.54/) | HSE University and Yandex
- [arXiv 2024.02] [APISERVE: Efficient API Support for Large-Language Model Inferencing](https://arxiv.org/pdf/2402.01869) | University of California, San Diego
- [Tech Report] [Task Scheduling for Decentralized LLM Serving in Heterogeneous Networks](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2024/EECS-2024-111.pdf) | UC Berkeley
- [arXiv 2024.01] [CARASERVE: CPU-Assisted and Rank-Aware LoRA Serving for Generative LLM Inference](https://arxiv.org/pdf/2401.11240) | HKUST
- [arXiv 2024.03] [DejaVu: KV-cache Streaming for Fast, Fault-tolerant Generative LLM Serving](https://arxiv.org/pdf/2403.01876) | ETH Zurich
- [arXiv 2024.04] [Mélange: Cost Efficient Large Language Model Serving by Exploiting GPU Heterogeneity](https://arxiv.org/abs/2404.14527) | UC Berkeley
- [MLSys 2024] [HeteGen: Heterogeneous Parallel Inference for Large Language Models on Resource-Constrained Devices](https://arxiv.org/abs/2403.01164) | National University of Singapore
- [ICML 2024] [HEXGEN: Generative Inference of Large Language Model over Heterogeneous Environment](https://arxiv.org/pdf/2311.11514) | HKUST

## Other List
- [ml-systems-papers](https://github.com/byungsoo-oh/ml-systems-papers)
- [Awesome AI System](https://github.com/lambda7xx/awesome-AI-system)
- [Awesome LLM Systems Papers](https://github.com/AmberLJC/LLMSys-PaperList)
- [Awesome-LLM-System-Papers](https://github.com/AmadeusChan/Awesome-LLM-System-Papers?tab=readme-ov-file)
- [Efficient Large Language Model (LLM) Inferencing on GPUs](https://github.com/yinuotxie/Efficient-LLM-Inferencing-on-GPUs?tab=readme-ov-file)
- [Numbers every LLM Developer should know](https://github.com/ray-project/llm-numbers)
- [Tensor Parallelism and Sequence Parallelism: Detailed Analysis](https://insujang.github.io/2024-01-11/tensor-parallelism-and-sequence-parallelism-detailed-analysis/)
- [Efficient Diffusion Models: A Survey](https://github.com/AIoT-MLSys-Lab/Efficient-Diffusion-Model-Survey)
