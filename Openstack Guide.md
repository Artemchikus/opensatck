Мой дневник по практике

1\. Подготовительная работа для создания ВМ.

1.1 Проверка поддержки виртуализации, если виртуализация есть, то должен быть такой вывод.

![](media/99e0cd9fa03dea087f9a3fcc82d4b370.png)

1.2. Установка необходимых пакетов: [**bridge-utils**](https://linuxhint.com/bridge-utils-ubuntu/) (создание моста), [**libvirt-daemon-system**](https://packages.debian.org/unstable/libvirt-daemon-system) (создание ВМ), [**qemu**](https://redos.red-soft.ru/base/arm/os-virtual/qemu-kvm/qemu-kvm/)[ **и** ](https://redos.red-soft.ru/base/arm/os-virtual/qemu-kvm/qemu-kvm/)[**qemu**](https://redos.red-soft.ru/base/arm/os-virtual/qemu-kvm/qemu-kvm/)[**-kvm**](https://redos.red-soft.ru/base/arm/os-virtual/qemu-kvm/qemu-kvm/) (работа гипервизора).

![](media/13fe50de0f46232343408a6f09092292.png)

1.3. Активация **libvirtd** для дальнейшей работы.

![](media/7131abd148fa192cb55f228e2ede5701.png)

1.4. Во всяких [**гайдах**](https://ubuntu-admin.ru/ubuntu-kvm/) говорится, что есть три способа настройки доступа виртуальных машин в сеть: через одну из ВМ (самый трудно настраиваемый и еще доп ВМ нужна), через NAT (уже реализован по дефолту **virbr0**), через Bridge (самый обычный и удобный, его и реализуем, не зря ведь **bridge-utils** качали).

Для этого создаем сетевой интерфейс **br0** типа bridge.

Потом все удаляем так как [**оказывается**](https://unix.stackexchange.com/questions/554331/theoretical-tap-interface-w-wifi-parent-interface), что нельзя проводить мост к wifi сети, только к ethernet, потому что Wi-Fi требует дополнительных MAC-адресов для связи, из-за чего принято решение пользоваться NAT :-)![](media/aabb63e2051375d8d99fe9812179bda0.png)

1.5. Установка софта для обновления времени сервера [**chrony**](https://chrony.tuxfamily.org/), для синхронизации времени виртуалок.

![](media/04e5b3a67a5a369279b2f169ec78bcb4.png)

![](media/6be1dd7ebcb80edb6340859fdb733b7b.png)

2\. Установка образа Centos и создание 3 ВМ с помощью KVM.

2.1. Создание диска с помощью [**qemu-img**](https://docs.altlinux.org/ru-RU/alt-server-v/9.2/html/alt-server-v/ch46s03.html) формата [**qcow2**](https://ru.wikipedia.org/wiki/Qcow2) для монтирования в ВМ (**preallocation=metadata** – выделяет пространство, необходимое для метаданных, но не выделяет место для данных. Это самый быстрый способ представления, но самый медленный для гостевой записи). Папки создал сам, кроме **/mnt**.

![](media/d2e80dfe396b1f6abd735f19f6acb1cb.png)

2.2. Установка утилит [**virt-install**](https://docs.altlinux.org/ru-RU/alt-server-v/9.2/html/alt-server-v/ch46s02.html) (создание ВМ) и [**osinfo-query**](https://man.archlinux.org/man/community/libosinfo/osinfo-query.1.en) (получение информации о поддержке гипервизорами различных операционных систем) для для создания гостевых машин в среде kvm из командной строки.

![](media/524153ceb3ef82407b55a70c77f7e89a.png)

2.3. Просмотр доступных для KVM образов Centos с помощью команды **osinfo-query** **os**.

![](media/20915b4cba2d7bdcbc44c58b75d75015.png)

2.4. Установка дистрибутива операционной системы с сайта [**http://mirror.yandex.ru/centos/7.9.2009/isos/x86_64/**](http://mirror.yandex.ru/centos/7.9.2009/isos/x86_64/) В моем случае загружается образ CentOS-7-x86_64-Minimal-2009.iso. Оказалось, что **Centos** крутая штука, так как ворует все идеи у коммерческой **Red Hat Enterprise Linux**, но при этом абсолютно бесплатная.

2.5. Cоздание ВМ с Centos7.0 с помощью **virt-install** ([**сайт**](https://libvirt.org/formatdomain.html#elementsOS) с документацией параметров).

![](media/8ac3ef7b54aee9e3502be61b7e29faf4.png)

-   **--virt-type kvm** — тип гипервизора
-   **--name VM1** — имя виртуалки
-   **--ram 4096** — оперативка
-   \-**-arch=x86_64** - имитируемая архитектура процессора
-   \-**-disk /mnt/kvm/disk/vm1.qcow2,size=6,format=qcow2** — пространство хранения данных, **size** — размер, **format** — формат тома.
-   **--network network=default** — сетевой интерфейс (**default=virbr0**)
-   \-**-os-type=linux** — тип операционной системы
-   **--os-variant=centos7.0** — конкретный вариант ОС
-   **--location=/mnt/kvm/iso/CentOS-7-x86_64-Minimal-2009.iso** — путь до установочного образа.
-   **--graphics none** — настройки экрана ВМ (**none** — без графического интерфейса)
-   **--console pty,target_type=serial** — тип подкюченя к гостевой консоли (**pty** - обеспечивают работу терминала в оконном интерфейсе, **target_type=serial** — дефолтная консоль, не открывается новое окно)
-   **--extra-args 'console=ttyS0,115200n8 serial'** — дополнительные аргументы, в моем случае параметры консоли в которую будет выводится информация иначе не видать меню загрузки.

2.6. Настройка с помощью меню установки **Centos** без графического интерфейса (в первый раз не понял,что за фигня и как этим пользоваться, оказывается циферками). Все загрузится нормально если на месте восклицательных знаков будут крестики.

![](media/3032f11940217764187ee5ef135d8f1c.png)

Настройка сети такая (почему-то скрин поплыл, но там **DNS** и **Gateway** одинаковые — **192.168.122.1**):

![](media/f59b1caf79e15690064e8f09d451b7e2.png)

Заполненное меню загрузки такое, пользователя не создаем:

![](media/1b7be7e322fd275b943b59759d24e981.png)

Готовая ВМ:

![](media/cd4d015b09dd5e2dffa3fc68d9e6f90c.png)

2.7. Полезные команды для работы с ВМ:

-   **virsh list --all** — просмотр всех ВМ
-   **virsh start vm** — запустить неактивную ВМ
-   **virsh shutdown vm** — выключить ВМ через гостевую ОС
-   **virsh destroy vm** — выключить ВМ принудительно (выдернуть из розетки)
-   **virsh domblklist vm** — информация о дисках ВМ
-   **virsh dominfo vm** — информация о ВМ
-   **virsh nodeinfo** — информация о хостовой машине
-   **virsh undefine --domain vm** — удалить всю информацию о ВМ, например чтобы переиспользовать диск который ей был занят (я то 3 раза ее пересоздал)
-   **virsh console vm** — подключение к консоли ВМ
-   **virsh dumpxml vm** — информация о настройках вм
-   **virsh snapshot-list --domain vm** — просмотр снапшотов ВМ
-   **virsh snapshot-create-as vm vm-snapshot** — создание снапшота ВМ
-   **virsh snapshot-info --domain vm --snapshotname vm-snapshot** — информация о снапшоте ВМ
-   **virsh snapshot-create-as --domain vm --name vm-snapshot** — создание снапшота ВМ
    -   **--disk-only** — создание снапшота только диска (без состояния оперативки)
-   **virsh snapshot-revert --domain vm --snapshotname vm-snapshot**  — откатить ВМ к снапшоту
    -   **--running —** запустить ВМ
-   **virsh snapshot-delete --domain vm --snapshotname vm-snapshot** — удалить снапшот ВМ
-   **virsh net-create path_to_net_conf** — создать сеть для ВМ
-   **virsh net-list** — список сетей
-   **virsh net-autostart net** — настройка автостарта сети
-   **virsh attach-interface vm --type bridge --source interface --model virtio --config --live --persistent** — присоединение сетевого интерфейса к ВМ
    -   **--source —** имя сетевого интерфейса на хосте
    -   **--model —** тип интерфейса
    -   **--config —** через файл конфига
    -   **--live —** ВМ не выключать
    -   **--persistent —** подключить навсегда
-   **virsh detach-interface vm3 --mac '52:54:00:3b:6e:3f' --type bridge --persistent** — отсоединение сетевого интерфейса от ВМ
-   **virsh net-info net** — информация о сети
-   **qemu-img info disk1.qcow2** — информация об образе диска
-   **qemu-img resize /mnt/kvm/disk/vmserver01-disk1.qcow2 +1G** — увеличение размера диска
-   **qemu-img info /mnt/kvm/disk/vmserver01-disk1.qcow2** — информация о диске
-   **Ctrl + ]** - выход из гостевой консоли
-   **virt-install --parameter=?** - информация о параметре (лучше сайтом пользоваться с команды пользы 0)
-   **nmcli connection show** — показывает все сетевые интерфейсы кроме loopback
-   **nmcli connection delete interface** — удаляет сетевой интерфейс (если вы выяснили, что мост в wifi не прокидывается)
-   **sudo mv src dst** — перенести файл, если у вас почему-то нет прав на доступа к папке **/mnt** через графический интерфейс Ubuntu

Остальные 2 ВМ поднимаются по аналогичной схеме только с другими ip адресами, требованиями к оперативке и размеру диска.

3\. Установка и настройка RabbitMQ на Control узел.

3.1. Зачем он вообще нужен? Оказывается **RabbitMQ** или какой-нибудь другой брокер сообщений (**Apache Qpid** или **ZeroMQ**) используется для координирования операций и обмена информацией между сервисами **OpenStack**, такими как **Glance**, **Cinder**, **Nova**, **Neutron**, **Heat** и **Ceilometer** по протоколу AMQP (Advanced Message Queuing Protocol).

3.2. [**Установка RabbitMQ**](https://gist.github.com/fernandoaleman/fe34e83781f222dfd8533b36a52dddcc) на ВМ с помощью yum (-y — автоответ yes, если не знали).

Сноска от автора (дочитайте до конца!), пока я пытался установить RabbitMQ по гайдам, я повидал многое, но после долгих поисков мне все таки удалось найти [**нормальное руководство**](https://computingforgeeks.com/installing-rabbitmq-on-centos-6-centos-7/), при помощи которого **RabbitMQ** все таки удалось установить. Но я же проделал такую большую работу качая **RabbitMQ** по другим гайдам, так что было принято решение сохранить мои первые попытки в описание процесса установки, нормальный порядок установки находится во [**тут**](#bookmark=id.1fob9te).

**ВНИМАНИЕ!!!!!** Я публично извиняюсь перед Андреем Маркеловым (автор книги) за [**эти слова**](#bookmark=id.2et92p0), и снимаю перед ним шляпу, так как его способ установки основных утилит для развертывания **openstack** является самым удобным в интернете, достаточно ввести 3 следующие команды после запуска ВМ и все остальные пакеты (сервисы **Openstack** и **RabbitMQ**) будут загружаться через обычный **yum** (таким образом настройка **RabbitMQ** начинается [**здесь**](#bookmark=id.3znysh7)):

![](media/bb3f4cccff418e16e13fcf3496b6622a.png)

3.3. Установка **epel-release** (открытое бесплатное хранилище пакетов от Fedora, видимо нужно для RabbitMQ) и обновление компонентов системы.

![](media/ec4853828cb641b2a727ec3ddfd5fd6f.png)

3.4. Установка **Erlang** - язык, на котором написан **RabbitMQ**. Для этого надо установить [**wget**](https://habr.com/ru/company/ruvds/blog/346640/), так как он не предустановлен, после чего загрузить **rpm** репозиторий с помощью wget, потом добавить его с помощью [**rpm**](https://www.inp.nsk.su/~bolkhov/teach/inpunix/setup_rpm.ru.html) **-Uvh** (**U** - апгрейд пакета, **vh** - для статус бара и дополнительной информации), и в конце установить необходимые пакеты ([**erlang**](https://habr.com/ru/post/50028/), [**socat**](https://linux-notes.org/ustanovka-socat-v-unix-linux/), [**logrotate**](https://1cloud.ru/help/linux/upravlenie-logami-s-pomoshch'yu-logrotate-na-ubuntu-16-04)) с помощью yum, на сколько я понял без них **RabbitMQ** не заработает, как минимум без **erlang**.

![](media/4698f122a01e395b2ccdb551f0dc732f.png)

3.5. Установка **rabbitmq-server**. Для этого устанавливаем rpm пакет с **RabbitMQ** с помощью **wget**, потом добавляем ключ подписи с помощью **rpm** (без него вылезет ошибка), после чего устанавливаем [**rabbitmq-server**](https://habr.com/ru/post/149694/) с помощью **rpm**, запускаем его и настраиваем автозапуск.

Понимаем, что **rabbitmq-server** не запускается, а точнее находится в состоянии activating пока не словит timeout, и решаем эту проблему путем настройки **firewall-cmd** и [**SELinux**](https://habr.com/ru/company/kingservers/blog/209644/).![](media/dfaebd6f5bb778999f558b99ac56218f.png)

![](media/4a987c554ba9d9e8c84ede132409e2d8.png)

![](media/25a2fd9d5d8729bb6bcad59d02ae89e3.png)

Для этого освобождаем следующие [**порты**](https://www.rabbitmq.com/networking.html) (где-то говорится, что достаточно сделать публичным только базовый порт **RabbitMQ**, но на всякий и другие тоже опубликуем, хотя наверное можно просто вырубить фаерволл):

-   **4369** — порт сервиса **epmd** (Erlang Port Mapper Daemon), служба обнаружения одноранговых узлов, используемая узлами RabbitMQ и инструментами CLI
-   **25672** — порт сервера распространения **Erlang**, используется для связи между узлами и инструментами CLI
-   **5671** — порт сервиса [**amqp**](https://habr.com/ru/post/62502/) (Advanced Message Queuing Protocol), тот самый протокол обмена сообщениями, который использует **RabbitMQ**
-   **5672** — порт сервиса **amqps** (amqp protocol over TLS/SSL), из расшифровки и так понятно (как говорится на официальном сайте, **RabbitMQ** не работает если на машине нет OpenSSL 1.1, конфликты с Erlang 24 версии, но может быть все работает, я не проверял)
-   **15672** — порт клиентов [**HTTP API**](https://www.rabbitmq.com/management.html), [**пользовательского интерфейса управления**](https://www.rabbitmq.com/management.html) и [**rabbitmqadmin**](https://www.rabbitmq.com/management-cli.html) без TLS/SSL, почему-то с TLS/SSL не публикуется
-   **61613** — порт сервиса [**stomp**](http://onreader.mdl.ru/RabbitMQInDepth/content/Ch09.html) (Simple Text Oriented Messaging Protocol), альтернатива AMQP для специфических случаев
-   **61614** - порт сервиса **stomps** (stomp protocol over TLS/SSL), аналогично **amqp** и **amqps**
-   **1883** — порт сервиса [**mqtt**](http://onreader.mdl.ru/RabbitMQInDepth/content/Ch09.html) (Message Queue Telemetry Transport) альтернатива AMQP для специфических случаев
-   **8883** — порт сервиса **mqtts** (mqtt protocol over TLS/SSL), аналогично **amqp** и **amqps**

![](media/7ff39f0c52513f7ba751f4c7c2065b6a.png)

-   **--zone=public** — публичный доступ к порту
-   **--permanent** — останется после перезагрузки машины

Перезагрузка фаервола, порты добавились:

**![](media/e29eb7358e3cf3195ea9edf93bc6420f.png)**

Включаем **nis** (Network Information Service - распространяет карты имен, паролей и другую важную информацию компьютерам своего домена) в **SELinux**. Совершенно без понятия зачем, видимо как-то связано с работой **Erlang**, если это не сделать, то порт для **RabbitMQ** не выделится.

![](media/a6459671b85f39d983dfdee7738a7909.png)

После чего пробуем различные фиксы из интернета (около 6), обращаемся к гадалке, от отчаяния пробуем оригинальный гайд по установке с официального сайта, и в конце концов понимаем, что ничего не работает. **RabbitMQ** мучает людей…

3.3. Итак вот оно — описание установки при помощи другого руководства. Первым делом меняем **hostname**, потому что **RabbitMQ** может не запустится, если hostname поменяется в будущем.

![](media/e7796f5d1e7443c6edf284349fb9b63c.png)

3.4. Далее как по [**старым гайдам**](#bookmark=id.gjdgxs) устанавливаем **wget** и **epel-release**.

![](media/f83411fc48a21fdb68930b81c80ecb77.png)

3.5. Потом как по [**старым гайдам**](#bookmark=id.30j0zll) устанавливаем **Erlang**, но на этот раз без **socat** и **logrotate** (видимо не так они и нужны), а также понимаем, что в прошлый раз скачали старую версию (**erlang-solutions-1.0-**1), из-за чего могли и возникнуть трудности.

![](media/9871a5313132a3785de6700bb62d4ed7.png)

3.6. На этом шаге уже существенные отличия, а именно нужно создать и заполнить файл **/etc/yum.repos.d/rabbitmq.repo** (конфигурация репозитория), возможно в старых гайдах это производилось автоматически. **Nano** на **Centos** не предустановлено так что пользуемся **vi**.

![](media/5fb2915f098dd031cd5fe389ed5c5285.png)

Обновляем кэш репозиториев yum и уже можно наблюдать как все красиво:

![](media/3ac1cf44bbf77ed2e34ad20be1a42876.png)

3.7. Момент истины, загружаем **rabbitmq-server** и запускаем его. Загрузилось нормально ухх, включалось долго (чуть инфаркт не схватил), и….. (драматическая пауза) да Высшие Силы сжалились надо мной и все запустилось, ни фаерволл ни **SELinux** настраивать (пока) не пришлось.

![](media/5cfd043db3ca58ab9dacaa2d69bd7669.png)

Если что вот так выглядело описание установки из книги (если вы сюда телепортировались, то сначала телепортируйтесь [**сюда**](#bookmark=id.2et92p0)):

![](media/357555aa93e110820acbdef55062de8d.png)

3.8. Постоянная рубрика «Удобные команды» на этот раз **RabbitMQ** (правда они вряд ли понадобятся, напрямую с **RabbitMQ** работает только **Openstack**, а нет понадобятся для настройки пользователя):

-   **rabbitmqctl delete_user user** — удалить пользователя
-   **rabbitmqctl change_password user strongpassword** — сменить пароль пользователя
-   **rabbitmqctl add_vhost /my_vhost** — добавить [**virtualhost**](http://virtualhost/)
-   **rabbitmqctl list_vhosts** — список virtualhosts
-   **rabbitmqctl delete_vhost /my_vhost** — удалить virtualhost
-   **rabbitmqctl set_permissions -p /my_vhost user ".\*" ".\*" ".\*"** - дать разрешение пользователю внутри virtualhost
-   **rabbitmqctl list_permissions -p /my_vhost** — список разрешений внутри virtualhost
-   **rabbitmqctl list_user_permissions user** — список всех разрешений пользователя
-   **rabbitmqctl clear_permissions -p /my_vhost user** — удалить разрешение пользователя внутри virtualhost
-   **rabbitmqctl status** — статус сервиса rabbitmq-server
-   **rabbitmqctl create_uers user strongpassword** — создать пользователя
-   **rabbitmqctl list_users** — список пользователей
-   **rabbitmqctl set_user_tags user tag** — присвоит тэг пользователю

3.9. Оказывается я пропустил некоторые настройки из начала книги, а именно обновление всех установленных пакетов:

![](media/7d72709c64516a0a2d0602718e84c4d0.png)

Отключение network manager и редактирование файла **ifcfg-eth0** (поправить параметры **NM_CONTROLLED** и **ONBOOT**, а также проверить настройку ip-адреса и тд) в папке **/etc/sysconfig/network-scripts/**, так как **NetworkManager** будет мешать работе **Openstack** **Neutron**, судя по [**документации RedHat**](https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/10/html/manual_installation_procedures/chap-prerequisites):

![](media/1da4ef047d8f930f44a111045ac2e600.png)

![](media/c8f77a33fb6baa0b3676a163245160e9.png)

![](media/8d43eddddc6fbc141540c225e5fd4a2b.png)

Отключение фаервола, а я то мучался ([**firewalld не используется даже когда хочется**](https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/10/html/manual_installation_procedures/chap-prerequisites), вместо него используется **iptables**, будем их настраивать во время работы с сервисом **Neutron**):

![](media/a410ac018d14666b0daf56b66409ca5e.png)

Установка пакета **openstack-selinux** (**sudo** **yum -y install openstack-selinux**) или отключение **SELinux** (я решил отключить (изменить поле **SELINUX** в файле **/etc/sysconfig/selinux**, потом перезагрузиться), так как не особо разбираюсь в данной технологии, но может быть изучу, года наконец разверну кластер):

![](media/2c7c680bf043c145b6c65cccd0c8ae90.png)

Проверка после перезагрузки:

![](media/edfae534889e228fc8e152b4f6ecc61f.png)

Установка **chrony**, как на хосте, у меня он уже был предустановлен и настроен (вроде бы устанавливал только на хост машину):

![](media/9c5ecb5642182a2fa55b66cd6e61161a.png)

Установка [**crudini**](http://www.pixelbeat.org/programs/crudini/) для редактирования конфигурационных файлов:

![](media/d07b249f097fd907bc42a2ff832a4f49.png)

Пример команды для редактирования конфига (не буду же я одну команду выносить в рубрику):

![](media/28728aa1785e49e50c4cc29b5f2186a5.png)

НО это все равно не меняет того, что способ установки **RabbitMQ** представленный в книге неполный, хотя и была показана установка репозитория **epel-release** (выполните две команды ниже и [**назад**](#bookmark=id.3znysh7)).

![](media/9fad61af5a67b5d8ba8ac96d88cf713f.png)

3.10. Ну теперь можно заняться настройкой **RabbitMQ**. Для начала настроим аутентификацию, в руководствах говорится что есть два способа аутентификации: с использованием имени и пароля **SASL-аутентификации** (Simple Authentication and Security Layer), обеспечиваемой фреймворком **Erlang**, и при помощи сертификатов и SSL. Я решил (посмотрел в руководствах), что буду реализовывать первый способ + все сервисы будут работать под одним пользователем **RabbitMQ** (да я знаю, что так не очень безопасно, но я и фаерволл уже вырубил). Что касается **SASL-аутентификации** в **RabbitMQ**, то есть два метода: [**PLAIN**](https://www.rabbitmq.com/access-control.html#:~:text=Description-,PLAIN,-SASL%20PLAIN%20authentication) и [**AMQPLAIN**](https://www.rabbitmq.com/access-control.html#:~:text=most%20other%20clients.-,AMQPLAIN,-Non-standard%20version) и дефолтный пользователь **guest**.

Создаем пользователя **openstack** (пароль - **openstack**) для настройки сервисов **Openstack** и добавляем ему права на настройку, чтение и запись:

![](media/650cd311301c7a1af2a466a7d189d67b.png)

Для дальнейшего удобства активируем графическую консоль **RabbitMQ**, она будет работать на порту **15672**:

![](media/35a2df32bf69cd6abaad6ca959260b47.png)

Для того чтобы получить доступ к в браузере обращаемся к **192.168.122.200:15672** (к моему большому удивлению проброс портов осуществлять не надо было). Для регистрации необходимо присвоить один из тэгов (**management**, **policymaker**, **monitoring**, **administrator**) созданному пользователю openstack, так как за пользователя guest зайти будет нельзя, из-за того что он может коннектиться [**только через loopback интерфейс**](https://www.rabbitmq.com/access-control.html#:~:text=). Я пока не знаю какой тег больше подойдет для пользователя **openstack**, когда узнаю нужный подчеркну выше (в этот web-интерфейс я больше ни разу не зашел). А пока по-быстрому создал пользователя **test** для проверки работоспособности web-интерфейса.

![](media/eecc2aad1051624bb8334c87adb1bd82.png)

Все нормально работает, главное выставить тэг:

![](media/833fc42699084e9781f91636cd8a9129.png)

4\. Установка и настройка БД MariaDB и PyMySQL.

4.1. Итак мы наконец закончили с конфигурацией **RabbitMQ**, теперь можно приступить к конфигурирования БД для использования сервисами **Openstack**. Качаем **MariaDB** и клиентскую библиотеку **PyMySQL** с помощью одной команды!

![](media/9852877a365c2d58c8f583d52c48da31.png)

4.2. Создаем файл **/etc/my.cnf.d/openstack.cnf** с конфигом **mysqld** и запускаем **MariaDB**:

![](media/17399c04459b2c0095db24851cff02af.png)

-   [**bind-address**](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_bind_address:~:text=%D0%B2%20MySQL%C2%BB%20.-,bind_address,-%D0%A4%D0%BE%D1%80%D0%BC%D0%B0%D1%82%20%D0%BA%D0%BE%D0%BC%D0%B0%D0%BD%D0%B4%D0%BD%D0%BE%D0%B9%20%D1%81%D1%82%D1%80%D0%BE%D0%BA%D0%B8) = **192.168.122.200** — ip-адрес контроллера БД
-   [**default-storage-engine**](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_bind_address:~:text=%D0%A3%D0%BF%D1%80%D0%B0%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5%20%D0%BF%D0%B0%D1%80%D0%BE%D0%BB%D1%8F%D0%BC%D0%B8%C2%BB%20.-,default_storage_engine,-%D0%A4%D0%BE%D1%80%D0%BC%D0%B0%D1%82%20%D0%BA%D0%BE%D0%BC%D0%B0%D0%BD%D0%B4%D0%BD%D0%BE%D0%B9%20%D1%81%D1%82%D1%80%D0%BE%D0%BA%D0%B8) = **innodb** - механизм хранения для таблиц по умолчанию, в данном случае [**innodb**](https://animatika.ru/info/gloss/innodb.html) (данные хранятся в больших совместно используемых файлах).
-   [**innodb_file_per_table**](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_file_per_table:~:text=WITH_DEBUG%20CMake%20option.-,innodb_file_per_table,-Command-Line%20Format) = **on** — для каждой таблицы будет создаваться новый файл
-   [**max_connections**](http://max_connections/) = **4096** — максимальное количество подключений
-   [**collation-server**](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_collation_server:~:text=the%20default%20database.-,collation_server,-Command-Line%20Format) = **utf8_general_ci** — тип сервера сравнения и сортировки строк, [**utf8_general_ci**](https://webcache.googleusercontent.com/search?q=cache:5FVk_T2r3uoJ:https://dev.mysql.com/doc/refman/8.0/en/charset-unicode-sets.html&cd=2&hl=ru&ct=clnk&gl=ru#:~:text=%C3%9C%20%3D%20Y%20%3C%20%C3%96-,_general_ci%20Versus%20_unicode_ci%20Collations,-For%20any%20Unicode) — дефолт для **utf8**.
-   [**character-set-server**](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_collation_server:~:text=and%20error%20messages.-,character_set_server,-Command-Line%20Format) = **utf8** — тип набора символов на сервере

![](media/a2c9f1bcf21998c1f54dcc505d41b8ee.png)

4.3. Запускаем супер-пупер мега скрипт [**mysql_secure_installation**](https://mariadb.com/kb/en/mysql_secure_installation/), который поможет безопасно установить **MariaDB** (что в принципе видно из названия), во время скрипта можно:

-   Установить пароль для учетных записей **root**.
-   Удалить корневые учетные записи, доступные из-за пределов локального хоста.
-   Удалить учетные записи анонимных пользователей.
-   Удалить тестовую базу данных, к которой по умолчанию могут обращаться анонимные пользователи.

Я везде отвечал **yes** (возможно это была критическая ошибка (нет)).

![](media/7a8c591f4e60e0e5f7961a77ae064e8f.png)

Проверить работу сервиса можно так:

![](media/8d097168f0bcd28e37bb5553b2cb6e24.png)

5\. Установка и настройка сервиса идентификации Keystone

5.1. Теперь можно и потихоньку начать настраивать сервисы **Openstack**, все начинается в **Keystone**. Что такое **Keystone** — это сервис идентификации **Openstack**, представляет собой централизованный каталог пользователей и сервисов, к которым они имеют доступ. Если хотите еще одно синонимичное описание то вот: **Keystone** - единая система аутентификации и авторизации облачной операционной системы.Сервис поддерживает следующие типы аутентификации:

-   Аутентификация по токенам
-   Аутентификация при помощи пары имя пользователя/пароль
-   **AWS**-совместимая аутентификация (Amazon Web Services)

Также **Keystone** хранит в себе список всех доступных сервисов в **Openstack** и реквизитов для обращения к их API (без привязки к пользователю), из чего следует, что и поднимать его нужно первым, ведь потом каждый последующий сервис надо в нем регистрировать.

Что такое **Сервис**, **Пользователь** и **Токен** и так понятно, а вот **Проект** что-то новое — это объединение **Ресурсов** (виртуальные машины, образы и т. д.), придуман чтобы контролировать права пользователей, чтобы они могли видеть и работать с ВМ только внутри одного проекта (изначально думал, что это типо **Namespace**, но есть еще **Домен**). Также в **Openstack** как и в **Kubernetes** **Пользователи** напрямую не имеют доступ к **Ресурсам** **Проектов**, а получают его через так называемые **Роли** (понятно для чего так придумано, чтоб можно было создавать кастомные права доступа и быстро их назначать пользователям). Думали все, а нет, еще есть **Домены** — объединения **Проектов**, уже полные аналоги **Namespace**. Ладно по теории вроде бы все теперь качаем.

5.2. Качаем пакеты **openstack-keystone** (сам **Keystone**) [**python-openstack-client**](https://github.com/openstack/python-openstackclient) (удобный CLI для работы с **Openstack**) [**httpd**](https://httpd.apache.org/) (**http** сервер **Apache**) [**mod_wsgi**](https://modwsgi.readthedocs.io/en/master/) (модуль **http** сервера для связи сервера и программой на **python**) с помощью **yum**. **Проблема** подкралась незаметно — пакеты **openstack-keystone** и pythonopenstackclient **не скачались**, но хоть фиксится просто во первых в файле **/etc/yum.repos.d/epel.repo** дизейблим все репозитории (**enabled=0**), а во-вторых я дефис забыл в названии пакета **python-openstackclient**.

![](media/760e27112b909aa8a6773281871ae031.png)

![](media/c4677cd02802e10910796f7fffd1eb8c.png)

5.3. Как уже говорилось, для каждого сервиса **Openstack** необходима отдельная БД, которую надо создавать руками, если происходит установка каждого сервера отдельно, так что создаем **keystone** БД, также надо выдать привилегии, чтобы сервис мог нормально работать с БД.

![](media/2389b43ee6b7ae1be7e0b6206f332db3.png)

5.4. В конфиг **Keystone** (**/etc/keystone/keystone.conf**) пишем путь к БД, можно с помощью **crudini**, можно с **vi** и тд.

![](media/fe56f19bc0420fcb4b261fac2ae797ff.png)

5.5. Задаем формат и провайдер токенов генерируемых сервисом **Keystone**. Формат токенов на выбор:

-   **UUID** - строка из 32 символов, которую удобно использовать в вызовах **OpenStack** API, например применяя команду **curl**. Круто - небольшой размер, не круто – токен не содержит информации, достаточной для того, чтобы произвести локальную авторизацию (сервисы **OpenStack** каждый раз должны отправлять токен сервису **Keystone**, для того чтобы получить информацию, какие операции разрешены с этим токеном).
-   **PKI** - содержат всю необходимую для локальной авторизации информацию и, кроме того, содержат в себе цифровую подпись и информацию об устаревании, а значит сервисы **OpenStack** могут локально кэшировать эти токены. Круто — нет теперь дудоса токенами, не круто — токены большие (могут быть больше 8 Кб), и некоторые сервисы не поддерживают HTTP-заголовки такого размера, и еще как с **curl** работать, когда у тебя заголовок на 8 Кб.
-   **PKIz** — пытались пофиксить проблемы **PKI**, но ничего не вышло, и они ушли в небытие.
-   **Fernet** - небольшого размера (до 255 символов), но содержат достаточно информации для локальной авторизации. Их не требуется синхронизировать между регионами, для них не нужна база данных (токены без сохранения состояния), и процесс генерации их быстрее, чем в первых двух реализациях. Дополнительным плюсом будет отсутствие необходимости настройки memcached (можно не кешировать, но по итогу все равно надо, если хочешь быть крутым). **Их и используем.**

Выставляем провайдера токенов в файле **/etc/keystone/keystone.conf** сами знаете как.

![](media/74469056d56d89e2861c38fb61bfcabd.png)

5.6. Инициализируем базу данных (скрипт **keystone-manage db_sync**) репозитории ключей **Fernet** (2 команды - **«keystone-manage** **fernet_setup»** и **«keystone-manage credential_setup»**).

![](media/d0704bdc975411da49b86bc8b5fe9e66.png)

5.7. Конфигурируем доменное имя сервера **Apache** **httpd** - сервис идентификации для сетевого взаимодействия, путем редактирования файла **/etc/httpd/conf/httpd.conf** (без использования **cruduini**).

![](media/1688d98474cd0238338ee218f2dc6654.png)

Также создаем и редактируем файл **/etc/httpd/conf.d/wsgi-keystone.conf** (это конфигурация модуля **http-сервера** для связи сервера и программой на **python**).

Ну чтож вот [**оп**](https://www.opennet.ru/docs/RUS/apache_dir/#Location)**исание полей**:

-   **Listen** — на каком порту работает
-   **VirtualHost** — имя **virtualhost** (любое на таком-то порту)
-   **WSGIDaemonProcess** — создать отдельные процессы демона, которым будет делегирован запуск приложений WSGI.
-   **WSGIProcessGroup** — в какой группе процессов будет выполняться приложение WSGI.
-   **WSGIScriptAlias** — помечает каталог как содержащий сценарии WSGI
-   **WSGIApplicationGroup** — какой группе приложений принадлежит приложение WSGI.
-   **WSGIPassAuthorization** - передаются ли заголовки авторизации HTTP в приложение WSGI.
-   **LimitRequestBody** - максимальное количество байт, которое может быть в запросе.
-   **IfVersion** — если версия **Apache** больше/меньше
-   **ErrorLogFormat** — формат лога ошибки
-   **ErrorLog** — файл сохранения логов
-   **CustomLog** — файл сохранения логов доп хоста
-   **Directory** — местонахождение проекта
-   **Require** **all** **granted** — нет IP-адресов, заблокированных для доступа к сервису (новая версия)
-   **Order** **allow,deny** — [**настройка**](https://habr.com/ru/post/81858/) системы контроля доступа
-   **Allow from all** — нет IP-адресов, заблокированных для доступа к сервису (старая версия)
-   **Alias** — сопоставление URL-адреса с путем файловой системы
-   **Location** — внутри настройки управления доступа к URL
-   **SetHandler** — просмотр определенного хендлера
-   **Options** — разрешенные особенности сервера

![](media/958e7c7c204411984e40bb8d24be994d.png)

Наконец-то запускаем веб-сервер:

![](media/b0e282e6c49c292836e14ac67af7fe91.png)

5.8. Теперь инициализируем **Keystone**. Опять же есть два варианта инициализации:

-   Использовать команду **«keystone-manage** **bootstrap»**, которая выполнит инициализацию за нас (рекомендовано разработчиками, после того как попробовал второй способ использовал этот):

Данная команда создаст за нас записи о сервисе Keystone, точки входа (**admin**, **internal** и **public**) в сервис Keystone и регион в котором расположен сервис Keystone, а также **default** домен, проект **admin**, пользователь **admin** и роль **admin** и соединить их:

![](media/43da8a4a014a52dbcd379e2e1b6ae50c.png)

Измененный скрипт (создаем сами, **PS1** — для красивого названия в консоли):

![](media/9bba12c5f18b07bb53d812d21694bd81.png)

Созданные ресурсы:

**![](media/777725fde90525fad9c1c7af3591a516.png)**

![](media/2794b1ce0ae7fdb766b846921d92d351.png)

-   Воспользоваться авторизационным токеном (общий секрет между **Keystone** и другими сервисами, а также вход в админку без пароля), долго, но зато идеально для понимания процессов (мой выбор, который был изменен на первый способ, так как этот способ был [**вырезан**](https://docs.openstack.org/keystone/rocky/admin/identity-auth-token-middleware.html#:~:text=The%20admin_token%20option%20is%20deprecated%20and%20no%20longer%20used%20for%20configuring%20auth_token%20middleware.)):

Для начала надо сгенерировать токен с помощью **OpenSSL** и поместить его в файл конфига **Keystone**:

![](media/b2027d70128ba75aeec10ea282b7f294.png)

Немножко займемся оптимизацией и напишем скрипт для задания значений переменным окружения (**OS_TOCKEN** — наш токен, **OS_URL** — URL **Keystone**, **OS_IDENTITY_API_VERSION** — версия API **Keystone**)

![](media/c850adaec0890c8e8abebc4e2d4618d1.png)

После попытки отправки запроса на сервер, было получено много различных ошибок.

5.9. После того как помучились с инициализацией создадим непривилегированного пользователя и все что для него надо (admin пользователь был создан автоматически).

Создаем проект **demo**:

![](media/b3b893f632a4b13dabadd5e08bb19f89.png)

Создаем пользователя **demo** с собственным **email**:

![](media/5d7434348d3a81e4f67f508ba5dddc90.png)

Создаем роль **user**:

![](media/374d6f57a8a39b3954807a3b0fe78bd1.png)

Добавляем роль **user** в проекте **demo** пользователю **demo**:

![](media/8d40be6037a4517b1af04b91f760c8db.png)

Скрипт для входа за пользователя **demo** (**keystonerc_usr**):

![](media/64ae20b62075362f83c2fdf2578f05fa.png)

5.10. Создаем проект **service** для всех сервисов **Openstack**:

![](media/1281f9d8592d2fcb81877c9072954b8d.png)

С **Keystone** пока закончили, теперь можно перечислить полезные команды **openstack cli** (сюда будут добавляться все команды использованные в процессе конфигурирования кластера):

-   **openstack user list** — список пользователей
-   **openstack role list** — список ролей
-   **openstack role assignment list** — список назначения ролей
-   **openstack domain list** — список доменов
-   **openstack volume list** — список томов
-   **openstack stack list** — список стеков
-   **openstack stack event list имя_стека** — список событий при создании стека
-   **openstack stack resource list имя_стека** — список ресурсов стека
-   **openstack volume backup list** — список бэкапов томов
-   **openstack server add volume имя_ВМ имя_тома** — подключение тома к ВМ
-   **openstack server remove volume имя_ВМ имя_тома** — отключение тома от ВМ
-   **openstack service list** — список сервисов
-   **openstack server list** — список ВМ
-   **openstack metric resource-type list** — список типов ресурсов метрик
-   **openstack metric resource list** — список ресурсов метрик
-   **openstack metric resource show имя** — список метрик связанных с ресурсом
-   **openstack metric measures show --resource-id id_ресурса имя_метрики** — список значений определенной метрики ресурса
-   **openstack alarm list** — список триггеров
-   **openstack orchestration service list** — список сервисов оркестрации
-   **openstack backup list** — список резервных копий
-   **openstack volume type list** — список типов томов (зашифрованный или нет)
-   **openstack security group list** — список групп безопасности
-   **openstack aggregate list** — список агрегаторов узлов
-   **openstack availability zone list** — список зон доступности
-   **openstack security group rule list имя_группы —** список правил брандмауэра для группы безопасности
-   **openstack security group rule create --protocol протокол --dst-port номер_порта имя_группы** — добавление правила брандмауэра в группу безопасности
-   **openstack hypervisor list** — список гипервизоров
-   **openstack hypervisor stats show** — статистика по гипервизорам
-   **openstack usage list** — использование ресурсов объектами
-   **openstack project list** — список проектов
-   **openstack quota list** — список квот на ресурсы
-   **openstack catalog list** — список каталогов
-   **openstack network agent list** — список сетевых агентов (dhcp, open vswitch и тд)
-   **openstack compute service list** — список служб Nova
-   **openstack host list** — список узлов
-   **openstack floating ip list** — список плавающих ip
-   **openstack floating ip create имя_сети** — создание плавающего ip из сети
-   **openstack server add floating ip имя_ВМ ip_адрес** — присоединение плавающего Ip к ВМ
-   **openstack hypervisor show имя_узла** — информация о гипервизоре узла
-   **openstack endpoint list** — список конечных точек
-   **openstack image list** — список образов
-   **openstack image save имя_образа \> путь_куда_сохранять** — скачать образ на локальную машину
-   **openstack image create имя_ВМ имя_образа** — создать образ из ВМ в Openstack
-   **openstack volume snapshot create --volume имя_тома** **имя_снимка** — создать снимок тома
-   **openstack volume backup create имя_тома** — создать бэкап тома
-   **openstack volume backup restore id_бэкапа имя_тома —** откатить том к бэкапу
-   **openstack console url show имя_вм** — показать URL по которому можно подключится к ВМ через noVNC
-   **openstack тип show имя_объекта** — информация о конкретном объекте
-   **openstack тип delete имя_объекта** — удаление объекта
-   **openstack service create --name имя_сервиса —description "описание" тип_сервиса** — создание сервиса
-   **openstack endpoint create тип_точки тип_интерфейса url --region регион** — создание конечной точки
-   **openstack domain create --description "описание" имя_домена** — создание домена
-   **openstack project create --domain имя_домена --description "описание" имя_проекта** — создание проекта
-   **openstack user create --domain имя_домена --email почта_пользователя --password пароль_пользователя имя_пользователя** — создание пользователя
-   **openstack role create имя_роли** — создание роли
-   **openstack role add --project имя_проект --user имя_пользователя имя_роли** — создание назначения роли (привязка роли к пользователю)
-   **openstack тип set --имя_поля значение_поля имя_объекта** — изменение полей объекта
-   **openstack тип unset --имя_поля значение_поля имя_объекта** — удаление поля объекта
-   **openstack console log show имя_ВМ** — показать логи ВМ
-   **openstack --debug** — полезный флаг для просмотра вызовов к Openstack API
-   **openstack token issue** — запрос токена для аутентификации
-   **openstack keypair create имя_пары \> файл_сохранения_пары** — создание пары ssh ключей
-   **source keystonerc_adm** — запуск скриптов (в моем случае задача глобальный переменных для авторизации под ролью **admin**)
-   **env** — просмотр переменных окружения
-   **/etc/имя_сервиса/имя_сервиса(или)службы.conf** — файлы конфигов
-   **/var/log/имя_сервиса/имя_сервиса-имя_службы.log** — файлы логов (некоторые тут **/var/log/httpd/\***)

6\. Установка и настройка сервиса хранения образов Glance

Перед началом хочется отметить, что обращения к **Openstack** API начали проходить достаточно медленно, после перезагрузки машины, возможно я дал ВМ слишком мало системных ресурсов, а может так и должно быть, вобщем пугаться долгих ответов сервера не надо, ниче не умерло.

6.1. Что ж для чего нам нужен **Glance**? **Glance** - ведет каталог, регистрация рует и доставляет образы виртуальных машин (как известно образ представляет собой шаблон для ВМ, может быть просто ОС, а может и содержать предустановленные пакеты). НО **Glance** самостоятельно не хранит образы, а использует для этого систему хранения данных (**Swift**, **Ceph** или просто **локальн**о на узле), информация же о размере, формате, имени образа и т. д. хранится в БД. Поддерживаются следующие форматы образов, еще есть образы контейнеров, но это уже другая история:

-   **vhd** (Virtual Hard Disk) — виртуальный жесткий диск от Microsoft
-   **vmdk** (Virtual Machine Disk) — диск виртуальной машины от VMware
-   **vdi** (Virtual Disk Image) — образ виртуального диска от VirtualBox(Oracle)
-   **iso** (International Organization for Standardization) — образ оптического диска от International Organization for Standardization
-   **qcow2** (QEMU Copy On Write 2) — образ QEMU
-   **ami** — образ Amazon Machine
-   **ari** — образ Amazon Ramdisk
-   **aki** — образ Amazon Kernel
-   **vhdx** — улучшенная версия vhd
-   **raw** — диск неструктурированного формата RAW

Стоит отметить, что данный сервис состоит из двух служб:

-   **glance-api** – предоставляет доступ к REST API сервиса образов для поиска, хранения и получения образов;
-   **glance-registry** – хранит, обрабатывает и предоставляет информацию. Непосредственно пользователи не взаимодействуют с этим сервисом, только сервисы **Glance**.

Вот красивая картинка компонентов **Glance**:

![](media/ad031fdc9175645ce60b9e0c12c1232f.png)

Что происходит когда мы отправляем запрос на создание ВМ — **Nova** (еще не созданный сервис по управлению виртуальными машинами и сетью) отправляет GET-запрос по адресу **http://путь-к-сервису-glance/images/идентификатор-образа**. В случае если образ найден, то glance-api возвращает URL, ссылающийся на образ. **Nova** передает ссылку драйверу гипервизора, который напрямую скачивает образ и запускает ВМ.

6.2. Теперь начинается практика, первым делом добавляем все необходимые компоненты в наш **Keystone**:

Создаем пользователя **glance**:

![](media/23a6ecf13d5f6e2053d237cecd3ee106.png)

Присваиваем пользователю **glance** роль **admin** в проекте **serivce**:

![](media/0f5c5261f1100aae23f13062f2772d11.png)

Создаем сервис **glance**:

![](media/34c3c7d4e3f86051c30658dff848ebb8.png)

Создадим точки входа для сервиса **glance**:

![](media/d68c266720c21e9472a94f214a9ef8df.png)

6.3. Теперь устанавливаем сервис **Glance** с помощью **yum**:

![](media/98fabd4b673152f1391dda72c9934187.png)

6.4. Создаем БД **glance** и выдаем на нее права:

![](media/9dcacfb87cdbb7f0a3c3f0f26371444b.png)

6.5. Прописываем строку подключения к базе данных в файлах конфигов служб **glance-api** (**/etc/glance/glance-api.conf**) и **glance-registry** (**/etc/glance/glance-registry.conf**):

![](media/39250994a5eecb3dc00a1ebaee7764d9.png)

6.6. Завершаем настройку БД **glance** с помощью скрипта (**db_sync**):

![](media/f58e3ebfafbb417dcd013abd8d805368.png)

6.7. Все что касается БД **glance** настроено, теперь настроим сам службу **glance-api** через конфиг (**/etc/glance/glance-api.conf**):

Настраиваем тип сервис аутентификации:

![](media/4d525362e02cba3c51ab576dd906976c.png)

Настраиваем разрешенный ip отправителя запросов к сервису **Glance**:

![](media/4207264c7803b780aaa74e73be9e0fdb.png)

Настраиваем URL аутентификации:

![](media/12ef6d9a9770aed6cba35ff4451442e4.png)

Настраиваем тип авторизации:

![](media/43cbf187af2d79a407cd01e4fa9a5aab.png)

Настраиваем имя домена в котором находится проект **glance**:

![](media/535bc038dc7405a60b401fa4cee00837.png)

Настраиваем имя домена в котором находится пользователь **glance**:

![](media/0a73481e426af7e2ad9c4c0b5e19df42.png)

Настраиваем имя проекта в котором находится пользователь **glance**:

![](media/e6039d5b35521f4d0d41b606ace199f7.png)

Настраиваем имя пользователя:

![](media/e2c3febe2ffe520082c1ad9261043cff.png)

Настраиваем пароль пользователя **glance**:

![](media/6406d3ad84e0b1e4eb6043b07fa5a6b4.png)

Настраиваем тип хранения образов (локальная файловая система):

![](media/74fbe340c39fe94ff0594f031137adf8.png)

Настраиваем путь к папке, гдк будут хранится образы:

![](media/dc1f5367be0b50fe3551ecf28eb2bb5e.png)

Также стоит отметить, что можно добавить несколько хранилищ образов с приоритетом (больше цифра - больше приоритет сохранения образа) по использованию, но я так не делал:

![](media/efb132b485112f4e56536d5778a66e3b.png)

Поля файла **/etc/glance/glance-api.conf** после исполнения команд (легко найти через **vi**, потом /текст, который надо найти):

![](media/ffd63faee7a5070e18a91d12c9392a36.png)

6.8. Теперь настроим сам службу **glance-registry** через конфиг (**/etc/glance/glance-registry.conf**), те же самые названия полей, кроме секции **glance_store**:

![](media/ac5a6854e44d5574a6208b92bcc8e712.png)

![](media/824769a5434ab404e68bdac9347c91a2.png)

6.9. Настраиваем параметры подключения к сервису **RabbitMQ** для службы glance-api (**/etc/glance/glance-api.conf**):

Настраиваем пароль пользователя **RabbitMQ**:

![](media/087bd5ae46be724a086850b6861e6e05.png)

Настраиваем имя пользователя **RabbitMQ**:

![](media/c365893c9b978aa8baeb3f55c6759091.png)

Настраиваем имя хоста **RabbitMQ**:

![](media/8509da8033d1c4cd09ba01a591bf487b.png)

![](media/10e5a30d3327b10af9fb649310aec923.png)

6.10. Запускаем сконфигурированные службы **glance-api** и **glance-registry**:

![](media/c163599e225f187a31b6759864f24980.png)

Папку для хранения образов нужно создать, а то **glance-api** не запустится (словил прикол, что у **glance-api** не было разрешений на файл, в котором хранятся логи - **/var/log/glance/api.log**). Решение прикола с разрешениями:

![](media/410e78212e15ade82d52ee5447cffcff.png)

![](media/dc299ad183f0e2e5290f3d0279c59761.png)

6.11. Итак мы все настроили и запустили, теперь можно затестить

Для начала скачаем образ **CirrOS** – минималистская операционная система с помощью **wget**:

![](media/0039f9e4f0a60433d066b307b2b4881d.png)

Необходимо также установить **qemu-img**, а потом посмотреть информацию о скачанном образе:

![](media/301226e92314fdc5839ae00956500692.png)

Какую информацию qemu-img:

-   **file format** – формат диска
-   **virtual size** – размер диска виртуальной машины
-   **disk size** – действительный размер файла
-   **cluster_size** – размер блока (кластера) qcow
-   **format specific information** – специфичная для формата информация (**compat** — версия QEMU, l**azy refcounts** — если on, то отключен ввод вывод метаданных, **refcount bits** — размер refcount table (но я не уверен), **corrupt** — искаженные данные или нет)

Далее надо скачать какую-нибудь утилиту автоматизированного создания образов, я выбрал **Oz**, которая использует заранее подготовленные файлы ответов неинтерактивной установки операционной системы. Если во время установки Keystone, были ошибки как и у меня и поэтому пришлось отключить epel repo, то его (и только его) надо опять [**включить**](#bookmark=id.tyjcwt).

![](media/8d23966ed66620bc43413237fcd31e7c.png)

Проверяем, что сеть libvirtd (перед этим **libvirtd** надо врубить), используемая по умолчанию с помощью команды **virsh net-list**, если нет, то определяем ее и задаем автозапуск сети:

Врубаем **libvirtd**:

![](media/ec546b31b61502d311c1f3b21238300f.png)

У меня после запуска **libvirtd** дефолтная сеть задалась автоматически:

![](media/48942a0d38d205ee4b70d5e82b695a9f.png)

Если у вас команда **virsh net-list** вывела что-то другое, то выполните две команды **net-define** (инициализация сети) и **net-autostart** (автостарт сети):

![](media/90522adf5c1db177b4cffbe1ad9033af.png)

6.12. Теперь, после того как скачали **oz** и настроили дефолтную сеть, можно начать настраивать **oz**.

Сначала зададим в качестве типа образа формат qcow2 в файле конфигурации **oz** (**/etc/oz/ oz.cfg**):

![](media/216942468106f702e04800cc1fda1bf7.png)

Теперь выбираем TDL шаблон для создания образа, примеры TDL шаблонов находятся в папке **/usr/share/doc/oz-0.15.0/examples**. Я списал самый простой TDL шаблон, а именно тот в котором определяются только путь к дистрибутиву (**\<install\>**) и пароль пользователя root (**\<rootpw\>**):

![](media/0677261c58f7eafa2577abe73c76f9bb.png)

Полезные секции кроме **\<install\>** и **\<rootpw\>:**

-   **\<packages\>** - установка пакетов
-   **\<repositories\>** - добавление репозиториев
-   **\<files\>** - создание файлов
-   **\<commands\>** - выполнение команд

После создание TDL шаблона можно создать образ с помощью oz-install (**-t —** через сколько секунд инсталлятор должен прервать установку, **-d** — показывает уровень сообщений об ошибках (**2** — норм, **3** — подробно)). Создается очень долго:

![](media/026946b6aba6d3c5227cb4f634dc236e.png)

Образ должен появится в папке /**var/lib/libvirt/images**, но он весит 11GB, так что я его так и не скачал (диск на виртуалке 11 GB), дальше если хотите подготовить образ к использованию в **Openstack** надо с помощью **virt-sysprep** (надо скачать) убрать специфичную для конкретного экземпляра машины информацию, потом уменьшить размер образа (превратить в тонкий диск) при помощи **virt-sparsify**. Если надо отредачить уже готовый образ то можно скачать **guestfish**:

![](media/9221d2188d9dc596dcc644e44313aef4.png)

6.13. Наконец-то после того как мы поработали с образами сами, можно поработать с образами с помощью **Openstack**. Для начала надо добавить в оба рабочих файла **keystonerc_adm** и **keystonerc_usr** переменную среды, определяющую версию **Glance** API, с которой мы будем работать (**OS_IMAGE_API_VERSION**=2):

![](media/3865f6858f55563cd0938ffbfc8d087b.png)

![](media/4561cac9f0c52b07eaf88f8fe412bdd9.png)

Загружаем наш скачанный образ **Cirros** в **Glance**:

![](media/bac46a0d3906acb8f9060f0bb435fbce.png)

Используемые параметры:

**--file** — путь до файла с образом

**--disk-format** — формат образа диска

**--container-format** — формат контейнера (bare — не контейнер)

**--public** — образ доступен всем

Для вас с прошлой строчки прошла одна секунда, а для меня - два дня, так как команда на скрине не выполнялась из-за того, что **Glance** не мог авторизоваться в **Keystone** (я так думал), в результате оказалось, что **Glance** не мог отправить запрос на URL <http://controller.test.local:35357/v3/>, так как это доменное имя не было прописано в **/etc/hosts** (но ведь запросы к **Keystone** шли нормально!!!), чтобы все работало надо каждое имя узла добавлять в **/etc/hosts** (зато я поработал с файлами логов **keystone** и **glance**, а также флагом **--debug**):

![](media/d02ac7c7f13b1ec049446bdeadf2472c.png)

После того как образ был загружен, он должен появится в этой папке **/var/lib/glance/images/**:

![](media/f1c199c78664ed5bd3bca6629b6e358f.png)

На этом все, но можно добавить, что образы популярных ОС можно найти тут:

-   Ubuntu: [**http://cloud-images.ubuntu.com/**](http://cloud-images.ubuntu.com/)
-   Fedora: [**https://cloud.fedoraproject.org/**](https://cloud.fedoraproject.org/)
-   Debian: [**http://cdimage.debian.org/cdimage/openstack/**](http://cdimage.debian.org/cdimage/openstack/)
-   CentOS: [**http://cloud.centos.org/centos/7/**](http://cloud.centos.org/centos/7/)

7\. Установка и настройка сервиса блочного хранилища Cinder

7.1. **Cinder** придуман для того чтобы хранить модифицированные в ходе работы тома, созданных образов, зачем? Ну, например, когда мы создали ВМ с помощью сервиса **Nova** (потом установим) мы с ней работаем (что-то устанавливаем, изменяем и тд), а что если узел на котором была создана ВМ выйдет из строя, и нам нужно ее срочно восстановить, **Glance** у нас хранит только базовые образы и никак не поможет, вот для из-за таких ситуаций и придуман **Cinder**, который записывает все изменения тома на узлах, где установлен **cinder-volume**. Также в **Cinder** можно не только хранить тома, но и создавать их снимки в режиме «только для чтения», с помощью которых создавать новые тома, доступные на запись.

Архитектура Cinder:

**![](media/9abe435ce0baf762444ba59f5c56d505.png)**

Службы Cinder:

-   **cinder-api** – точка входа для запросов в сервис по протоколу HTTP. Приняв запрос, сервис проверяет полномочия на выполнение запроса и переправляет запрос брокеру сообщений для доставки другим службам
-   **cinder-scheduler** – сервис-планировщик принимает запросы от брокера сообщений и определяет, какой узел с сервисом **openstack-cinder-volume** должен обработать запрос
-   **cinder-volume** – сервис отвечает за взаимодействие с бэкэндом – блочным устройством. Получает запросы от планировщика и транслирует непосредственно в хранилище. **Cinder** позволяет одновременно использовать несколько бэкендов. При этом для каждого из них запускается свой openstack-**cinder-volume**. И при помощи параметров **CapacityFilter** и **CapacityWeigher** можно управлять тем, какой бэкенд выберет планировщик
-   **cinder-backup** – сервис отвечает за создание резервных копий томов в объектное хранилище

7.2. К этому пункту свободное место на control узле равнялось 1GB, так что надо увеличивать размер:

Первым делом вырубаем **VM2** и увеличиваем размер диска **qcow2** на 3GB:

![](media/c7b81ac360d0b92f11d15c6cadd09ed4.png)

Потом запускаем **VM2**:

![](media/11504910c741676a85e07756baf14537.png)

Теперь с помощью **lsblk** проверяем, что появился диск **/dev/vda2**, а с помощью **pvs**, что на нем стоит система:

![](media/14bb56eb22ed3d386d1f9582b5356c38.png)

Скорее всего увеличить размер партиции можно и другим способом, но я побоялся экспериментировать и скачал утилиту **growpart**:

![](media/bb2454feb83670619dacbfab2f601b8c.png)

После этого увеличиваем размер раздела **/dev/vda2**:

![](media/ae1841501bbc013cb9b7c7f645e45925.png)

И наконец увеличиваем раздел **/dev/mapper/centos-root**:

![](media/7fe976461256d2c87856e8c8aa95dec8.png)

7.3. Теперь можно начать настройку окружения для сервиса **Cinder**, для этого надо добавить к виртуальной машине еще один диск в качестве блочного устройства, созданного с помощью [**Linux LVM**](https://habr.com/ru/post/67283/) (использует протокол [**iSCSI**](https://community.fs.com/ru/blog/iscsi-storage-basics-plan-iscsi-san.html)), на который будут писаться все изменения томов ВМ параллельно их записи на локальную систему рабочего узла.

Создадим новый диск и прикрепим его к запущенной **VM2**:

![](media/8c6096864b094e221e916a8dae762647.png)

В **VM2** проверяем присоединенный диск:

**![](media/c18ba71803eb76f47a08bebd3ad91104.png)**

Теперь создаем **LVM-группу** на присоединенном диске:

![](media/e8c294b9ebcaa5628bd9aad323786848.png)

![](media/c2b5eddc4efee1b2eaa95f3430e28962.png)

7.3. Теперь качаем пакет openstack-cinder и редачим его конфигурационный файл **/etc/cinder/cinder.conf**.

![](media/82686aba9f97c0941da2c569aaddf3d7.png)

Настраиваем подключение к БД:

![](media/e7a1ec667636fab9483c68580bb0e606.png)

Настраиваем URL, имя и пароль **RabbitMQ**:

![](media/8ef905b9b215b895ace20c4581cd0fab.png)

Как раньше настраиваем реквизиты пользователя и проекта в **Keystone**:

![](media/211604b746e37e1a1b42aca0a1f2f2eb.png)

Настройка имени **lvm-группы**:

![](media/d1105d4e4a23180dc02f91b02ad815e4.png)

Настраиваем бэкенд хранения **Cinder**, а также драйвер (отвечает за хранение данных при помощи локального менеджера логических томов и протокола транспорта iSCSI) для него:

![](media/328ad65730562b65f7459e889f841dc3.png)

Настраиваем используемый протокол транспорта и возможность использовать команду cinder-rtstool для управления томами (**lioadm** — поддержка [**LIO iSCSI**](https://habr.com/ru/post/200466/#:~:text=%D0%92%D1%81%D1%82%D1%80%D0%B5%D1%87%D0%B0%D0%B9%D1%82%D0%B5%2C%20%D0%B2%D0%BE%D1%82%20%D0%BE%D0%BD%D0%B8.-,LIO,-linux-iscsi.org)):

![](media/eb3c5b1fdb658042a2578726b339e434.png)

Настройка URL **Glance** API:

![](media/0a593989b8aaa271f40f6931530ca753.png)

Настройка пути к папке, где будут хранится [**lock-файлы**](https://docs.openstack.org/oslo.concurrency/ussuri/admin/index.html) (файлы, создаваемые сервисами для межпроцессорной блокировки):

![](media/47076d9df834ebf174a9fd6ee0ce7f39.png)

7.4. Теперь аналогично прошлым настройка создаем БД **cinder**:

![](media/d8154547a0e77d4084ba8c1ff1a2e779.png)

Во время выполнения скрипта синхронизации вылезают сообщения об устаревших полях, полностью их игнорируем, БД и так поднимется:

![](media/e6e2fa2bbfdbb50c684655491788c2ed.png)

7.5. Теперь регистрируем сервис **Cinder** в **Keystone**:

Создаем пользователя **cinder** и предоставляем ему роль:

![](media/17c0b61327345184a1b667f2d4df09ce.png)

Создаем сервисы **cinderv2** и **cinderv3**. Раньше надо было иметь две версии **cinder** для полной совместимости с остальными сервисам, но вроде как это уже [**не необходимо**](https://docs.openstack.org/cinder/latest/install/cinder-controller-install-rdo.html#:~:text=Beginning%20with%20the%20Xena%20release) и можно ограничится только сервисом **cinderv3** (я боюсь и разверну 2 сервиса):

![](media/a33d58294429f31a6d4eb6487474a61a.png)

Соответственно раз сервиса два, то и точек входа в два раза больше:

![](media/55d6c89b9d10de3ac31caeadcf59d12f.png)

![](media/d1e844c11cc25284e1c9fdcae95c1b0c.png)

7.6. Теперь устанавливаем и настраиваем сервис **iSCSI**, по идее следующие действия надо производить на других узлах для большей доступности и отказоустойчивости, поэтому некоторые команды могут повторяться.

Устанавливаем [**targetcli**](https://www.saqwel.ru/articles/linux/nastrojka-linux-iscsi-posredstvom-targetcli/) (оболочка для управления **LIO iSCSI**):

![](media/5bdee26c10e43e45b280580a4ba4801b.png)

Задаем ip машины на которой будет находится **cinder-volume**:

![](media/b7cd1795bb2a49e0b98417a9e3e9bb41.png)

Задаем бэкенд хранения **Cinder**:

![](media/c8079ccdfcd446ced65b33310bcc8153.png)

По итогу всех махинаций наш конфиг (**/etc/cinder/cinder.conf**) выглядит так:

![](media/5aa9ca46e4da81b06929d4c323df4545.png)

7.7. Теперь запускаем все что можно:

![](media/93825464c23dc479123a473d37c8c71d.png)

![](media/d89d66d10ab0c74f45913a53a1207c8c.png)

![](media/c9a670e1f41531d4c6505b429bab29f5.png)

![](media/6094964a822bf7c7ce4d9a88a2482275.png)

![](media/ebbce541571da8726c3bf25ca573a55f.png)

Также все эти службы можно было запустить одной командой с помощью пакета [**openstack-utils**](https://github.com/redhat-openstack/openstack-utils), который надо скачать, так как с помощью него также можно удобно селдить за статусами всех сервисов кластера:

![](media/73a9e01251a6a9f13de81beaef10caea.png)

Запуск всех служб **Cinder** одной командой:

![](media/e874a9cfb0b57498615fd35b6b0b9055.png)

Просмотр статуса всех сервисов одной командой (если что **keystone** и должен быть в **inactive**, мы ведь его напрямую не поднимали, а через **RabbitMQ**):

![](media/da9993f2bf275d61b5a5e251bfc5369b.png)

Также можно посмотреть состояние служб **Cinder** (**cinder-backup** по идее должен был быть остановлен, так как ему не задана конечная точка бэкенда хранения этих самых бэкапов (то самый сервис **Swift** из будущего), он почему-то работает, но в логах пишет, что могут быть проблемы):

![](media/7911137d48292d07040d9624ac126ca0.png)

Содержимое файла логов **/var/log/cinder/backup.log** **cinder-backup**:

![](media/99d1b90e48348add698b9f25575dc727.png)

На этом настройка закончена, теперь тестим функционал. Для этого попробуем создать том (**volume**) **testvol1**:

![](media/d5d3980003e9e98add1860c74d1a4477.png)

Пока с помощью **openstack cli** посмотреть на созданный том (**openstack volume list**) нельзя, так как мы еще не развернули сервис nova, но можно посмотреть с помощью **cinder cli**:

![](media/90d07bb5b92894707ea936693c4c0a21.png)

Зря я сделал диск размером всего в 2GB:

![](media/71cf997cc46b32402058dd78d95d864d.png)

7.8. По идее на этом можно закнчивать настройку сервиса **Cinder**, но надо кое-что уточнить, а именно, что в реальной жизни бэкендом **cinder-volume** и cinder-backup должен быть сервис **Swift** или **Ceph**, которые нужно разворачивать на других узлах. Мой ноут с 8GB оперативки и в перспективе двумя рабочими узлами такое точно не потянет, но все-таки было принято решение сделать бэкендом **cinder-backup NFS-сервер** расположенный на одном из рабочих узлов в качестве эксперимента.

Итак для начала нужно этот рабочий узел поднять (когда я в первом пункте сказал, что поднял 3 ВМ, я соврал):

![](media/bc4d8dc52417c80fa71e1e795ecb347a.png)

![](media/458130efe042ee0ecf2b93f6809fb3b3.png)

Теперь наконец-то понятная настройка ВМ для работы с **Openstack** (если что файл **/etc/hosts** должен выглядеть да всех ВМ одинаково, так что редачим его на каждой ВМ):

![](media/234112b1ef3b3d0b89164c421966dc12.png)

![](media/7be906780512eb28f0df56167cb14cf4.png)

Теперь настраиваем все что касается [**nfs сервера**](https://winitpro.ru/index.php/2020/06/05/ustanovka-nastrojka-nfs-servera-i-klienta-linux/):

Скачиваем и запускаем **nfs-server**:

![](media/cb4bae7bdcaf4a5cd1ce50ba08958e20.png)

Создаем папку в которая будет общая для всех **nfs-клиентов**;

![](media/f74db4ebc82716b81dfc8636de6a55dc.png)

Редактируем файл **/etc/exports**, указывая каким ip-адресам будет доступна папка, после чего применяем настройка и перезапускаем **nfs-сервер**:

![](media/b6394821a15ae3c9718e703d3a23a4bf.png)

-   **rw** – права на запись в каталоге (**ro** – доступ только на чтение)
-   **sync** – синхронный режим доступа (**async** – подразумевает, что не нужно ждать подтверждения записи на диск)
-   **no_root_squash** – разрешает root пользователю с клиента получать доступ к NFS каталогу (обычно не рекомендуется)
-   **no_all_squash** — включение авторизации пользователя (**all_squash** – доступ к ресурсам от анонимного пользователя)

Теперь попробуем настроить **NFS** бэкенд для **cinder-backup** на **controllet.test.local** (оказалось, что не обязательно настраивать **nfs-клиента** самостоятельно, монтировать папки и тд, достаточно просто добавить параметры в конфиг **Cinder** **/etc/cinder/cinder.conf**, и все cмонтируется автоматически после перезагрузки сервиса **Cinder**):

Настраиваем драйвер для **cinder-backup** и папку **nfs-сервера**, которую надо смонтировать:

![](media/747a44fa017ae1e52226192a73002cad.png)

![](media/fa2763a673f83b66ee713090ca5b19ec.png)

Все, поздравляю **Swift** можно не настраивать как и **Ceph**, но сами понимаете, что если мощность позволяют надо обязательно ставить или **Swift** или **Ceph**, желательно на трех отдельных узлах. Пока нет желания про них что-то расписывать, может только в самом конце, когда все будет настроено, я разверну **Swift** или **Ceph** на рабочих узлах.

8\. Установка и настройка сервиса управления виртуальными машинами и сетью Nova

8.1. Сервис **Nova** у нас отвечает за за управление запущенными экземплярами виртуальных машин, а также сетью (раньше да, теперь за сеть отвечает **Neutron**) как видно из названия раздела. Состоит из следующих служб (помимо **RabbitMQ** (брокера сообщений) и собственной БД):

-   **openstack-nova-api** – как и подобные службы других рассмотренных сервисов, отвечает за обработку пользовательских вызовов API.
-   **openstack-nova-scheduler** – сервис-планировщик. Получает из очереди запросы на запуск виртуальных машин и выбирает узел для их запуска. Выбор осуществляется, согласно весам узлов после применения фильтров (необходимый объем оперативной памяти, определенная зона доступности и т. д.). Вес рассчитывается каждый раз при запуске или миграции виртуальной машины.
-   **openstack-nova-conductor** – служба выступает в качестве посредника между базой данных и nova-compute, позволяя осуществлять горизонтальное масштабирование (нельзя развертывать на тех же узлах, что и nova-compute).
-   **openstack-nova-novncproxy** – выступает в роли [**VNCпрокси**](https://habr.com/ru/post/76237/) (такую опцию можно было выбрать во время запуска виртуалок при помощи **virsh**) и позволяет подключаться к консоли виртуальных машин при помощи браузера.
-   **openstack-nova-consoleauth** – отвечает за авторизацию для сервиса **openstack-nova-novncproxy** (вроде как больше не поддерживается).
-   **openstack-nova-placement-api** – сервис отвечает за отслеживание списка ресурсов и их использование (раньше был частью nova, потом отпочковался и стал назваться **openstack-placement-api**, а по-благородному **Placement**).
-   **openstack-nova-compute** – демон, управляющий виртуальными машинами через API гипервизора. Как правило, запускается на узлах, где располагается сам гипервизор.

Все сервисы кроме **nova-compute** располагаются на **controller** узле, а **nova-compute** на рабочих узлах **compute** и **compute-opt** (пока не создан).

8.2. Приступаем к установке сервиса **Nova**:

Как всегда базовые начальные шаги (и хорошо, что они у всех сервисов очень похожие):

Устанавливаем все службы **Nova**, кроме **openstack-nova-compute** (Placement раз уж он был когда-то частью **Nova** тоже скачаем и потом настроим, также возможно не надо качать **openstack-nova-console**, так как **openstack-nova-consoleauth** больше не поддерживается, но это не точно):

![](media/92952a9b70f3bff3d34bd9e0ac954fd9.png)

Создаем три бд (**nova** — содержит всю информацию о ВМ, которые удалось запланировать, **nova-api** — содержит информацию о местонахождении и временном (еще только создаются) местонахождении ВМ и **nova-cell0** — содержит ВМ, которые не удалось запланировать) и выдаем всем привилегии:

![](media/0fb7352acfc39d9c5e350d4b347d1334.png)

Регистрируем сервис **Nova** в **Keystone**:

![](media/f156b3c51dec45b08c62e52fc6906f1e.png)

![](media/43f13009a5827bd2170ea8f5a0a25028.png)

Теперь регистрируем сервис **Placement** в **Keystone**:

![](media/6aa4fe013e849b4ce533b16578ebafec.png)

![](media/8ed2bb8aa7e611bb06fe87b93da81fbe.png)

Далее конфигурируем файл **/etc/nova/nova.conf** (все параметры (я напишу какие не надо) ниже также надо будет добавить в аналогичные файлы на рабочих узлах):

Базовая конфигурация (**enabled_apis** — разрешенные API):

![](media/61e7dfb3201107e624e229e51de5b6ea.png)

Теперь на будущее включаем поддержку сервиса Neutron (**firewall_driver** — должно быть такое значение, если используется **Neutron** вместо **nova-network**):

![](media/99b36da30a3121a7468b77b262d31004.png)

Настраиваем URL для использования сервиса **Glance**:

![](media/4d29a7a8eb0ed2e907bf8e77c82a19b8.png)

Указываем имя региона для сервиса **Cinder**:

![](media/20e437c5d0c42401e49d29fb0f215a38.png)

Указываем путь к файлам блокировок:

![](media/ed2e6735cd0a7cd0761502f4e9f0b100.png)

Настраиваем параметры для подключения к сервису **Placement**:

![](media/149f5edd21ce2e9bb39da909d58790bb.png)

![](media/a751dcfc11d2151999d495ae70635ebc.png)

На этом общие параметры заканчиваются и начинаются специфичные для **controller.test.local**:

Настраиваем пути к созданным БД (почему только к 2 из 3 я без понятия):

![](media/053d29e502067ddb1adb3002957e3fc9.png)

Указываем ip **controller.test.local**:

![](media/a4dc1aa65601d0583f162b1dee5cf8f4.png)

Указываем ip адрес на котором будет работать **VNC-сервер**:

![](media/fde21837d8171490aed8c9545c6c128c.png)

![](media/211c5dbe3d10a709cda0bd4818edee05.png)

Теперь можно инициализировать базу данных **nova-api**:

![](media/13e47465e8186c39919f58d46f2964a7.png)

Также надо зарегистрировать базу данных **cell0** в БД **nova** и создать ячейку **cell1**:

![](media/2e0b6f2dc51e491839dd8a6e2433544d.png)

Теперь заполняем (очень долго) БД **nova** (я уже сошел с нити понимания конфигурации):

![](media/bcaf2705dde22657d4223642e6f0f770.png)

8.3. Перед тем как запустить все сервисы **Nova** cконфигурируем и запустим сервис **Placement** (оказывается зарегистрировать его в **Keystone** было недостаточно):

Для этого создадим БД **placement** и выдадим разрешения:

![](media/a6d654993c4e4f1206ad73944ce592f5.png)

Так-как в **Keystone** мы **placement** зарегистрировали, пропускаем этот шаг и заполняем файл конфига **/etc/placement/placement.conf**:

![](media/0eba53bfc550fa8513f2f5c31bf0c18e.png)

![](media/b811ddc65a84cfc3dfdad06e1b5dee6b.png)

Синхронизируем БД **placement**:

![](media/9e2d8f807aebe966ad51d3f4fdba0897.png)

8.4. Запускаем все службы **Nova** и перезапускаем **httpd** (насчет **openstack-nova-console** пока не знаю, поэтому не запускаю, так как в официальном гайде **Openstack** (не из книги) **openstack-nova-console** даже не скачивается):

![](media/9184884d611ab701589d5a756e68080e.png)

![](media/85c02bc7a2526c0d63e50b1520788112.png)

8.5. На этом настройка сервиса **Nova** на **controller.test.local** закончена и можно переходить к настройке рабочих узлов (все настройки проводятся на **compute.test.local**, **compute-opt.test.loca**l аналогично, но с другими ip).

Первым делом устанавливаем пакеты **openstack-nova-compute** (служба **Nova** для рабочих узлов) **sysfsutils** (утилита получения информации о ВМ) **libvirt** (утилита виртуализации), если не качается фикс [**здесь**](#bookmark=id.tyjcwt):

![](media/52c7884471668dd53bc137637781d223.png)

Далее необходимо проверить поддержку аппаратной виртуализации на рабочем узле (**svm** — поддержка от Amd, **vmx** — от Intel):

![](media/46752ac0475afd10f139c49e047761a4.png)

Если вдруг аппаратная виртуализация не поддерживается (пустой вывод команды выше), то необходимо включить эмуляцию с помощью **QEMU**:

![](media/6d4d5e0db101e14462c53d54bb8eac9a.png)

Теперь настраиваем конфигурационный файл сервиса **Nova** **/etc/nova/nova.conf**, для начала базовые параметры для всех узлов, о которых упоминалось [**ранее**](#bookmark=id.2s8eyo1):

![](media/a751dcfc11d2151999d495ae70635ebc.png)

Теперь устанавливаем параметры **/etc/nova/nova.conf** специфичные для рабочих узлов:

Указываем адрес вычислительного узла (**compute-opt** — **192.168.122.215**):

![](media/dde0ced28ed6b086625980dc9822d508.png)

Включаем поддержку vnc, а также указываем что **VNC-серве**р будет слушать подключения на всех интерфейсах и URL прокси-сервера, где будет доступен браузеру интерфейс виртуальной машины:

![](media/3cbc8c6b6824d3798aef60213e7398a1.png)

Указываем адрес узла-клиента nvcproxy (**compute-opt** — **192.168.122.215**):

![](media/1ba2c857a65294777b72a1d624aecfba.png)

На этом работа с конфигом заканчивается и можно запускать сервисы **libvirtd** **openstack-nova-compute** (**openstack-nova-compute** не запустится, если **controller** узел не в сети):

![](media/e8e48496be35dad6c30ed0ced19bd29a.png)

8.6. Теперь проверяем что мы там настроили и заканчиваем синхронизацию на **controller** узле:

Проверяем видит ли **controller** узел наш **compute** узел:

![](media/84338dcb526d1ad19dffe174f291c68d.png)

Если видит, то регистрируем гипервизоры с помощью скрипта (вывод примерно такой, честно скажу, свой вывод для **compute.test.local** я потерял, так что взял вывод после подключения **compute-opt.test.local**):

![](media/fc39344907b860aa7f7e576b5deacccd.png)

Теперь можно проверить работу **Placement** API с помощью еще одного скрипта:

![](media/9611986cafbcc6a353471a24fa5ea578.png)

**Если скрипт выводит ошибку 403** — то вы такой же как и я (хотя как может быть по другому, если вы шли по этому гайду), так вот ошибка решается добавлением секции **\<Directory /usr/bin\>** в конфигурационный файл **httpd** для **Placement** API **/etc/httpd/conf.d/00-placement-api.conf** и последующим перезапуском сервиса **httpd**:

![](media/a4b8597e4b90e2bd47fb5cacb133c905.png)

На этом настройка сервиса **Nova** закончилась (нашел [**крутой современный гайд**](https://www.informaticar.net/openstack-compute-installation-tutorial-centos/) на установку **Openstack**), теперь экспромтом покажу как настроить **nfs-клиента** на **compute-opt** узле, так как автоматически он его не создаст:

Во-первых вот так по итогу выглядит итоговый **/etc/hosts**, каждого узла нашего проекта:

**![](media/9d049f484d4c2a213eaebdd1604726c2.png)**

Скачиваем и запускаем **nfs-server**:

![](media/fad90f9d707279c2a6e303eb2445dfcd.png)

Создаем папку, в которую мы смонтируем **nfs-каталог**, и собственно монтируем в нее **nfs-каталог** нашего **nfs-сервера**:

![](media/db93ddccd6adc59a47be078db66db804.png)

Для того чтобы каталог автоматически монтировался, добавим соответствующую строку в файл **/etc/fstab** и применим конфигурацию:

![](media/8bd67a049baeccbd7cfb5631bc83b744.png)

-   **rw** — разрешение на чтение и запись
-   **sync** — все изменения сразу сбрасываются на диск без использования кэша
-   **hard** — клиент будет пинговать nfs-сервер, если тот станет недоступен
-   **intr** — монтирование можно прервать

9\. Установка и настройка сетевого сервиса Neutron

9.1. Про сервис Neutron написано много, так что введение будет большим. Для начала **Neutron** — реализует коммутацию (**NetFlow**, **RSPAN**, **SPAN**, **LACP**, **802.1q**, туннелирование **GRE** и **VXLAN**), балансировку нагрузки, брандмауэр и **VPN** в кластере **Openstack** (раз он сетевой сервис). Определим основные абстракции сервиса:

-   **сеть** – есть внутренние, виртуальные сети, которых может быть много, и как минимум одна внешняя. Доступ к виртуальным машинам внутри внутренней сети могут получить только машины в этой же сети или узлы, связанные через виртуальные маршрутизаторы. Внешняя сеть представляет собой отображение части реально существующей физической сети для обеспечения сетевой связанности виртуальных машин внутри облака и «внешнего мира».
-   **подсеть** – сеть должна иметь ассоциированные с ней подсети. Именно через подсеть задается конкретный диапазон IP-адресов
-   **маршрутизатор** – как и в физическом мире, служит для маршрутизации между сетями.
-   **группа безопасности** – набор правил брандмауэра, применяющихся к виртуальным машинам в этой группе
-   **«плавающий IP-адрес»** (Floating IP) – IP-адрес внешней сети, назначаемый экземпляру виртуальной машины. Он может быть выделен только из существующей внешней сети (по умолчанию каждый проект имеет квоту в 50 адресов)
-   **порт** – подключение к подсети. Порт на виртуальном коммутаторе. Включает в себя MAC-адрес и IP-адрес

Соответственно в кластере приходится иметь дело со следующими видами трафика:

-   **внешний трафик** — публичная сеть, которой принадлежат «реальные» IP-адреса виртуальных машин, а также демилитаризованная зона
-   **внутренний трафик** — сеть для операционных систем и служб (ssh и мониторинг), сеть для сетевой установки узлов под сервисы OpenStack, сеть для **IPMI**, **DRAC**, **iLO**, консолей коммутаторов и сеть передачи данных для служб **SDS** (**Swift**, **Ceph**, **iSCSI**, **NFS**)
-   **трафик виртуальных машин** — сеть связи внешнего трафика с ВМ
-   **трафик API и управления** — сеть, предоставляющая наружу публичный API и веб-интерфейс **Horizon**

Для функционирования Neutron нужно минимум три узла:

-   **узел управления** — узел с брокером сообщений (**controller.test.local**)
-   **сетевой узел** — **производительный** физический сервер или физический коммутатор
-   **вычислительный узел** — некоторые службы Neutron добавляются на каждый рабочий узел (**compute.test.local** и **compute-opt.test.local**)

Почему все используют **Neutron** вместо **nova-networ**k? Потому что в **Neutron** можно расширять API при помощи подключаемых модулей (в **nova-network** для настройки сети использовались только возможности ОС на которой работает узел), при этом у **Neutron** есть супер модуль **Modular Layer 2** (**ML2** – **OVS**/**LB**), позволяющий использовать сразу несколько технологий 2 уровня (раньше можно было только **LinuxBridge** или только **Open vSwitch**) путем использования специальных драйверов (присутствуют из коробки):

-   драйвера агентов, например **LinuxBridge** или **OVS**
-   драйвера контроллеров **SDN**, например **OpenDaylight**, **OpenContrail**, **VMware** **NSX** или **PLUMgrid**
-   драйвера аппаратных коммутаторов, например **Cisco** **Nexus** или **Extreme** **Networks**

Также можно качать сторонние [**драйвера**](https://wiki.openstack.org/wiki/Neutron_Plugins_and_Drivers) или разворачивать сервисы:

-   **маршрутизатор**
-   **балансировщик** **нагрузки** (LBaaS)
-   **брандмауэр** (FaaS)
-   **виртуальные** **частные** **сети** (VPNaaS)

Наконец на 65 странице можно частично понять смыл проектов **Openstack** — каждый проект может иметь свою сеть, соответственно один узел может иметь несколько адресных пространств которые даже могут пересекаться (несколько ARP таблиц и таблиц маршрутизации, наборов правил брандмауэра, сетевых устройств и т. д).

Что касается служб **Neutron**:

-   **neutron-server** – центральный управляющий компонент. Не занимается непосредственно маршрутизацией пакетов. С остальными компонентами взаимодействует через брокер сообщений **(сидит на узле управления)**
-   **neutron-openvswitch-agent** – взаимодействует с **neutron-server** через брокер сообщений и отдает команды **OVS** для построения таблицы потоков **(сидит на сетевом и рабочих узлах)**
-   **neutron-l3-agent** – обеспечивает маршрутизацию и NAT, используя технологию сетевых пространств имен **(сидит на сетевом узле)**
-   **openvswitch** – программный коммутатор, используемый для построения сетей **(сидит на сетевом и рабочих узлах)**
-   **neutron-dhcp-agent** – сервис отвечает за запуск и управление процессами dnsmasq, легковесного **dhcp-сервера** и сервиса кэширования DNS. Также **neutron-dhcp-agent** отвечает за запуск прокси-процессов сервера предоставления метаданных (каждая сеть, создаваемая агентом, получает собственное пространство имен **qdhcp-UUID_сети**) **(сидит на сетевом узле)**
-   **neutron-metadata-agent** – данный сервис позволяет виртуальным машинам запрашивать данные о себе, такие как имя узла, открытый **ssh-ключ** для аутентификации и др (ВМ получают эту информацию во время загрузки скриптом, обращаясь на адрес **http://169.254.169.254**. Агент проксирует соответствующие запросы к **openstack-nova-api** при помощи пространства имен маршрутизатора или **DHCP**) **(сидит на сетевом узле)**
-   **neutron-ovs-cleanup** – отвечает во время старта за удаление из базы данных **OVS** неиспользуемых мостов «старых» виртуальных машин **(сидит на сетевом узле)**

Итак что происходит в **Neutron** при создании ВМ через сервис **Nova**:

1.  Сервис **Nova** отправляет запрос на сервис neutron-server, отвечающий за API
2.  **Neutron-server** отправляет запрос агенту DHCP (**neutron-dhcp-agent**) на создание IP-адреса
3.  **Neutron-dhcp-agent** обращается к сервису **dnsmasq**, отвечающему за подсеть, в которой создается виртуальная машина
4.  **Dnsmasq** возвращает первый свободный IP-адрес из диапазона адресов подсети
5.  **Neutron-dhcp-agent** отправляет этот адрес сервису **neutron-server**
6.  После того как за виртуальной машиной закреплен IP-адрес, сервис **Neutron** отправляет запрос на **Open vSwitch** для создания конфигурации, включающей IP-адрес в существующую сеть.
7.  **Open vSwitch** возвращает обратно параметры конфигурации на сервис **Neutron** при помощи шины сообщений
8.  **Neutron** отправляет параметры конфигурации сервису **Nova**
9.  Конец

Небольшая сноска перед следующим пунктом, я тут нашел рекомендуемые требования к узлам… (у меня всего 8GB, возможно в скором времени мне это все еще как аукнется, как минимум **Swift** и **Ceph** даже если бы хотел развернуть не смог), в такие моменты внезапно понимаешь, что по идее эти узлы должны одновременно запущены 24/7 и их не надо останавливать, чтобы перестать ловить фризы всей системы, а ведь впереди еще как минимум 3 сервиса.

![](media/efcb9a2891cad38a52a0f46d276bfe66.png)

9.2. Начинаем установку и настройку сервиса **Neutron** на узле управления (**controller.test.local**):

На всякий случай сделаем снимок состояния виртуалки, а то мне кажется,что грань великих тормозов все ближе:

![](media/58d87565d5c5a26bc5e2be4d83cc1540.png)

Устанавливаем пакеты **openstack-neutron** (службы **Neutron**) **openstack-neutron-ml2** (тот самый [**волшебный модуль**](#bookmark=id.17dp8vu)):

![](media/dc04b47d58eb90f8051145468b2120a9.png)

Классическое создание БД для сервиса, на это раз как нетрудно понять БД зовется **neutron**:

![](media/63df452a5ea9d6d8e9eb2e8d0fb65f34.png)

Далее… да вы угадали — регистрируем сервис в **Keystone**:

![](media/73db145f3550302e4672dd774e94a1ac.png)

![](media/f0116def4106fd2f1c4928065f64d52d.png)

Далее идут общие параметры конфигурационных файлов для сетевого, рабочих и управляющего узлов. **Neutron** — король по количеству настроек, вот описание файлов конфигураций, которые мы будем изменять:

-   **/etc/neutron/neutron.conf** — основной конфиг **Neutron**
-   **/etc/neutron/plugins/ml2/ml2_conf.ini —** конфиг модуля **ML2**
-   **/etc/neutron/plugin.ini —** ссылка на конфиг **ML2**, там прикол, что если не будет ссылки то конфиг **ML2** не подключится, так как находится в другой папке.
-   **/etc/neutron/l3_agent.ini** — конфиг **L3-агента** (отвечает за маршрутизацию)
-   **/etc/neutron/dhcp_agent.ini** — конфиг **DHCP-агента**
-   **/etc/neutron/metadata_agent.ini** — конфиг агента, предоставляющего метаданные виртуальным машинам
-   **/etc/neutron/plugins/ml2/openvswitch_agent.ini** — конфиг агента **Open vSwitch** для модуля **ML2**

Ну что приступаем к настройке **/etc/neutron/neutron.conf**, первым делом настраиваем базовые вещи (подключение к брокеру сообщений и аутентификацию **Keystone**):

![](media/4b219c2b711cecdc2263908801fdc4b2.png)

Указываем название основного подключаемого модуля второго уровня модели OSI (также есть : **hyperv**, **cisco**, **brocade**, **embrance**, **vmware**, **nec** и др):

![](media/9d3450d2fd070676d896d0533b0e5802.png)

Указываем погружаемые модули (из **router**, **firewall**, **lbaas**, **vpnaas**, **metering**), подгружаем только маршрутизатор:

![](media/ee86f392597bea572e0ef69bd3539165.png)

Настройка разрешения пересечения IP внутри проектов:

![](media/137953fad4b225310d9d81c97ef30917.png)

Забыл настроить путь к файлам блокировки:

![](media/8dd03dd05a200effccf7944402662072.png)

Настройка **/etc/neutron/neutron.conf** закончена (вроде как), теперь настраиваем **/etc/neutron/plugins/ml2/ml2_conf.ini**:

Настраиваем используемые сегменты сетей (выбор из: **flat**, **GRE**, **VLAN**, **VXLAN** и **local**):

![](media/9f6d84b6e5761bacd5602b0552ea0067.png)

Настраиваем технологии используемые для сетей проектов (поддерживающие пересечение IP: **GRE**, **VLAN**, **VXLAN** и **flat**):

![](media/6daee08fe44e13978a88fa9fc8a447b6.png)

Настраиваем тип драйвера механизма **ML2** (выбор из: **openvswitch**, **mlnx**, **arista**, **cisco**, **logger**, **linuxbridge** и **brocade**):

![](media/8d6fbd19080c055160a6416d57da038e.png)

Настраиваем диапазон ID для [**GRE-туннелей**](https://wiki.dieg.info/gre):

![](media/f80394e43afc850ae97c689ac14b480c.png)

Включаем группы безопасности [**ipset**](https://www.dmosk.ru/terminus.php?object=ipset):

![](media/fcafeeb32a3afda7bca2dedfbee2d22b.png)

![](media/2be059b77f4d2a31906fa9cc71326b42.png)

Выше были настройки общие для всех узлов, теперь пойдут настройки только для управляющего узла:

Настраиваем параметры подключения к БД:

![](media/adaeda5fed09b2be45dc8fef79f40eb3.png)

Настраиваем отправку уведомлений сервису **Nova** при изменении статуса портов и IP-адресов:

![](media/8bac8a3c29ade930ca95fc7ef0c4b44e.png)

Настраиваем параметры подключения к **Keystone** сервиса **Nova** (видимо **Neutron** нужно в определенных обстоятельствах нужно выдавать себя за **Nova**):

![](media/bb8db2761de9ad20f5dca94486a451f9.png)

Переносим все заботы о сети с сервиса **Nova** на **Neutron** в файле **/etc/nova/nova.conf** (еще брандмауэр **Nova** вырубаем), третью команду отдельно выносить не буду, там мы задаем поддержку **Open vSwitch**:

![](media/849bcda170115feb36dd194436667ff5.png)

Настраиваем параметры аутентификации **Neutron** для **Nova**, если раньше мы писали их в **/etc/neutron/neutron.conf**, то теперь в **/etc/nova/nova.conf**:

![](media/684b942b7caba08f4ff54f343c9df829.png)

Теперь можно синхронизировать БД:

![](media/46a7dc27a096755ef8268ffa5988f0ab.png)

У меня вылезла ошибка **«No module named MySQLdb»**, почему только сейчас известно только разрабам **Openstack** (хотя вывод скрипта огромный, мб он единственный использует это модуль), но фиксится она как нетрудно догадаться скачиванием данного модуля (после снапшота, я теперь готов все что угодно качать):

![](media/762425d2aa665f89bc5fc16949499bd5.png)

Теперь создаем ссылку на конфиг ML2 в папке настроек Neutron **/etc/neutron/**:

![](media/5ecdde3591a2bedfe275e42dce62cb04.png)

Указываем использование прокси для сервера метаданных и общий секрет, которым будут подписываться запросы к серверу метаданных в конфиге **/etc/nova/nova.conf:**

**![](media/67b2d15221a51bb718790ecf751bf1e9.png)**

По итогу на узле управления в конфиги добавляются следующие параметры:

![](media/e26d4786c34485a242d6b6e43bb6536d.png)

Перезапускаем сервисы **nova** и стартуем сервис **neutron-server**:

![](media/4dd35371ed9496e38095fff788d2a081.png)

Проверка итога настройки:

![](media/dc71f4b20a07ebfb8a67b9ae59435c41.png)

9.3. Теперь я оказался на перепутье ведь надо настраивать сетевой узел как отдельный узел, а ноутбук уже просит пощады, так что мой выбор пал на разворачивание сетевого узла на узле управления, может выйти гениально, а может и нет (но я ведь не зря снимок сделал). Далее я буду писать как будто настраиваю отдельный узел, но делать пометки лишних действий, так как некоторые уже делал при настройке узла управления:

Начнем с установки необходимых пакетов (если как и я устанавливаете на узел управления, то **openstack-neutron** и **openstack-neutron-ml2** качать не надо) **openstack-neutron-openvswitch** (модуль openvswitch для openstack) и [**openvswitch**](https://habr.com/ru/post/325560/) (программный коммутатор):

![](media/edb49d7c32444ee4f4d4e3dc95a466e4.png)

Далее идет установка всех общих параметров конфигов (на узле управления уже установлены) и создание ссылки на конфиг в **/etc/neutron/plugins/ml2/ml2_conf.ini**:

![](media/2be059b77f4d2a31906fa9cc71326b42.png)

![](media/5ecdde3591a2bedfe275e42dce62cb04.png)

Дальше уже страшно, так как нужно внести изменения в параметры ядра, а именно включить маршрутизацию пакетов (**net.ipv4.ip_forward**) и выключить фильтрацию пакетов по их исходящему адресу (**net.ipv4.conf.all.rp_filter** и **net.ipv4.conf.default.rp_filter**):

![](media/f70743de4e15bb712d3789d28326ef69.png)

Настраиваем конфиг **ML2** **/etc/neutron/plugins/ml2/ml2_conf.ini**, указывая [**flat**](https://wiki5.ru/wiki/Flat_network) провайдер для внешней сети:

![](media/3dd47dfc0b52a9ec5bac487968a66126.png)

Настраиваем конфиг **Open vSwitch** **/etc/neutron/plugins/ml2/openvswitch_agent.ini**:

Указываем IP-адрес для конечной точки туннеля (ip узла управления):

![](media/c10e2da385c13c7c11c0845545fcac3f.png)

Сопоставляем внешнюю сеть созданному мосту **br-ex**:

![](media/f051c7cdc0fd778ed177ae11b8e9e31b.png)

Настраиваем включение **GRE-туннелирования**:

![](media/a72f1759af8bfd8f3d24c15e5c6ba703.png)

Настраиваем конфиг **L3-агента** (маршрутизация) **/etc/neutron/l3_agent.ini**:

Указываем драйвер **Open vSwitch**:

![](media/025e398f6aa8413590442593c129b030.png)

Настраиваем использование **namespace**:

![](media/4d75f33436755f94f5bfa5850300c5bf.png)

Указываем имя, используемого моста:

![](media/eb362b3fe2356c3ec6fe1f7b9153a687.png)

Настраиваем удаление неиспользуемых **namespace**:

![](media/e22e9f6321beb5e33d8f36c1baf42f92.png)

Настраиваем конфиг **DHCP-агента** **/etc/neutron/dhcp_agent.ini**:

Указываем драйвер **Open vSwitch**:

![](media/4c1c8f2bea4651731a774b7f8c12b2c6.png)

Указываем **Dnsmasq** DHCP-драйвер:

![](media/8a2f02f15acd6293bd86d3196aa0c37a.png)

Настраиваем использование и удаление **namespace**:

![](media/c544f79ea835f2aa466e84ac6476ec2f.png)

Настриваем конфиг агента-метаданных **/etc/neutron/metadata_agent.ini**:

Указываем IP-адрес управляющего узла:

![](media/2b788c3e02430fd8d51f05e4f83e10d2.png)

Указываем общий секрет, используемый для получения метаданных;

![](media/e18f81cb96ef101baaa037e8deb3688b.png)

Дополнительные параметры конфигов сетевого (управляющего) узла:

![](media/291a2a4379527c5247eecd0a11248fd3.png)

Запускаем и включаем **Open vSwitch**:

![](media/ec5c450437d3bdb677280cd7a159adf7.png)

Я тут понял, что в определенный момент перестал показывать скрины статусов, но не переживайте, все работает без ошибок.

Ухх, господа, теперь [**надо создать сетевой интерфейс**](https://bozza.ru/art-266.html) для нашей так называемой внутренней сети (в книге он был с первой главы, в то время как у меня был только **eth0** на всех ВМ), так что выходим из нашей виртуалки на хост и начинаем создавать новую сеть типа bridge:

Первым делом создаем файл конфигурации сети **private.xml** (я просто скопировал содержимое файла **/etc/libvirt/qemu/networks/default.xml** и поменял ip и удалил части, которые генерируются автоматически), потому что **virsh** почему-то не может создать сеть через параметры командной строки (Привет это Артем и будущего, я ошибся в ip адресах и фиксил это один день своей жизни, не беспокойтесь скрины уже исправлены):

![](media/f576eada66819763444635495745e8ba.png)

Создаем сеть с помощью **virsh**:

![](media/0db82c5e5720b5b0659b13869e9bb5dc.png)

Включаем автостарт сети private (перед настройкой автостарта, нужно открыть конфиг сети через **net-edit**, чтобы первоначальные настройки сети применились):

![](media/c5a4c5904f3aedae6841d0306cd6580a.png)

Должен появится сетевой интерфейс **virbr3** на хосте:

![](media/37866c4fc0f1a4e5f5a4675f1549c190.png)

Теперь можно запускать **VM2** (**controller.test.local**) и присоединить к ней сетевой интерфейс (команда выполняется на хосте):

![](media/1980b451a64422f687cfffa75c623443.png)

Внутри ВМ должен появится новый интерфейс **eth1**:

![](media/4057f70becc5e6b07c9ae015e9b8a154.png)

На этом все, сетевой интерфейс к машине присоединен, теперь можно перейти к его настройке для сетевого (у меня управляющего) узла для сервиса **Neutron**:

Создаем файл **/etc/sysconfig/network-scripts/ifcfg-eth1** и пишем в него следующие настройки и перезапускаем сеть (Опять Артем из будущего, короче с конфигами были тоже проблемы и пришлось добавить файл **/etc/sysconfig/network-scripts/ifcfg-br-ex** специально для моста):

![](media/6e0d0e212d9d9e7d425ecd38c11f024e.png)

Теперь надо добавить мост **br-ex** для Open vSwitch, который будет находится на созданном интерфейсе **etn1**:

![](media/f26c91e7e3ab3ec90bae7a4a0c26effe.png)

Проверяем все ли подключилось нормально при помощи **ovs-vsctl**:

![](media/35f530f72a7bf7d7baa6ddcec1e64a53.png)

Вроде все круто, так что можно запускать все службы **Neutron** для сетевого (сами знаете какого) узла (насколько я понял ovscleanup стартовать не надо):

![](media/0f3634997f0f78dbbb1c9e5a5e4e1c19.png)

Вместо того, чтобы проверять статус каждой службы можно посмотреть все одной командой:

![](media/2af3252af99072e44ce0b18e57f8f5b0.png)

Теперь переходим к настройке служб **Neutron** на рабочих узлах **compute** и **compute-opt**:

Качаем необходимые пакеты (все уже обсуждались раньше):

![](media/9a6cccd6bd706fb9992279f0bd46c106.png)

Теперь заполняем общие для всех узлов параметры конфигов для сервиса **Neutron**, и создаем ссылку на конфиг **ML2**:

![](media/2be059b77f4d2a31906fa9cc71326b42.png)

![](media/5ecdde3591a2bedfe275e42dce62cb04.png)

Базу выполнили, теперь изменяем параметры ядра **net.ipv4.conf.all.rp_filter** и **net.ipv4.conf.default.rp_filter** (отключение фильтрации пакетов), **net.bridge.bridge-nf-call-iptables** (пакеты с сетевого моста передаются на обработку **iptables**):

![](media/0a7b3569dcb238f55b52c335413cdb48.png)

Если последняя строка не выполняется то выполните следующую команду (включает модуль ядра [**br_netfilter**](https://ebtables.netfilter.org/documentation/bridge-nf.html)), а потом опять **sysctl**:

![](media/e1f52983861cb34c32464e5679f7c08e.png)

Так, ну теперь пошли специфичные для рабочих узлов параметры конфигов:

Настраиваем конфиг **Open vSwitch** **/etc/neutron/plugins/ml2/openvswitch_agent.ini**:

Указываем локальный ip вычислительного узла (**compute-opt** — **192.168.122.215**):

![](media/9fb6faa5e7eb91011de67af61f729a81.png)

Указываем тип используемой технологии туннеля:

![](media/8592e9de7e399960d8bc656e6f98491f.png)

Указываем драйвер брандмауэра (**iptables** на **Open vSwitch**):

![](media/71f77651ab8fdb277c0b20b8b2462561.png)

Запускаем **Open vSwitch**, так как его настройка закончена:

![](media/38c1590e6b33102262bbef3aaa2eb052.png)

Настраиваем конфиг сервиса **Nova** **/etc/nova/nova.conf**:

Передаем полномочия (первые 3 команды) с службы **nova-network** на **Neutron**, а также отключаем службу брандмауэра в **Nova** (последняя команда):

![](media/849bcda170115feb36dd194436667ff5.png)

Указываем параметры аутентификации **Neutron** (как и на управляющем узле):

![](media/684b942b7caba08f4ff54f343c9df829.png)

Персональные параметры рабочих узлов:

![](media/a48ef41ca91d8b0be068d58fcbc33b7c.png)

Перезапускаем службу **nova-compute** и запускаем агента **Open vSwitch**:

![](media/03f1fe5dc2b4368b3addeb971e0a3f20.png)

Должны появится новые сетевые агенты в выводе команды, по одному на рабочий узел (я пока настроил только один):

![](media/9ae2b40c004a8b2f70a0c566cdcae255.png)

Все, с сервисом **Neutron** пока покончено, как и с основными (точнее 6 из 7, без Swift) сервисами **Openstack**. Осталось только поработать с созданием ВМ и сетей для них, а также установить доп сервисы для удобства (**Horizon**, **Gnocchi**. **Ceilometer**, **Aodh** и **Heat**). Есть также некоторые сомнения касательно подключения дополнительных сетевых интерфейсов для рабочих узлов (пока не подключаю), если их надо будет добавить, естественно, сделаю сноску (**не надо**).

10\. Тестим все то, что на разворачивали.

10.1. Для начала создадим все необходимые компоненты сетевой инфраструктуры для запуска первой ВМ:

Создаем внешнюю сеть **ext-net**:

![](media/a8137d09adf26446bcfe0ecac580f463.png)

Из-за опечатки в книге (вместо **--provider-physical-network** **external** было написано **--provider-physical-network datacentre**) я потратил очередные 2 часа своей жизни на исправление бага… А ведь это уже 4 издание книги.

-   **--external** — сеть является внешней (есть средство внешней маршрутизации)
-   **--share** — сеть доступна всем проектам
-   **--provider-network-type** — физический механизм, с помощью которого реализуется виртуальная сеть (**flat**, **geneve**, **gre**, **local**, **vlan**, **vxlan**)
-   **--provider-physical-network** — имя физической сети, в которой реализована виртуальная сеть (мы мапили это название с созданным мостом, а потом объявляли его возможным для использования в типе сети **flat**)

Теперь создадим подсеть, из которой будут выделяться плавающие IP-адреса (в реальности они должны быть видны из интернета).

![](media/4cf102c68e876c4111b029e55db8771c.png)

-   **--network** — сеть родитель
-   **--no-dhcp** — без DHCP
-   **--allocation-pool** — доступные ip адреса
-   **--gateway** — дефолтный шлюз во внешнюю сеть (ip маршрутизатора)
-   **--subnet-range** — маска подсети

Теперь переквалифицируемся в пользователя demo и создаем сеть в его личном проекте:

![](media/9e1c85229d9c9fc8daf2ffb7a570060d.png)

Создаем подсеть (**demo-subnet**) нашей **demo-net**:

![](media/5d0385e6a20a80f3dc967a321a85b71b.png)

Создаем роутер (**demo-router**) для **demo-subnet**:

![](media/55f27b362382013e57a971c79264feaf.png)

Добавляем подсеть **demo-subnet** к роутеру **demo-router**:

![](media/3d82f27b7ffff3dcd5afa4d29bb97441.png)

**Отступление от темы**: у меня уже начинаются убиваться процессы на виртуалке из-за нехватки оперативки.

Теперь выставляем внешний шлюз для роутера **demo-router**:

![](media/468a1dadde55b74117e2218e4167dc5f.png)

Вот примерные параметры роутера **demo-router**:

![](media/4de7b72e779f50ec089a30127d8b4049.png)

10.2. Теперь переходим к созданию экземпляра ВМ:

А, попались, думали уже можно создать ВМ, неа еще надо создать **flavour** (шаблон виртуальной машины по выделяемым ресурсам) перед этим. Шаблоны по умолчанию может создавать только админ:

![](media/c426705fd43aaa59f8fb0ab9bec7dbdf.png)

-   **--ram** — оперативная память
-   **--disk** — размер диска
-   **--vcpu** — количество виртуальных ядер CPU
-   **--public** — доступен всем пользователям
-   **--rxtx-factor** — пропускная способность сети (только для гипервизора **XEN**)
-   **--ephemeral** — размер второго диска (в отличии от **--disk** всегда удаляется при удалении ВМ)
-   **--swap** — размер опционального [**swap-раздела**](https://fornex.com/ru/help/swap/#:~:text=SWAP%20(%D1%81%D0%B2%D0%BE%D0%BF)%20%E2%80%94%20%D1%8D%D1%82%D0%BE%20%D0%BC%D0%B5%D1%85%D0%B0%D0%BD%D0%B8%D0%B7%D0%BC,%2C%20%D0%BD%D0%B0%D0%B7%D1%8B%D0%B2%D0%B0%D0%B5%D0%BC%D1%8B%D0%B5%20%D1%81%D1%82%D1%80%D0%B0%D0%BD%D0%B8%D1%86%D0%B0%D0%BC%D0%B8%20(pages).)

Выделил контроллер узлу еще 2GB оперативки, надеюсь хватит, но я уже вхожу в зону «выделил ресурсов больше чем на самом деле» (2+2+6=8)

Теперь создадим пару **ssh-ключей**, которую служба метаданных сервиса **Nova** будет передавать созданным ВМ:

![](media/1c5e9bae58e61e2e8f4eb1f60c77f681.png)

Вхожу в теневую зону (запускаю сразу 3 ВМ) и пытаюсь создать ВМ по созданному шаблону **m2.tiny**, передав в нее созданную пару ключей **demokey1** (если на этом отчет закончился, то это потому что у меня взорвался ноутбук):

При запуске обоих рабочих узлов, узел управления сразу падает, видимо kvm не любит избыточность ресурсов, поэтому прощай **compute-opt**. Лагануло при создании ВМ у меня знатно.

![](media/cc7bd0b94de8112ae1e907aa4a10d359.png)

**Изливаю очередной бомбеж:** я весь день (5 часов подряд) пытался пофиксить ошибку **UnicodeDecodeError** (ошибки можно найти в **логах**, выводе команд **journalctl** и **openstack server show имя_ВМ**), которая вылезала при создании ВМ, к 5 часу я уже редачил питоновские файлы по гайдам на китайском языке, но все равно ошибка коварно вылезала, а потом я перезагрузил (в 3 раз за 5 часов) виртуалки и все пофиксилось, вот как-то так...

Посмотреть статус созданной ВМ можно как с помощью **openstack cli**, так и при помощи **virsh** на рабочем узле:

![](media/0e3b94bdac5244aa879a5f640f83be0f.png)

На рабочем узле:

![](media/2b80deb8de41d677c0d6c9b7b52152fd.png)

Теперь можно подключится к ВМ через **noVNC** (**VNC** через браузер), получив URL с помощью **openstack cli**:

![](media/c00cff5b39799f2dfd8d3773ecf3fd0c.png)

Пишем полученный URL в строку браузера и получаем доступ к консоли ВМ **myinctance1** (у меня на хосте не настроено сопоставление DNS с ip, так что я заменил **controller.test.local** на **192.168.122.200**):

![](media/82421171ce116e3c50d5efa9efa88280.png)

Вот скрины из книги о том как создается ВМ (запрос путешествует по всем сервисам):

![](media/4b41c6dcc43a8a6d3730495f0e737fbf.png)

![](media/56428e603ef51c45fc24202fe5e3b71c.png)

Теперь раз ВМ включена то должна начать записываться статистика по потреблению ей ресурсов:

![](media/7fb17165d60bf4e3d985b5bf63883df6.png)

Теперь надо добавить к ВМ сеть, чтобы к ней например можно было подключаться через SSH:

Первым делом надо создать группу безопасности **demo-sgrpoup** (набор правил брандмауэра, разрешающих доступ к тем или иным портам виртуальной машины), так как к ВМ автоматически добавилась группа **default**, которая ничего не разрешает:

![](media/b510d187b63b20ce4187fa3d9be80026.png)

Теперь добавим 2 правила: первое разрешает доступ по SSH, второе разрешает протокол ICMP:

![](media/0bde0f8a477180a09f6d6b69ee7f5f41.png)

![](media/7aa17c0504ca1cd141a49ad8b53f805a.png)

Посмотреть установленные правила можно так:

![](media/a546c2f581ccace6a94f27f92e961ee5.png)

Теперь можно добавить группу безопасности **demo-sgroup** ВМ **myinsatnce1** и удалить группу **default** (также можно указывать группу безопасности при запуске ВМ при помощи флага **--security-groups**), заодно держите удобный способ просмотра отдельных параметров вывода команды:

![](media/8e2520ead8f6596992e95c59a8913fa4.png)

Группу безопасности добавили теперь надо выделить ip из внешней сети **ext-net** и добавить его нашей ВМ для подключения:

![](media/146a302c96cd9094522e4ef02dcb29fe.png)

Не переживайте, что у меня ВМ вырублена, это просто чтобы ноут не зависал:

![](media/46b99d67b9d7e2b3e2a0965b8c547e2f.png)

Теперь с помощью SSH можно подключится к созданной виртуалке по предоставленному ей плавающему ip (только перед этим надо выдать другие разрешения на файл ключей **demokey1**, а то SSH его проигнорирует из-за недостаточной безопасности):

![](media/da78bb3387fca7397c2ad0db9e21c9f2.png)

10.3. Переходим к тестированию создания снимков и резервных копий ВМ:

Когда-то там мы создавали мы создавали том **testvlo1**, его и используем:

![](media/7560e1a3dc2f4d49e1336b212c437629.png)

Добавим том **testvol1** к ВМ **myintsance1** (можно подключать во время создания ВМ при помощи флага **--block-device-mapping**), добавляем к ВМ по id (не по имени ВМ), потому что **admin** не видит ВМ **myintsance1**, так как она находится в другом проекте (опять китайцы помогли исправить баг, на этот раз **OSError: no such file or directory**, не знаю из-за устаревшей книги или из-за чего еще но в файл конфига сервиса **Cinder /etc/cinder/cinder.conf** надо добавить параметр **target_helper** то ли вместо **iscsi_helper** то ли вместе (че поделать все на китайском написано), я добавил вместе и все заработало):

![](media/279ce03bdf91b4c92b64e2e8f23d67f0.png)

![](media/860428523f7644cd6420dc9d04762751.png)

Теперь он должен стать помечен как использующийся (**in-use**):

![](media/f0f78284677c24c8a1ce603d46cbe147.png)

Заходим в ВМ и смотрим наличие тома (потом можно смонтировать на него файловую систему и тд, но я делать этого не буду):

![](media/d43b2f0c2e2d585ebd956c69292acc30.png)

С томами все теперь переходим к снимкам. Важное уточнение оказывается снимок в **openstack** не является точкой восстановления для существующей ВМ (как в KVM), а является самостоятельным образом (как и скачанный нами **cirros**). Поэтому сники в **Openstack** используются для резервного копирования или создания нового базового образа. Так теперь надо сделать какое-нибудь видное изменение в ВМ **myinstance1**, которое можно было бы увидеть в новой ВМ **myinstance2** созданной из снимка ВМ **myinstance1** (можно сделать снимок тома (**volume snapshot**) и ВМ (**image create**)).

Создадим файл тест в **root** каталоге:

![](media/b8f852c77b1a9126f7aa9d5d8ae9d9a9.png)

Теперь создаем снимок **myinstance1_sn1** ВМ **myinstance1**, и проверяем его появление в сервисе **Glance**:

![](media/7f49c449294bb3284576a2481d5c7017.png)

![](media/749c417d0e760f86f577f16e6986fbfa.png)

Теперь запускаем новую ВМ **myinstance2**, используя созданный снимок **myinstance1_sn1**:

![](media/79390ae911e13b21ec6234746e1d5c3e.png)

Заходим в нее через браузер и смотрим присутствуют ли изменения в ВМ **myinstance2**:

![](media/e88dcad5eba052cef725b39cf4f7edc0.png)

Два совета из книги: если вы работаете с виртуальными машинами **Fedora**, **CentOS**, **RHEL** или им подобными, необходимо перед созданием снимка удалить правило udev для сети, иначе новый экземпляр не сможет создать сетевого интерфейса:

![](media/2cd49011d94f92841ef630e0b833e8ae.png)

Если виртуальная машина запущена во время создания снимка, необходимо сбросить на диск содержимое всех файловых буферов и исключить дисковый ввод/вывод каких-либо процессов:

![](media/b67010e1994b53264744aa24ceb28f6f.png)

Отличия **резервных копий** от **снимков**:

-   резервная копия сохраняется в объектное хранилище, а не в сервис работы с образами
-   резервная копия занимает столько места, сколько занимают данные, а моментальный снимок копирует весь образ полностью
-   резервную копию можно восстановить на новый том, а из моментального снимка можно запустить новый экземпляр виртуальной машины
-   в отличие от создания моментального снимка, нельзя делать резервную копию используемого тома

Узнав отличия снимков от резервных копий, можно перейти созданию резервной копии тома **testvol1** (**--force** нужен чтобы бэкапить том в состоянии **in-use**):

![](media/16ba2f4a13886d88b54ac3a66aa17c79.png)

Теперь тестируем восстановление тома (статус восстановления можно смотреть как и статус запуска ВМ), у меня бэкап не вышел (вылезает ошибка **MemoryError**, так что скорее всего просто мощности не хватает узлу), но команда вот:

![](media/197e9021c58bc5f499fee5d655dee1bd.png)

10.4. Особая тема шифрования томов **Cinder**. Шифрование необходимо при общении сервисов **Cinder** и **Nova** и реализуется либо при помощи общего секрета (делаем так), либо при помощи Barbican (сервис управления ключами, у меня нет и не будет в ближайшем будущем). Понятное дело, что если секрет скомпрометирован, то злодей может получить доступ ко всем томам, так что в реальной жизни его хорошо прячем.

Задаем значение секрета в конфигах сервисов **Nova** (**/etc/nova/nova.conf**) и **Cinder** (**/etc/cinder/cinder.conf**) и перезапускаем сервисы:

На вычислительных узлах:

![](media/e34ddf83d582d4760c451333d2252c66.png)

На управляющем узле:

![](media/fbb2951d82a952045ee03b25a28f8803.png)

Создаем новый тип тома [**LUKS**](https://en.wikipedia.org/wiki/Linux_Unified_Key_Setup):

**![](media/448307b75a4e1c09f910d12d7ac68e00.png)**

-   **--encryption-provider** — драйвер обеспечивающий поддержку шифрования
-   **--encryption-cipher** — алгоритм или режим шифрования
-   **--encryption-key-size** — размер ключа шифрования
-   **--encryption-control-location** — служба в которой выполняется шифрование

Теперь создаем сам зашифрованный том (создать его не получится, так как фича с ключами без использования сервиса Barbican устарела, и теперь надо обязательно его разворачивать, чего я пока делать не буду, но команда с примерным выводом вот):

![](media/d97d23b06b3a3e9fa26532fe352cef5b.png)

-   **--size** — размер тома в ГБ
-   **--type** — тип тома

С томами все.

10.4. Переходим к тестированию квот на ресурсы. Они используются для управления вычислительными ресурсами и дисковым пространством в разрезе проекта, но можно и для всех пользователей.

Дефолтные квоты всех проектов:

![](media/9e9f75679f45996b84a50ec8e0de73f9.png)

Также можно смотреть квоты для определенного проекта, например **demo**:

![](media/33ce5084d92d50436e7539915d14faf2.png)

Можно кенчно самому повыставлять квот, но опыт **LimitRange** и **ResourceQuota** при работе с **Kubernetes** говорит мне переходить к следующему пункту.

10.5. Итак зоны доступности и агрегаторы вычислительных узлов. Зоны доступности позволяют группировать узлы **OpenStack** в логические группы с общим элементом доступности (например одна стойка или одно здание) и по умолчанию все узлы находятся в зоне **nova**. Агрегаторы узлов же – это логическое объединение узлов с привязкой дополнительных метаданных. Агрегаторы позволяют группировать узлы по различным специфическим признакам (потом **Nova** автоматически будет разворачивать ВМ на более подходящих узлах). У меня один узел уже умер, но я решил все равно рассказать про эти объекты.

Для примера и тестирования создадим агрегатор **MyAggregate**:

![](media/9e208bda59dc907bfac865581478ea15.png)

Добавим в агрегатор **MyAggregate** рабочий узел **compute.test.local** (один узел может состоять в нескольких агрегаторах):

![](media/b60d5a059fca8fb9bbf78a679950d48e.png)

Добавим метаданные агрегатору **MyAggregate**:

![](media/db6679a29b00d0e23fd5b6963ad96def.png)

Создадим новый тип (**flavour**) ВМ **m2.mynewmdvm** (с помощью него затестим агрегатор):

![](media/70f9e5061782d0f43815dede4554de87.png)

Добавим к созданному типу ВМ **m2.mynewmdvm** метаданные, чтобы ВМ этого типа могли располагаться только на узлах определенного агрегатора **MyAggregate** и проверим получившийся тип:

![](media/6d36d42aa5b8b478ab57a6dcbc5b9c89.png)

Все теперь ВМ этого типа будет стартовать только на узле **compute.test.local**, так как метаданные типа ВМ **m2.mynewmdvm** и агрегатора **MyAggregate** в котором состоит узел совпадают (поверьте на слово ибо одновременно два рабочих узла поднять я не могу).

Зона доступности, как можно понять, является недоагрегатором, и ее можно создать если приписать агрегатору определенное имя зоны доступности, вот пример (агрегатор — **MyAggregateZ**, имя зоны — **MyZone**):

![](media/eef8ca9f69637ef46a4d82b45f7c9602.png)

Потом можно добавить узел **compute-opt.test.local** в данный агрегатор **MyAggregateZ**, и зона доступности узла изменится:

![](media/801a0632e048adcea9a7882a9a43c267.png)

![](media/6abbe226b2eff1b8583a929d3de51e90.png)

Теперь можно запустить ВМ с указанием зоны доступности (**--availability-zone**) и она развернется исключительно на узле **compute-opt.test.local** (если он у вас конечно работает, у меня просто уже нет **compute-opt** узла так что держите только пример):

![](media/857d16d3b2aff4746265ae84d33b3f51.png)

Также можно изменить дефолтную зону доступности для кластера при помощи параметра **default_availability_zone** в **/etc/nova/nova.conf** на управляющем узле.

Тест закончен, удаляем узлы из агрегаторов и сами агрегаторы **MyAggregate** и **MyAggregateZ**:

![](media/eb7a55aaa6de0a5be91399bd76ace5f3.png)

![](media/5633042f91ee52e0f6d3a83e568775f3.png)

![](media/ec406dbe120fe0d254b04a22299f22b8.png)

Также стоит отметить зоны доступности **Cinder**, чтобы монтировать подходящие для ВМ тома (принцип действия абсолютно такой же).

10.6. Тестируем живую миграцию ВМ. Есть два типа миграции:

-   с общей системой хранения данных. Виртуальная машина перемещается между двумя вычислительными узлами с общим хранилищем, к которым оба узла имеют доступ (например **NFS**, **Ceph** или вариант, когда не используются временные диски (которые удаляются при удалении ВМ), а в качестве единственной системы хранения при создании виртуальных машин используется **Cinder**)
-   без общей системы хранения данных. В этом случае на миграцию требуется больше времени, поскольку виртуальная машина копируется целиком с узла на узел по сети.

У меня миграцию произвести не выйдет, так как не могу одновременно запустить два рабочих узла, но шаги я опишу (правда без выводов команд).

Первым делом проверяем видят ли рабочие узлы **compute** и **compute-opt** друг друга:

На **compute-opt** узле:

![](media/bb2e8bd0aed1aec51a01a99628d48e76.png)

На **compute** узле:

![](media/f6359bba863b6256bc90071fef55f111.png)

Теперь запускаем ВМ **test** и смотрим на каком узле она находится, а также достаточно ли ресурсов на другом узле:

![](media/55f206f53da7cbac62af29216ee62584.png)

![](media/340bfb6f96170939a97333c7537c08da.png)

![](media/b5170146bddf20970c899304efba7677.png)

Разрешаем демону **libvirtd** слушать входящие подключения по сети, добавив следующую опцию в файл **/etc/sysconfig/libvirtd.conf** на обоих рабочих узлах:

**![](media/c76327d583a0a4f1cdaca996495dffb5.png)**

Разрешаем подключение без аутентификации и шифрования к демону **libvirtd**, редактируя файл конфига **/etc/libvirt/libvirtd.conf** (можно также использовать сертификаты или протокол [**Kerberos**](https://ru.bmstu.wiki/Kerberos)), после чего перезапускаем демон:

**![](media/fffec3daac14fb3eb40dd90d81ac3e42.png)**

Далее надо добавить флаги миграции в файл конфига **/etc/nova/nova.conf** на обоих рабочих узлах (параметр уже устарел, так что я не смог найти подробную информацию) и перезапустить службу **nova-compute**:

**![](media/6af1a7d770fc6a59a6608989f973e329.png)**

**![](media/50befb5bd7f7513d92c3121e68d46364.png)**

Теперь можно осуществить живую миграцию на другой узел (**--block-migrate** — миграция без общего дискового хранилища):

**![](media/bbe9e65b84d3d1fed2aa79d736c9eb1a.png)**

Смотрим, где теперь работает ВМ (если все получилось, то на другом узле):

![](media/10b2dc1fa27aab54d5a82bf65ace6795.png)

Информацию о миграции можно найти в параметрах ВМ, которая мигрирует:

![](media/f97a5be1643bafed73a723e0c8660d78.png)

10.7. Удобный (или нет) способ конфигурировать ВМ при запуске по названием [**cloud-init**](https://cloudinit.readthedocs.io/en/latest/topics/examples.html#yaml-examples). Заключается в создании **YAML-файлов** с синтаксисом **cloud-config**, в которые записываются описываются модули, исполняемые во время загрузки ВМ (например модуль установки пакетов), после содержимое **YAML-файла** передаются при создании ВМ (при помощи флага **--user-data**) и конфигурирует виртуалку (например задает имя хоста или предустанавливает пакеты).

Вот пример содержимого YAML-файла **cloud-config**, задающего имя ВМ:

![](media/c0c75d5ebb751528d8000715ce6f6b44.png)

После чего этот файл можно передать при создании ВМ **testvm**, и на ней как нетрудно догадаться имя хоста будет **testvm.test.local** (зайти в ВМ можно через **noVNC**):

![](media/7f84db14416a5f0fd3353cb9e0cf1a5f.png)

![](media/70e7df2c3097c0f849d84962157173de.png)

Все на этом можно закончить тесты базовой работы кластера и перейти к установке оставшихся сервисов.

11\. Установка и настройка сервиса веб-панели управления Horizon.

11.1. Сервис представляет собой удобный графический интерфейс в виде веб-приложения (на **Python/Django**), через который можно работать с Openstack. Больше собственно сказать о нем нечего, разве что на нем пока представлено только 70% функционала **openstack cli**.

11.2. Теории не много, так что приступаем к установке и конфигурации.

Первым делом устанавливаем необходимые пакеты **httpd** (Apache HTTP сервер) **mod_ssl** (модуль **Apache**, обеспечивающий криптографию через протоколы SSL и TLS) **mod_wsgi** (модуль **Apache**, предоставляющий интерфейс для работы с **web-приложениями**, написанными на **Python**) **memcached** (сервис кэширования данных в оперативной памяти на основе хеш-таблицы) **python-memcached** (Python интерфейс для работы с **memcached**) **openstack-dashboard** (сервис **Horizon**) на **controller.test.local узел** (некоторые пакеты уже установлены, но все равно их перечислю, так как **Horizon** можно установить и на другие узлы и даже ВМ):

![](media/d533c0ebc4d02fccc64a6a1fa5f4ada2.png)

Теперь конфигурируем конфиг **/etc/openstack-dashboard/local_settings** сервиса **Horizon**, изменяя опции

-   **OPENSTACK_HOST** — IP-адрес узла, где запущен Keystone (или имя)
-   **ALLOWED_HOSTS** — разрешенные ip для пользования **Horizon** (**[“\*“]** - все могут, но желательно перечислить избранных через запятую)
-   **SESSION_TIMEOUT** — время жизни сессии в **Horizon** в секундах, можно увеличить тут и в конфиге **Keystone** (e**xpiration=3600** секции **[Token]**) если образы ВМ очень большие и долго грузятся
-   **WEBROOT** — URL панели мониторинга относительно ip хоста (этот параметр я вымучил кровью и потом, почему-то с определенного момента ни один шаг не проходит без каких-либо ошибок)
-   **SESSION_ENGINE** — указание модуль обработки сеансов ([**Django**](https://django.fun/ru/docs/django/4.1/topics/http/sessions/)) пользователя
-   **CACHES** — сервис кэширования и его местоположение (ip адрес)
-   **OPENSTACK_KEYSTONE_URL** — URL сервиса **Keystone**, но мы его редачим, чтобы включить третью версию API (у меня редачить не пришлось)
-   **OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT** — поддержка доменов (openstack доменов, не путать с обычными)
-   **OPENSTACK_API_VERSIONS** - поддерживаемые версии API сервисов
-   **OPENSTACK_KEYSTONE_DEFAULT_DOMAIN** — имя домена по умолчаниЮ

![](media/f1f34672276cf8f2b76efa72fc29558c.png)

Теперь указываем параметр **WSGIApplicationGroup** (группа приложений в которой будет запускаться **Horizon**, в книге написано, что это фиксит некоторые баги) в файле конфига **/etc/httpd/conf.d/openstack-dashboard.conf** и перезапускаем службы **httpd** и **memcached**:

**![](media/bba365a31be4504310176878c86b3c94.png)**

**![](media/a4d317056e171cabec4f9c8071ab22ff.png)**

**![](media/c86979906b826de88c03daad7dbc39cd.png)**

Веб-интерфейс Horizon будет нас ждать по адресу [**http://controller.test.local/dashboard**](http://controller.test.local/dashboard) (в моем случае <http://192.168.122.200/dashboard>):

Как же тяжело даются последние сервисы, короче у меня при авторизации вылезала ошибка «**InternalServerError**», и мало, что она вылезала дак к этому еще файл логов ошибок **httpd /var/log/httpd/error_log** решил сломать вывод времени ошибки (выводил время ошибки на три часа раньше чем на самом деле). Вообщем оказалось, что проблема в **memcached**, который блокал весь трафик, кроме **localhost**, по итогу надо было поменять параметр **OPTIONS** и все заработало (еще я почистил кэш браузера мб это тоже помогло):

![](media/045103ec0953d9e05d892784f2620489.png)

![](media/62a7d26cde2333a775bd86393c8a53c7.png)

11.3. **Horizon** запустили теперь смотрим что внутри.

Заходим за пользователя **demo** и видим следующую картину:

![](media/898eece13bbcc1032a2073007fac4851.png)

Содержимое главной вкладки **Проект**:

-   **Вычислительные ресурсы**:
    -   **Обзор** – общий обзор ресурсов пользователя. Можно посмотреть текущее потребление ресурсов, а также получить краткий отчет за это время
    -   **Инстансы** – список виртуальных машин для текущего проекта, а также возможность производить с виртуальными машинами различные действия, например создание новой виртуальной машины или подключение к консоли и просмотр вывод терминала виртуальной машины
    -   **Образы** – управление образами **Glance**. Тут же из образа можно стартовать виртуальную машину и создать том
    -   **Ключевые** **пары** — управление ключами доступа по SSH
    -   **Доступ** **к API** — URL, по которым доступны API соответствующих служб облака
    -   **Группы** **серверов** — управление группами серверов (придуманы, чтобы напрмер запретить ВМ внутри группы подниматься на одном узле)
-   **Диски**
    -   **Диски** — управление томами (все тома у нас в проекте **admin**, так что **demo** пользователь их не видит)
    -   **Снимок** — управление снимками томов
    -   **Группы** — управление [**группами томов**](http://xgu.ru/wiki/%D0%93%D1%80%D1%83%D0%BF%D0%BF%D0%B0_%D1%82%D0%BE%D0%BC%D0%BE%D0%B2)
    -   **Снимки** **групп** — управление снимками групп томов
-   **Сеть**
    -   **Сетевая** **топология** – топология сети проекта.
    -   **Сети** – список сетей проекта. Тут же создаются подсети, а также находится список портов с возможностью редактирования
    -   **Роутеры** – список маршрутизаторов проекта и управление ими
    -   **Плавающие** **IP** - управление «плавающими» IP-адресами
    -   **Группы** **безопасности** — управление группами безопасности
-   **Объектное** **хранилище** (у меня нет, так как нет **Swift**)
    -   **Контейнеры** – создание контейнеров и псевдопапок, а также загрузка в объектное хранилище файлов и их скачивание.
-   **Оркестрация** (пока нет потом появится с установкой **Heat**)
    -   **Стеки** – управление стеками, включая их запуск и мониторинг.

Содержимое вкладки **Администратор** (будет выводить инфу только для пользователя **admin**):

-   **Обзор** – тут располагается статистика по всем проектам за заданный период времени;
-   **Использование** **ресурсов** – вывод информации от **OpenStack** **Telemetry** (**Ceilometer**), поэтому пока ее нет
-   **Вычислительные** **ресурсы**:
    -   **Гипервизоры** – информация по всем гипервизорам, управляемым **OpenStack** (кликнув по имени узла, можно посмотреть, какие виртуальные машины он обслуживает)
    -   **Агрегаты** **узлов** – управление агрегатами, настройка группировок узлов как на логические группы, так и группировка их по физическому месторасположению
    -   **Инстансы** – управление инстансами, прямо как во вкладке Проект только отображает все виртуальные машины
    -   **Типы** **инстансов** – управление типами экземпляров виртуальных машин
    -   **Образы** – управление образами **Glance**. Отображаются все образы
-   **Диски** – аналогична одноименной во вкладке **Проект**, однако включает дополнительные подвкладки:
    -   **Типы** **дисков** — определение типов томов и соответствующих им параметров качества обслуживания (**QOS**)
    -   **Типы** **групп** — аналогично типам томов
-   **Сеть**
    -   **Сети** – аналогична одноименной во вкладке **Проект**, но включает в себя сети всех проектов, а также тут администратор также может задать внешнюю сеть
    -   **Маршрутизаторы** – опять же как и во вкладке Проект, но включает в себя все маршрутизаторы
    -   **Плавающие** **IP** — как во вкладке Проект, только все IP
    -   **Политики** **RBAC** — управление политиками доступа [**RBAC**](https://habr.com/ru/company/custis/blog/248649/) к сетям
-   **Система**
    -   **Параметры по умолчанию** – можно задать квоты для проектов, действующие по умолчанию
    -   **Определения** **метаданных** — управление собственными метаданными, которые потом можно использовать при создании ВМ (и не только) по умолчанию или по просьбе пользователей
    -   **Системная** **информация** – информация о сервисах облака и их текущем состоянии

Последняя вкладка **Идентификация**:

-   **Проекты** — управление всеми проектами
-   **Пользователи** — управление всеми пользователями
-   **Группы** — управление всеми группами пользователей
-   **Роли** — управление всеми ролями
-   **Доступ** **для** **приложений** — управление учетными данными приложений (**application credentials** придуманы, чтобы чтобы приложения могли проходить аутентификацию в **Keystone**)

На этом с **Horizon** все. Также можно еще менять тему интерфейса, но я пожалуй откажусь.

12\. Установка и настройка сервиса сбора телеметрии Telemetry (Ceilometer, Aodh, Gnocchi)

12.1. Итак сервис **Openstack Telemetry** состоит из трех отдельных проектов связанных с мониторингом:

-   **Ceilometer** – отвечает за мониторинг и сбор данных, поддерживает два способа сбора:
    -   при помощи очереди сообщений через службу **ceilometer-collector** (запускается на одном или более управляющих узлах и отслеживает очередь сообщений, получает уведомления от других сервисов **Openstack** и, преобразовывая их в сообщения телеметрии, отправляет обратно в очередь сообщений)
    -   через опрашивающие инфраструктуру агенты (через вызовы API агенты периодически запрашивают у сервисов **Openstack** необходимую информацию)

Состоит из следующих служб:

-   **openstack-ceilometer-notification** – агент, отправляющий по протоколу AMQP метрики сборщику от различных сервисов
    -   **openstack-ceilometer-central** – агент, запускаемый на центральном сервере для запроса статистики по загрузке, не связанной с экземплярами виртуальных машин или вычислительными узлами (можно запустить несколько штук)
    -   **openstack-ceilometer-compute** – агент, запускаемый на всех вычислительных узлах для сбора статистики по узлам и экземплярам виртуальных машин
-   **Aodh** – отвечает за обработку триггеров (**alarms**). Триггеры срабатывают при достижении метрикой определенного значения, после чего в **Openstack** можно обратится по HTTP на определенный адрес или записать событие в журнал. Состоит из следующих служб:
    -   **openstack-aodh-api** – API-сервер, который запускается на одном или более узлах. Служит для предоставления доступа к информации о сработавших триггерах (**alarms**)
    -   **openstack-aodh-evaluator** – сервис, определяющий, сработал ли триггер при достижении метриками заданных значений в течение определенного измеряемого периода
    -   **openstack-aodh-notifier** – сервис, запускающий те или иные действия при срабатывании триггера
    -   **openstack-aodh-listener** – сервис, определяющий, когда триггер сработает (срабатывание определяется сравнением правил срабатывания триггера и событий, полученных агентами сбора телеметрии)
-   **Gnocchi** – бэкэнд для хранения метрик, собранных сервисом телеметрии. Придуман чтобы хранить данные в **Ceph** (или файловой системе), так как это удобнее. Состоит из двух служб:
    -   **openstack-gnocchi-api** — сервер приема API запросов на управляющем узле
    -   **openstack-gnocchi-metricd** — агент на управляющем узле, который в реальном времени вычисляет статистику приходящих данных и записывает их в файловую систему, а также индексирует эти данные и записывает их индексы в БД (у нас **MariaDB**), для быстрого доступа к ним.

Сами же [**метрики**](http://docs.openstack.org/admin-guide/telemetry.html) бывают трех типов (если работали с **Prometeus**, то узнаете этих троих как родных):

-   накопительные счетчики (**cumulative**) – постоянно увеличивающиеся со временем значения
-   индикаторы (**gauge**) – дискретные и плавающие значения, например ввод/вывод диска или присвоенные «плавающие IP»
-   дельта (**delta**) – изменение со временем, например пропускная способность сети

Грустно это признавать, но данный сервис (или сервисы) развернуть мне не удастся. Вы представляете 6 ГБ оперативки на управляющем узле уже не хватает (а ведь только в прошлом году я совершенно без проблем разворачивал **Kubernetes** на трех узлах), так что опять описание шагов, без собственного выполнения (может в новом году). Вот пруф плачевного состояния:

![](media/9978baf8a57487c58fc74b8a4b4ecdf9.png)

12.2. Приступаем к установке служб **Gnocchi**, **Ceilometer** и **Aodh** на управляющем (**controller.test.local**) узле (можно на разных).

Установим службы **Gnocchi**:

![](media/45d08af99a146be6f04944d17696d50e.png)

Проведем классическую регистрацию сервисов **Gnocchi** и **Ceilometer** в **Keystone**:

![](media/8c5133d9f3c1d8e3a7e02846daa85a52.png)

Создаем БД **gnocchi** для хранения индексов службы **Gnocchi**:

![](media/9488df25abb809738b88fd5dbc23d406.png)

Переходим к настройке файла конфига **/etc/gnocchi/gnocchi.conf** сервиса **Gnocchi**:

Указываем параметры авторизации в **Keystone**:

![](media/a4c76f528625021c77ff4045775ae517.png)

Указываем параметры подключения к БД:

![](media/7a113bde6db17080087c15eba3fabbd1.png)

Указываем место в файловой системе, где будут хранится данные метрик (можно хранить локально, можно например в **Ceph**, если крутой, но я локально):

![](media/a40ecabbbfaffae0facc622429015895.png)

Инициализируем **Gnocchi**:

![](media/eb0d49a91662b31afc643da7113a1204.png)

Запускаем и включаем все службы **Gnocchi**:

![](media/21d8112ea2e470c53cd73f1290602ddf.png)

Переходим к сервису **Ceilometer** и качаем его службы:

![](media/7cda923209c16324b264d18154381dbc.png)

Настраиваем файл конфига **/etc/ceilometer/ceilometer.conf** сервиса **Ceilometer**:

Указываем информацию по подключению к сервису **RabbitMQ**:

![](media/28b2cfbb3ec5091052af32e65cddd5d8.png)

Указываем реквизиты сервиса **Ceilometer**:

![](media/44bfb11fd292833f9c8da6cfacd3ec3e.png)

Инициализируем **Ceilometer**:

![](media/9ef1402fd18022e86f3867f55b4332cc.png)

Запускаем и включаем все службы **Ceilometer**:

![](media/ba0252c7be0f9cd5b247b58c11a512c1.png)

Теперь качаем службы для сервиса **Aodh** (**python-aodhclient** модуль для работы с **Python**):

![](media/b91195595b55cd33c05c992236ca3f05.png)

Создаем БД **aodh** для хранения информации о триггерах сервиса **Aodh**:

![](media/75bdea5c18f76eaf4114ae0a71975cb0.png)

Настраиваем файл конфига **/etc/aodh/aodh.conf** сервиса **Aodh**:

Регистрируем сервис **Aodh** в **Keystone**:

![](media/4a7ae125ab086d53ca1a6c830ff4bb52.png)

Указываем параметры авторизации в **Keystone**:

![](media/1f8df12dc8c271224b35c87e3f428ee0.png)

Теперь указываем реквизиты самого **Aodh**:

![](media/b05db11c395a6a0d4d44ea1d80addd90.png)

И параметры подключения к БД **aodh** и **RabbitMQ**:

![](media/f1463161b363dcf62ad89b2329d06266.png)

Синхронизируем БД **aodh** с сервисом:

![](media/9fe20cf6db359bae5dc74086a6d2d341.png)

Теперь можно запустить все службы:

![](media/4606d5e75f62187813b2041ed34a7299.png)

12.3. Переходим к установке службы **openstack-ceilometer-compute** на рабочих (**compute.test.local**) узлах (для всех узлов действия одинаковы):

Устанавливаем службу **openstack-ceilometer-compute**:

![](media/30d46c563a6e93a224146503c615862b.png)

Задаем параметры файлу конфига **/etc/ceilometer/ceilometer.conf** сервиса **Ceilometer**:

URL брокера сообщений **RabbitMQ**:

![](media/12f550a8bbd9f0f8e887e95aee85e748.png)

Указываем собственные реквизиты сервиса **Ceilometer**:

![](media/388ba6976a1a647866ccb76622632ea2.png)

Указываем в конфиге **/etc/nova/nova.conf** сервиса **Nova**, чтобы он начал отправлять сообщения через брокер сообщений:

![](media/ddd5be196dd60bedcb21f0029a09743a.png)

Настраиваем получение статистики по «**memory.usage**», «**disk.usage**» и «**disk.device.usage**»:

![](media/7992495f9a2bc1b657584d576065eedf.png)

Запускаем службу **openstack-ceilometer-compute** и перезапускаем службу **openstack-nova-compute**:

![](media/f9803c094ef708b77168e8e1b8fda9de.png)

12.4. Далее можно настраивать сервисы для отправки сообщений в сервис **Telemetry**. Я сделаю настройку сервисов **Glance** и **Cinder** (остальные сервисы [**тут**](https://docs.openstack.org/mitaka/install-guide-ubuntu/ceilometer.html)):

Настраиваем файлы конфигов служб (**/etc/glance/glance-api.conf** и **/etc/glance/glance-registry.conf**) **Glance**:

Настраиваем службу отправки сообщений для **Ceilometer** по AMQP и URL принимающего брокера сообщений (**RabbitMQ**) в конфиге (**/etc/glance/glance-api.conf**) **glance-api**:

![](media/3f500c2f56205a2223774f464740f311.png)

Настраиваем URL принимающего брокера сообщений (**RabbitMQ**) в конфиге (**/etc/glance/glance-registry.conf**) **glance-registry**:

![](media/aa325d14aef202e64113de446c4b4d89.png)

Перезапускаем настроенные службы **glance-api** и **glance-registry**:

![](media/5ce3e0627a730bf4b7579dd6a5db1c19.png)

Переходим к настройке конфига **/etc/cinder/cinder.conf** сервиса **Cinder** (если данный сервис находится на других узлах, то там его настройка аналогична):

Настраиваем службу отправки сообщений для **Ceilometer** по AMQP:

![](media/011272f810433b0a936ad62c79ee1fad.png)

Можно создать **cron-службу** (выполняет что-то периодически), которая будет раз в 5 минут собирать информацию о сервисе (в официальной документации такого пункта нет):

![](media/10d6d299fdd973c8eec438967d83889f.png)

Перезапускаем службы **Cinder**:

![](media/57ef992a292a4639035f120bc3a7b1d2.png)

Все, теперь должны пойти метрики.

12.5. Посмотреть метрики можно при помощи [**следующих команд**](#bookmark=id.1t3h5sf), а мы перейдем к созданию воображаемого триггера.

Перед созданием немного инфы о триггерах. Во-первых, триггер может быть в 3 состояниях: «**Ok**», «**Тревога**» и «**Недостаточно** **данных**». Во-вторых, правила триггеров могут объединяться с помощью **AND** и **OR**. В-третьих, к триггеру можно привязать два типа действий: отправление **POST-запроса** на какой-то URL или запись в файл журнала (только при отладке, в реальной жизни нет).

Теперь создаем триггер **myalarm1** для ВМ c id **35923211-ec9d-4f0a-8d5c-2cec96a427d8** (у вас может быть другой id), подробная информация по созданию [**тут**](https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/11/html/command-line_interface_reference/telemetry_data_collection_service_ceilometer_command_line_client#ceilometer_alarm_gnocchi_aggregation_by_metrics_threshold_update):

![](media/b7505c564da583eef05b27e7777ef77f.png)

-   **--type** — тип триггера (в нашем случае срабатывание по определенному порогу метрик)
-   **--name** — имя триггера
-   **--metric** — имя метрики
-   **--aggregation-method** — метод объединения значений метрик (в нашем случае берется среднее значение за период)
-   **--comparison-operator** — тип сравнения (в нашем случае **greater equal** — больше равно )
-   **--threshold** — сравниваемое значение
-   **--evaluation-periods** — количество периодов после возникновения события, которые нужно подождать перед совершением действия
-   **--granularity** — длительность периода в секундах
-   **--alarm-action** — действие, совершаемое триггером
-   **--resource-type** — тип ресурса openstack
-   **--query** — параметры передаваемые действию (в нашем случае в журнал запишется id ВМ)

С триггерами все переходим к последнему сервису (ну как так-то их еще минимум 5, но в гайде в ближайшее время их не будет).

13\. Установка и настройка сервиса оркестрации Heat

13.1. Сервис **Heat** отвечает за автоматизацию управления жизненными циклами наборов облачных сервисов, объединяя их в так называемые стеки (**stack**). С помощью него можно как развернуть ВМ, так и стартовать комплексное приложение из многих машин и масштабировать его в зависимости от информации, передаваемой модулем телеметрии. Для описания стеков используются специальные описания ресурсов, их ограничений, зависимостей и параметров:

-   **HOT** (Heat Orchestration Template) – формат, предназначенный исключительно для OpenStack. Представляет собой документ формата YAML (с ним и работаем)
-   **CFТ** (AWS CloudFormation) – документ формата JSON в формате, совместимом с шаблонами сервиса [**CloudFormation**](http://aws.amazon.com/ru/cloudformation/). Нужен, чтобы работать с уже существующими для AWS [**шаблонами**](http://../%E2%80%A2%20https://aws.amazon.com/cloudformation/aws-cloudformation-).

Службы **Heat**:

-   **openstack-heat-engine** – основной сервис, обеспечивающий обработку шаблонов и отправляющий события пользователям API
-   **openstack-heat-api** – сервис, отвечающий за предоставление основного REST API Heat. Взаимодействует с **openstack-heat-engine** через вызовы RPC
-   **openstack-heat-api-cfn** – аналогичен **openstack-heat-api**, но обеспечивает работу с API, совместимым с **AWS CloudFormation**.
-   **heat cli** – интерфейс взаимодействия с **Heat** API. Помимо командной строки, разработчики могут напрямую вызывать REST API или использовать веб-интерфейс **Horizon**.

13.2. Приступаем к воображаемой установке и настройке служб **Heat**:

Устанавливаем необходимые службы на узел управления (python2-heatclient модуль для взаимодействия **Python** с **Heat**):

![](media/699c1431059af48c0d79bf56a302c7e7.png)

Создаем БД **heat**:

![](media/d89fc008ec3ca547d93eaee4ee79644d.png)

Регистрируем **Heat** в сервисе **Keystone**:

![](media/375baa4317d03c0fc725d40ba261731e.png)

Уже что-то интересное, для **Heat** необходим отдельный домен **heat** для проектов и пользователей стека:

![](media/b030fdfbfafc76aec0bf907d3190ed13.png)

В новом домене heat создаем админа **heat_domain_admin**:

![](media/96d84e080c493d089b9e76f03b6e3cad.png)

Создадим роли владельца стека **heat_stack_owner** и пользователя стека **heat_stack_user**. Также добавим роль владельца стека нашему пользователю **demo**:

![](media/477a2c412a6c43d77009d51a74ebfa8e.png)

Закончим регистрацию в **Keystone**, добавив два сервиса: **heat** (обычный **heat**) и **heat-cfn** (**heat** для **CloudFormation**). Ну и добавим точки входа в добавленные сервисы:

![](media/0e801757410c735fe03b8092473d32dc.png)

Приступаем к редактированию конфигурационного файла **/etc/heat/heat.conf**:

Прописываем путь к БД **heat**:

![](media/f5ad867634beac269c7dd2720e4242d0.png)

Указываем URL брокера сообщений (**RabbitMQ**):

![](media/1d224b534aa544b0dd06f5b13b922acf.png)

Добавляем параметры авторизации в сервисе **Keystone**:

![](media/901500607c23bcd3ba3e24cafcbde20b.png)

Добавляем реквизиты авторизации доверенного лица (**trustee**) в Keystone:

![](media/f47f62e5026349b1375455f5f3d24f37.png)

Указываем, что в запросах клиентах должен быть заголовок **authenticate_uri:**

![](media/ce23a35c322e39fc095781f9d0081659.png)

Задаем URL сервера метаданных и сервера доступности ресурсов (показывает, когда можно создать ресурс, создание которого зависит от существования других ресурсов):

![](media/f4ea536c430b396f7f0ff419ff7cb7a5.png)

Указываем выделенный под запуск стеков домен и реквизиты администратора:

![](media/14f222111c8dbc9c5bd20346ae3677bd.png)

Синхронизируемся с БД **heat**:

![](media/fe60216ff35d7b00fabbbdbd815a7b72.png)

Запускаем службы **Heat**:

![](media/6d4c6dd9a5040212034cc8c42888ef6f.png)

Команда проверки работы сервисов — [**вот**](#bookmark=id.4d34og8).

13.3. Теперь можно создать первый стек, для этого на создать YAML-файл **test-server.yml** следующего содержания (cоздает стек, состоящий из одной виртуальной машины, которой во время старта передается скрипт, выводящий сообщение «Instance STARTED!» на стандартный вывод):

![](media/4d81c879ec3a006847e4e061290074e8.png)

-   **heat_template_version** — версия шаблона **Heat** (соответствует дате релиза актуальной версии Openstack, в файле версия устарела)
-   **description** — описание шаблона
-   **parameters** — определение переменных окружения (параметров), для дальнейшего использования
    -   **network** — параметр сети
        -   **type** — тип параметра (в данном случае строка)
        -   **description** — описание параметра
        -   **default** — дефолтное значение параметра, если через флаги команды создания стека не передано другое значение
    -   **image** — параметр образа
-   **resources** — описание ресурсов шаблона
    -   **my_server** — имя ресурса
        -   **type** - тип ресурса (в данном случае ВМ (**server**))
        -   **properties** — определение параметров ВМ my_server
            -   **flavor** — тип тома ВМ
            -   **key_name** — пара SSH ключей ВМ
            -   **networks** — массив сетей подключаемых к ВМ
                -   **network** — сеть подключенная к ВМ
                    -   **get_param** — получение переменных окружения (параметров)
            -   **image** — базовый образ ВМ
            -   **user_data** — действия выполняемые при запуске ВМ (в данном случае запуск скрипта)
            -   **user_data_format** — формат, в котором были записаны действия (в данном случае в виде строки)
-   **outputs** — параметры выводимые пользователю в **Horizon** или через API
    -   **instance_name** — название ВМ
        -   **description** — описание параметр
        -   **value** — значение параметра
            -   **get_attr** — получение значения атрибута ресурса (задается имя ресурса (**my_server**) и конкретный атрибут (**name**))
    -   **private_ip** — плавающий IP ВМ

Теперь можно создать стек используя созданный YAML-файл:

![](media/37105bbcd086c5d92ce28a69dfba25b2.png)

-   **--parameter** — передать значение параметра
-   **-t** — использовать файл шаблона

[**Данная команда**](#bookmark=id.3dy6vkm) показывает статус создания стека.

Также можно посмотреть создалась ли ВМ, а также информацию о ней:

![](media/b0718241cd9056575cdf9b4df59fa804.png)

Проверка срабатывания скрипта:

![](media/4424658b5bf5702adf86e559e4096049.png)

После чего можно посмотреть список ресурсов стека и потом его удалить:

![](media/aaf133b8a01ecbb678107310f179277d.png)

14\. Что происходит под капотом Neutron (бонусная глава)

14.1. Давайте выясним как у нас в кластере на самом деле создаются сети и почему на рабочих узлах достаточно одного сетевого интерфейса. Для начала разберемся, что из себя представляет **Open vSwitch** (виртуальный коммутатор), а также **сетевые пространства имен**:

14.2. **Сетевые пространства имен** являются одной из разновидностей **пространств имен Linux** (придуманы, чтобы изолировать друг от друга процессы), которые включают в себя:

-   **PID**, Process ID – изоляция иерархии процессов
-   **NET**, Networking – изоляция сетевых интерфейсов
-   **PC**, InterProcess Communication – управление взаимодействием между процессами
-   **MNT**, Mount – управление точками монтирования
-   **UTS**, Unix Timesharing System – изоляция ядра и идентификаторов версии

Так вводим три следующие команды (список **сетей** и **маршрутизаторов** **openstack**, а также список **сетевых пространств Linux**) и анализируем вывод;

![](media/b66c3112d85b6c3e55689a835836c7b3.png)

Что же мы видим, а именно, что служба **neutron-l3-agent** создает сетевые пространства для каждой сети (**qdhcp-id**) и маршрутизатора (**qrouter-id**) **Openstack** (id совпадают), но только для тех, к которым подключены ВМ напрямую (сетевого пространства **ext-net** нет). Также стоит отметить, что сетевые пространства автоматически не удаляются.

С сетевыми пространствами все.

14.3. Итак, **Open vSwitch**, данная технология состоит из следующих компонентов:

-   **Openswitch_mod.ko** — модуль ядра, отвечающий за работу с пакетами
-   **Ovs-vswitchd** — демон, отвечающий за управление, программирование логики пересылки пакетов, VLAN’ы и объединение сетевых карт
-   **Ovsdb-server** — сервер базы данных, отвечающий за ведение базы данных с конфигурацией
-   **Контроллер SDN** (Software Defined Network) — API сервер для работы с коммутатором
-   OpenFlow — протокол, при помощи которого **контроллер SDN** может удаленно контролировать таблицы потоков на коммутаторах и маршрутизаторах

В совокупности все это выглядит так:

![](media/5105f72b6d6a535fc9465059446c24ce.png)

Основные команды для работы с Open vSwitch:

-   **ovs-vsctl show** – вывод общей информации по коммутатору
-   **ovs-vsctl add-br/del-br** – добавить или удалить мост
-   **ovs-vsctl add-port/del-port** – добавить или удалить порт
-   **ovs-ofctl dump-flows** – вывод запрограммированных потоков для конкретного коммутатора (за их определение отвечает служба **neutron-openvswitch-agent**)
-   **ovsdb-tool show-log** – все команды настройки, отданные OVS, при помощи утилит пространства пользователя (с ее помощью **Neutron** работает с Open vSwitch)

Теперь сравним вывод списка портов **openstack** с выводом конфигурации **Open vSwitch** (на **controller** и **compute** узлах). Вывод **Open vSwitch** я немного подрезал:

![](media/c07b358b51a814fbf469903a3cf89d69.png)

![](media/267c46c3ce57c1a2aa31e4fbc7a460a1.png)

![](media/ed21fe7904c95d618a33efa843d8d906.png)

Тааак, в первую очередь можно заметить, что на узле управления у нас находятся три моста (**br-int**, **br-tun** и **br-ex**), а на рабочем узле, где запущена ВМ два моста (**br-int** и **br-tun**). Что из себя представляют данные мосты:

-   **Br-int** – интеграционный мост, предназначенный для подключения ВМ. Он осуществляет VLAN-тегирование трафика приходящего с/на вычислительные узлы. Находится как на вычислительных так и сетевых (у меня управляющем) узлах и создается автоматически при первом старте **neutron-openvswitch-agent**. На управляющем узле к нему подключены шесть портов:
    -   **tapef1fc61d-56** — порт моста к ВМ **myinstance1** (ip моста — **172.16.0.2**), для **testvm** нет моста, так как она в данный момент выключена
    -   **patch-tun** – соединение с мостом **tun**
    -   **qr-a3b98aca-13** – подключение к единственному маршрутизатору (ip маршрутизатора в подсети — **172.16.0.1**)
    -   **qg-106645c0-cd** — подключение к внешнему шлюзу маршрутизатора (ip внешнего шлюза в настройках маршрутизатора — **10.100.0.121**)
    -   **int-br-ex** – подключение к мосту **br-ex**
    -   **br-init** — собственный порт моста **init**
-   На рабочем узле есть четыре порта:
    -   **qvo87118d9b-84** — интерфейс ВМ **myinstance1** (локальный ip ВМ — **172.16.0.25**)
    -   **qvo9c0640c5-92** — интерфейс ВМ **testvm** (локальный ip ВМ — **172.16.0.72**)
    -   **patch-tun** — аналогично управляющему узлу соединение с мостом **tun**
    -   **br-int** — аналогично управляющему узлу порт моста **init**
-   **br-tun** – в нашем случае это GRE-туннель. Связывает сетевые и вычислительные узлы, передавая тегированный трафик с интеграционного моста, используя правила OpenFlow. Имеет следующие порты (на обоих узлах):
    -   **gre-c0a87ad7** — gre-туннель до узла **compute-opt**
    -   **gre-c0a87ac8** — gre-туннель до узла **controller**
    -   **gre-c0a87ad2 —** gre-туннель до узла **compute**
    -   **br-tun** — собственный порт моста **tun**
    -   **patch-int** — подключение к мосту **br-int**
-   **Br-ex** – мост, осуществляющий взаимодействие с внешним миром. Существует только на сетевых (управляющих) узлах. Содержит порты:
    -   **eth1** — физический сетевой интерфейс
    -   **phy-br-ex** — порт связи моста и физического сетевого интерфейса
    -   **br-ex** — собственный порт моста **ex**

Вот так можно посмотреть сетевые пространства имен нашего виртуального маршрутизатора:

![](media/d62d65ad9c61c49f5f40e348ab5dbe19.png)

14.4. Из-за того, что **Open vSwitch** не может работать с правилами iptables, которые применяются на виртуальный интерфейс, непосредственно подключенный к порту коммутатора, группы безопасности (набор настраиваемых разрешающих правил прохождения трафика, которые возможно назначать на порты) применяются к мосту **qbr-id** с помощью [**LinuxBridge**](https://kbespalov.medium.com/%D0%B2%D0%B8%D1%80%D1%82%D1%83%D0%B0%D0%BB%D1%8C%D0%BD%D1%8B%D0%B5-%D1%81%D0%B5%D1%82%D0%B5%D0%B2%D1%8B%D0%B5-%D1%83%D1%81%D1%82%D1%80%D0%BE%D0%B9%D1%81%D1%82%D0%B2%D0%B0-%D0%B2-linux-linux-bridge-7e0e887edd01) (что-то типо костыля), выведем все мосты рабочего узла:

![](media/e07b9f9f13eb8c06183acf2e22929021.png)

Тут видно три моста:

-   **qbr87118d9b-84** — мост к ВМ **testvm**
-   **qbr9c0640c5-92** — мост к ВМ **myinstace1**
-   **virbr0** — мост к хост машине

Посмотрим правила брандмауэра (на самом деле команды, которые автоматически ввела служба **neutron-openvswitch** при создании ВМ) для интерфейса **tap9c0640c5-92** (сетевой интерфейс ВМ **myinstace1**) с помощью [**iptables**](https://linux.die.net/man/8/iptables) (**-S** — все правила):

![](media/be7bb3730256a6da3fbbc859fa4276f8.png)

-   **-A** — добавить правило в цепочку с именем (например **neutron-openvswi-FORWARD**)
-   **-m** — имя дополнительного используемого модуля (**physdev** — модуль устройств ввода и вывода порта моста, **comment** — модуль добавления комментариев)
-   **--physdev-out** — имя порта моста, через который будет отправлен пакет
-   **--physdev-in** — имя порта моста, через который принимается пакет
-   **--physdev-is-bridged** — пакет передается по мосту и не маршрутизируется
-   \-**-comment** — содержимое комментария к правилу
-   **-j** - имя цели (цепочки) правила (что делать с пакетом, в нашем случае прейти к цепочке с таким-то именем). Вот вывод инфы о цепочке входящего в ВМ трафика:

![](media/46b457219adee47b8121f75e86a70371.png)

Тут видно нашу настройку приема **icmp** пакетов, **ssh** подключений, а также дефолтный прием всех **udp** пакетов на ip ВМ, ну и правила работы с разными тегами трафика (**RELATED** (пакет начинает новое соединение (например **FTP**), но связан с существующим соединением. **принимаем**), **ESTABLISHED** (пакет связан с соединением, которое видело пакеты в обоих направлениях. **принимаем**), **INVALID** (пакет связан с неизвестным соединением. **отбрасываем**) и **все остальное** (отправляет в цепочку **neutron-openvswi-sg-fallback**)).

Вот так должна выглядеть наша сеть (**qvb** - подключение в сторону моста **qbr**, **qvo** - подключение в сторону **OVS**):

![](media/505b6b9f438095ef54f5753cd25087da.png)

Ну вроде все единственное, что можно сказать это что у **openstack** есть служба openstack-neutron-fwaas (файерволл как сервис), которая помогает снизить нагрузку на сетевые интерфейсы ВМ, оправляя им уже проверенный трафик.

14.4. Для того чтобы красиво визуализировать сеть узла можно скачать специальную утилиту сканирования сети **plotnetcfg**, которая создаcт конфиг сети, после чего визуализировать этот конфиг в виде схемы при помощи утилиты **dot**, входящей в пакет **graphviz** (можно скачать на каждый узел, чтобы составить их схемы, но я скачаю только на узел **compute**).

Перед установкой пакетов надо опять [**включить**](#bookmark=id.tyjcwt) репозиторий epel, который мы вырубали. После чего надо скачать пакеты **plotnetcfg** и **graphviz**:

![](media/b907951b216a2702977c14dfa04716b5.png)

Теперь можно визуализировать сетевую конфигурацию узла **compute** (-T — тип файла, в который сохранится схема):

![](media/8504e133990566cfddf8570f28d809fe.png)

Скачиваем на хост машину файл и смотрим че вышло:

![](media/619d6346d7f4337e61cee725f30062ee.png)

Скрин части схемы (**eth1** мне было просто лень убрать, так-то его не должно быть на вычислительном узле):

![](media/4ff41ec616ae38d558290c7349f2495d.png)

14.5. Еще интересная тема так называемый мониторинг трафика ВМ. Попробуем проанализировать трафик нашей ВМ **myinstance1**:

Запускаем из ВМ поток пакетов ICMP на ip виртуального маршрутизатора (и не останавливаем):

![](media/36ae198019cbd8ea9d52e85975be3a01.png)

Теперь заходим на узел, на котором она поднялась, качаем [**tcpdump**](https://andreyex.ru/linux/komanda-tcpdump-v-linux/) (**-n** — начать сразу же, **-p** — выводить все в консоль, **-i** — имя интерфейса **-vvv** — очень детальный вывод, **-s** — размер снимка пакета, **-w** — записать в файл) и пытаемся перехватить трафик (интерфейсы **patch-tun** и **br-int**) ВМ (должны вылезти ошибки):

![](media/332fbd80b761a0dccf03624acc505ff3.png)

Почему же вылезают ошибки, а потому что внутренние устройства **Open vSwitch** невидимы для большинства утилит за пределами OVS, так как **Open vSwitch** не может работать с правилами **iptables**, которые применяются на виртуальный интерфейс, непосредственно подключенный к порту коммутатора (из-за этого все больше любят **Linux Bridge**, с которым таких проблем нет). Поэтому люди придумали костыль, а именно **dummy-интерфейс** (локальный интерфейс, тоже самое что и **loopback**), который будет зеркалировать весь трафик моста  **br-int**.

Создаем интерфейс **br-int-tcpdump** и поднимаем его:

![](media/cee9e4b0e505cd9ed25514b49b449f1e.png)

Добавляем созданный интерфейс **br-int-tcpdump** к мосту **br-int**:

![](media/283562bb22ad5d85bd89bb64f54b330c.png)

Он должен появится в конфиге OVS в секции моста **br-int** (управляющего узла):

![](media/103127b7362abb630f09f8c0a3c7e257.png)

Теперь настраиваем зеркалирование всего трафика с мста **br-int** на интерфейс **br-int-tcpdump** (эта сложная команда включает в себя 4 подкоманды, которые передают друг другу значения):

![](media/4a093735666fa99e2d5da01a2422038d.png)

-   **mirrors** — список зеркалирований интерфейса
-   **--id=@имя_переменной** — локальная переменная, в которую запишется вывод подкоманды
-   **name** — имя зеркалирования
-   **select-dst-port** — интерфейс, на который приходят пакеты
-   **select-src-port** — интерфейс, с которого приходят пакеты
-   **output-port** — интерфейс, на который пакеты будут зеркалироваться
-   **select_all** — все пакеты, если 1

Теперь трафик должен начать перехватываться (его можно записать в файл, скинуть на хост машину и проанализировать, например, в [**Wireshark**](https://habr.com/ru/post/204274/)):

![](media/742aadfb148e6be41c679fe1387d1ace.png)

14.6. Ну что ж последний пункт затрагивает такое понятие как балансировщик нагрузки как сервис (**LbaaS**). Вроде у **Openstack** уже есть свой сервис балансировки нагрузки по имени **Octavia**, но я последую книге и разверну [**HAProxy**](https://wiki.astralinux.ru/pages/viewpage.action?pageId=61573337) (подключаемый модуль **Neutron**). Балансировщик нагрузки отвечает за балансировку входящих сетевых подключений между экземплярами виртуальных машин, входящих в один кластер, чтобы не было такого, что одна ВМ задыхается от запросов, пока вторая не знает чем ей заняться. Далее в книге была показана конфигурация HAProxy, в ходе которой я выяснил, что его уже никто почти не использует, так как есть **Octavia**, так что пункт отменяется до момента настройки мной данного сервиса.

Пока можно сказать три вещи:

-   Есть три типа политики балансировки нагрузки:
    -   подключения принимаются по очереди всеми виртуальными машинами
    -   с одного IP-адреса подключения принимает одна и та же виртуальная машина
    -   подключение принимает машина с наименьшим числом подключений
-   Lbaas создает сетевые пространства имен для ВМ в так называемом пуле балансировки
-   Порядок организации балансировки нагрузки для нескольких ВМ:
    -   Создается специальный пул, в котором все ВМ будут делить запросы
    -   В пул добавляются ВМ по их ip
    -   Создается сервис мониторинга доступности ВМ
    -   Создается виртуальный ip пула
    -   Виртуальному ip пула добавляется плавающий ip, чтобы к кластеру можно было подключаться извне
    -   Таким образом запросы приходят на плавающий ip и распределяются согласно политике между участниками пула

Поздравляю господа, на этом минимальную настройку можно объявить законченной.

![](media/e2a97b3103efaa3ae04c1c9ba7a6ebbb.png)
