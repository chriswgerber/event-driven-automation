# Event-Driven Automation

## Setup

> Assumes the IP of the host is `192.168.5.1`, which is likely the case if you're using Colima.

1. K3s Config
    1. Update systemd-resolved
        - /etc/hosts - Adds working hosts to local VM

          ```Shell
          $ colima ssh # Optional

          $ sudo tee -a /etc/hosts <<EOF
          192.168.5.1 cgerber.local
          192.168.5.1 awx.cgerber.local
          192.168.5.1 eda.cgerber.local
          192.168.5.1 mqtt.cgerber.local
          192.168.5.1 webapp.cgerber.local
          EOF
          ```

        - /etc/rancher/k3s/resolv.conf - Change K3s Resolver to point to local DNS

          ```Shell
          $ colima ssh # Optional

          $ sudo tee /etc/rancher/k3s/resolv.conf <<EOF
          nameserver 192.168.5.1
          EOF 
          ```

        - Update systemd service config

          ```Shell
          sudo tee -a /etc/systemd/system/k3s.service <<EOF
                '--resolv-conf' '/etc/rancher/k3s/resolv.conf'
          EOF 
          ```

    2. Update K3s Resolv Config

        ```Shell
        sudo tee /etc/systemd/resolved.conf.d/k3s.conf <<EOF
        [Resolve]
        DNSStubListenerExtra=192.168.5.1
        ReadEtcHosts=yes
        EOF
        ```

    3. Restart Service
        - `sudo systemctl restart systemd-resolved.service`

    4. Restart CoreDNS
        - `kubectl -n kube-system rollout restart deployment coredns`

    5. Verify
        - `resolvectl status`

2. Helm
    - `helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/`
    - `helm repo add awx-operator https://ansible.github.io/awx-operator/`

3. K8s Dashboard
    - `helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard`
    - `kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443`

4. Cert Authority
    - Create Authority Key and Cert

5. AWX
    1. Postgres data source directories
        - `colima ssh` (optional)
        - `sudo mkdir -p /data/postgres-15/`
        - `sudo chown -R 26 /data/postgres-15/`

    2. Project data dir
        - `sudo mkdir -p /data/projects`

    3. Certificate

        ```Shell
        openssl x509 -req \
          -days 825 \
          -sha256 \
          -CAcreateserial \
          -CA ca/cacert.pem \
          -CAkey ca/cacert.key \
          -in awx/ssl/awx.cgerber.csr \
          -extfile awx/ssl/awx.cgerber.ext \
          -out awx/ssl/awx.cgerber.crt
        ```

    4. Update Config values and Deploy via Helm
        - `helm -n awx upgrade -i awx awx-operator/awx-operator -f awx/values.yml`

    5. Create Project, Job Template, Credential (Git, K8s), Inventory

6. EDA
    1. Certificate
    2. AWX Access token
    3. Postgres data source directories
        - `colima ssh` (optional)
        - `sudo mkdir -p /data/eda/postgres-15`
        - `sudo chown -R 26 /data/eda/postgres-15/`
    4. Deploy
        - Operator `kubectl apply -k eda/operator`
        - Server `kubectl apply -k eda/server`
        - Message Queue `kubectl apply -k eda/mqtt`
    5. Create Project, Decision Env, AWX Access Token, Activation

7. MQTT & Webapp
    1. Deploy Webapp
        - `kubectl apply -k webapp`

    2. Install Client
        - `brew install mqttui`

## Sections

```Plaintext
ansible
  ├── collections
  ├── playbooks
  └── rulebooks
```

Ansible code for the demo. Collections installs required collection utilizing Kubernetes in the playbook.

```Plaintext
awx
  └── ssl
```

Installs and deploys AWX.

```Plaintext
bin
```

Includes scripts for demoing the app.

```Plaintext
eda
  ├── mqtt
  ├── operator
  └── server
```

Deploying EDA and dependencies. Includes the Operator for EDA, the EDA instance, and an MQTT topic used to demo sending event streams to EDA.

```Plaintext
k8s-dashboard
```

Dashboard for viewing the resources deployed to the K8s Cluster.

```Plaintext
webapp
  └── files
```

The example web app being acted upon by the automation.
