# cmd for creating control node:
```
multipass launch -c 2 -m 4G -d 20G --cloud-init control/cloud-config.yaml -n control
```

# cmd for creating worker nodes:
```
for i in 1 2 3
do
multipass launch -c 2 -m 4G -d 10G --cloud-init wroker/cloud-config.yaml -n node-$i
done
```
