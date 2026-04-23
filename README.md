
# 🌟 Project IRIS
**Retinal Image Deblurring & Denoising via Variational, End-to-End, Generative, and Hybrid Approaches**

> **Corso:** Computational Imaging 2025/26  
> **Docenti:** Prof.ssa Elena Loli Piccolomini, Prof. Davide Evangelista (DISI)  
> **Gruppo:** F (3 studenti)  
> **Repository:** [GitHub Link]  

---

## 📖 Descrizione del Progetto
Questo progetto affronta il problema inverso di **deblurring e denoising** su immagini mediche della retina, formulato come:
$$y = Ax + n$$
dove $A$ è un operatore di motion blur (angolo $45^\circ$, kernel size $9$) e $n$ è rumore Gaussiano additivo con livelli $\sigma \in \{0.005, 0.01, 0.05, 0.1\}$. 

L'obiettivo è confrontare criticamente quattro famiglie metodologiche sugli stessi input degradati, valutando punti di forza, limiti e compromessi tra qualità visiva, metriche quantitative (PSNR, SSIM) e costo computazionale.

---

## 📂 Struttura della Repository
```
computational-imaging-groupF/
├── README.md                  # Documentazione principale
├── environment.yml            # Ambiente Conda con dipendenze
├── .gitignore                 # File esclusi (dati, checkpoint, cache)
│
├── data/
│   ├── raw/                   # Dataset originale (gitignored)
│   ├── processed/             # Immagini 256x256, split train/val/test (gitignored)
│   └── download_prepare.py    # Download, resize, normalizzazione e split
│
├── configs/
│   ├── degradation.yaml       # Parametri di degradazione (angolo, kernel, σ)
│   ├── training.yaml          # Iperparametri di training (lr, batch, epochs, seed)
│   └── methods/
│       ├── fista_wavelet.yaml # λ, n_iter, tipo wavelet, soglia
│       ├── nafnet.yaml        # Architettura, ottimizzatore, scheduler
│       ├── diffpir.yaml       # N_steps, guidance, denoiser checkpoint
│       └── weighted_tv.yaml   # λ_tv, α (pesi appresi), solver params
│
├── src/
│   ├── __init__.py
│   ├── data/                  # Dataset, DataLoader, transform
│   ├── models/                # Architetture NN (NAFNet)
│   ├── methods/
│   │   ├── variational/       # FISTA + prox operatore Wavelet
│   │   ├── end_to_end/        # Training/valutazione NAFNet
│   │   ├── generative/        # DiffPIR adattato a deblur+denoise
│   │   └── hybrid/            # Weighted Total Variation (Morotti et al., 2025)
│   ├── utils/
│   │   ├── metrics.py         # PSNR, SSIM (vettorizzati)
│   │   ├── visualization.py   # Plot, crop, difference maps, grid comparative
│   │   ├── degradation.py     # Generazione deterministica di y = Ax + n
│   │   └── logging.py         # Salvataggio CSV/JSON, tracking metriche
│   └── pipeline.py            # Orchestrazione: degradazione → ricostruzione → salvataggio
│
├── scripts/
│   ├── 01_preprocess.py       # Setup dataset e split
│   ├── 02_train_end_to_end.py # Training NAFNet
│   ├── 03_run_variational.py  # Esecuzione FISTA su test set
│   ├── 04_run_generative.py   # Esecuzione DiffPIR su test set
│   ├── 05_run_hybrid.py       # Esecuzione Weighted TV su test set
│   ├── 06_evaluate_all.py     # Calcolo PSNR/SSIM e generazione tabelle
│   └── 07_generate_figures.py # Creazione figure comparative per le slide
│
├── notebooks/
│   ├── 01_eda.ipynb           # Analisi esplorativa dataset
│   ├── 02_hyperparam_tuning.ipynb # Scelta euristica parametri (λ, step diffusione, ecc.)
│   └── 03_results_analysis.ipynb  # Analisi finale e export risultati
│
├── results/
│   ├── checkpoints/           # Pesi modelli (.pt)
│   ├── metrics/               # CSV/JSON con PSNR/SSIM per metodo e σ
│   └── figures/               # Ricostruzioni, crop, residuali, tabelle
│
└── presentation/
    ├── slides.pdf / slides.pptx
    └── speaker_notes.md
```

### 🔍 Logica Organizzativa
| Cartella | Scopo | Consiglio d'uso |
|----------|-------|-----------------|
| `configs/` | Separazione logica/parametri | Modifica qui i valori senza toccare il codice. Usa `seed` fissato per riproducibilità. |
| `src/methods/` | Isolamento metodologie | Ogni metodo espone `run(y, cfg) -> x_hat`. Interfaccia uniforme per cicli automatici. |
| `utils/degradation.py` | Garanzia di input identici | Genera `y` una sola volta. Tutti i metodi leggono lo stesso file/seed. |
| `results/` | Output puliti | Nessuno script scrive qui se non `06_` e `07_`. Pronto per slide e report. |
| `notebooks/` | Sperimentazione | Usali solo per tuning e analisi. Non committare output `.ipynb` pesanti. |

---

## ⚙️ Setup e Installazione
```bash
# 1. Clona la repository
git clone https://github.com/<your-username>/project-iris.git
cd project-iris

# 2. Crea l'ambiente Conda
conda env create -f environment.yml
conda activate iris-env

# 3. (Opzionale) Scarica il dataset manualmente se lo script non funziona
# Link: https://www.kaggle.com/datasets/... (Egyptian Medical Retinal Images)
```

---

## 🚀 Esecuzione della Pipeline
Gli script sono numerati per seguire un flusso lineare e riproducibile:

```bash
# 1. Preprocessing dataset (resize 256x256, normalizzazione [0,1], split 70/15/15)
python scripts/01_preprocess.py

# 2. Training metodo End-to-End (NAFNet)
python scripts/02_train_end_to_end.py

# 3-5. Esecuzione metodi su test set (generano risultati in results/)
python scripts/03_run_variational.py
python scripts/04_run_generative.py
python scripts/05_run_hybrid.py

# 6. Calcolo metriche PSNR/SSIM e salvataggio tabelle
python scripts/06_evaluate_all.py

# 7. Generazione figure comparative per presentazione
python scripts/07_generate_figures.py
```

---

## 🎛️ Configurazione e Scelta dei Parametri
Ogni metodo richiede la taratura di iperparam specifici. La scelta è stata condotta **euristicamente** ed è documentata in `notebooks/02_hyperparam_tuning.ipynb`. I valori finali sono fissati nei file `configs/methods/*.yaml`.

| Metodo | Parametri Critici | Strategia di Scelta |
|--------|-------------------|---------------------|
| **FISTA + Wavelet** | $\lambda$, `n_iter`, `wavelet_type` | Grid search su val set, monitoraggio SSIM |
| **NAFNet** | `lr`, `batch_size`, `scheduler` | Early stopping su val loss, warmup cosine |
| **DiffPIR** | `N_steps`, `guidance_scale`, $\rho$ | Bilanciamento fedeltà dati vs prior, analisi PSNR/σ |
| **Weighted TV** | $\lambda_{tv}$, `α_init`, solver steps | Adattività basata su gradiente locale, convergenza relativa |

---

## 📊 Risultati e Riproducibilità
- **Metriche:** Tutti i valori PSNR e SSIM sono esportati in `results/metrics/final_table.csv`.
- **Visualizzazioni:** Immagini ricostruite, crop significativi e mappe di differenza sono in `results/figures/`.
- **Determinismo:** 
  - Il degradazione è generata con `torch.manual_seed(42)` e salvata su disco.
  - Tutti i metodi leggono lo stesso `y`.
  - I seed per PyTorch, NumPy e CUDA sono fissati in `src/utils/degradation.py` e `src/pipeline.py`.

---

## 👥 Gruppo F
| Nome | Ruolo | Contatto |
|------|-------|----------|
| `[Studente 1]` | Metodi Variazionali + Ibridi + Tuning Parametri | `email@unibo.it` |
| `[Studente 2]` | Pipeline Dati + End-to-End (NAFNet) + Training | `email@unibo.it` |
| `[Studente 3]` | Generativo (DiffPIR) + Metrics/Visualization + Slide | `email@unibo.it` |

---

