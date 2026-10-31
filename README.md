CSCI461 Introduction to Big Data — Assignment #1
Dataset: **YouTube Spam Collection** (Option 3) — 1,956 comments across 5 videos (Psy, Katy Perry, LMFAO, Eminem, Shakira), labeled spam (1) / ham (0).

## Project Structure

```
bd-a1/
├── Dockerfile
├── requirements.txt
├── README.md
├── data/
│   └── dataset.csv          
├── notebook/
│   └── analysis.ipynb       
├── output/
│   ├── cleaned_data.csv
│   ├── model_metrics.txt
│   ├── cluster_summary.txt
│   └── figures/
└── screenshots/             
```

## 1. Build the Custom Image

```bash
docker build -t hamzawaleed1/bd-a1:latest .
```

**Image vs. container:** an **image** is the read-only, versioned blueprint produced by `docker build` — it bundles the OS layer, JupyterLab, Python packages, and the notebook/dataset copied in by the `Dockerfile`. A **container** is a running (or stopped) *instance* of that image, with its own writable layer, network ports and mounted volumes. One image can start many independent containers.

## 2. Run the Container with a Bind Mount

```bash
docker run -it --rm \
  -p 8888:8888 \
  -v "$(pwd)/output:/home/jovyan/work/output" \
  hamzawaleed1/bd-a1:latest
```

- `-p 8888:8888` maps **HOST_PORT:CONTAINER_PORT** — JupyterLab listens on port `8888` *inside* the container; the first `8888` is the port exposed on the host machine. Requests to `http://localhost:8888` on the host are forwarded through Docker's networking into the container's internal port.
- `-v "$(pwd)/output:/home/jovyan/work/output"` is a **bind mount**, linking the host's `output/` folder directly to the container's `output/` folder. Anything the notebook writes to `output/` lands straight on the host disk, so results persist after the container is removed — unlike files written only inside the container's own writable layer, which disappear with it. (A *named volume*, by contrast, is storage Docker manages itself rather than a specific host path.)

JupyterLab prints a login URL with a token on startup. Opening that URL, then `notebook/analysis.ipynb`, followed by **Kernel → Restart Kernel and Run All Cells**, executes the full pipeline.

## 3. Persistent Output

Final artifacts are written under `output/` from inside the container:
- `output/cleaned_data.csv`
- `output/model_metrics.txt`
- `output/cluster_summary.txt`
- `output/figures/*.png` (correlation heatmap, word-frequency charts, group comparison, confusion matrix)

Because `output/` is bind-mounted, these remain on the host after the container stops or is removed.

## 4. Container Lifecycle Evidence

```bash
docker images                        # the built bd-a1 image
docker ps                            # the running container
docker ps -a                         # all containers, including stopped ones
docker logs <container_id>           # JupyterLab startup log / token
docker inspect <container_id>        # full container configuration (mounts, ports, etc.)
```

Captured output for each command is in `screenshots/`.

## 5. Docker Hub

```bash
docker login
docker tag bd-a1 hamzawaleed1/bd-a1:latest
docker push hamzawaleed1/bd-a1:latest
```

Repository: `hamzawaleed1/bd-a1`
Tag: `latest`
Docker Hub: https://hub.docker.com/r/hamzawaleed1/bd-a1

Reproducibility is verified by pulling and running the published image on a separate machine:

```bash
docker pull hamzawaleed1/bd-a1:latest
docker run -it --rm -p 8888:8888 hamzawaleed1/bd-a1:latest
```

## Notes on Data Cleaning Decisions

- Exact duplicate rows (same `COMMENT_ID` recorded twice) were dropped as true data-entry duplicates.
- Rows with duplicate **comment text** but different authors/IDs were **kept** — identical spam text posted by many accounts is itself a meaningful spam signal, not an error.
- `DATE` is missing for ~12% of rows; these were left as missing 
