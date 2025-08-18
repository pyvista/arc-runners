FROM summerwind/actions-runner:v2.328.0-ubuntu-22.04

RUN sudo apt-get update && \
    sudo apt-get install -y \
    xvfb \
    libxrender1 \
    libgl1 \
    libx11-6 \
    libxi6 \
    libxext6 \
    libgl1-mesa-glx && \
    sudo rm -rf /var/lib/apt/lists/*
