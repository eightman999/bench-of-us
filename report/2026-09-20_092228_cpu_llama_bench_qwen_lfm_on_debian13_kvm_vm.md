# Debian 13 KVM VM（CPU のみ）で Qwen / LFM を llama-bench

- **作成者**: eightman999
- **作成日**: 2026-09-20

## 概要

KVM 仮想マシン（Debian GNU/Linux 13、Intel Xeon Processor 8 vCPU、メモリ 15 GiB、GPU なし）で、llama.cpp の `llama-bench` により Qwen2.5-3B-Instruct Q4_K_M、LFM2.5-2.6B QAD-Q4_0、Qwen3-4B-Instruct-2507 Q4_K_M を測定しました。
GPU が無いため llama-split-bench は使わず、CPU バックエンド（`ngl=0`、threads=8、pp512 / tg64、繰り返し 3 回）です。
同条件では prefill は Qwen2.5-3B が最速（pp512 308.83 t/s）、生成は LFM2.5-2.6B が最速（tg64 41.59 t/s）でした。

## ハードウェア

| 項目 | 内容 |
|------|------|
| コンピュータ / マザーボード | KVM 仮想マシン（DMI の製品名・マザーボード型番は取得不可） |
| GPU | なし（CPU のみ） |
| GPU 接続 | 該当なし |
| CPU | Intel Xeon Processor (KVM)、8 vCPU（1 ソケット、1 thread/core） |
| メモリ | 15 GiB、swap なし |
| 電源 | 不明（仮想マシンのため） |

## ソフトウェア環境

| 項目 | 内容 |
|------|------|
| OS | Debian GNU/Linux 13 (trixie) / Linux 6.12.94+ |
| GPU ドライバ | 該当なし（GPU なし） |
| llama.cpp | ggml-org/llama.cpp commit 6d9c82e（0.4.0-dev）、CPU バックエンド、`build-noamx`、Release |

## ベンチマーク

GPU が無いため llama-split-bench は未使用です。llama.cpp 同梱の `llama-bench` で測定しました。

### 条件

| 項目 | 内容 |
|------|------|
| ツール | llama-bench（llama.cpp 同梱、commit 6d9c82e） |
| モデル | Qwen2.5-3B-Instruct Q4_K_M（1.79 GiB、3.09B） / LFM2.5-2.6B QAD-Q4_0（1.48 GiB、2.70B） / Qwen3-4B-Instruct-2507 Q4_K_M（2.32 GiB、4.02B） |
| 測定モード | CPU のみ（`--n-gpu-layers 0`）、threads=8 |
| ctx / stages | 該当なし（llama-bench の固定長 pp512 / tg64） |
| KV キャッシュ | 既定（未指定） |
| 投機的デコード | なし |
| 繰り返し | 3 回（`r=3`） |

### 結果

単位は t/s（平均 ± 標準偏差）。

| モデル | 量子化 | サイズ | params | pp512 | tg64 |
|--------|--------|--------|--------|------:|-----:|
| Qwen2.5-3B-Instruct | Q4_K_M | 1.79 GiB | 3.09B | 308.83 ± 2.11 | 21.89 ± 0.35 |
| LFM2.5-2.6B | QAD-Q4_0 | 1.48 GiB | 2.70B | 234.59 ± 2.51 | 41.59 ± 0.98 |
| Qwen3-4B-Instruct-2507 | Q4_K_M | 2.32 GiB | 4.02B | 228.40 ± 1.09 | 17.30 ± 0.51 |

### 所感

- GPU 無しの 8 vCPU / 15 GiB KVM でも、Q4 系 2.6–4B は対話用途に耐える生成速度が出ました。
- 同条件では LFM2.5-2.6B QAD-Q4_0 の tg が最も高く（約 42 t/s）、Qwen2.5-3B は prefill が最速（約 309 t/s）でした。
- Qwen3-4B は tg が約 17 t/s と 3 モデル中いちばん遅い一方、パラメータ数は最大です。
- 常駐させるなら RAM 余裕を見て 3B 帯が無難です（測定時の空きはおおよそ 9.8 GiB available）。

## 添付

- [results-summary.json](attachment/2026-09-20_092228_cpu_llama_bench_qwen_lfm_on_debian13_kvm_vm/results-summary.json)
- [bench_qwen25_3b.md](attachment/2026-09-20_092228_cpu_llama_bench_qwen_lfm_on_debian13_kvm_vm/bench_qwen25_3b.md)
- [bench_lfm25.md](attachment/2026-09-20_092228_cpu_llama_bench_qwen_lfm_on_debian13_kvm_vm/bench_lfm25.md)
- [bench_qwen3_4b.md](attachment/2026-09-20_092228_cpu_llama_bench_qwen_lfm_on_debian13_kvm_vm/bench_qwen3_4b.md)
