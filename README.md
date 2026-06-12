# Content-Based Image Retrieval (CBIR)

A C++/OpenCV image search engine: given a target image and a database of ~1,100 images, it returns the most visually similar images. It compares six different feature representations — from raw pixels to deep neural-network embeddings — across a clean Model–View–Controller architecture.

---

## What it does

You hand the system a query image; it ranks every image in the database by similarity and returns the top N. The interesting part is *how* similarity is defined — the project implements and compares six feature types, each capturing something different about an image:

| Feature | What it captures | Distance metric |
|---|---|---|
| **Baseline** | raw 7×7 center pixels | sum of squared differences |
| **Color histogram** | rg-chromaticity (color ratio, brightness removed) | histogram intersection |
| **Multi-region histogram** | spatial color layout (top vs bottom) | weighted histogram intersection |
| **Texture + color** | color + Sobel gradient-magnitude histogram | histogram intersection |
| **Deep embedding** | ResNet18 semantic features (512-dim) | cosine distance |
| **Custom** | center color + whole-image texture blend | histogram intersection |

The core takeaway the project demonstrates: **a feature is a choice, and that choice defines what "similar" means.** The same query returns completely different neighbors depending on whether you look at color, spatial layout, texture, or learned content.

---

## Architecture

The system is organized as **Model–View–Controller** so that adding a new feature never requires touching the pipeline:

```
Model        features.cpp     image  -> feature vector
             distances.cpp    vector vs vector
             csv_util.cpp     vectors <-> disk (CSV cache)

Controller   cbir_engine.cpp  owns the 4-step retrieval flow:
                              1. feature the target
                              2. feature the database (offline, cached)
                              3. distance to every image
                              4. sort ascending, return top N

View         compute_features.cpp   builds the feature database
             match.cpp              command-line query
             gui.cpp                OpenCV window: target + ranked matches
```

Two-program design: `compute_features` builds the feature database once (cached to CSV); `match` and `gui` query it cheaply. Adding a feature is a single new function in the Model — the controller and all three views are untouched. The GUI was added as a third view with zero changes to the retrieval logic.

---

## Build

Requires OpenCV (tested with OpenCV 4 via Homebrew on macOS).

```bash
make
```

Produces three executables: `compute_features`, `match`, `gui`.

## Usage

```bash
# 1. build the feature database (run once per feature type)
./compute_features <imageDir> <featureType> <outCsv>

# 2. query for the top N matches
./match <targetImage> <featureType> <featureCsv> <N>

# or view results graphically
./gui <targetImage> <featureType> <featureCsv> <N> [metric] [imageDir]
```

Feature types: `baseline` · `hist` · `multihist` · `texture` · `dnn` · `custom`

**Example:**

```bash
./compute_features images baseline feat_baseline.csv
./match images/pic.1016.jpg baseline feat_baseline.csv 4
```

---

## Tech

C++17 · OpenCV · ResNet18 embeddings (pre-trained on ImageNet) · Make

## Notes

The CSV read/write utility is adapted from course-provided starter code. The ResNet18 embeddings were supplied as a precomputed CSV. All feature extraction, distance metrics, the MVC engine, both command-line programs, and the GUI are my own implementation.
