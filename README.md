# PyVista's Self-Hosted Runners

PyVista's Linux runners are deployed on an isolated self-hosted Kubernetes
cluster using [Kubernetes controller for GitHub Actions self-hosted
runners](https://github.com/actions/actions-runner-controller) (ARC). ARC was
chosen as it allows for the creation of scalable sets of runners rather than
relying on manually deploying and maintaining individual runners.

[https://k3s.io/](k3s) was chosen as the Kubernetes distribution due to its
ease of deployment and "smart defaults" approach to Kubernetes. However, the
instructions that follow will be generalized to any Kubernetes deployment
should you choose to also deploy your own scalable runners.


### Cluster Deployment

Deployment has been automated and simplified via an Ansible playbook. Details
of this deployment are within `ansible/README.md`.

### Repository Organization

Each scale set used by PyVista has a directory here, for example
``ubuntu-22.04``, which contain `README.md`, `Dockerfile`, and `values.yaml`
which defines the configuration of the runner scale set.


### Local Debug

One advantage of using ARC is the ability to debug a workflow
interactively. For example, debug the `ubuntu-22.04` image with:

```bash
docker run -it --rm ghcr.io/pyvista/arc-runners:ubuntu22.04 bash
```

Alternatively directly on the cluster:

```bash
kubectl run -i --tty pyvista-shell \
  --image=ghcr.io/pyvista/arc-runners:ubuntu22.04-gpu-latest \
  --restart=Never \
  --rm --overrides='
{
  "apiVersion": "v1",
  "spec": {
    "nodeSelector": {
      "kubernetes.io/hostname": "sr630-node-0"
    }
  }
}' -- bash
```

Next, within the container install a virtual environment, clone pyvista, and
run the unit tests:

```bash
sudo apt update
sudo apt install python3.10-venv -y
python3.10 -m venv .venv
. .venv/bin/activate
pip install pip -U
git clone https://github.com/pyvista/pyvista
cd pyvista
pip install -e . --group test
pytest
```


### Notes

To get EGL to work on Ubuntu24.04, it's critical to install the `libnvidia-gl`
package that matches the installed version of your NVIDIA drivers. For example,
`libnvidia-gl-575-server`. Without this, even with EGL installed on the
container, EGL will not be exposed to the container.

Only once the drivers are correctly installed will you be able to see the GPU
from docker when running `pyvista.Report()`:

```
  Date: Mon Aug 25 21:11:52 2025 UTC

                OS : Linux (Ubuntu 24.04)
            CPU(s) : 48
           Machine : x86_64
      Architecture : 64bit
       Environment : Python
        GPU Vendor : NVIDIA Corporation
      GPU Renderer : Quadro P2000/PCIe/SSE2
       GPU Version : 4.6.0 NVIDIA 575.57.08
     Render Window : vtkEGLRenderWindow
  MathText Support : True
```


```

docker pull kinetica/nvidia-opengl:ubuntu24.04
docker run --gpus all -it kinetica/nvidia-opengl:ubuntu24.04
```

Test for GPU visibility and EGL info:

```
nvidia-smi 
apt-get update && apt-get install mesa-utils-extra -y && eglinfo | grep NVIDIA
```


On the cluster, this was tested with:

```
apiVersion: v1
kind: Pod
metadata:
  name: glvnd-test
  namespace: default
spec:
  restartPolicy: Never
  runtimeClassName: nvidia
  containers:
  - name: pyvista-container
    image: kinetica/nvidia-opengl:ubuntu24.04
    command: ["/bin/bash"]
    stdin: true
    tty: true
    resources:
      limits:
        nvidia.com/gpu: 1
```
