# Jetson Installation Guide

This guide provides instructions for installing WhisperX on NVIDIA Jetson devices with compatibility for torchaudio 2.x.

## Prerequisites

- NVIDIA Jetson device (Nano, TX2, Xavier, Orin, etc.)
- PyTorch and torchaudio pre-installed (typically comes with JetPack)
- Python 3.9 or higher

### Verify Existing PyTorch Installation

Before installing WhisperX, verify your PyTorch and torchaudio versions:

```bash
python3 -c "import torch, torchaudio; print(f'PyTorch: {torch.__version__}'); print(f'torchaudio: {torchaudio.__version__}')"
```

**Important:** This branch is compatible with torchaudio 2.x (e.g., 2.0.0, 2.1.0, 2.4.0, 2.10.0). If you have a different version, you may need to upgrade your JetPack or PyTorch installation.

## Installation

### Method 1: Basic Installation (Recommended for Jetson)

This method installs WhisperX without reinstalling PyTorch/torchaudio, using your existing Jetson-optimized builds:

```bash
# Clone the repository
git clone https://github.com/sealambda/whisperX.git
cd whisperX
git checkout jetson-torchaudio-2.10-compat

# Install WhisperX (without torch dependencies)
pip install -e . --no-build-isolation

# If you encounter issues, you can also try:
# pip install . --no-build-isolation
```

### Method 2: Manual Dependency Installation

If you prefer more control over the installation process:

```bash
# Install core dependencies first
pip install ctranslate2>=4.5.0 faster-whisper>=1.1.1 nltk>=3.9.1
pip install "numpy>=2.0.2,<2.1.0" "pandas>=2.2.3,<2.3.0" "av<16.0.0"
pip install transformers>=4.48.0

# Install pyannote.audio (>=4.0.1 for torchaudio 2.x compatibility)
pip install "pyannote-audio>=4.0.1"

# Install speechbrain from git (fixes list_audio_backends issue)
pip install git+https://github.com/speechbrain/speechbrain.git@develop

# Install WhisperX without dependencies
pip install --no-deps -e .
```

## What's Fixed in This Branch?

This branch includes specific fixes for Jetson/torchaudio 2.x compatibility:

### Issue #1: AudioMetaData Compatibility
**Error:** `AttributeError: module 'torchaudio' has no attribute 'AudioMetaData'`

**Fixed by:** Upgrading to `pyannote-audio>=4.0.1`, which is compatible with torchaudio 2.x API changes.

### Issue #2: list_audio_backends() Removal
**Error:** `AttributeError: module 'torchaudio' has no attribute 'list_audio_backends'`

**Fixed by:** Installing speechbrain from git (develop branch), which includes the fix from [speechbrain PR #2988](https://github.com/speechbrain/speechbrain/pull/2988). This fix has not yet been released in a stable version.

### Issue #3: Jetson-Specific PyTorch Builds
**Problem:** Standard pip installations of PyTorch can override Jetson-optimized CUDA builds.

**Fixed by:** Making torch/torchaudio optional dependencies, allowing Jetson users to use their pre-installed versions without reinstallation.

## Testing Your Installation

### 1. Download Test Audio

```bash
wget https://www.kozco.com/tech/piano2.wav -O test_audio.wav
```

### 2. Test Basic Transcription (Without Diarization)

```bash
whisperx test_audio.wav --model tiny --compute_type int8
```

### 3. Test with Diarization

```bash
# Note: Diarization requires a Hugging Face token
export HF_TOKEN="your_huggingface_token_here"
whisperx test_audio.wav --model small --compute_type int8 --diarize --hf_token $HF_TOKEN
```

### 4. Verify Python Imports

```bash
python3 -c "from whisperx.asr import load_model; print('✓ WhisperX import successful')"
python3 -c "import pyannote.audio; print('✓ pyannote.audio import successful')"
python3 -c "import speechbrain; print('✓ speechbrain import successful')"
```

## Compute Type Recommendations for Jetson

Jetson devices have limited compute resources. Use these recommended compute types:

- **Jetson Nano:** `int8` (recommended), `float16` (slower)
- **Jetson TX2:** `int8` or `float16`
- **Jetson Xavier/Orin:** `float16` (recommended), `int8` (faster, lower quality)

Example:
```bash
whisperx audio.wav --model base --compute_type int8
```

## Troubleshooting

### CUDA Out of Memory

If you encounter CUDA OOM errors:
1. Use a smaller model (`tiny`, `base` instead of `small`, `medium`)
2. Reduce batch size: `--batch_size 8` or `--batch_size 4`
3. Use int8 compute type: `--compute_type int8`

### Import Errors

If you get import errors for torch/torchaudio:
1. Verify your PyTorch installation: `python3 -c "import torch; print(torch.__version__)"`
2. Check CUDA availability: `python3 -c "import torch; print(torch.cuda.is_available())"`
3. Reinstall JetPack if necessary

### pyannote.audio Errors

If you get errors related to `pyannote.audio`:
1. Ensure you have version 4.0.1 or higher: `pip show pyannote-audio`
2. Upgrade if needed: `pip install --upgrade "pyannote-audio>=4.0.1"`

### speechbrain Errors

If you get errors related to `list_audio_backends`:
1. Ensure speechbrain was installed from git: `pip show speechbrain`
2. Reinstall from git if needed: `pip install --force-reinstall git+https://github.com/speechbrain/speechbrain.git@develop`

## Standard (Non-Jetson) Installation

For standard x86_64/ARM64 systems with CUDA, you can install with torch dependencies:

```bash
pip install -e .[torch]
```

This will install PyTorch 2.8.x with CUDA support.

## Additional Resources

- [WhisperX README](README.md)
- [CUDNN Troubleshooting](CUDNN_TROUBLESHOOTING.md)
- [Examples](EXAMPLES.md)
- [Jetson Software Documentation](https://docs.nvidia.com/jetson/)

## Support

If you encounter issues specific to this Jetson compatibility branch, please report them with:
- Your Jetson model and JetPack version
- PyTorch and torchaudio versions
- Full error message and stack trace
