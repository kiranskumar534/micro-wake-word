# MicroWakeWord Training for ESP32-S3-BOX3

Train custom wake word detection models for ESP32-S3-BOX3 using Malayalam (or any language).

## 📋 Quick Start

1. **Open the notebook**: `notebooks/training_book_refactored.ipynb`
2. **Run all cells in order** (top to bottom)
3. **Wait 2-4 hours** for training to complete
4. **Download the model** and deploy to ESP32

## 🎯 What This Does

This project trains a wake word detection model that can recognize a specific word or phrase (like "രാഘവാ" in Malayalam) and run on ESP32-S3-BOX3 hardware.

## 📚 Prerequisites

### Hardware
- **GPU**: T4 or better recommended (or use Google Colab)
- **RAM**: 16GB+ recommended
- **Disk**: 15GB free space

### Software
- **Python**: 3.10 (required)
- **Jupyter**: Notebook or JupyterLab
- **Git**: For cloning dependencies

## 🚀 Training Workflow

The notebook guides you through these steps:

### 1. Environment Setup (5 min)
- Installs microWakeWord and dependencies
- Validates Python version and GPU availability
- Checks disk space

### 2. TTS Setup (10 min)
- Downloads Piper TTS engine
- Downloads Malayalam voice models (meera & arjun)

### 3. Sample Generation (20-30 min)
- Generates 1000 wake word samples
- Uses parallel processing for speed
- Creates variations in speed and tone

### 4. Augmentation Data (30-60 min)
- Downloads background audio datasets:
  - MIT RIR: Room impulse responses (~50MB)
  - ESC-50: Environmental sounds (~100MB)
  - FMA: Music (optional, ~2GB)

### 5. Feature Generation (20-30 min)
- Creates spectrogram features
- Applies augmentation (background noise, reverb, etc.)
- Generates training/validation/test sets

### 6. Training (1-2 hours)
- Trains neural network model
- Saves checkpoints every 500 steps
- Selects best model based on validation performance

### 7. Export (5 min)
- Converts to TFLite format
- Quantizes for ESP32 deployment
- Downloads model file

## 📁 Project Structure

```
micro-wake-word/
├── notebooks/
│   ├── training_book_refactored.ipynb  ← Use this!
│   ├── training_book_clened.ipynb      (old version)
│   └── basic_training_notebook.ipynb   (reference)
├── generated_samples/                   (created during training)
├── generated_augmented_features/        (created during training)
├── trained_models/                      (created during training)
└── README.md                            ← You are here
```

## ⚙️ Configuration

All configuration is in **Cell 4** of the notebook. Key settings:

```python
TARGET_WORD = "രാഘവാ"          # Your wake word
SAMPLES_PER_MODEL = 500          # Samples per voice (1000 total)
MAX_WORKERS = 8                  # Parallel workers for generation
TRAINING_STEPS = 10000           # Training iterations
BATCH_SIZE = 128                 # Batch size (optimized for T4)
```

### Adjusting for Your Needs

- **More accuracy**: Increase `TRAINING_STEPS` to 20000-30000
- **Faster training**: Decrease `SAMPLES_PER_MODEL` to 250 (but less accurate)
- **Different language**: Change `TTS_MODELS` to your language
- **CPU limitations**: Decrease `MAX_WORKERS` to 4

## 🎮 Using Google Colab

1. Upload `training_book_refactored.ipynb` to Google Colab
2. **Change runtime to GPU**:
   - Runtime → Change runtime type → Hardware accelerator → GPU (T4)
3. Run all cells
4. Model will auto-download when complete

## 🔧 Troubleshooting

### "Not enough disk space"
- Free up 15GB before starting
- Skip FMA dataset (set `DOWNLOAD_FMA = False`)

### "No GPU detected"
- Training will be VERY slow on CPU
- Use Google Colab with GPU runtime instead

### "Sample generation failed"
- Check Piper executable exists
- Verify TTS models downloaded correctly
- Check error messages in output

### "Training loss not decreasing"
- May need more samples (increase `SAMPLES_PER_MODEL`)
- Try longer training (increase `TRAINING_STEPS`)
- Check that augmentation data downloaded correctly

### "Model doesn't detect wake word"
- Adjust `probability_cutoff` in model JSON (start with 0.3-0.5)
- Record real voice samples and retrain
- Increase training data diversity

## 📦 Deploying to ESP32-S3-BOX3

### 1. Create Model Manifest

Create `raghava.json`:

```json
{
  "type": "micro",
  "wake_word": "raghava",
  "author": "Your Name",
  "website": "https://github.com/yourusername",
  "model": "stream_state_internal_quant.tflite",
  "version": 1,
  "micro": {
    "probability_cutoff": 0.5,
    "sliding_window_average_size": 10
  }
}
```

### 2. Copy Files to ESPHome

Copy both files to your ESPHome config directory:
- `stream_state_internal_quant.tflite`
- `raghava.json`

### 3. Update ESPHome YAML

```yaml
micro_wake_word:
  models:
    - model: raghava.json
```

### 4. Flash to Device

```bash
esphome run your-device.yaml
```

### 5. Tune Performance

Test the wake word and adjust `probability_cutoff`:
- **Too sensitive** (false positives): Increase to 0.6-0.7
- **Not sensitive enough**: Decrease to 0.3-0.4

## 🎯 Expected Results

### Good Model
- ✅ Detects wake word reliably (>90% of the time)
- ✅ Few false positives (<1 per hour)
- ✅ Works at different distances (1-3 meters)
- ✅ Works with background noise

### Poor Model (needs retraining)
- ❌ Misses wake word frequently
- ❌ Many false positives
- ❌ Only works when very close
- ❌ Fails with any background noise

## 🔄 Improving Your Model

If results aren't good:

1. **Generate more samples**
   - Increase `SAMPLES_PER_MODEL` to 1000-2000
   - Add more voice variations

2. **Train longer**
   - Increase `TRAINING_STEPS` to 20000-30000
   - Monitor validation metrics

3. **Add real recordings**
   - Record yourself saying the wake word
   - Add to `generated_samples/` directory
   - Retrain

4. **Adjust augmentation**
   - Modify `AUGMENTATION_DURATION_S`
   - Change SNR ranges
   - Add more background audio

## 📚 Resources

- [ESPHome microWakeWord Docs](https://esphome.io/components/micro_wake_word)
- [Model Examples](https://github.com/esphome/micro-wake-word-models/tree/main/models/v2)
- [microWakeWord GitHub](https://github.com/kahrendt/microWakeWord)
- [Piper TTS](https://github.com/rhasspy/piper)

## 🐛 Known Issues

### Issue: "impulse_paths = []" but MIT RIR downloaded
**Status**: Fixed in refactored notebook
**Solution**: Automatically detects and uses downloaded RIR files

### Issue: Cells out of order
**Status**: Fixed in refactored notebook
**Solution**: All cells now in logical execution order

### Issue: No progress indicators
**Status**: Fixed in refactored notebook
**Solution**: Added tqdm progress bars for all long operations

### Issue: Poor error messages
**Status**: Fixed in refactored notebook
**Solution**: Added validation and helpful error messages

## 📝 Notes

- **Training time**: 1-2 hours on T4 GPU, much longer on CPU
- **Model size**: ~200-500KB (fits easily on ESP32)
- **Languages**: Works with any language supported by Piper TTS
- **Customization**: All parameters configurable in notebook

## 🤝 Contributing

Found a bug or want to improve the notebook? Please open an issue or pull request!

## 📄 License

This project uses microWakeWord, which is open source. Check the original repository for license details.

---

**Happy training! 🎉**

For questions or issues, please check the troubleshooting section above or open an issue on GitHub.
