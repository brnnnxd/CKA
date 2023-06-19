## Dicas

### Doc Ref - [kubernetes.io](https://kubernetes.io/pt-br/docs/reference/kubectl/cheatsheet/)

#### Create Alias

- BASH

```sh
source <(kubectl completion bash) # configuração de autocomplete no bash do shell atual, o pacote bash-completion precisa ter sido instalado primeiro.
echo "source <(kubectl completion bash)" >> ~/.bashrc # para adicionar o autocomplete permanentemente no seu shell bash.
```

- ZSH

```sh
autoload -U +X bashcompinit && bashcompinit # Load and initialize bash completion compatibility in Zsh
autoload -U +X compinit && compinit # Load and initialize Zsh completion system
```

```sh
source <(kubectl completion zsh)  # configuração para usar autocomplete no terminal zsh no shell atual
echo '[[ $commands[kubectl] ]] && source <(kubectl completion zsh)' >> ~/.zshrc # adicionar auto completar permanentemente para o seu shell zsh
```

- Criando Alias (Pode ser utilizado em ambos os sh)

```sh
complete -o default -F __start_kubectl k # Configura o autocompletar para o comando 'k' usando a função '__start_kubectl'.
alias k='kubectl'  # Abreviação para o comando 'kubectl' como 'k'.
alias kgp='kubectl get pods'  # Atalho para 'kubectl get pods'.
alias kgs='kubectl get services'  # Atalho para 'kubectl get services'.
alias knd='kubectl describe'  # Atalho para 'kubectl describe'.
alias krm='kubectl delete'  # Atalho para 'kubectl delete'.
alias kaf='kubectl apply -f'  # Atalho para 'kubectl apply -f'.
alias kl='kubectl logs'  # Atalho para 'kubectl logs'.
alias kex='kubectl exec -it'  # Atalho para 'kubectl exec -it'.
```

```sh
export KUBE_EDITOR=vim # Define o editor padrão do Kubernetes como o Vim.
```

#### Ponto de Atenção

- Antes de responder a uma pergunta, é essencial validar o contexto atual para garantir que nossas respostas estejam corretas. Para verificar o contexto atualmente utilizado, podemos utilizar o seguinte comando no terminal.

```sh
kubectl config current-context
```

- Lista `SHORTNAMES` dos commandos.

```sh
kubectl api-resources
```
