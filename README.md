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


### Deployment

Follow the instructions at [Quickstart for Actions Runner
Controller](https://docs.github.com/en/actions/tutorials/use-actions-runner-controller/quickstart). For
the sake of brevity, here it is summarized:

- You should have access to Kubernetes cluster. If not, install
  [minikube](https://minikube.sigs.k8s.io/docs/start/) for a local single node
  cluster.
- Install [helm](https://github.com/helm/helm)
- Create a GitHub app to allow the ARC controller to create scale sets at will on GitHub. See [Authenticating ARC to the GitHub API](https://docs.github.com/en/actions/tutorials/use-actions-runner-controller/authenticate-to-the-api#deploying-using-personal-access-token-classic-authentication).

Create the scale set controller:

```
NAMESPACE="arc-systems"
helm install arc \
    --namespace "${NAMESPACE}" \
    --create-namespace \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller
```

Create a scale set. The values YAML file uses the `pre-defined-secret` from your GitHub application.

```
RUNNER_SCALE_SET_LABEL="ubuntu-22.04-self-hosted"
helm install \
  "${RUNNER_SCALE_SET_LABEL}" \
  --namespace "arc-runners" \
  --create-namespace \
  -f values.yaml \
  oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set
```

The above example uses the `values.yaml` from this `ubuntu-22.04` directory to
deploy a runner set like GitHub's `ubuntu-22.04`. This image is nearly
identical to GitHub's official image with the notable exception that the image
was modified to pre-install several `apt` packages used by PyVista.


### Repository Organization

Each scale set used by PyVista has a directory here, for example
``ubuntu-22.04``, which contain `README.md`, `Dockerfile`, and `values.yaml`
which defines the configuration of the runner scale set.


### Local Debug

One advantage of using ARC is the ability to debug a workflow
interactively. For example, debug the `ubuntu-22.04` image with:

```bash
docker run -it --rm docker pull ghcr.io/pyvista/arc-runners:ubuntu22.04 bash
```

Next, within the container install a virtual environment, clone pyvista, and
run the unit tests:

```bash
sudo apt update
sudo apt install python3.10-venv
cd /tmp
python3.10 -m venv .venv
. .venv/bin/activate
pip install pip -U
git clone https://github.com/pyvista/pyvista
cd pyvista
pip install -e . --group test
pytest
```
