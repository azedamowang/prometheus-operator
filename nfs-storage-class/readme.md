安装持久化nfs


# 服务端

$ yum install -y nfs-utils rpcbind

# 客户端

$ yum install -y nfs-utils

#启动NFS服务

systemctl start rpcbind && systemctl enable rpcbind

systemctl start nfs && systemctl enable nfs

#NFS服务配置/etc/exports

/data/share 10.222.77.0/24(rw,sync,insecure,no_subtree_check,no_root_squash)

showmount NFS服务器IP  查看是否有共享nfs目录



安装kubernetes1.20.X版本需要修改镜像 vbouchaud/nfs-client-provisioner
