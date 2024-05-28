# Event-Driven Automation

## Setup

1. K3s
    - systemd-resolv config
      - /etc/hosts
      - resolv.conf
      - Restart Services
      - Restart CoreDNS
    - Update K3s Resolv Config
      - Restart
2. Cert Authority
    - Create Authority
3. Helm
    - `helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/`
    - `helm repo add awx-operator ...`
4. K8s Dashboard
    - `helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard`
5. AWX
    - Postgres data source directories
    - Certificate
      - `openssl x509 -req -in awx.cgerber.csr -CA cacert.pem -CAkey cacert.key -CAcreateserial -out awx.cgerber.crt -days 825 -sha256 -extfile awx.cgerber.ext`
    - Create Project, Job Template, Credential (Git, K8s), Inventory
6. EDA
    - Certificate
    - AWX Access token
    - Postgres data source directories
    - Create Project, Decision Env, AWX Access Token, Activation
7. MQTT
    - Create Client
