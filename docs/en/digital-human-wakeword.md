# Digital Human Startup Method

## Overview

The test page integrates a high-precision voice wake-up function based on **Sherpa-ONNX**, supporting custom wake-up words and real-time detection. Using a lightweight keyword detection model, it provides millisecond-level response speed.

## Wake-up Word Model

### Model Download (Required)

**Important Notice**: The project does not include model files and needs to be downloaded and configured in advance.

### Official Model Download Address

- **Official Model List**: <https://csukuangfj.github.io/sherpa/onnx/kws/pretrained_models/index.html>
- **Recommended Model**: `sherpa-onnx-kws-zipformer-wenetspeech-3.3M-2024-01-01`

### Download and Configuration Steps

#### 1. Download Model Package

```bash
# Method 1: Direct download (Recommended)
cd main/digital-human/wakeword_runtime/
wget https://github.com/k2-fsa/sherpa-onnx/releases/download/kws-models/sherpa-onnx-kws-zipformer-wenetspeech-3.3M-2024-01-01.tar.bz2

# Extract
tar xvf sherpa-onnx-kws-zipformer-wenetspeech-3.3M-2024-01-01.tar.bz2

# Method 2: Using ModelScope
pip install modelscope
python -c "
from modelscope import snapshot_download
snapshot_download('pkufool/sherpa-onnx-kws-zipformer-wenetspeech-3.3M-2024-01-01', cache_dir='./models')
"
```

#### 2. Configure Model Files

After downloading the model package, it contains the following files:

```
sherpa-onnx-kws-zipformer-wenetspeech-3.3M-2024-01-01/
├── encoder-epoch-12-avg-2-chunk-16-left-64.int8.onnx    # Speed priority
├── encoder-epoch-12-avg-2-chunk-16-left-64.onnx
├── encoder-epoch-99-avg-1-chunk-16-left-64.int8.onnx    # Speed priority
├── encoder-epoch-99-avg-1-chunk-16-left-64.onnx         # Accuracy priority
├── decoder-epoch-12-avg-2-chunk-16-left-64.onnx
├── decoder-epoch-99-avg-1-chunk-16-left-64.onnx         # Accuracy priority
├── joiner-epoch-12-avg-2-chunk-16-left-64.int8.onnx     # Speed priority
├── joiner-epoch-12-avg-2-chunk-16-left-64.onnx
├── joiner-epoch-99-avg-1-chunk-16-left-64.int8.onnx     # Speed priority
├── joiner-epoch-99-avg-1-chunk-16-left-64.onnx          # Accuracy priority
├── tokens.txt                    # Token mapping table (required)
├── keywords_raw.txt              # May be included in the model package (optional, not required by runtime)
├── keywords.txt                  # Ready-made
├── test_wavs/                    # Test audio (optional)
├── configuration.json            # Model metadata (optional)
└── README.md                     # Documentation (optional)
```

#### 3. Choose Configuration Plan

**Plan 1: Accuracy Priority (Recommended)**

```bash
cd sherpa-onnx-kws-zipformer-wenetspeech-3.3M-2024-01-01

# Create model directory
mkdir -p ../models

# Copy accuracy priority epoch-99 fp32 triad
cp encoder-epoch-99-avg-1-chunk-16-left-64.onnx ../models/encoder.onnx
cp decoder-epoch-99-avg-1-chunk-16-left-64.onnx ../models/decoder.onnx
cp joiner-epoch-99-avg-1-chunk-16-left-64.onnx ../models/joiner.onnx

# Copy accompanying files
cp tokens.txt ../models/tokens.txt
# keywords_raw.txt if included in the model package, can be kept; runtime does not depend on it
```

**Plan 2: Speed Priority**

```bash
cd sherpa-onnx-kws-zipformer-wenetspeech-3.3M-2024-01-01

# Create model directory
mkdir -p ../models

# Copy speed priority epoch-99 int8 triad
cp encoder-epoch-99-avg-1-chunk-16-left-64.int8.onnx ../models/encoder.onnx
cp decoder-epoch-99-avg-1-chunk-16-left-64.onnx ../models/decoder.onnx
cp joiner-epoch-99-avg-1-chunk-16-left-64.int8.onnx ../models/joiner.onnx

# Copy accompanying files
cp tokens.txt ../models/tokens.txt
```

**Notes**:

- **Don't mix fp32 and int8**: All three model files must maintain consistent precision
- **Prefer epoch-99**: More thoroughly trained than epoch-12, with higher accuracy
- **Required files**: `encoder.onnx` + `decoder.onnx` + `joiner.onnx` + `tokens.txt` + `keywords.txt`

### Final Model File Structure

After configuration, the model files should be placed in the `wakeword_runtime/models/` directory, with the complete path being `main/digital-human/wakeword_runtime/models/`:

```
wakeword_runtime/models/
├── encoder.onnx      # Encoder model (renamed)
├── decoder.onnx      # Decoder model (renamed)
├── joiner.onnx       # Joiner model (renamed)
├── tokens.txt        # Pinyin Token mapping table (228-line version)
├── keywords.txt      # Keyword configuration file (auto-generated on first startup)
└── keywords_raw.txt  # Optional, not required by runtime
```

## Startup Methods

Execute in the `main/digital-human` directory:

```bash
pip install -r wakeword_runtime/requirements.txt
python start.py
```

After startup, the default addresses are:

- Page address: `http://127.0.0.1:8006/index.html`
- Event bridge address: `ws://127.0.0.1:8006/wakeword-ws`
- Health check: `http://127.0.0.1:8006/health`

Stop method:

- Press `Ctrl+C` in the running terminal
- This will simultaneously stop the static page service, event bridge, and wake-up word detection process

## Configuration File Explanation

The configuration file is located at [main/digital-human/wakeword_runtime/config.json](../main/digital-human/wakeword_runtime/config.json).

Current main configuration items:

```json
{
  "wakeword": {
    "enabled": true
  },
  "model_dir": "models",
  "audio": {
    "input_device": null,
    "sample_rate": 16000,
    "channels": 1
  },
  "detector": {
    "num_threads": 4,
    "provider": "cpu",
    "max_active_paths": 2,
    "keywords_score": 1.8,
    "keywords_threshold": 0.1,
    "num_trailing_blanks": 1,
    "cooldown_seconds": 1.5
  },
  "logging": {
    "level": "INFO",
    "dir": "logs",
    "file": "wakeword-runtime.log"
  }
}
```

Field meanings:

| Parameter | Description |
| --- | --- |
| `wakeword.enabled` | Whether to enable local wake-up word detection |
| `model_dir` | Directory where models and vocabularies are located |
| `audio.input_device` | Microphone input device, defaults to system default device |
| `audio.sample_rate` | Sampling rate, defaults to `16000` |
| `audio.channels` | Number of channels, defaults to `1` |
| `detector.num_threads` | Number of detector threads |
| `detector.provider` | Inference provider, usually `cpu` |
| `detector.max_active_paths` | Number of search paths |
| `detector.keywords_score` | Keyword enhancement score |
| `detector.keywords_threshold` | Detection threshold |
| `detector.num_trailing_blanks` | Number of trailing blanks |
| `detector.cooldown_seconds` | Cooling time between consecutive triggers |
| `logging.level` | Log level |
| `logging.dir` | Log directory |
| `logging.file` | Log filename |

## Recommended Usage Process

### First-time Use

1. Prepare model files and `tokens.txt` in the `models/` directory
2. Confirm `models/keywords.txt` exists
3. Run `python start.py` in the `digital-human` directory
4. Open browser to `http://127.0.0.1:8006/index.html`
5. Go to settings page to check "Wake-up Word" configuration

### Modify Wake-up Word

1. Open digital human page settings
2. Switch to "Wake-up Word" tab
3. Modify enable status or wake-up word list
4. Click "Apply Wake-up Word"
5. Follow prompts to decide whether to restart immediately

### Disable Wake-up Word

1. Change "Enable Local Wake-up Word" to disabled
2. Click "Apply Wake-up Word"
3. Suggest restarting once

After disabling:

- Page and event bridge still available
- Wake-up word detection no longer runs

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.