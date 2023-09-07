# **Введение**

Выполнение действий приведенных в методичке дает познакомиться с такими инструментами, как `Vagrant` и `Packer`, получить базовые навыки работы с системами контроля версий (`Github`). Получить навыки создания кастомных образов виртуальных машин и основам их распространения через репозиторий `Vagrant Cloud`, а так же навыки по обновлению ядра системы из репозитория.

Все ниже описанные действия производятся на компьютере с установленным `Ubuntu`.
Для выполнения работы потребуются следующие инструменты:

- **VirtualBox** - среда виртуализации, позволяет создавать и выполнять виртуальные машины;
- **Vagrant** - ПО для создания и конфигурирования виртуальной среды. В данном случае в качестве среды виртуализации используется *VirtualBox*;
- **Packer** - ПО для создания образов виртуальных машин;
- **Git** - система контроля версий

А так же аккаунты:

- **GitHub** - https://github.com/
- **Vagrant Cloud** - https://app.vagrantup.com

- # **Установка ПО**

### **Oracle VirtualBox**
Добавляем GPG-ключ репозитория: wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
Добавляем репозиторий VirtualBox: sudo add-apt-repository "deb [arch=amd64] http://download.virtualbox.org/virtualbox/debian $(lsb_release -cs) contrib"
Обновляем список пакетов: sudo apt update 
Установим VirtualBox: sudo apt install -y virtualbox-6.1 
Установим VirtualBox extension pack: sudo apt install -y virtualbox-ext-pack 
(Во время установки потребуется принять лицензионное соглашение)
На этом установка VirtualBox закончена.

### **Vagrant**

Добавляем GPG-ключ репозитория: curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
Добавляем репозиторий Hashicorp: sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
Обновляем список пакетов: sudo apt update
Установим Vagrant: sudo apt install -y vagrant
Проверим версию Vagrant: vagrant version \
Версия Vagrant должна быть 2.2.17 или выше
     На этом установка Hashicorp Vagrant завершена

### **Packer**

Packer также можно установить из репозитория Hashicorp

Установим packer: sudo apt install -y packer
Проверим версию packer: packer --version

После успешного окончания будет установлен Packer.


# **Запуск ВМ в Vagrant и обновление ядра**

Создаем Vagrantfile и запускаем ВМ командой- vagrant up

Подключаемся по ssh к созданной виртуальной машины. Для этого в каталоге с нашим Vagrantfile вводим команду vagrant ssh 

Перед работами проверим текущую версию ядра:
[vagrant@kernel-update ~]$ uname -r
4.18.0-277.el8.x86_64

Далее подключим репозиторий, откуда возьмём необходимую версию ядра:
sudo yum install -y https://www.elrepo.org/elrepo-release-8.el8.elrepo.noarch.rpm 

В репозитории есть две версии ядер:
kernel-ml — свежие и стабильные ядра
kernel-lt — стабильные ядра с длительной версией поддержки, более старые, чем версия ml.

Установим последнее ядро из репозитория elrepo-kernel:
sudo yum --enablerepo elrepo-kernel install kernel-ml -y

Параметр --enablerepo elrepo-kernel указывает что пакет ядра будет запрошен из репозитория elrepo-kernel.

Уже на этом этапе можно перезагрузить нашу виртуальную машину и выбрать новое ядро при загрузке ОС. 

Если требуется, можно назначить новое ядро по-умолчанию вручную:
1) Обновить конфигурацию загрузчика:
   sudo grub2-mkconfig -o /boot/grub2/grub.cfg
2) Выбрать загрузку нового ядра по-умолчанию:
   sudo grub2-set-default 0

Далее перезагружаем нашу виртуальную машину с помощью команды sudo reboot

После перезагрузки снова проверяем версию ядра (версия должа стать новее):
[vagrant@kernel-update ~]$ uname -r 
6.5.1-1.el8.elrepo.x86_64

На этом обновление ядра закончено

# **Packer**

Теперь необходимо создать свой образ системы, с уже установленым ядром 6й версии. Для это воспользуемся ранее установленной утилитой `packer`. В директории `packer` создаем все необходимые настройки (файл centos.json & ks.cgf)и скрипты (папка scripts) для создания необходимого образа системы.

Cоздадим образ системы с помощью команды: packer build centos.json

После успешного создания образа в Packer выдаст следующее сообщение:
==> Builds finished. The artifacts of successful builds are:
--> centos-8: 'virtualbox' provider box: centos-8-kernel-5-x86_64-Minimal.box 

Также после успешного создания образа в каталоге packer появился файл сentos-8-kernel-5-x86_64-Minimal.box

# **Vagrant cloud**

Публикация образа:

vagrant cloud auth login
vagrant cloud publish --release petriaevmaksim/centos8-kernel5 1.0 virtualbox centos-8-kernel-5-x86_64-Minimal.box

# **Заключение**

В результате выполнения ранее описанных действий получены: виртуальная машина из базового репозитория с помощью Vagrant, создан кастомный образаз с обновленным ядром с помощью packer, этот образ опубликован в vagrant cloud. Результаты проделанной работы с vagrantfile на опубликованный образ выложены в репозитории github.
