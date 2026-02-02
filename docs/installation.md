```
═══════════════════════════════════════════════════════════════════════════════
🐺 LXRCore-AI-Seek - Installation Guide
   The Land of Wolves 🐺 | მგლების მიწა
═══════════════════════════════════════════════════════════════════════════════
```

# Installation Guide

## 📋 Table of Contents

1. [System Requirements](#system-requirements)
2. [Prerequisites](#prerequisites)
3. [Installation Steps](#installation-steps)
4. [Verification](#verification)
5. [Troubleshooting](#troubleshooting)

---

## 💻 System Requirements

```
═══════════════════════════════════════════════════════════════════════════════
█████ MINIMUM REQUIREMENTS
═══════════════════════════════════════════════════════════════════════════════
```

### Hardware Requirements

#### For Inference (Minimum)
- **GPU**: NVIDIA A100 80GB (or equivalent) × 2
- **RAM**: 256GB system memory
- **Storage**: 2TB NVMe SSD
- **CPU**: 32+ cores recommended
- **Network**: 10Gbps for multi-node setups

#### For Inference (Recommended)
- **GPU**: NVIDIA H100 80GB × 2-4
- **RAM**: 512GB+ system memory
- **Storage**: 4TB+ NVMe SSD
- **CPU**: 64+ cores
- **Network**: 25Gbps+ for multi-node setups

### Software Requirements

- **Operating System**: Linux (Ubuntu 20.04+ or similar)
  - ⚠️ **Mac and Windows are NOT supported**
- **Python**: 3.10 or 3.11
- **CUDA**: 12.1+ recommended
- **Docker**: Optional, but recommended for containerized deployment

> [!IMPORTANT]
> This model requires significant computational resources. Ensure your hardware meets the minimum requirements before proceeding.

---

## 🔧 Prerequisites

```
═══════════════════════════════════════════════════════════════════════════════
█████ DEPENDENCIES & SETUP
═══════════════════════════════════════════════════════════════════════════════
```

### 1. Install Python 3.10+

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install python3.10 python3.10-venv python3.10-dev

# Verify installation
python3.10 --version
```

### 2. Install CUDA Toolkit

```bash
# Download and install CUDA 12.1+
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-keyring_1.0-1_all.deb
sudo dpkg -i cuda-keyring_1.0-1_all.deb
sudo apt-get update
sudo apt-get install cuda

# Verify CUDA installation
nvcc --version
nvidia-smi
```

### 3. Install Git LFS (for model weights)

```bash
sudo apt-get install git-lfs
git lfs install
```

---

## 📦 Installation Steps

```
═══════════════════════════════════════════════════════════════════════════════
█████ STEP-BY-STEP INSTALLATION
═══════════════════════════════════════════════════════════════════════════════
```

### Step 1: Clone the Repository

```bash
# Clone the LXRCore-AI-Seek repository
git clone https://github.com/iboss21/TheSigma.git
cd TheSigma
```

### Step 2: Create Virtual Environment

```bash
# Create a new virtual environment
python3.10 -m venv venv

# Activate the virtual environment
source venv/bin/activate

# Upgrade pip
pip install --upgrade pip
```

### Step 3: Install Dependencies

```bash
# Navigate to the inference directory
cd inference

# Install required packages
pip install -r requirements.txt

# Verify installations
python -c "import torch; print(f'PyTorch: {torch.__version__}')"
python -c "import torch; print(f'CUDA available: {torch.cuda.is_available()}')"
```

### Step 4: Download Model Weights

You have two options for obtaining model weights:

#### Option A: Download Original Weights (Recommended)

```bash
# Install Hugging Face CLI
pip install huggingface-hub

# Download model weights from Hugging Face
# Note: This is a large download (685GB)
huggingface-cli download deepseek-ai/DeepSeek-V3 --local-dir /path/to/model/weights

# Or download the base model
huggingface-cli download deepseek-ai/DeepSeek-V3-Base --local-dir /path/to/base/weights
```

#### Option B: Use Pre-downloaded Weights

If you already have the weights, ensure they're in the correct location:

```bash
# Verify weights directory structure
ls -lh /path/to/model/weights
# Should contain: config.json, model-*.safetensors, tokenizer files
```

### Step 5: Convert Weights (If Needed)

If you need to convert HuggingFace format to the inference format:

```bash
# Convert weights for distributed inference
python convert.py \
    --hf-ckpt-path /path/to/model/weights \
    --save-path /path/to/converted/weights \
    --n-experts 256 \
    --model-parallel 16
```

### Step 6: Optional - Convert FP8 to BF16

If your hardware doesn't support FP8:

```bash
# Convert FP8 weights to BF16
python fp8_cast_bf16.py \
    --input-fp8-hf-path /path/to/fp8/weights \
    --output-bf16-hf-path /path/to/bf16/weights
```

---

## ✅ Verification

```
═══════════════════════════════════════════════════════════════════════════════
█████ TESTING INSTALLATION
═══════════════════════════════════════════════════════════════════════════════
```

### Test 1: Import Check

```python
# test_imports.py
import torch
import triton
import transformers
from safetensors.torch import load_model

print("✓ All required packages imported successfully")
print(f"✓ PyTorch version: {torch.__version__}")
print(f"✓ CUDA available: {torch.cuda.is_available()}")
print(f"✓ Number of GPUs: {torch.cuda.device_count()}")

for i in range(torch.cuda.device_count()):
    print(f"  - GPU {i}: {torch.cuda.get_device_name(i)}")
```

Run the test:

```bash
python test_imports.py
```

### Test 2: Model Loading

```bash
# Test model loading (adjust paths as needed)
python generate.py \
    --ckpt-path /path/to/converted/weights \
    --config configs/config_671B.json \
    --interactive
```

Expected output:
```
✓ Model loaded successfully
✓ Tokenizer initialized
Ready for interactive chat...
```

### Test 3: Simple Generation

```bash
# Test with a simple prompt
echo "The Land of Wolves is" > test_prompt.txt

python generate.py \
    --ckpt-path /path/to/converted/weights \
    --config configs/config_671B.json \
    --input-file test_prompt.txt
```

---

## 🔧 Troubleshooting

```
═══════════════════════════════════════════════════════════════════════════════
█████ COMMON ISSUES & SOLUTIONS
═══════════════════════════════════════════════════════════════════════════════
```

### Issue 1: CUDA Out of Memory

**Symptoms:**
```
RuntimeError: CUDA out of memory
```

**Solutions:**
1. Reduce batch size
2. Use FP8 quantization instead of BF16
3. Add more GPUs for model parallelism
4. Enable gradient checkpointing

```bash
# Set environment variable to reduce memory fragmentation
export PYTORCH_CUDA_ALLOC_CONF=max_split_size_mb:512
```

### Issue 2: Import Errors

**Symptoms:**
```
ModuleNotFoundError: No module named 'triton'
```

**Solutions:**
```bash
# Reinstall dependencies
pip install --force-reinstall -r requirements.txt

# Or install specific package
pip install triton==3.0.0
```

### Issue 3: Model Loading Fails

**Symptoms:**
```
FileNotFoundError: model weights not found
```

**Solutions:**
1. Verify weights directory structure
2. Check file permissions
3. Ensure git-lfs pulled large files

```bash
# Pull LFS files
cd /path/to/model/weights
git lfs pull

# Verify files
ls -lh *.safetensors
```

### Issue 4: Multi-GPU Issues

**Symptoms:**
```
RuntimeError: NCCL error
```

**Solutions:**
```bash
# Set NCCL debugging
export NCCL_DEBUG=INFO
export NCCL_IB_DISABLE=1  # If using non-InfiniBand network

# Check GPU visibility
nvidia-smi

# Verify all GPUs are accessible
python -c "import torch; print(torch.cuda.device_count())"
```

### Issue 5: Performance Issues

**Symptoms:**
- Slow inference speed
- High latency

**Solutions:**
1. Enable FP8 mode
2. Adjust batch size
3. Use more GPUs
4. Check thermal throttling

```bash
# Monitor GPU utilization
watch -n 1 nvidia-smi

# Check GPU temperature
nvidia-smi --query-gpu=temperature.gpu --format=csv
```

---

## 📞 Getting Help

If you encounter issues not covered here:

1. **Check Documentation**: Review other docs in this directory
2. **Search Issues**: Check [GitHub Issues](https://github.com/iboss21/TheSigma/issues)
3. **Discord Support**: Join us at https://discord.gg/CrKcWdfd3A
4. **Create Issue**: Report new bugs on GitHub

---

## 🔄 Next Steps

After successful installation:

1. Read the [Configuration Guide](configuration.md)
2. Explore [Framework Integration](frameworks.md)
3. Review [Security Guidelines](security.md)
4. Optimize [Performance Settings](performance.md)

---

<div align="center">
  <p><strong>🐺 The Land of Wolves - Installation Complete! 🐺</strong></p>
  <p>Made with ❤️ by iBoss21 & The Lux Empire</p>
</div>
