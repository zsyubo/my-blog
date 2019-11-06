```shell
[root@localhost ~]# systemctl get-default   
graphical.target
// 设置非图形化
[root@localhost ~]#systemctl set-default multi-user.target
```

如果需要还原，设置回原值就行。

