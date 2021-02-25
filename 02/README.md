# Дисковая подсистема, RAID, партиции и SWAP
убедимся, что список RAID-дисков пуст:
```
sudo cat /proc/mdstat
```
## RAID-0
список подключенных дисков (из файла [vagrantfile](vagrantfile))
```
lsblk
```
соберем RAID-0:
```
mdadm --create --verbose /dev/md0 --level=0 --raid-device=2 /dev/sdb /dev/sdc
```
убедимся, что RAID подцепился и на нем собираем файловую систему:
```
lsblk
sudo mkfs.xfs /dev/md0
sudo mount /dev/md0 /media/
df -Th
```
## RAID-1
соберем RAID-1:
```
mdadm --create --verbose /dev/md1 --level=1 --raid-device=2 /dev/sdd /dev/sde
```
собираем файловую систему:
```
lsblk
sudo mkfs.ext4 /dev/md1
sudo mount /dev/md1 /mnt
df -Th
```
убедимся, что во 2 варианте размер диска в 2 раза меньше - это же зеркало.  
список RAID-дисков:  
```
sudo cat /proc/mdstat
или так:
sudo mdadm -D /dev/md0
```
вывод:
```
Personalities : [raid0] [raid1]
md1 : active raid1 sde[1] sdd[0] - raid-1, 2 устройства
      254976 blocks super 1.2 [2/2] [UU] - оба юнита работают

md0 : active raid0 sdc[1] sdb[0]
      507904 blocks super 1.2 512k chunks
```
## сымитируем фейл диска
разделим окно терминала пополам и запустим в одном из них цикл:
```
sudo watch cat /proc/mdstat
```
а во 2 окне зафейлим один диск:
```
sudo mdadm /dev/md1 --fail /dev/sdd
```
вывод:
```
Personalities : [raid0] [raid1]
md1 : active raid1 sde[1] sdd[0](F)
      254976 blocks super 1.2 [2/1] [_U] - один юнит стал недоступен

md0 : active raid0 sdc[1] sdb[0]
      507904 blocks super 1.2 512k chunks
```
например, возникла подобная ситуация с недоступностью юнита. проверим:
```
sudo cat /proc/mdstat
```
похоже RAID сломался физически. исключим его из массива рейдов перед извлечением:
```
sudo mdadm /dev/md1 --remove /dev/sdd
```
починили и воткнули рабочий диск в стойку, запускаем:
```
sudo mdadm /dev/md1 --add /dev/sdd
```
прошел recovery:
```
Personalities : [raid0] [raid1]
md1 : active raid1 sdd[2] sde[1]
254976 blocks super 1.2 [2/2] [UU]

md0 : active raid0 sdc[1] sdb[0]
507904 blocks super 1.2 512k chunks
```
## SWAP
посмотрим текущие настройки:
```
free -mh
```
увеличим swap:  
```
sudo dd if=/dev/zero of=/swapfile bs=3072 count=1048576
mkswap /swapfile
swapon /swapfile
free -mh
```
можем удалить:
```
swapoff /swapfile
free -mh
```
## Создание GPT-разделов, партиций и их монтирование (не погружался)
создать GPT раздел, пять партиций и смонтировать их на диск:
```
parted -s /dev/md0 mklabel gpt Создаем партиции
parted /dev/md0 mkpart primary ext4 0% 20% 
parted /dev/md0 mkpart primary ext4 20% 40% 
parted /dev/md0 mkpart primary ext4 40% 60% 
parted /dev/md0 mkpart primary ext4 60% 80% 
parted /dev/md0 mkpart primary ext4 80% 100%
```
создать на этих партициях файловые системы:
```
for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done
```
смонтировать их по каталогам:
```
mkdir -p /raid/part{1,2,3,4,5}
for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done
```
