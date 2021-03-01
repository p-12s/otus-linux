# Управление пакетами

## RPM (redhat package manager)
установим RPM:
```
sudo yum install epel-release -y
```
скачаем RPM-пакет
```
yumdownloader nginx
```
проходим в скачанный пакет с midnight commander, и видим все файлы (мета-инфо, файлы, скриплеты).  
установим пакет nginx (не искал как ставить с rpm):
```
sudo yum install nginx -y
```
посмотрим инфу о пакете:
```
rpm -qi nginx
```
команды:
```
rpm -q {name}               — проверить установлен ли пакет {name}
rpm -qi {name}/{pkg}        — показать мета-информацию о пакете
rpm -qp --queryformat %{VERSION}-%{RELEASE} {pkg} 
                            — формат вывода
информа􏰁ии
rpm -e {name}               — удалить пакет
rpm -i {pkg}                — установить пакет
rpm -ql {name}/{pkg}        — вывести список файлов пакета
rpm -q --scripts            — показать скриптлет􏰃
rpm -qR                     — показать от каких пакетов зависит 􏰄тот
rpm -q --provides {name}    — показать ресурс􏰃 предоставл􏰀ем􏰃е пакетом
rpm -qf {file}              — показать какому пакету принадлежит {file}.
rpm2cpio {pkg} | cpio -idmv — распаковать содержимое пакета
```

## тест сборки пакепа
в одном окне скачаем пакет для примера:
```
yumdownloader tree
```
начнем сборку:
```
sudo yum install rpmdevtools rpm-build rpmdev-setuptree
tree -d -L 1 ~/rpmbuild
~/rpmbuild
```
содержимое:
```
├── BUILD # директория в которой происходит сборка 
├── RPMS # директория с собранными пакетами
├── SOURCES # директория с исходными файлами 
├── SPECS # директория с spec-файлами
└── SRPMS # директория с SRPM-пакетами
```
по-аналогии с пакетом tree добавим spec-файл:

```
sudo
```
