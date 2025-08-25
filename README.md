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
  --image=ghcr.io/pyvista/arc-runners:ubuntu22.04-gpu \
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
