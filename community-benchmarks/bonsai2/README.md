# Bonsai 2 (ternary)

Results for the **Bonsai 2** family (27B and smaller, as they get released). Bonsai 2 27B is
built on Qwen3.8-27B and keeps ternary weights end to end; it is the current generation, separate
from the older `bonsai/` (1-bit) and `ternary-bonsai/` (first-generation ternary) families.

## Formats

| File | ggml type | Runs on |
|---|---|---|
| `Ternary-Bonsai-2-27B-PTQ1_0.gguf` (5.53 GiB) | `PTQ1_0` (143, dense trits, group 128) | PrismML fork only |
| `Ternary-Bonsai-2-27B-PQ2_0.gguf` (7.21 GiB) | `PQ2_0` (142, 2-bit slots, group 128) | PrismML fork only |
| `Ternary-Bonsai-2-27B-Q2_0-prism-fork-required.gguf` (dev repo, 7.10 GiB) | `Q2_0` (42, group 64) | loads on mainline llama.cpp but produces garbage until the activation transform lands upstream; correct on the fork |

The two production files live in [prism-ml/Ternary-Bonsai-2-27B-gguf](https://huggingface.co/prism-ml/Ternary-Bonsai-2-27B-gguf);
the dev file is in [prism-ml/Ternary-Bonsai-2-27B-gguf-dev](https://huggingface.co/prism-ml/Ternary-Bonsai-2-27B-gguf-dev).

## Results

Sorted by decode speed (TG128). Format column shows the packing used.

| Hardware | Backend | Format | PP512 (t/s) | TG128 (t/s) | TG with MTP (t/s) | Details |
|---|---|---:|---:|---:|---:|---|
| NVIDIA RTX 5070 Ti Laptop 12 GB | llama.cpp CUDA (Windows) | PQ2_0 | 1,135 | 49.0 | 70-75 (code) | [link](cuda-rtx5070ti-laptop-windows.md) |
| NVIDIA RTX 5070 Ti Laptop 12 GB | llama.cpp CUDA (Windows) | PTQ1_0 | 527 | 49.0 | | [link](cuda-rtx5070ti-laptop-windows.md) |
| NVIDIA RTX 5070 Ti Laptop 12 GB | llama.cpp CPU (Windows) | PTQ1_0 | (see entry) | 4.9 | | [link](cuda-rtx5070ti-laptop-windows.md) |

## Submitting

Same flow as the other families, with a couple of Bonsai 2 specific notes:

- `setup.sh` defaults to `BONSAI_FAMILY=bonsai2` (Bonsai 2 27B, `PQ2_0`). To get the second packing run
  `hf download prism-ml/Ternary-Bonsai-2-27B-gguf Ternary-Bonsai-2-27B-PTQ1_0.gguf --local-dir models`, and for
  the mainline-format file see the `-dev` repo above.
- Both production packings need the PrismML fork binaries; mainline llama.cpp rejects the types.
- Report which packing you used. `PQ2_0` is faster at prompt processing, `PTQ1_0` is smaller and is the one that
  fits long contexts on 12 GB cards.
- If you measured speculative decoding, say which path: the embedded MTP head (`--spec-type draft-mtp` with a
  file that contains the MTP layer, or a separate `-md` drafter) or a DSpark/DFlash drafter. Note that MTP forces
  a single server slot (`-np 1`).
- Long-context numbers are welcome: the 4-bit KV cache with a calibrated `--kv-mean-center` bias is what makes
  128K fit on 12 GB.

Open a PR with your file in this folder and a row in this table (and in the root index if you like).
