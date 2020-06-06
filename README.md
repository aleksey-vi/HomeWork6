# Стенд для работы по домашнему заданию № 6 "Загрузка Системы"

## Задание:

 - Попасть в систему без пароля несколькими способами
 - Установить систему с LVM, после чего переименовать VG
 - Добавить модуль в initrd

Начнем с того, что я немного отредактировал Vagrantfile с прошлого домашнего задания, а именно дописал в начало: <br>

    MACHINES = {
        :lesson6 => {
              :box_name => "centos/7",
              :box_version => "1804.02",
              :ip_addr => '192.168.11.106',
                    },
               }

после чего добавил строку:

    vb.gui = true


## 1. Попасть в систему без пароля несколькими способами

  Для получения доступа открываем GUI VirtualBox, запускаем вм. При выборе ядра для загрузки нажать e - в данном контексте edit. Попадаем в окно где мы можем изменить параметры загрузки.

### 1.1 rw init=/sysroot/bin/sh

  В строке начинающейся с linux16 заменяем
  **ro** нa **rw init=/sysroot/bin/sh**, удаляем параметр **console=ttS0,115200n8** и нажимаем сtrl+x для загрузки в систему.

  Файловая система сразу смонтирована в режим Read-Write

### 1.2 rd.break

  В конце строки начинающейся с linux16 добавляем **rd.break console=tty1** и нажимаем сtrl-x для загрузки в систему. Попадаем в emergency mode. Корневая файловая система смонтирована. Попасть в нее и поменять пароль администратора:

    mount -o remount,rw /sysroot
    chroot /sysroot
    passwd root
    touch /.autorelabel

После чего можно перезагружаться и заходить в систему с новым паролем.

### 1.3 init=/bin/sh

В конце строки начинающейся с linux16 удаляем параметр **console=ttS0,115200n8**, добавляем **init=/bin/sh**, нажимаем сtrl-x для загрузки в систему. Рутовая файловая система при этом монтируется в режиме Read-Only. Если вы хотите перемонтировать ее в режим Read-Write можно воспользоваться командой:

    mount -o remount,rw /

После чего можно убедиться записав данные в любой файл или прочитав вывод команды: **mount | grep root**


## 2. Установить систему с LVM, после чего переименовать VG


Первым делом посмотрим текущее состояние системы (нас интересует вторая строка с именем Volume Group):

    vgs

Приступим к переименованию:

    vgrename VolGroup00 OtusRoot

Далее правим **/etc/fstab**, **/etc/default/grub**, **/boot/grub2/grub.cfg**.И тут есть два способа: <br>
1) Открываем все эти 3 файла по очереди в редакторе и меняем ***VolGroup00*** на ***OtusRoot***. <br>
2) Для правки используем sed, как например для редактирования /boot/grub2/grub.cfg команда будет иметь вид:

    cat /boot/grub2/grub.cfg | grep VolGroup00
    sed -i 's/VolGroup00/OtusRoot/g' /boot/grub2/grub.cfg

Пересоздаем initrd image, чтобы он знал новое название Volume Group:

    mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)

После чего можем перезагружаемся с новым именем Volume Group и проверяем

    vgs

## 3 Добавить модуль в initrd

В каталоге с скриптами модулей создаем папку с именем 01test:

    mkdir /usr/lib/dracut/modules.d/01test

В нее помещаем два скрипта:

1. [https://gist.github.com/lalbrekht/e51b2580b47bb5a150bd1a002f16ae85] module-setup.sh - который устанавливает модуль и вызывает скрипт test.sh

2. [https://gist.github.com/lalbrekht/ac45d7a6c6856baea348e64fac43faf0] test.sh - собственно сам вызываемый скрипт, в нём у нас рисуется пингвинчик

Затем командой **mv** мы переименовываем данные файлы, по названиям указанным выше.

Пересобираем образ initrd    

    dracut -f -v

Можно проверить/посмотреть какие модули загружены в образ:

    lsinitrd -m /boot/initramfs-$(uname -r).img | grep test

Отредактируем grub.cfg убрав эти опции:rghb, quiet Для редактирования

    vim /etc/default/grub
    grub2-mkconfig -o /boot/grub2/grub.cfg

В итоге при загрузке будет пауза на 10 секунд и пингвин в выводе терминала ^__^

# При выполнении данного задания я пользовался следующей информацией: <br>

1. https://otus.ru/media-private/49/90/%D0%9F%D1%80%D0%B0%D0%BA%D1%82%D0%B8%D0%BA%D0%B0_%D0%97%D0%B0%D0%B3%D1%80%D1%83%D0%B7%D0%BA%D0%B0_Linux-5373-499096.pdf?hash=WKV2V_-xOZUiUlM8hEtthg&expires=1591468342 <br>
2. https://docs.google.com/document/d/1c6DM3vJ06-SSESpWpWk_vaZy4bvL1CUrFV81cPNGy4c/edit#heading=h.shtebjpns1m1<br>
3. https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/sec-terminal_menu_editing_during_boot#sec-Changing_and_Resetting_the_Root_Password <br>
4. https://habr.com/ru/post/404511/ <br>
5. http://pyatilistnik.org/kak-pereimenovat-faylyi-v-centos/ <br>
6. https://help.ubuntu.ru/wiki/vagrant <br>
7. https://www.vagrantup.com/docs/vagrantfile <br>
8. https://www.vagrantup.com/docs/networking/private_network.html <br>
9. https://gist.github.com/Jekins/2bf2d0638163f1294637#Emphasis <br>
