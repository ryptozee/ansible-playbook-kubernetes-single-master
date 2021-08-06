# An Ansible playbook for setting up a Kubernetes cluster with a single master control plane

## Preparation
- ### Put your hosts and IPs in ```hosts``` file
```
[nodes:children]
master
worker

[master]
master-1 ansible_host=192.168.1.95

[worker]
node-1 ansible_host=192.168.1.91
node-2 ansible_host=192.168.1.92
...
node-N ansible_host=...
```

- ### Put your ssh username to ```remote_user``` in ```ansible.cfg```
```
remote_user = <user_name>
```

- ### Put the etcd, kubernetes gz files under the ```bin``` folder
    ### You can find the gz files from

    etcd: https://github.com/etcd-io/etcd/releases

    kubernetes: https://github.com/kubernetes/kubernetes/tree/master/CHANGELOG

- ### Also put the cfssl files under the ```bin``` folder
    ### You at least need ```cfssl``` ```cfssljson``` ```cfssl-certinfo``` for this playbook
    ### These binaries can be found from
    https://github.com/cloudflare/cfssl/releases

- ### Change the keys in ```vars.yml``` to the file names you have downloaded
```
etcd_gz_file: etcd-v3.4.16-linux-amd64.tar.gz
kubernetes_gz_file: kubernetes-server-linux-amd64.tar.gz
cfssl_binary: cfssl_1.4.1_linux_amd64
cfssl_certinfo_binary: cfssl-certinfo_1.4.1_linux_amd64
cfssljson_binary: cfssljson_1.4.1_linux_amd64
```

- ### Set ```master_node``` to the hostname of your master-node

## Customization
- ### Modify ```pause_image_url``` if you want to use a different version or remote registry for your pause image
- ### Put ```config.toml``` under folder ```cfg/containerd/``` and Uncomment line 77-85 in ```init.yml``` if you want to your own configuration for containerd
```
  # - name: Import containerd configuration file
  #   copy:
  #     src: cfg/containerd/config.toml
  #     dest: /etc/containerd/
  # - name: Restart containerd
  #   systemd:
  #     name: containerd
  #     enabled: yes
  #     state: restarted
```
- ### Comment line 220-229 in ```master.yml``` if you don't want to run kubelet and kube-proxy on the master node
```
  - name: Enable and Start Worker Nodes
    systemd:
      name: "{{ item }}"
      enabled: yes
      state: started
    with_items:
    - kubelet
    - kube-proxy
    tags:
    - start_worker_plane
```

## Play
- ### ```ansible-playbook init.yml -kK```

## Note
- ### Based on Ubuntu 20.04
- ### You can find the kubeconfig file with cluster:admin previledge under ```ca/<master_node>/opt/kubernetes/ssl```, use this kubeconfig to connect to the kube-apiserver
- ### Only support single master node