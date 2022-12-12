安装持久化nfs


# 服务端

$ yum install -y nfs-utils rpcbind

# 客户端

$ yum install -y nfs-utils

#启动NFS服务

systemctl start nfs && systemctl enable nfs

systemctl start rpcbind && systemctl enable rpcbind

#NFS服务配置/etc/exports

/data/share 10.222.77.0/24(rw,sync,insecure,no_subtree_check,no_root_squash)

showmount NFS服务器IP  查看是否有共享nfs目录



安装kubernetes1.20.X版本需要修改镜像 vbouchaud/nfs-client-provisioner


在NFS服务器端(RHEL6.5)使用命令 showmount -e 127.0.0.1 报错，提示：clnt_create: RPC: Program not registered

解决办法：

在服务器上先停止rpcbind，
          service rpcbind stop

     2. 然后在停止nfs

         service nfs stop

     3. 最后在重启rpcbind和nfs，一定要按顺序启动和停止

          service rpcbind start

          service nfs start
