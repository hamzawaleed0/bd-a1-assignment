# CSCI461 Assignment #1 - Dockerized Data, Text Analysis and Machine Learning
# Base image: official Jupyter scipy-notebook (comes with Python, JupyterLab,
# and most of the scientific stack already installed)
FROM quay.io/jupyter/scipy-notebook:latest

# The scipy-notebook image already runs as a non-root user ("jovyan").
# Create the project folder inside the image and make it the working directory.
WORKDIR /home/jovyan/work

# Install any additional/updated Python packages listed in requirements.txt.
# --chown keeps this owned by jovyan (uid 1000), the non-root user Jupyter
# actually runs as, instead of the root ownership a plain COPY would leave.
COPY --chown=jovyan:users requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy the (empty, unrun) notebook and the dataset into the image, so the
# image is reproducible on its own even before any bind mount is attached.
COPY --chown=jovyan:users notebook/analysis.ipynb ./notebook/analysis.ipynb
COPY --chown=jovyan:users data/dataset.csv ./data/dataset.csv

# Pre-create the output folder so it's ready to receive a bind mount at
# `docker run` time, or to be written to directly even without one.
RUN mkdir -p output/figures

# JupyterLab listens on 8888 inside the container by default
EXPOSE 8888