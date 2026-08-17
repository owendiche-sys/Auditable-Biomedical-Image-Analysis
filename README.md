# Auditable Biomedical Image Analysis

## Project overview

This repository contains a complete biomedical image-analysis assignment using synthetic fluorescence-microscopy images of cell nuclei. The project compares three complementary approaches:

1. Direct image description with a local vision-language model.
2. Classical segmentation and quantitative feature extraction.
3. Learned segmentation with a compact U-Net.

The final hybrid pipeline follows:

```text
image -> U-Net mask -> connected components -> region measurements
      -> validated JSON record -> short narrative
```

The design prioritises auditability. Quantitative measurements and validated JSON records are treated as the source of truth, while generated narratives are retained only as explanatory outputs.

## Repository structure

```text
.
├── README.md
├── .gitignore
├── notebook/
│   └── Auditable_Biomedical _Image_Analysis.ipynb
├── final_outputs/
│   ├── prompts.txt
│   ├── preprocessed/
│   ├── task1/
│   ├── task2/
│   ├── task3/
│   ├── task4/
│   └── robustness/
└── report/
    └── report.md
```

## Tasks

### Task 1: Data preparation and direct VLM description

The notebook downloads the public assignment dataset, verifies image-mask pairing, converts images to grayscale and resizes images and masks to 256 x 256 pixels. It produces an exploratory figure containing representative training images and the intensity distribution.

A representative image is sent to `llama3.2-vision` through a local Ollama server. A naive prompt is compared with an optimised, descriptive-not-diagnostic prompt that requires JSON and permits uncertainty. Three repeated optimised calls are retained to demonstrate run-to-run variation.

### Task 2: Classical features and numbers-first interpretation

Otsu thresholding and morphological cleanup create a classical foreground mask. Connected components are labelled and measured using region properties including area, eccentricity, solidity, circularity and mean intensity.

The text-only `llama3.2` model receives the numerical summary rather than the image. Its structured response is checked against the measured object count and calculated categories.

### Task 3: U-Net segmentation

Three compact U-Nets are trained using BCE, Dice and combined BCE + Dice loss. The architecture, seed, split, optimiser and training budget are held constant so that the loss function is the main experimental variable.

Checkpoint selection and probability-threshold calibration use validation data only. Performance is evaluated using Dice and intersection over union (IoU), with prediction panels provided for three validation images.

### Task 4: Hybrid test pipeline

The selected U-Net is applied to all 12 unseen test images. For each image, the notebook saves:

- A predicted binary mask.
- A region-properties feature table.
- A validated JSON record.
- The raw language-model response and validation errors.
- A short narrative.

The structured records are aggregated into `test_image_records.csv`.

### Robustness analysis

Blurred and low-contrast images are traced through input-quality checks, segmentation, measurements and narrative generation. This identifies the earliest stage at which corruption becomes detectable and demonstrates how image degradation can propagate into apparently fluent reporting.

## Key results

| Method | Loss | Validation Dice | Validation IoU | Threshold |
|---|---:|---:|---:|---:|
| Otsu + morphology | Not applicable | 0.9697 | 0.9412 | Not applicable |
| U-Net | BCE | 0.9936 | 0.9874 | 0.60 |
| U-Net | Dice | 0.9913 | 0.9827 | 0.65 |
| **U-Net** | **BCE + Dice** | **0.9945** | **0.9891** | **0.70** |

The combined-loss U-Net was selected. On the unseen test set it achieved:

- Mean Dice: **0.9943**
- Mean IoU: **0.9887**
- Test images processed: **12**

These high scores should be interpreted cautiously because the dataset is small, synthetic and visually consistent. They do not demonstrate clinical generalisation.

## Running the notebook in Google Colab

The notebook is self-contained; no Python source files outside the notebook are required.

1. Open [`Auditable_Biomedical _Image_Analysis.ipynb`](notebook/Auditable_Biomedical _Image_Analysis.ipynb) in Google Colab.
2. Select **Runtime -> Change runtime type**.
3. Choose a **T4 GPU** runtime.
4. Confirm that `FULL_RUN = True` in the configuration cell.
5. Select **Runtime -> Run all**.
6. Keep the session open while the dataset, Ollama and the local models download.
7. When the final cell completes, download `/content/final_outputs.zip` from the Colab Files panel.

The setup cell performs the following automatically:

- Installs the required Python libraries.
- Downloads and validates the assignment dataset.
- Installs and starts Ollama 0.17.1 inside the Colab virtual machine.
- Downloads `llama3.2-vision` and `llama3.2`.
- Displays the available GPU information.

Colab storage is temporary. A new runtime may therefore need to download the dataset and models again.

## Reproducibility settings

| Setting | Value |
|---|---:|
| Random seed | 42 |
| Image size | 256 x 256 |
| Training images | 80 |
| Validation images | 20 |
| Test images | 12 |
| Epochs | 10 |
| Batch size | 8 |
| Learning rate | 0.001 |
| VLM repeats | 3 |

The unseen test set is not used for model selection. Prompts, raw replies, parsed JSON, validation errors, checkpoints, figures and numerical tables are saved under `/content/outputs` during execution.

## Evidence and report

- [GitHub-readable report](report/report.md)
- [Saved prompts](final_outputs/prompts.txt)
- [Task 1 VLM comparison](final_outputs/task1/vlm_prompt_comparison.json)
- [Task 2 numbers-first output](final_outputs/task2/numbers_first_llm_output.json)
- [Task 3 evaluation metrics](final_outputs/task3/evaluation_metrics.csv)
- [Task 4 aggregate records](final_outputs/task4/test_image_records.csv)
- [Robustness trace](final_outputs/robustness/robustness_trace.csv)

## Limitations

- The dataset is synthetic and much smaller than a clinical dataset.
- All images come from a narrow and consistent distribution.
- Only one random seed was used for the loss comparison.
- Pixel-level overlap can remain high when touching nuclei are merged into one object.
- Valid JSON does not guarantee factual language-model output.
- No external, prospective or expert-reader clinical validation was performed.

The results should therefore be treated as an educational demonstration of an auditable pipeline rather than evidence of clinical safety or effectiveness.

