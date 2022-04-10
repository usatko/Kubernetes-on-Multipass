# cmd for creating master node:
```
multipass launch -c 2 -m 4G -d 20G --cloud-init ctrl-cloud-config.yaml -n master
```

# cmd for creating worker nodes:
```
for i in 1 2 3
do
multipass launch -c 2 -m 4G -d 10G --cloud-init cloud-config.yaml -n node-$i
done
```

# cmd for join workers to cluster:
```
sudo kubeadm token create --print-join-command > join-command.sh
```

