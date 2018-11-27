# note

1. 修改主机名: 

```bash
hostnamectl 
hostnamectl --static set-hostname <new_hostname>
hostnamectl --transient set-hostname <new_hostname>
hostnamectl --pretty set-hostname <new_hostname>
# 不需要重启
```

