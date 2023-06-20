## mock exam 02

### Questão 1

Realizar o backup do etcd do cluster, salvar em `/opt/etcd-backup.db`

<details>
  <summary><b>Resposta</b></summary>

1. A primeira, utilizando somente a linha de comando:

[DOC](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)

```sh
# Para pegar o endpoint, entrar no arquivo abaixo em spec > containers > command > --listen-client-urls= endpoints.
vi /etc/kubernetes/manifests/etcd.yaml
# Para pegar o ceritifcado utilizar o comando abaixo.
cat /etc/kubernetes/manifests/etcd.yaml | grep file

ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 snapshot save /opt/etcd-backup.db \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \ 
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

Output esperado.

`Snapshot saved at /opt/etcd-backup.db`

</details>

---

### Questão 2

Crie um pod chamado `redis-storage` usando a image `redis:alpine` com um volume do tipo `emptyDir` que dura a vida util do pod.

Host Path: `/data/redis`

<details>
  <summary><b>Resposta</b></summary>

- Dica: pegar um exemplo na documentação [Link](https://kubernetes.io/docs/concepts/storage/volumes/)

1. Utilizar a linha de comando para criar o manifesto:

```sh
k run redis-storage --image=redis:alpine --dry-run=client -o yaml > quest-2.yaml
vi quest-2.yaml # adicione o bloco do volume e volumeMounts como na doc.
```

- Exemplo de yaml.

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: redis-storage
  name: redis-storage
spec:
  containers:
  - image: redis:alpine
    name: redis-storage
    resources: {}
    volumeMounts:
    - mountPath: /data/redis
      name: cache-volume
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
  - name: cache-volume
    emptyDir: {}
status: {}
```

- Validando se o pod esta running com o volume.

```sh
k get po 
k describe po redis-storage
```

</details>

---

### Questão 3

Realize a criação de um novo pod chamado `super-user` com a image `busybox:1.28`. Permitir que o pod seja capaz de definir o system_time.

pod sleep for 4800 seconds.

<details>
  <summary><b>Resposta</b></summary>

- Dica: pegar um exemplo da securityContext na documentação [Link](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#set-capabilities-for-a-container)

1. Utilizando linha de comando para criar o manifesto:

```sh
k run super-user --image=busybox:1.28 --dry-run=client -o yaml > quest-3.yaml
vi quest-3.yaml # adicione o command: sleep 4800 e securityContext.
```

- Validando se o namespace foi criado.

```sh
k get po 
k describe po redis-storage
```

</details>

---

### Questão 4

Em um arquivo de definição de pod criado em `/root/CKA/use-pv.yaml`. Use esse arquivo de manifesto e monte um volume persistente chamado `pv-1`. certifique-se de que o pod esteja em execução e o PV eteja vinculado.

mountPath: `/data`
persistentVolumeClaim Name: `my-pvc`

<details>
  <summary><b>Resposta</b></summary>

1. Utilizando linha de comando para criar o arquivo com o output do `k get no -o json`:

```sh
vi /root/CKA/use-pv.yaml
```

- Exemplo de yaml.

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: use-pv
  name: use-pv
spec:
  containers:
  - image: nginx
    name: use-pv
    resources: {}
    volumeMounts:
    - mountPath: "/data"
      name: my-pvc
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
  - name: mypd
    persistentVolumeClaim: 
      claimName: my-pvc
status: {}
```

- Realizando a criacao do pod.

```sh
k create po -f /root/CKA/use-pv.yaml
```

- Validando se o namespace foi criado.

```sh
k get po 
k describe po use-pv
```

</details>

---

### Questão 5

Criar um novo deployment chamado `nginx-deploy` com a imagem `nginx:1.16` e uma replica. Em seguida atualize o deployment para versão `1.17` usando rolling update.

<details>
  <summary><b>Resposta</b></summary>

1. Utilizando linha de comando:

```sh
k create deploy nginx-deploy --image=nginx:1.16 --replicas=1
k get deploy
k set image deployment/nginx-deploy nginx=nginx:1.17
```

- Validando se o namespace foi criado.

```sh
k describe deploy nginx-deploy
```

</details>

---

### Questão 6

Criar um novo usuario chamado `John`, com acesso ao cluster com as permissoes de `create, liste, get, update e delete pods` no namespace `development`.
A chave privada ja existe, `/root/CKA/john.key` csr `/roo/CLA/john.csr`.

- CSR: `john-developer`
- Role Name: `developer`
- namespace: `development`
- Resources: `Pods`
- Access: User `john`

<details>
  <summary><b>Resposta</b></summary>

[Documentação](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/)

1. Realizar a criacao do CSR basiado no exemplo da doc.

Crie um arquivo com exemplo abaixo.

`vi john-csr.yaml`

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john-developer
spec:
  request: # Deve ser coloca o valor resultante do /root/CKA/john.csr
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
```

- Para obter o valor da chave utilziar o comando abaixo.

```sh
cat /root/CKA/john.csr | base64 | tr -d "\n"
```

- Realizar a criacao do CSR

```sh
k create -f john-csr.yaml
k get csr # o CSR estará pendente de aprovação
k certificate approve john-developer
k get csr
```

- Criando a role com os acessos.

```sh
k create role --help
k create role developer --verb=create,get,list,update,delete --resource=pods -n development
```

- Validando se a role foi criado.

```sh
k get role -n development
k describe role -n development
```

- Rolebinding.

```sh
k create rolebinding --help
k create rolebinding john-developer --role=developer --user=john -n development
```

- Validando se a rolebinding foi criado.

```sh
k get rolebindings -n development
k describe rolebindings -n development
k auth can-i create po -n development --as john # retorno deve ser yes
```

</details>

---

### Questão 7

- Foi criado um pod nginx chamado `nginx-resolver` usando a imagem `nginx`, expondo internamente com o servico chamado `nginx-resolver-service`.

- Teste se voce consegue pesquiser os nomes de servico e pod dentro do cluster. user a imagem `busybox` para criar um pod para pesquisa de dns.

- Registre o resultado em `/root/CKA/nginx.svc` e `/root/CKA/nginx.pod` para resoluções de nome de serviços e pod respectivamente

<details>
  <summary><b>Resposta</b></summary>

1. Utilizando linha de comando:

```sh
k run busybox --image=busybox:1.28 -- sleep 4000

k exec busybox -- nslookup nginx-resolver-service > /root/CKA/nginx.svc

k get pods -o wide # pegar o ip do pod nginx-resolver

k exec busybox -- nslookup 10-244-0-4.default.pod.cluster.local > /root/CKA/nginx.pod
```

- Validando se o pod foi criado.

```sh
cat /root/CKA/nginx.svc
cat /root/CKA/nginx.pod
```

</details>

---

### Questão 8

Criar um pod no node01 chamado `nginx-critical` com a imagem  `nginx`, que em caso de falha recrie ou restart.

Use o `/etc/kubernetes/manifests` path de exemplo.

<details>
  <summary><b>Resposta</b></summary>

1. Utilizando linha de comando:

```sh
k get nodes -o wide #pegar ip do node01 
ssh ip-do-node01
k run nginx-critical --image=nginx --restart=Always --dry-run=client -o yaml
```

- Validando se o pod foi criado.

```sh
k get po 
k describe po nginx-critical
```

</details>

---
