# Ban Topics Scanner

It is designed to inspect the outputs generated by Language Learning Models and to flag or restrict responses that delve
into predefined banned topics, such as religion. This ensures that the outputs align with community guidelines and do
not drift into potentially sensitive or controversial areas.

## Attack

Even with controlled prompts, LLMs might produce outputs touching upon themes or subjects that are considered sensitive,
controversial, or outside the scope of intended interactions. Without preventive measures, this can lead to outputs that
are misaligned with the platform's guidelines or values.

## How it works

It relies on the capabilities of the following models:

- [MoritzLaurer/deberta-v3-base-zeroshot-v1](https://huggingface.co/MoritzLaurer/deberta-v3-base-zeroshot-v1)
- [MoritzLaurer/deberta-v3-large-zeroshot-v1](https://huggingface.co/MoritzLaurer/deberta-v3-large-zeroshot-v1)

These models aid in identifying the underlying theme or topic of an output, allowing the scanner to cross-check it against
a list of banned topics.

## Usage

```python
from llm_guard.output_scanners import BanTopics

scanner = BanTopics(topics=["violence"], threshold=0.5)
sanitized_output, is_valid, risk_score = scanner.scan(prompt, model_output)
```

## Optimizations

### ONNX

The scanner can run on ONNX Runtime, which provides a significant performance boost on CPU instances. It will fetch Laiyer's ONNX converted models from [Hugging Face Hub](https://huggingface.co/laiyer).

To enable it, install the `onnxruntime` package:

```sh
pip install llm-guard[onnxruntime]
```

And set `use_onnx=True`.

### Use smaller models

You can rely on base model variant (default) to reduce the latency and memory footprint.

## Benchmarks

Environment:

- Platform: Amazon Linux 2
- Python Version: 3.11.6

Run the following script:

```sh
python benchmarks/run.py output BanTopics
```

Results:

| Instance                         | Input Length | Test Times | Latency Variance | Latency 90 Percentile | Latency 95 Percentile | Latency 99 Percentile | Average Latency (ms) | QPS    |
|----------------------------------|--------------|------------|------------------|-----------------------|-----------------------|-----------------------|----------------------|--------|
| AWS m5.xlarge                    | 89           | 5          | 2.39             | 485.00                | 509.32                | 528.78                | 435.82               | 204.21 |
| AWS m5.xlarge with ONNX          | 89           | 5          | 0.09             | 165.61                | 170.05                | 173.60                | 155.90               | 570.87 |
| AWS g5.xlarge GPU                | 89           | 5          | 35.44            | 331.25                | 425.26                | 500.46                | 142.77               | 623.37 |
| Azure Standard_D4as_v4           | 89           | 5          | 3.91             | 547.06                | 577.87                | 602.53                | 483.73               | 183.99 |
| Azure Standard_D4as_v4 with ONNX | 89           | 5          | 0.06             | 176.34                | 179.65                | 182.30                | 168.16               | 529.25 |
