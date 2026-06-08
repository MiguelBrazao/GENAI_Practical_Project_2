# Image-to-Prompt Inversion (Generative AI - TP2)

This repository contains the implementation of the **Image-to-Prompt Inversion** pipeline for the Generative AI (TP2) project. The objective is to reconstruct text prompts that guide a text-to-image generator (LCM) to reproduce target images as closely as possible, using a hybrid pipeline of vision-language captioning, prompt retrieval, and iterative LLM-based refinement.

---

## 📁 Expected Folder Structure

For the project to run correctly (both locally and on Google Colab), organize your files according to the structure below:

```text
GENAI_Practical_Project_2/
├── statements/
│   ├── GENAI_TP2_Enunciado.pdf
│   └── TP2_Image-to-Prompt_Inversion.pdf
├── students/                           # Student subdirectory (contains outputs/ placeholders)
│   └── outputs/
├── tp2-chosen/                         # Target images to invert (exactly 6 pngs)
│   ├── 1159_25.png
│   ├── 1159_29.png
│   ├── 1159_3.png
│   ├── 1159_7.png
│   ├── 7836.png
│   └── 9338.png
├── tp2-chosen.zip                      # Optional compressed archive of target images
├── outputs/
│   └── final_submission/               # Folder to save results
├── FINAL_TP2_StarterPack_Students.ipynb # Main pipeline Jupyter Notebook
└── README.md                           # Project documentation (this file)
```

---

## 🛠️ Installation & Dependencies

The project relies on PyTorch (with CUDA support), Diffusers, Transformers, LPIPS, and auxiliary libraries.

### Option A: Local Run using Conda (Recommended)

To set up a local environment using Anaconda/Miniconda, run the following commands in your terminal:

1. **Create and Activate Environment:**
   ```bash
   conda create -n GEN_AI_TP2 python=3.11 -y
   conda activate GEN_AI_TP2
   ```
2. **Install PyTorch with CUDA Support:**
   Select the command matching your CUDA driver version. For CUDA 12.1:
   ```bash
   conda install pytorch pytorch-cuda=12.1 -c pytorch -c nvidia -y
   ```
3. **Install Required Packages:**
   Install diffusers, transformers, LPIPS, and specific pinned versions of pillow/pandas to prevent notebook crashes:
   ```bash
   python -m pip install "diffusers==0.35.2" "transformers<5" accelerate safetensors matplotlib torchvision ipywidgets "pandas<3" "Pillow<12" numpy lpips notebook
   ```
4. **Run Jupyter Notebook:**
   Launch the notebook server in the project directory:
   ```bash
   jupyter notebook
   ```
   Select `FINAL_TP2_StarterPack_Students.ipynb` and choose the `GEN_AI_TP2` conda kernel.

### Option B: Running on Google Colab (Free Tier - T4 GPU)

1. **Upload Notebook:** Upload the `FINAL_TP2_StarterPack_Students.ipynb` file to Google Colab.
2. **Upload Target Images:** Zip your `tp2-chosen/` directory and upload the `tp2-chosen.zip` file directly to the root of your Colab runtime or mount Google Drive and lay it out under `MyDrive/GENAI_TP2/tp2-chosen.zip`.
3. **Execution Environment:** Set the Runtime type to **T4 GPU** (highly recommended, as LCM rendering and metric computation require GPU acceleration).
4. **Notebook Setup Cell:** The first cell in the notebook handles the installation of all required packages automatically (`diffusers`, `transformers`, `accelerate`, `lpips`, and pinned versions `Pillow<12` / `pandas<3`).

---

## 🚀 Execution & Main Commands

The pipeline is fully automated and runs sequentially cell-by-cell in the Jupyter Notebook:

### Step 1: Initialization & Environment Configuration
*   Loads libraries, validates the GPU, and configures the Latent Consistency Model (LCM) configuration (Dreamshaper v7, 8 inference steps, guidance scale 8.0).
*   **Mandatory Colab CLIP Patch:** Evaluates the `transformers` version in Colab and applies a patch to prevent type attribute errors with `BaseModelOutputWithPooling` outputs from the CLIP model.

### Step 2: BLIP Captioning
*   Runs the BLIP model to generate `NUM_CAPTIONS = 100` candidate prompts per target image using nucleus sampling (`top_k = 50`, `top_p = 0.95`, `temperature = 0.85`).

### Step 3: CLIP Text-Image Retrieval
*   Compares the text embeddings of all 100 captions to the visual embedding of the target image. It retrieves the top `CLIP_RETRIEVAL_TOP_K = 100` candidates based on text-image cosine similarity.

### Step 4: CLIP Rendered Reranking (Visual Reranking)
*   Renders the top `CLIP_RERANK_TOP_K = 20` candidates using the LCM renderer.
*   Calculates the visual CLIP similarity between the generated render and the target image to rerank the candidates based on actual visual fidelity.

### Step 5: Iterative LLM-based Refinement
*   Refines the top 3 prompts for each image over `iterations = 6` cycles.
*   **API Configuration:** Provide your IAEDU API parameters in the configuration cell:
    ```python
    IAEDU_API_URL = "https://api.iaedu.pt/agent-chat/api/v1/agent/.../stream"
    IAEDU_API_KEY = "sk-usr-..."
    IAEDU_CHANNEL_ID = "cmq..."
    ```
*   **Rate-Limit Protections:** The code implements a `time.sleep(10)` delay between iterations and a robust backoff-retry loop to automatically handle `Rate limit reached (429)` errors. 
*   **Robust Parsing:** Stream token accumulation and regex parser fallbacks are active to prevent data loss in case the LLM response is truncated or malformed.

### Step 6: Evaluation & Export
*   Computes and saves final results to `outputs/final_submission/` including the top-3 images per target, their respective prompts, and a `metrics_summary.json` containing the CLIP similarity, LPIPS distance, and pixel MSE.
*   Prints mean and standard deviation of target metrics.

---

## 📝 Key Design Configuration Parameters
Adjust these constants in the notebook configuration cells before launching a full run:
*   `dtype = torch.float32` (Ensures consistency with the professor's setup)
*   `use_fast = False` (Keeps the slow processor active to match the BLIP model's default behavior)
*   `CLIP_RETRIEVAL_TOP_K = 100` (Filters candidates in text-space)
*   `CLIP_RERANK_TOP_K = 20` (Reranks candidates in visual-space)