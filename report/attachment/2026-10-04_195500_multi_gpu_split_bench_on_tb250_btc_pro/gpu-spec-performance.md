# tb250 全 GPU 公称性能・実測対比

BIOSTAR TB250-BTC PRO (Celeron G3930) に搭載した全 GPU の公称性能と実測ベンチ結果の対比。

## 公称性能（実機 CU 数・最大クロックから算出）

| GPU | Arch | コア数 | Max clk | FP32 (公称) | VRAM | メモリ帯域 | TDP |
|---|---|---:|---:|---:|---:|---:|---:|
| **RX 6400** | Navi24 RDNA2 | 768 (12CU) | 2320MHz | **3.57 TFLOPS** | 4GB GDDR6 | 128 GB/s | 53W |
| **Pro WX 2100** | Polaris12 GCN4 | 512 (8CU) | 1219MHz | **1.25 TFLOPS** | 2GB GDDR5 | 48 GB/s | 35W |
| **GT 730** | GK208B Kepler | 384 (2SM) | 954MHz | **0.73 TFLOPS** | 1GB GDDR5 | 40 GB/s | ~25W |
| **GT 710** | GK208B Kepler | 192 (1SM) | 954MHz | **0.37 TFLOPS** | 2GB DDR3 | 14.4 GB/s | ~19W |
| **GT 430** | GF108 Fermi | 96 (2SM) | 1400MHz | **0.27 TFLOPS** | 1GB DDR3 | 28.8 GB/s | 49W |
| HD 610 (iGPU) | KBL GT1 | 96 (12EU) | 1050MHz | ~0.20 TFLOPS | shared | — | — |
| Celeron G3930 | x86 SSE4.2 | 2C | 2900MHz | ~0.02 TFLOPS | — | 34 GB/s (DDR4-2133) | 51W |

算出根拠: 各デバイスの `clinfo` 実測値（Max compute units / Max clock frequency）に、SM/CU あたりコア数（RDNA2・GCN4=64、Kepler GK208B=192、Fermi GF108=48、KBL GT1 EU=8）を乗じて FP32 = コア数 × 2 × 最大クロックで計算。

## 実測 decode（各系統の最深 depth）

| GPU | 実測 decode | モデル / 経路 | FP32比 (RX6400=100) |
|---|---:|---|---:|
| RX 6400 | 58.7 t/s | Qwen3-1.7B Q4_K_M / Vulkan | 100 |
| WX 2100 | 15.7 t/s | 〃 / Vulkan | 35 |
| GT 730 | 9.8 t/s | TinyLlama-1.1B Q4_0 / CUDA sm_35 | 20 |
| GT 710 | 4.5 t/s | 〃 / CUDA sm_35 | 10 |
| GT 430 | 1.4–2.4 t/s | OpenLLaMA-3B Q4_0 / CLBlast ngl 4–10 | 8 |
| CPU | 1.39 t/s | Qwen3-1.7B Q4_K_M | — |

## 所感

- Kepler 2 枚は **FP32 比 (GT730:GT710 = 2:1) と decode 比 (9.8:4.5 ≈ 2.2:1) がほぼ一致** — 演算律速に近い挙動。
- RX 6400 系は FP32 比でも帯域比でも圧倒。Vulkan 系の直接比較は可能。
- GT 430 は公称 0.27 TFLOPS だが、CLBlast オフロードは PCIe 2.0 x1 の転送レイテンシ (1層あたり +55 ms/token) が支配的で、公称性能比以上に実効が悪い。
