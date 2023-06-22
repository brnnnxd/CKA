## mock exam 03

### Questão 1

Criar uma nova service account com nome  `pviewer`. Conceda a esta conta acesso de listar todos os `PersistentVolumes` no criando uma role `pviewer-role` e u ClusterRoleBinding chamada `pviewer-role-bindig`.

Em seguida, crie um pod chamado `pviewer` com a imagem: `redis` e  serviceAccount: `pviewer` no namespace `default`
cd 
<details>
  <summary><b>Resposta</b></summary>

1. Utilizando somente a linha de comando:

```sh
k create sa pviewer
k get sa 
k create clusterrole --help
k create clusterrole pviewer-role --verb=list --resource=persistentvolumes
k get clusterrole pviewer-role
k create clusterrolebinding --help
k create clusterrolebinding pviewer-role-bindig --clusterrole=pviewer-role --serviceaccount=default:pviewer
k describe clusterrolebinding  pviewer-role-bindig
k run pviewer --image=redis --dry-run=client -o yaml > pviewer.yaml
vim pviewer.yaml
k create apply -f pviewer.yaml
```
- Exemplo de yaml.

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pviewer
  name: pviewer
spec:
  serviceAccountName: pviewer
  containers:
  - image: redis
    name: pviewer
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

- Validando se o pod esta running com o volume.

```sh
k get po 
k describe po pviewer |grep -i service
```

</details>

---

### Questão 2

Liste o `InternalIP`de todos os nodes do cluster. salve o resultado em um arquivo em `/root/CKA/node_ips` 

A resposta de estar formatada: `InternalIP do controlplane`<space>`InteralIP `

<details>
  <summary><b>Resposta</b></summary>
  
[DOC pesquisando externalIP tem um exemplo que pode ser usado apenas alterando para InternalIP](https://kubernetes.io/pt-br/docs/reference/kubectl/cheatsheet/)

- Utilizar a linha de comando:

```sh
k get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}' > /root/CKA/node_ips
```

- Validando se o pod esta running com o volume.

```sh
cat /root/CKA/node_ips
```

</details>

---

### Questão 3

Realize a criação de um pod chamado `multi-pod` com dois containers 

Container 1: Nome `alpha` Imagem `nginx`
Container 1: Nome `beta` Imagem `busybox` sleep 4800 seconds.

configurar environment variables:
container 1 
`name: alpha`
container 2
`name: beta`

<details>
  <summary><b>Resposta</b></summary>

- Utilizando linha de comando:

```sh
k run multi-pod --image=nginx --dry-run=client -o yaml > multipod.yaml
vim multipod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: multipod
  name: multipod
spec:
  containers:
  - image: nginx
    name: alpha
    env:
      - name: "name"
        value: "alpha"      
  - image: busybox
    name: beta
    command:
      - sleep
      - "4800"
    env:
      - name: "name"
        value: "beta"      
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

- Validando se o namespace foi criado.

```sh
k get po 
k describe po multipod
```

</details>

---

### Questão 4

Crie um pod chamado `non-root-pod`. usando a imagem `redis:alpine`.

runAsUser: `1000`
fsGroup: `2000`

[DOC referencia](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

<details>
  <summary><b>Resposta</b></summary>

- Utilizando linha de comando.

```sh
k run non-root-pod --image=redis:alpine --dry-run=client -o yaml > non-root-pod.yaml
vim non-root-pod.yaml
```

- Exemplo de yaml.

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: non-root-pod
  name: non-root-pod
spec:
  securityContext:
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - image: redis:alpine
    name: non-root-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

- Realizando a criacao do pod.

```sh
k apply -f non-root-pod.yaml
```

- Validando se o namespace foi criado.

```sh
k get po 
k get po non-root-pod -o yaml
```

</details>

---

### Questão 5

foi implementado um novo pod chamado `np-test-1` e um service com nome `np-test-service`. As conexoes de entrada para esse servico nao estao funcionando. (Corrija o problema).

Crie uma a NetworkPolicy, com nome `ingress-to-nptest` que permite conexoes de entrar para o servico na porta `80`.

<details>
  <summary><b>Resposta</b></summary>

[DOC referencia](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

- Utilizando linha de comando:

```sh
vi  network-policy.yaml
```
- Exemplo de yaml.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ingress-to-nptest
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: np-test-1
  policyTypes:
    - Ingress
  ingress:
    - 
      ports:
        - protocol: TCP
          port: 80
```

- Validando se o namespace foi criado.

```sh
#subir um container para validar via curl
k run curl --image=alpine/curl --rm -- sh
```

</details>

---

### Questão 6

Defina o worker `node01` como Unschedulable. Depois de concluir crie um pod `dev-redis`, image `redis:alpine`, para garantir que as cargas de trabalho nao sejam planejadas para esse worker node, por fim crie um novo pod chamado prod-redis com a image `redis:alpine` com toleration schedulable no node01.

Criar um novo usuario chamado `John`, com acesso ao cluster com as permissoes de `create, liste, get, update e delete pods` no namespace `development`.
A chave privada ja existe, `/root/CKA/john.key` csr `/roo/CLA/john.csr`.

- key: `env_type`
- Value: `production`
- Effect: `NoSchedule`
- pod: `dev-redis` no run node01
- Create a pod: `prod-redis` to run on node01

<details>
  <summary><b>Resposta</b></summary>

[Documentação](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/)

- Utilizando linha de comando:

Crie um arquivo com exemplo abaixo.

```sh
k get no
k taint nodes node01 env_type=production:NoSchedule
k describe node node01
k run dev-redis --image=redis:alpine
k get pods -o wide
k run prod-redis --image=redis:alpine --dry-run=client -o yaml > prod-redis.yaml
vi prod-redis.yaml
```

- Exemplo de yaml.

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: prod-redis
  name: prod-redis
spec:
  tolerations:
  - key: "env_type"
    operator: "Equal"
    value: "production"
    effect: "NoSchedule"
  containers:
  - image: redis:alpine
    name: prod-redis
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

- Validando se o namespace foi criado.

```sh
k get pods -o wide
```
</details>

---

### Questão 7

Criar um pod chamado `hr-pod` no namespace `hr` pertencendo ao ambiente de `production` e `frontend`. image `redis:alpine`.

<details>
  <summary><b>Resposta</b></summary>

- Utilizando linha de comando:

```sh
k create ns hr
k run hr-pod -n hr --image=redis:alpine --labels="environment=production,tier=frontend"
```

- Validando se o pod foi criado.

```sh
k get pods -n hr
```

</details>

---

### Questão 8

Um arquivo kubeconfig kubeeconfig chamado `super.kubeconfig` foi criado em `/root/CKA`. Há algo errado com a configuracao deve ser corrigido.

<details>
  <summary><b>Resposta</b></summary>

- Utilizando linha de comando:

```sh
k get nodes --kubeconfig /root/CKA/super.kubeconfig # validar o erro.
cat .kube/config # validar o endpoint
vim /root/CKA/super.kubeconfig # corrigir o endpoint
k get nodes --kubeconfig /root/CKA/super.kubeconfig # validar o erro.
```

</details>

---
