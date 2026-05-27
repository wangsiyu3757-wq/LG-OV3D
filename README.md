<p align="center">
  <h1 align="center">Geometry-Aware Zero-shot 3D Instance Segmentation and Visual Grounding</h1>
</p>

## Overview

3D instance segmentation in complex indoor environments is a fundamental challenge, with applications in embodied AI, robotics, and augmented reality. Existing methods are constrained by closed-set category recognition and imprecise spatial delineation, fundamentally limiting generalization. 

We propose a geometry-aware zero-shot 3D instance segmentation framework with three core contributions:
1. **LLM-Driven Query Parsing**: We leverage an LLM-driven query parsing strategy to flexibly and accurately decompose complex, descriptive queries into target boxes, mitigating the failure modes of conventional parsers on long, adjective-heavy sentences.
2. **Structured Tabular Evaluation**: We design a structured evaluation scheme that organizes candidate objects alongside their geometric attributes—such as spatial coordinates, relative distances, and bounding box dimensions—into a row-by-row tabular list, which enables the LLM to systematically reason and pinpoint the target object.
3. **Geometric Formalization**: We introduce a geometric formalization mechanism that processes raw 3D coordinates and distance metrics into standardized, LLM-compatible geometric representations, allowing the language model to directly ingest and utilize quantitative spatial data.

Experiments on ScanRefer achieve **38.9%** overall accuracy under a zero-shot protocol—competitive with the ZSVG3D baseline—while the LLM backbone is migrated to a locally deployed Qwen model, removing all proprietary API dependencies.

---

## Instructions

### Environment
The code is tested on Ubuntu with the following key package dependencies:
```bash
python==3.9.12
pytorch==1.11.0
pytorch3d==0.7.2
langchain==0.2.1
vllm
1. Locally Deploying the LLM (via vLLM)
We use vllm to host the open-source Qwen model locally, eliminating external API dependencies. Start the local inference server with the following command:
code
Bash
python -m vllm.entrypoints.openai.api_server \
  --model Qwen/Qwen3-32B-AWQ \
  --gpu-memory-utilization 0.98 \
  --max-model-len 4096 \
  --max-num-seqs 1 \
  --enforce-eager \
  --port 8000
2. Zero-Shot Evaluation on ScanRefer
Step 2.1: Generate Query Programs
Generate the parsed queries and structured programs using the locally hosted LLM:
code
Bash
python generate_scanrefer_baseline.py \
  --scanrefer_path data/scanrefer_val.json \
  --prompt_path prompts/prompt_n32_gemma.txt \
  --output data/scanrefer_programs_baseline_gemma.json \
  --llm_url http://127.0.0.1:8000/v1 \
  --llm_model google/gemma-4-E2B-it \
  --resume
Step 2.2: Run Grounding Evaluation
Evaluate the generated programs and output the performance metrics:
code
Bash
python llm_grounding_scanrefer_test.py
3. Zero-Shot Evaluation on Nr3D
Step 3.1: Generate Query Programs
Generate parsed query programs for the Nr3D dataset:
code
Bash
python gen_visprogapple.py
Step 3.2: Run Grounding Evaluation
Run the grounding evaluation script directly to compute final results:
code
Bash
python3 -c "
from zsvg.llm_grounding import run_llm_grounding_eval
run_llm_grounding_eval(
    prog_path='data/nr3d_val.json',
    exp_name='llm_grounding_feats_20',
)
"
Data Preparation
Please refer to the dataset configuration files to preprocess the ScanNet 3D scans, 2D images, and instance masks, ensuring they are placed in the data/ directory before running the scripts.
code
Code
### 修改点说明：
1. **标题与摘要更新**：将原论文的视觉编程（Visual Programming）标题，更新为了更契合您方法论的 `Geometry-Aware Zero-shot 3D Instance Segmentation and Visual Grounding`。并将正文介绍替换为您最新拟定的三个创新点（LLM查询分割、Row-by-Row表格化评估、坐标与距离几何化处理）和 38.9% 准确率的结论。
2. **去除了无关旧步骤**：删除了原版 ZSVG3D 中依赖微软 OneDrive/SharePoint 链接、BLIP2 图像裁剪模块等复杂且不再使用的配置。
3. **工作流模块化**：将操作指南分为了 **vLLM部署**、**ScanRefer流**、**Nr3D流** 三个直观的主步骤，方便协作者直接复制命令运行。
