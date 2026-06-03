# Vietnamese Mathematical Word Problem Solving with GPT-2

## Overview

This project investigates the use of a Vietnamese GPT-2 language model for solving **Mathematical Word Problems (MWPs)**. Instead of modifying the model architecture, we focus on improving reasoning performance through a collection of data-centric and training-centric techniques, including:

* Chain-of-Thought (CoT) Reformation
* Sample Filtering
* Curriculum Learning
* Prompt Design
* Self-Consistency Decoding

The system was developed as part of a study on mathematical reasoning in Vietnamese language models.

---

## Project Objectives

The primary goal of this project is to improve the mathematical reasoning capability of a compact Vietnamese language model while maintaining reasonable training and inference costs.

Specifically, we investigate:

* Whether structured reasoning traces improve learning.
* The impact of curriculum learning on mathematical reasoning.
* The effectiveness of prompt engineering strategies.
* The contribution of self-consistency decoding.
* The importance of data quality in fine-tuning small language models.

---

## Resources

The system is built upon **NlpHUST/gpt2-vietnamese**, a Vietnamese GPT-2 model pretrained on the OSCAR Vietnamese corpus using causal language modeling. The model follows the standard GPT-2 architecture (12 layers, hidden size 768) and was selected due to its lightweight size and suitability for fine-tuning under limited training resources.

### Dataset
https://www.kaggle.com/datasets/kimanh2002/dataset-math

### Base Model
https://www.kaggle.com/datasets/kimanh2002/nlphustgpt2-vietnamese

### Hugging Face Model Card
https://huggingface.co/NlpHUST/gpt2-vietnamese

---

## Dataset

The training and validation data consist of Vietnamese mathematical word problems derived from datasets commonly used in mathematical reasoning research, including variants originating from the MetaMath framework.

The corpus contains multiple problem categories such as:

* Arithmetic
* Ratios and proportions
* Percentages
* Algebra
* Geometry
* Multi-step reasoning

The dataset includes several transformed variants:

| Variant   | Description           |
| --------- | --------------------- |
| AnsAug    | Answer augmentation   |
| Rephrased | Question paraphrasing |
| SV        | Backward reasoning    |
| FOBAR     | Backward reasoning    |

Each sample contains:

* Problem statement
* Reasoning process
* Final answer

---

## Methodology

### 1. Sample Filtering

The original training dataset contained **95,400 samples**. To satisfy the project's training-time constraint (maximum 3 hours) and reduce redundancy, a filtering pipeline was applied before training.

The filtering process included:

- Removing extremely short or excessively long samples.
- Verifying the existence of both problem statements and solutions.
- Ensuring responses contain a recognizable final-answer marker (e.g., `####`, `Đáp án là`, `Câu trả lời là`).
- Removing exact duplicate questions.

A large portion of the dataset consisted of duplicated questions generated through data augmentation. After filtering, the dataset size was reduced from **95,400** to **53,184** training samples (approximately **44\% reduction**).

This preprocessing step significantly reduced training cost while preserving mathematical diversity and reasoning coverage.

---

### 2. Chain-of-Thought Reformation

Original reasoning traces contain highly heterogeneous natural language explanations.

To reduce reasoning variability, we transform solution traces into compact equation-oriented reasoning chains.

Example:

Original:

```text
Nếu Alexis có 60 quả xoài và cô ấy có số xoài gấp bốn lần Dilan và Ashley cộng lại thì Dilan và Ashley có tổng cộng 60/4 = 15 quả xoài. Tổng số xoài mà tất cả các bạn cộng lại là 60 + 15 = 75. #### 75 Đáp án là: 75
```

Reformatted:

```text
60/4 = 15\n60 + 15 = 75\n#### 75
```

This process reduces linguistic noise while preserving mathematical transformations.

---

### 3. Curriculum Learning

Training data are sorted according to an estimated difficulty score.

Difficulty is determined using:

* Solution length
* Problem category
* Backward-reasoning indicators

The dataset is divided into:

* Easy
* Medium
* Hard

Training proceeds through three cumulative stages:

| Stage   | Data                 |
| ------- | -------------------- |
| Stage 1 | Easy                 |
| Stage 2 | Easy + Medium        |
| Stage 3 | Easy + Medium + Hard |

Each stage uses its own optimization schedule.

| Stage  | Learning Rate | Warmup Ratio |
| ------ | ------------- | ------------ |
| Easy   | 4e-3          | 0.15         |
| Medium | 3e-3          | 0.10         |
| Hard   | 2e-3          | 0.05         |

---

### 4. Prompt Design

A lightweight prompt template is used during training.

#### Prompt Instruction

```text
Tính kết quả theo từng bước phép tính toán học.
Không dùng lời giải thích.
Kết thúc bằng '#### <số>'.
```

Since problem types are not provided during evaluation, only one unified instruction style is employed.

---

### 5. Self-Consistency Decoding

During inference, the model does not rely on a single generated solution. Instead, it generates multiple reasoning paths and selects the most consistent answer through majority voting.

For each problem:

- Temperature = 0.5
- Top-p = 0.85
- Number of generated solutions = 20

Each response is processed by an answer extraction module to obtain the final numerical answer. The answer that appears most frequently among all valid generations is selected as the final prediction.

Example:

```text
42, 42, 41, 42, 40, ...
```

Final prediction:

```text
42
```

After voting, the system returns the first reasoning path that produced the selected answer. If no valid answer can be extracted, the first generated response is used as a fallback.

---

## Experimental Results

### Incremental Improvements

| Configuration         | Validation Score |
| --------------------- | ---------------- |
| Baseline              | 0.0344           |
| + CoT Reformation     | 0.1979           |
| + Self-Consistency    | 0.2023           |
| + Sample Filtering    | 0.2744           |
| + Curriculum Learning | 0.3084           |
| Final System          | 0.3663           |

Results indicate that data quality and curriculum design contribute substantially to overall performance.

---

## Limitations

Several limitations remain:

* Rule-based CoT extraction cannot perfectly process all reasoning styles.
* GPT-2 has limited reasoning capacity compared to larger modern language models.
* Some mathematical expressions remain difficult to normalize automatically.
* The curriculum schedule was only partially explored due to computational constraints.

---

## Future Work

Potential future directions include:

* LLM-assisted CoT reformation
* More advanced curriculum scheduling strategies
* Larger Vietnamese language models
* Tool-augmented reasoning systems
* Evaluation on additional mathematical benchmarks

---

## Authors

This project was developed as part of a study on Vietnamese Mathematical Word Problem Solving using GPT-2 and data-centric reasoning enhancement techniques.
