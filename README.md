# Unofficial OpenVINO INT4 conversion of z-lab/Qwen3.5-4B-DFlash

**This is not the authors' release.** The model is
[z-lab/Qwen3.5-4B-DFlash](https://huggingface.co/z-lab/Qwen3.5-4B-DFlash), a DFlash draft model for
[Qwen/Qwen3.5-4B](https://huggingface.co/Qwen/Qwen3.5-4B), which its model card describes as "a joint retrain from
Z-Lab and Modal". The research and the training are theirs. This repository only converts their published weights to
OpenVINO's format, so that OpenVINO Model Server can use them.

## Credit

- **DFlash**, the method: Jian Chen, Yesheng Liang and Zhijian Liu, *DFlash: Block Diffusion for Flash Speculative
  Decoding*, [arXiv:2602.06036](https://arxiv.org/abs/2602.06036) (ICML 2026). Code:
  [github.com/z-lab/dflash](https://github.com/z-lab/dflash). Project page:
  [z-lab.ai/projects/dflash](https://z-lab.ai/projects/dflash/).
- **The draft model**: [Z-Lab](https://z-lab.ai) and [Modal](https://modal.com).
  [z-lab/Qwen3.5-4B-DFlash](https://huggingface.co/z-lab/Qwen3.5-4B-DFlash), converted here from revision
  [`9a1996cc`](https://huggingface.co/z-lab/Qwen3.5-4B-DFlash/tree/9a1996ccf887b79ab3af4fcbf8c1d1f4b5658bcf); the same
  weights are published as [modal-labs/Qwen3.5-4B-DFlash](https://huggingface.co/modal-labs/Qwen3.5-4B-DFlash).
- **The base model**: the Qwen team, Alibaba Cloud. [Qwen/Qwen3.5-4B](https://huggingface.co/Qwen/Qwen3.5-4B).
- **The target model this draft was checked with**: OpenVINO's INT4 conversion of the base model,
  [OpenVINO/Qwen3.5-4B-int4-ov](https://huggingface.co/OpenVINO/Qwen3.5-4B-int4-ov).

If you use this draft, please cite DFlash, as its authors ask:

```bibtex
@article{chen2026dflash,
  title   = {{DFlash: Block Diffusion for Flash Speculative Decoding}},
  author  = {Chen, Jian and Liang, Yesheng and Liu, Zhijian},
  journal = {arXiv preprint arXiv:2602.06036},
  year    = {2026}
}
```

and Qwen3.5, as the Qwen team asks:

```bibtex
@misc{qwen3.5,
    title  = {{Qwen3.5}: Towards Native Multimodal Agents},
    author = {{Qwen Team}},
    month  = {February},
    year   = {2026},
    url    = {https://qwen.ai/blog?id=qwen3.5}
}
```

## Not affiliated

This conversion is not affiliated with or endorsed by Z-Lab, Modal, the OpenVINO project or Intel, the Qwen team or
Alibaba Cloud. Any fault in it is this repository's: please report it here, not to them.

**To the authors:** if you would like anything here changed (the credit, the wording, the files) or the repository
taken down, please [open an issue](https://github.com/jmwrockon/qwen35-4b-dflash-openvino/issues) and the maintainer
will do it.

## What it is for

A draft model for DFlash speculative decoding in [OpenVINO Model Server](https://github.com/openvinotoolkit/model_server)
(OVMS), with [OpenVINO/Qwen3.5-4B-int4-ov](https://huggingface.co/OpenVINO/Qwen3.5-4B-int4-ov) as the target. As
z-lab's model card says, it "is not a standalone language model": the draft proposes several tokens at once and the
target verifies them.

OVMS 2026.4.1 loads a draft only as OpenVINO IR. This conversion is published so that the student environment
installer of an applied machine learning course at Drexel University (ECE 310 / ECE 612) can download it, pinned by
SHA-256, instead of converting it on every student's machine.

## Using it with OVMS

Put the files in a folder beside the target's:

```
models/
  OpenVINO/Qwen3.5-4B-int4-ov/    the target
  z-lab/Qwen3.5-4B-DFlash/        this repository's files, plus openvino_model.bin from the release
```

For example:

```
git clone --depth 1 --branch v2026.10.07 https://github.com/jmwrockon/qwen35-4b-dflash-openvino models/z-lab/Qwen3.5-4B-DFlash
curl -L -o models/z-lab/Qwen3.5-4B-DFlash/openvino_model.bin https://github.com/jmwrockon/qwen35-4b-dflash-openvino/releases/download/v2026.10.07/openvino_model.bin
cd models/z-lab/Qwen3.5-4B-DFlash && sha256sum -c SHA256SUMS && cd ../../..
```

Then serve the target with the draft. This is the command line it was checked with; the draft path is relative to the
target's folder:

```
ovms --rest_port 8000 --rest_bind_address 127.0.0.1 --source_model OpenVINO/Qwen3.5-4B-int4-ov --model_repository_path models --task text_generation --target_device GPU --cache_dir cache --draft_model_path ../../z-lab/Qwen3.5-4B-DFlash --enable_prefix_caching false
```

OVMS logs `Draft model strategy: DFlash` when it has picked the draft up. Two things seen while checking it:
- On OVMS 2026.4.0, a request without `max_tokens` crashed the server. On 2026.4.1 it was answered normally.
- With the draft, OVMS returned its greedy answer even when the request set a temperature.

## What was changed

Converted on 2026-10-07 (20:58 UTC) on Windows 11 x64 with OpenVINO Model Server 2026.4.1's pull mode, which exports
through optimum-intel:

```
ovms --pull --source_model z-lab/Qwen3.5-4B-DFlash --model_repository_path <dir> --task text_generation --target_device GPU --weight-format int4 --extra_quantization_params "--all-layers --task text-generation-with-past"
```

OVMS ran, as its log records:

```
optimum-cli export openvino --model z-lab/Qwen3.5-4B-DFlash --trust-remote-code --weight-format int4 --all-layers --task text-generation-with-past <dir>\z-lab\Qwen3.5-4B-DFlash
```

- **Source:** z-lab/Qwen3.5-4B-DFlash at `9a1996ccf887b79ab3af4fcbf8c1d1f4b5658bcf`; its `model.safetensors`
  (bfloat16, 1,268,859,081 bytes) has SHA-256 `1eb221d36abb13a5f1b972f8d031a9723fad8cbb7d275abe548b60e77577eb42`.
- **Tools**, installed into OVMS's own Python 3.12.10 from the export requirements its documentation names
  (`docs/pull_optimum_cli.md`): OVMS 2026.4.1 (build cfc9ac72), optimum-intel 2.2.0, optimum 2.3.0, openvino 2026.4.0
  (2026.4.0-22959-99c81491cc3), openvino-tokenizers 2026.4.0.0, nncf 3.4.0, transformers 5.5.4, torch 2.14.1+cpu,
  huggingface_hub 1.21.0.
- **Compression**, as recorded in `openvino_model.xml`: NNCF weight compression to `int4_asym`, group size 128,
  ratio 1.0, all layers. Data-free: no calibration set, no AWQ, GPTQ or scale estimation.
- **Reproducible:** an earlier export on the same machine (2026-10-06, `optimum-cli` run directly, optimum-intel
  2.2.0.dev0 with the same openvino, nncf and transformers) produced a byte-identical `openvino_model.bin`.
- **Files:** every model file here was written by the conversion. [NOTICE](NOTICE) lists what each one is and how
  `config.json` differs from the original.
  - `config.json` has one change by hand: the exporter had pointed `auto_map` at optimum-intel's model patcher by a
    path on the machine that converted it, and that key is removed. `auto_map` is only for remote-code loading in
    Python `transformers`; OVMS does not use it.
  - OVMS's pull also wrote a `graph.pbtxt`, which would serve the folder as a model of its own. It is left out, because
    a draft is not served on its own.

## Files

The weights are a release asset, because GitHub does not accept files over 100 MiB. `SHA256SUMS` lists the same
hashes.

| File | Bytes | SHA-256 | Where |
|---|---:|---|---|
| `config.json` | 1,253 | `a6d4f93a8ed8da52c90290c80684a0f4992358e5872b7ee6341005a84a5593f5` | repository |
| `generation_config.json` | 121 | `a5258b78e12c64e8e98a9d49e08230388f3763f49af3aaa44d0f9617b34c30c8` | repository |
| `openvino_config.json` | 476 | `2e672785585d26dd2347331b9a52848cce46d197561b0200e896602410345e65` | repository |
| `openvino_detokenizer.bin` | 21 | `107e29a1c6741ecb589447fdf29bcea64db8eb77f89822713afa47597c29eaa5` | repository |
| `openvino_detokenizer.xml` | 7,255 | `640fd4a3a73280065a297bb074537061e8ec9deda5c8da42784575070a7f162c` | repository |
| `openvino_model.bin` | 329,595,604 | `cfcf6b3cfa6423c156e1be9f5daf3e4f540b900ef6fc1ad2a7d16968b6726457` | [release v2026.10.07](https://github.com/jmwrockon/qwen35-4b-dflash-openvino/releases/tag/v2026.10.07) |
| `openvino_model.xml` | 878,354 | `29d2188db31b65f38529e7f7e7eac30ff382e33c753adfd0672460a1cd8c771d` | repository |
| `openvino_tokenizer.bin` | 192 | `c419adbd3a5a9820a42acc58e98192563d267a448b88fab8a03f60ab2e71ed50` | repository |
| `openvino_tokenizer.xml` | 25,069 | `ba5e4909708a60e9a9aeec49b38a7d569467ccbee9394bb78ee786640cb68ebb` | repository |
| `tokenizer.json` | 1,629 | `716702a1fcdec1149657dc57cd79277661fd38d632ea2bb1f3bc45ea94a8d539` | repository |
| `tokenizer_config.json` | 301 | `41efc93f4aee1c5abd4737c1df5dd7ef274102acf433da9770b92f2106975903` | repository |

## How it was checked

On one machine: Intel Core Ultra 5 225H with Arc 130T integrated graphics, Windows 11, OVMS 2026.4.1, the target
OpenVINO/Qwen3.5-4B-int4-ov (revision `f2377291`) on the GPU, served with the command line above.

- **Correct:** 50 of 50 on the course's 50-question check set, and the installer's probe passed in 3 runs of 3.
  Served from exactly the files published here, every one of those answers was identical to the earlier export's.
- **Speed:** in two sessions on 2026-10-07, before and after a reboot, with that earlier export: for a 767-token
  answer to the installer's probe with thinking on, OVMS with this draft was 1.40 to 1.45 times as fast as OVMS with
  the MTP head built into OpenVINO/Qwen3.5-4B-int4-ov, and 2.52 to 2.69 times as fast as Ollama serving Qwen3.5-4B on
  the same GPU. Served from these files, it fell inside the range of those sessions. Other hardware will differ.

For the draft's design and the authors' own benchmarks (SGLang on an NVIDIA B200), see
[z-lab's model card](https://huggingface.co/z-lab/Qwen3.5-4B-DFlash).

## Licence

[Apache License 2.0](LICENSE), the licence z-lab gives the original model in its model card. The original repository
has no LICENSE file of its own, so [LICENSE](LICENSE) is the Apache Software Foundation's text
(<https://www.apache.org/licenses/LICENSE-2.0.txt>). The base model, Qwen3.5-4B, is also Apache-2.0
(Copyright 2026 Alibaba Cloud). DFlash's code, which is not included here, is under the MIT License. [NOTICE](NOTICE)
states the changes made, as section 4 of the licence requires.
