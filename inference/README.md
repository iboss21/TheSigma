```
═══════════════════════════════════════════════════════════════════════════════
██╗     ██╗  ██╗██████╗  ██████╗ ██████╗ ██████╗ ███████╗      █████╗ ██╗    
██║     ╚██╗██╔╝██╔══██╗██╔════╝██╔═══██╗██╔══██╗██╔════╝     ██╔══██╗██║    
██║      ╚███╔╝ ██████╔╝██║     ██║   ██║██████╔╝█████╗ █████╗███████║██║    
██║      ██╔██╗ ██╔══██╗██║     ██║   ██║██╔══██╗██╔══╝ ╚════╝██╔══██║██║    
███████╗██╔╝ ██╗██║  ██║╚██████╗╚██████╔╝██║  ██║███████╗     ██║  ██║██║    
╚══════╝╚═╝  ╚═╝╚═╝  ╚═╝ ╚═════╝ ╚═════╝ ╚═╝  ╚═╝╚══════╝     ╚═╝  ╚═╝╚═╝    
                                                                                
███████╗███████╗███████╗██╗  ██╗                                              
██╔════╝██╔════╝██╔════╝██║ ██╔╝                                              
███████╗█████╗  █████╗  █████╔╝                                               
╚════██║██╔══╝  ██╔══╝  ██╔═██╗                                               
███████║███████╗███████╗██║  ██╗                                              
╚══════╝╚══════╝╚══════╝╚═╝  ╚═╝                                              
═══════════════════════════════════════════════════════════════════════════════
🐺 LXRCore-AI-Seek - Inference Scripts
   Powered by The Land of Wolves 🐺 | მგლების მიწა
═══════════════════════════════════════════════════════════════════════════════
```

# Inference Directory

## 📂 Directory Contents

This directory contains the core inference implementation for LXRCore-AI-Seek:

### Python Scripts

- **`model.py`** - Core Transformer model architecture
  - Multi-head Latent Attention (MLA)
  - Mixture-of-Experts (MoE) implementation
  - FP8 quantization support
  - Distributed processing

- **`generate.py`** - Text generation script
  - Interactive chat mode
  - Batch inference
  - Temperature-based sampling
  - Multi-GPU support

- **`convert.py`** - Weight conversion utility
  - HuggingFace format → Inference format
  - Model parallelism sharding
  - Expert distribution

- **`fp8_cast_bf16.py`** - Precision conversion
  - FP8 → BF16 conversion
  - Hardware compatibility
  - Weight dequantization

- **`kernel.py`** - Custom CUDA kernels
  - Triton-based kernels
  - FP8 quantization ops
  - Optimized GEMM operations

### Configuration Files

- **`configs/`** - Model configuration files
  - `config_671B.json` - Full 671B parameter model
  - `config_236B.json` - Smaller variant
  - `config_16B.json` - Compact variant
  - `config_v3.1.json` - Latest version config

### Dependencies

- **`requirements.txt`** - Python package dependencies
  - PyTorch 2.4.1
  - Triton 3.0.0
  - Transformers 4.46.3
  - Safetensors 0.4.5

## 🚀 Quick Start

### Installation

```bash
# Navigate to this directory
cd inference

# Install dependencies
pip install -r requirements.txt
```

### Basic Usage

#### Interactive Chat

```bash
torchrun --nnodes 2 --nproc-per-node 8 \
    --node-rank $RANK --master-addr $ADDR \
    generate.py \
    --ckpt-path /path/to/model \
    --config configs/config_671B.json \
    --interactive \
    --temperature 0.7 \
    --max-new-tokens 200
```

#### Batch Inference

```bash
torchrun --nnodes 2 --nproc-per-node 8 \
    --node-rank $RANK --master-addr $ADDR \
    generate.py \
    --ckpt-path /path/to/model \
    --config configs/config_671B.json \
    --input-file prompts.txt
```

### Weight Conversion

```bash
# Convert HuggingFace weights
python convert.py \
    --hf-ckpt-path /path/to/hf/weights \
    --save-path /path/to/converted \
    --n-experts 256 \
    --model-parallel 16
```

### Precision Conversion

```bash
# Convert FP8 to BF16
python fp8_cast_bf16.py \
    --input-fp8-hf-path /path/to/fp8 \
    --output-bf16-hf-path /path/to/bf16
```

## 🏗️ Architecture Overview

### Model Structure

```
LXRCore-AI-Seek (671B parameters)
├── Embedding Layer (0.9B params)
├── Transformer Layers × 61
│   ├── Multi-head Latent Attention
│   ├── MoE Feed-Forward
│   │   ├── Routed Experts: 256
│   │   ├── Active Experts: 6 per token
│   │   └── Shared Experts: 2
│   └── Layer Normalization
├── Output Layer (0.9B params)
└── MTP Modules (11.5B params)
```

### Key Features

- **FP8 Precision**: Native support with 128×128 block scaling
- **Model Parallelism**: Distributed across multiple GPUs
- **Expert Routing**: Efficient MoE implementation
- **Long Context**: 128K token context window
- **Multi-Token Prediction**: Speculative decoding support

## 📊 Performance Characteristics

### Hardware Requirements

| Configuration | GPUs | VRAM | Notes |
|--------------|------|------|-------|
| Minimum | 2× A100 80GB | 160GB | FP8 mode required |
| Recommended | 2× H100 80GB | 160GB | Best performance |
| Optimal | 4× H100 80GB | 320GB | Multi-node capable |

### Inference Speed

- **Latency**: ~50-100ms per token (depends on hardware)
- **Throughput**: ~1000-2000 tokens/sec (batch inference)
- **Context**: Supports up to 128K tokens

## 🔧 Configuration

### Model Configurations

Different model sizes are available:

```bash
# Full model (671B)
--config configs/config_671B.json

# Medium model (236B)
--config configs/config_236B.json

# Compact model (16B)
--config configs/config_16B.json
```

### Generation Parameters

```python
--temperature 0.7      # Sampling temperature (0.0-2.0)
--max-new-tokens 200   # Maximum tokens to generate
--top-p 0.95           # Nucleus sampling threshold
--top-k 50             # Top-k sampling
```

## 🛠️ Development

### Code Structure

```
inference/
├── model.py           # Core model implementation
├── generate.py        # Generation utilities
├── convert.py         # Weight conversion
├── fp8_cast_bf16.py  # Precision conversion
├── kernel.py          # Custom kernels
├── requirements.txt   # Dependencies
└── configs/          # Configuration files
```

### Adding Custom Kernels

Custom Triton kernels can be added to `kernel.py`:

```python
@triton.jit
def custom_kernel(...):
    # Your kernel implementation
    pass
```

## 📚 Related Documentation

- [Main README](../README.md) - Project overview
- [Installation Guide](../docs/installation.md) - Setup instructions
- [Configuration Guide](../docs/configuration.md) - Detailed config
- [Performance Guide](../docs/performance.md) - Optimization tips

## ⚠️ Important Notes

1. **Linux Only**: Mac and Windows are not supported
2. **Python 3.10+**: Required for all scripts
3. **CUDA 12.1+**: Recommended for best performance
4. **Memory**: Ensure sufficient VRAM for your model size

## 🐛 Troubleshooting

### Common Issues

**Out of Memory**
```bash
export PYTORCH_CUDA_ALLOC_CONF=max_split_size_mb:512
```

**NCCL Errors**
```bash
export NCCL_DEBUG=INFO
export NCCL_IB_DISABLE=1
```

**Import Errors**
```bash
pip install --force-reinstall -r requirements.txt
```

## 📞 Support

- **Discord**: https://discord.gg/CrKcWdfd3A
- **GitHub Issues**: https://github.com/iboss21/TheSigma/issues
- **Documentation**: https://github.com/iboss21/TheSigma/tree/main/docs

---

<div align="center">
  <p><strong>🐺 Inference Engine for The Land of Wolves 🐺</strong></p>
  <p>Made with ❤️ by iBoss21 & The Lux Empire</p>
</div>
