### 1. Подготовка
версия дистрибутива:  
```
cat /etc/*release*
```
версия ядра:
```
uname -r
```
подготовимся к работе с дистрибутивом:
```
sudo yum groupinstall "Development Tools"
# псевдо-графическая оболочка:
sudo yum install ncurses-devel
sudo yum install hmaccalc zlib-devel binutils-devel elfutils-libelf-devel bc openssl-devel
sudo yum install wget top vim
```
идем на [сайт](https://www.kernel.org) с ядрами и копируем ссылку на дистрибутив:
```
sudo wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.11.1.tar.xz
sudo tar -Jxf linux-5.11.1.tar.xz
sudo rm -rf linux-5.11.1.tar.xz
sudo ln -s linux-5.11.1/ linux
ls -la
```
 
### 2. Сборка ядра
сбросить настройки, если что-то пошло не так:
```
sudo make mrproper
```
конфигурация настроек:  
"M" - будет модулем;  
"*" - модуль встроен в ядро;  
" " - отключен.  
```
sudo make menuconfig
```
после выбора настроек проверяем, нужно ли установить дополнительные модули:
```
sudo make dep
sudo make bzImage
```
ядро будет собираться около получаса.

### 3. Сборка модулей
чтобы процесс работал, если прервется ssh-соединение:
```
sudo yum install screen
screen
sudo make modules 
```
отсоединимся от процесса:  
Ctrl + A  
d
если процесс прерывается - перезапускаем:
```
screen -x
sudo make modules 
```
модули будут собираться около часа.
### 4. Итоговая сборка
перекладываем собранные модули:
```
sudo make modules_install
```
можно запустить установку ядра так, но мы не будем:
```
sudo make install
```
настройки grub2:
```
sudo cat /boot/grab2/grub.cfg
vim /etc/default/grub
```
установить настройку grub - чтобы грузилось первое по-списку:
```
GRUB_DEFAULT=0
```
перезапускаем сборку grub:
```
grub2-mkconfig -o /boot/grub2/grub.cfg
```
перезапуск:
```
uname -r
reboot
uname -r
```
### 5. Возможные проблемы
версия gcc устарела (мне не удалось поднять версию с 4.8.1 до 4.9.0):
```
gcc -v
sudo yum install libmpc-devel mpfr-devel gmp-devel

cd ~/Downloads
curl ftp://ftp.mirrorservice.org/sites/sourceware.org/pub/gcc/releases/gcc-4.9.2/gcc-4.9.2.tar.bz2 -O
tar xvfj gcc-4.9.2.tar.bz2

cd gcc-4.9.2
./configure --disable-multilib --enable-languages=c,c++
make -j 4
make install
```
