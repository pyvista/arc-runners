### k3s deployment using Ansible

Instructions from: https://github.com/k3s-io/k3s-ansible

Install the k3s ansible playbook:

```
ansible-galaxy collection install git+https://github.com/k3s-io/k3s-ansible.git
```

Set up the cluster with:

```bash
ansible-playbook -i inventory.yml wrapper.yml
```

### Cluster Configuration

All host IP information is within `inventory.yml`.

Static IPs assigned using MAC address on router using DHCP.

##### Control Nodes
Using [High Availability Embedded etcd](https://docs.k3s.io/datastore/ha-embedded).

Hardware: [Raspberry 5 8Gb](https://www.raspberrypi.com/products/raspberry-pi-5/).

Nodes:
- 10.30.0.4:  # rasp5-0
- 10.30.0.5:  # rasp5-1
- 10.30.0.6:  # rasp5-2

##### Agent Nodes

Hardware: 
- [Lenovo SR630 v2](https://lenovopress.lenovo.com/lp1391-thinksystem-sr630-v2-server)
- 2x Intel(R) Xeon(R) Gold 6246 CPU @ 3.30GHz
- 188Gi DDR4 RAM
- Quadro P2000 with 5120MiB VRAM
- 500 GB NVME SSD

Nodes:
- 10.30.0.8: # sr630-node-0
