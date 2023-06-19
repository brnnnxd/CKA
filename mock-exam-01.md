## mock exam 01

### Questão 1

Criar um pod com nome `nginx-pod` usando a imagem `nginx:alpine`
<details>
  <summary><b>Resposta</b> <em>(clique para ver a resposta)</em></summary>

- Nesse caso, temos duas formas.

1. A primeira, utilizando somente a linha de comando:

```sh
k run nginx-pod --image=nginx:alpine
```

2. A segunda, e mais recomendada. Eu acho ela melhor pelo fato de você
poder analisar com mais tranquilidade o que você está criando. Mas é a minha opinião apenas.

```sh
k run nginx-pod --image=nginx:alpine --dry-run=client -o yaml > quest-1.yaml

k create -f quest-1.yaml
```

</details>

### Questão 2

Realizar o deploy de um pod com nome de `messaging` usando a imagem `redis:alpine` com o labels `tier=msg`.  

<details>
  <summary><b>Resposta</b> <em>(clique para ver a resposta)</em></summary>
1. Utilizando somente a linha de comando:

```sh
k run messaging --image=redis:alpine --labels="tier=msg"
```

2. Criando o manifesto e aplicando.

```sh
k run messaging --image=redis:alpine --labels="tier=msg" --dry-run=client -o yaml > quest-2.yaml

k create -f quest-2.yaml
```

- Validando se o pod subiu com labels.

```sh
k describe po messaging |grep Labels
k get po
```

</details>

### Questão 3

Realize a criação de um namespace com nome de `aptx-1020`.
<details>
  <summary><b>Resposta</b> <em>(clique para ver a resposta)</em></summary>

1. Utilizando linha de comando para criar o namespace:

```sh
k create ns aptx-1020
```

- Validando se o namespace foi criado.

```sh
k get ns
```

</details>

### Questão 4

Obtenha a lista de nodes no formato `JSON`, armazenar em um arquivo no diretório `/opt/outputs/nodes-xpto.json`.

<details>
  <summary><b>Resposta</b> <em>(clique para ver a resposta)</em></summary>

1. Utilizando linha de comando para criar o arquivo com o output do `k get no -o json`:

```sh
k get no -o json > /opt/outputs/nodes-xpto.json
```

- Validando se o namespace foi criado.

```sh
cat /opt/outputs/nodes-xpto.json
```

</details>

### Questão 5

Criar um servico `messaging-service` para expor a porta `6379` do pod `messaging`

<details>
  <summary><b>Resposta</b> <em>(clique para ver a resposta)</em></summary>

1. Utilizando linha de comando:

```sh
k expose --help
k expose po messaging --port 6379 --name=messaging-service
```

- Validando se o namespace foi criado.

```sh
k get svc
k describe svc messaging-service
```

</details>

### Questão 6

Criar um `Implantação(deployment)` com nome `web-app` usando a imagem `nginx:alpine` com `2` replicas.

<details>
  <summary><b>Resposta</b> <em>(clique para ver a resposta)</em></summary>

- Nesse caso, temos duas formas.

1. A primeira, utilizando somente a linha de comando:

```sh
k create deployment web-app --image=nginx:alpine --replicas=2
```

2. A segunda, e mais recomendada. Eu acho ela melhor pelo fato de você
poder analisar com mais tranquilidade o que você está criando. Mas é a minha opinião apenas.

```sh
k create deployment web-app --image=nginx:alpine --replicas=2 --dry-run=client -o yaml > quest-6.yaml

k create -f quest-6.yaml
```

- Validando se o namespace foi criado.

```sh
k get deploy
```

</details>

### Questão 7

Criar um pod statico com nome `static-busybox` no controlplane node, usando a imagem `busybox` com o command `sleep 1000`.

<details>
  <summary><b>Resposta</b> <em>(clique para ver a resposta)</em></summary>

- Nesse caso, temos duas formas.

1. Utilizando linha de comando:

```sh
k run static-busybox --image=busybox --command -- sleep 1000
```

2. Utilizando dry-run.

```sh
k run static-busybox --image=busybox --command -- sleep 1000 --dry-run=client -o yaml > quest-7.yaml

cp quest-7.yaml /etc/kubernets/manifests/
```

- Validando se o pod foi criado.

```sh
k get po
k describe po static-busybox
```

</details>

### Questão 8

Criar um pod no namespace `financeiro` com nome `temp-bus` com a imagem  `redis:alpine`.

<details>
  <summary><b>Resposta</b> <em>(clique para ver a resposta)</em></summary>

1. Utilizando linha de comando:

```sh
k run temp-bus --image=redis:alpine -n financeiro
```

- Validando se o pod foi criado.

```sh
k get po -n financeiro
k describe po temp-bus -n financeiro
```

</details>

### Questão 9

Expor o deployment `web-app` criando um servico com nome `web-app-service` para expor a porta `30082` para o nodes do cluster.

A aplicacao esta ouvindo na porta `8080`.

<details>
  <summary><b>Resposta</b> <em>(clique para ver a resposta)</em></summary>

1. Utilizando linha de comando:

```sh
k expose --help
k expose deploy web-app --name=web-app-service --type NodePort --port 8080 
k describe svc web-app-service
k edit svc web-app-service # alterar NodePort para `30082`.
```

- Validando se o namespace foi criado.

```sh
k get svc
k describe svc web-app-service
```

</details>

### Questão 10

Use a consulta JSON PATH para recuperar o `osImage` de todos os nodes e armazene em um arquivo em `/opt/ooutputs/nodes_os_xpto.txt`.

<details>
  <summary><b>Resposta</b> <em>(clique para ver a resposta)</em></summary>

1. Utilizando linha de comando:

```sh
k describe nodes
k get nodes -o jsonpath='{.items[*].status.nodeInfo.osImage}' > /opt/ooutputs/nodes_os_xpto.txt
```

- Validando se o namespace foi criado.

```sh
cat /opt/ooutputs/nodes_os_xpto.txt
```

</details>

### Questão 11

Crie um `Persistent Volume` com as especificacoes abaixo

Nome do Volume: `pv-analytics`
Storage: `100Mi`
modo de acesso: `ReadWriteMany`
Host Path: `/pv/data/analytics`

<details>
  <summary><b>Resposta</b> <em>(clique para ver a resposta)</em></summary>

1. Utilizando linha de comando:
Criado um arquivo apartir da documentacao [Link](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-analytics
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /pv/data/analytics
```

```sh
k vim pv-analytics.yaml
k create -f pv-analytics.yaml

```

- Validando se o namespace foi criado.

```sh
k get pv 
```

</details>
