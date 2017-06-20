### Некоторые условия

В примерах команд мы будем использовать пакет package-1.rpm, если явно
не указано иное. Если мы говорим, что пакет не установлен, значит он
располагается в текущем каталоге. В противном случае, следует указать
полный путь. Пакет может содержать скрипты, которые выполняются при
установке или удалении пакета.

### Простые манипуляции

  ---------------------------------------------------------------------------------------------------------
  Чтобы ...                                                          Нужно выполнить
  ------------------------------------------------------------------ --------------------------------------
  Импортировать GPG ключ для проверки пакетов                        1.  rpm --import RPM-GPG-KEY
                                                                     

  Посмотреть информацию о пакете, который лежит в текущем каталоге   \$ rpm -qip package-1.rpm

  Посмотреть список файлов из неустановленного пакета                \$ rpm -qlp package-1.rpm

  Посмотреть скрипты %pre- %post- install (uninstall)                \$ rpm -qp --scripts package-1.rpm

  Посмотреть changelog пакета                                        \$ rpm -qp --changelog package-1.rpm

  Установить пакет                                                   \$ rpm -ivh package-1.rpm

  Обновить или установить пакет                                      \$ rpm -Uvh package-1.rpm

  Обновить пакет, если его предыдущая версия установлена в системе   \$ rpm -Fvh package-1.rpm

  Узнать, какому пакету принадлежит данный файл                      \$ rpm -qf /etc/sysctl.conf

  Посмотреть информацию об уже установленном пакете                  \$ rpm -qi initscripts

  Посмотреть список всех установленных пакетов                       \$ rpm -qa

  Найти все пакеты, начинающиеся на gnome                            \$ rpm -qa "gnome\*"

  Удалить пакет                                                      \$ rpm -e package

  Посмотреть даты инсталляции пакетов                                \$ rpm -qa --last

  Посмотреть зависимости необходимые пакету                          \$ rpm -qR yum
  ---------------------------------------------------------------------------------------------------------

\

### Манипуляции посложнее

#### Использование ключа подсказки --aid

При инсталляции нового пакета иногда возникают ситуации, в которых
требуется добавить еще один или несколько пакетов для удовлетворения
зависимостей. К сожалению, по-умолчанию RPM не сообщает какой именно
пакет нужен, сообщает только имя библиотеки и версию. Дело в том, что
информация о все пакетах, входящих в официальный дистрибутив, содержится
отдельно - в специальном пакете rpmdb-\* (для версий дистрибутивов
RedHat это будет rpmdb-redhat-<дата-релиза>, для CentOS -
rpmdb-CentOS-<дата-релиза> и т.д.).

Например: rpmdb-CentOS-4.4-0.20060828

Т.е. необходимым условием является наличие этого пакета,
проинсталлированного в системе.

Проверяем:

    # rpm -ivh --aid samba-swat-3.0.9-1.3E.7.x86_64.rpm
    error: open of /var/spool/up2date/samba-3.0.9-1.3E.7.x86_64.rpm failed: No such file or directory
    error: Failed dependencies:
            samba = 0:3.0.9 is needed by samba-swat-3.0.9-1.3E.7

Здесь нам указали, что для инсталляции samba-swat потребуется
обязательный пакет samba с версией 0:3.0.9

Обратите внимение на строчку "**error: open of
/var/spool/up2date/samba-3.0.9-1.3E.7.x86\_64.rpm failed: No such file
or directory**". RPM сам попытался проинсталлировать пакет, но не нашел
его в каталоге */var/spool/up2date*. Если мы заглянем в конфигурационный
файл */etc/rpm/macros.solve* и исправим параметр %\_solve\_pkgsdir на
местоположение всех пакетов дистрибутива, то после повторного выполнения
команды, RPM найдет необходимый пакет и автоматически его
проинсталлирует.

Например, все пакеты дистрибутива у нас живут в каталоге
*/opt/linux/redhat/as30u7/x86-64*, который в свою очередь смонтирован по
NFS с сервера. Тогда в файле */etc/rpm/macros.solve* должно быть:

*%\_solve\_pkgsdir /opt/linux/redhat/as30u7/x86-64*

А вот результат выполнения команды:

    # rpm -ivh --aid samba-swat-3.0.9-1.3E.7.x86_64.rpm
    Preparing...                ########################################### [100%]
       1:samba                  ########################################### [ 50%]
       2:samba-swat             ########################################### [100%]

\
==== Чтобы добавить новые ключи, расширяющие возможности стандартного
вывода: ====

    $ cat >~/.popt 
    rpm     alias --arch --qf '%{NAME}  %{VERSION}-%{RELEASE} %{ARCH}\n' \
            --POPTdesc=$\"list package name, version, release and arch\"
    ^D

Посмотреть оригинальные запросы можно в /usr/lib/rpm/rpmpopt-\*.\
 Альтернативные форматы запросов можно помещать в /etc/popt или в
\~/.popt

------------------------------------------------------------------------

`` Чтобы увидеть результат выполним комнду: `rpm -q --arch postfix`  ``\
``  и сравним ее вывод с выводом этой команды: `rpm -q --arch kernel` ``

  ------------------------------------------------------------ ------------------------------------------------------------------------------------
  Посмотреть список полей                                      \$ rpm -q --querytags
  Вывод дополнительных полей                                   \$ rpm -q --qf '%{NAME}\\t%{VERSION}-%{RELEASE}.%{ARCH}\\t%{LICENSE}\\n' package-1
  Проверить целостность установленного пакета                  \$ rpm -V postfix
  Проверить целостность всех установленных пакетов в системе   \$ rpm -Va
  Откатить пакеты по состоянию на 1 мая                        \$ rpm -Uhv --rollback 'may 1'
  ------------------------------------------------------------ ------------------------------------------------------------------------------------

\
----

-   расшифровка результатов проверки целостности:\
     \`S.......\` отличаются размеры файлов\
     \`.M......\` отличаются флаги и права доступа\
     \`..5.....\` отличается контрольная сумма\
     \`...D....\` несоответствуют major/minor номер устройств\
     \`....L...\` readLink(2) path mismatch\
     \`.....U..\` отличаются владельцы\
     \`......G.\` отличаются группы-владельцы\
     \`.......T\` отличается время модификации\
     \` c\` - файл из секции %config, то есть явно конфигурационный
    файл\
     \` d\` - файл из секции %doc, аналогично - явно принадлежит к
    документации\
     \` g\` - файл-призрак, т.е. содержимое файла явно не существует в
    пакете (но возможно создается скриптами при инсталляции пакета)\
     \` l\` - файл из секции %license\
     \` r\` - файл из секции %readme\

### Сложные манипуляции

#### Как откатить систему пакетов по состоянию на определенное время

1\. Добавить в файл /etc/rpm/macros\
%\_repackage\_all\_erasures 1\
2. Для отката состояния использовать опцию --rollback\
rpm -Uhv --rollback '12 hours ago'

требуется \`%\_repackage\_all\_erasures 1\` в /etc/rpm/macros