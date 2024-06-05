Ctrl+Shift V

### ip
```console
ip -c a
ip addr show enp3s0
ip addr change dev enp3s0 192.168.1.100
```

### ifconfig
```console
sudo apt install net-tools
sudo ifconfig <port name> 192.168.1.201
```

### ssh
```console
sudo apt-get install openssh-server
sudo service ssh status
```

### gcc
```console
sudo apt install g++-10 gcc-10
gcc-10 --version
```

### tmux
```console
sudo apt install tmux
tmux attach [tmux a]
tmux a -t <name>
tmux detach-client
tmux ls
tmux kill-session -t <name>
```

### alias
```console
alias # print list of aliases
```

### узнать pid процесса, который занимает тот или иной порт
```console
netstat -tulpn
```

### tcpdump
```console
sudo tcpdump -i enp3s0
sudo tcpdump -i enp3s0 src 192.168.1.50
sudo tcpdump -i enp3s0 port 25200 -c 20
sudo tcpdump -i enp3s0 ip multicast -v
sudo tcpdump host
```

