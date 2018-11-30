1. 无法远程管理节点

```bash
# 报错
workerer576.hadoop.spacestar.com | FAILED! => {
    "failed": true,
    "msg": "Using a SSH password instead of a key is not possible because Host Key checking is enabled and sshpass does not support this.  Please add this host's fingerprint to your known_hosts file to manage this host."
}
```

解决：

修改 ansible 配置文件

```bash
vim /etc/ansible/ansible.cfg
# 这一行注释取消 host_key_checking = False
```

2. playbook 用法

```
ansible-playbook -i inventory/[qa/prod] ./deploy.yml -e tag=$ver -e env=[qa/pro]
```

