# Streamlit-only deployment folder

This folder is prepared only for Streamlit deployment.

It includes the app code, dependencies, theme config, benchmark result files, and model instructions.
It does not include training notebooks, dataset, training runs, videos, or model weights.

## Important
- Do not commit `.pt` or `.pth` model weights to the normal GitHub repository.
- Upload model weights to GitHub Releases instead.
- For a stable online demo, run YOLO online first. SSD and Faster R-CNN can be demonstrated locally if Streamlit Cloud is too slow.

## Minimum files for Streamlit Cloud
- `ui.py`
- `requirements.txt`
- `.streamlit/config.toml`
- `results/model_comparison.csv` if the benchmark tab reads from it

## Optional folders
- `models/` contains only README instructions, not weights.
- `sample_data/` can contain a few small demo images.
