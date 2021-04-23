# Setup kubectl autocomplete in bash

```bash
sudo apt install bash-completion
source <(kubectl completion bash) # setup autocomplete in bash into the current shell
echo "source <(kubectl completion bash)" >> ~/.bashrc # add autocomplete permanently to bash shell

# optional: use a shortcut alias for kubectl
alias k=kubectl
complete -F __start_kubectl k
```