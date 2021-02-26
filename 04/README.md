# Установка сервера и клиента NFS на CentOS 7
Network File System (NFS) позволяет монтировать удаленные каталоги на своем сервере, и обращаться к ним как к своим директориям.
## Установка на NFS на сервере
пакет:
```
sudo yum install nfs-utils -y
```
папка для шары:
```
mkdir /var/nfsshare
chmod -R 777 /var/nfsshare/
```
запуск служб и добавление их в автозагрузку:
```
systemctl enable rpcbind
systemctl enable nfs-server
systemctl enable nfs-lock
systemctl enable nfs-idmap
systemctl start rpcbind
systemctl start nfs-server
systemctl start nfs-lock
systemctl start nfs-idmap
```
теперь поделимся каталогом:
```
nano /etc/exports
```
и опишем шару (в примере - ip клиентской машины):
```
/var/nfsshare    192.168.0.101(rw,sync,no_root_squash,no_all_squash)
если вместо ip использовать * - смогут подключаться все
```
перезапустим службу:
```
запустим службу NFS
```
возможно понадобится добавить записи в firewall:
```
firewall-cmd —permanent —add-port=111/tcp
firewall-cmd —permanent —add-port=54302/tcp
firewall-cmd —permanent —add-port=20048/tcp
firewall-cmd —permanent —add-port=2049/tcp
firewall-cmd —permanent —add-port=46666/tcp
firewall-cmd —permanent —add-port=42955/tcp
firewall-cmd —permanent —add-port=875/tcp
firewall-cmd —permanent —zone=public —add-service=nfs
firewall-cmd —permanent —zone=public —add-service=mountd
firewall-cmd —permanent —zone=public —add-service=rpc-bind
firewall-cmd --reload
```
## Установка на NFS на клиенте
пакет:
```
sudo yum install nfs-utils -y
```
точка монтирования:
```
mkdir -p /mnt/nfs/home
```
запуск служб и автозагрузка:
```
systemctl enable rpcbind
systemctl enable nfs-server
systemctl enable nfs-lock
systemctl enable nfs-idmap
systemctl start rpcbind
systemctl start nfs-server
systemctl start nfs-lock
systemctl start nfs-idmap
```
монтируем:
```
mount -t nfs 192.168.0.100:/home /mnt/nfs/home/
```
перепроверим состояние (в списке устройств должна буть примонтированная директория):
```
df -kh
```
## Автомонтирование NFS при перезапуске
на клиенте добавим запись в файл:
```
nano /etc/fstab

192.168.0.100:/home    /mnt/nfs/home   nfs defaults 0 0
```
## SSHFS
FUSE - модуль для ядер Linux/Unix.  
Запуск кода файловой системы в пользовательском пространстве,   
что в свою очередь позволяет монировать эти ФС непривилегированным пользователям.  
Используется, например, в - SSHFS, NTFS-3G, GlusterFS, ZFS.  
```
sudo yum install sshfs
sshfs your.server.name:/path /mountpoint
fusermount -u /mountpoint
/etc/fstab

sshfs#user@host:/ /mnt/host fuse defaults 0 0
```
