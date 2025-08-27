## PyVista Ubuntu 22.04 NVIDIA GPU ARC Runner Image

Docker image based on `kinetica/nvidia-opengl:ubuntu22.04` with NVIDIA driver
capabilities enabled. Includes git, Python, OpenGL libraries, and utilities for
running CI/CD workflows with GPU support.


### Build

```
docker build -t ghcr.io/pyvista/arc-runners:ubuntu22.04-gpu .
```

### Local Debug

```
docker run -it --rm ghcr.io/pyvista/arc-runners:ubuntu22.04-gpu bash
```
