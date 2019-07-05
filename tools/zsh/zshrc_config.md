# zshrc config

### iternmal 显示ip
```
IP=$(ip addr | grep -w "inet" | grep "192.168.21" | awk '{ print $2; }' | sed 's/\/.*$//')
PS1='${ret_status} %{$fg[cyan]%}[%n@$IP %~] $ %{$reset_color%} $(git_prompt_info)'
```