# Лабораторная работа 1

## Цель работы:

Пользуясь терминалом на компьютере А перенести файл с компьютера Б на компьютер С, находящиеся в одной локальной сети.

## Шаги:

1.  Используя `ipconfig getifaddr en0` для MacOS и `ipconfig` для Windows (найти еще что-то линуксоподобное было невозможно, а поднимать виртуалку лень), выясняем локальные IP.

    | ПК  |     Это     |
    | --- | :---------: |
    | 1   | 192.168.3.9 |
    | 2   | 192.168.3.6 |
    | 3   | 192.168.3.5 |

    Проверим, действительно ли можно к каждому из них подключиться:

    ПК1 -> ПК2:

    ```
    ping 192.168.3.6
    PING 192.168.3.6 (192.168.3.6): 56 data bytes
    64 bytes from 192.168.3.6: icmp_seq=0 ttl=64 time=81.468 ms
    64 bytes from 192.168.3.6: icmp_seq=1 ttl=64 time=102.787 ms
    64 bytes from 192.168.3.6: icmp_seq=2 ttl=64 time=22.239 ms
    ```

    ПК1 -> ПК3

    ```
    ping 192.168.3.5
    PING 192.168.3.5 (192.168.3.5): 56 data bytes
    64 bytes from 192.168.3.5: icmp_seq=0 ttl=128 time=6.669 ms
    64 bytes from 192.168.3.5: icmp_seq=1 ttl=128 time=3.965 ms
    64 bytes from 192.168.3.5: icmp_seq=2 ttl=128 time=3.249 ms
    ```

2.  Для Mac было необходимо разрешить remote-подключение к диску. На ПК1 и ПК2 команда `sudo systemsetup -getremotelogin`, выдывала `off`. Из-за того, что ПК1 рабочий и, видимо, администратор отключил для моего user'a часть прав, попытка разрешить подключение выдавала следующее:

    ```
       sudo systemsetup -setremotelogin on
       setremotelogin: Turning Remote Login on or off requires Full Disk Access privileges.
    ```

    Пришлось через System Preferences -> Sharing -> Remote Login выдывать себе доступ.

    Для Windows все было не так удачно, ведь для MacOS ssh-сервер предустановлен изначально, и достаточно ввести одну команду/кликнуть галочку в удобном GUI, а при попытке подключению к ПК3 по SSH выдывало `connection refused`.

    Но используя [этот](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse?tabs=powershell) гайд, удалось поднять SSH сервер и на Windows

3.  Подключаемся с ПК3 к ПК2:
    `ssh 192.168.3.9@leshafatin`, вводим пароль и получаем доступ:

    ```
     Last login: Sun Sep 17 23:59:20 2023 from 192.168.3.9
     ➜ /Users/leshafatin
    ```

4.  На ПК2 создаем файлик, который будем передавать и передадим ПК1:

    ```
    ➜ mkdir lab1
    ➜ cd lab1
    ➜ echo Hello from PC3 > test.txt

    ➜ scp  ./test.txt a.fatin@192.168.3.9:media/

    Password:
     test.txt                                                          100%   15     2.2KB/s   00:00
    ```

5.  Проверим, дошел ли он до ПК1:

    ```
    ➜ ssh a.fatin@192.168.3.9
    (a.fatin@192.168.3.9) Password:
    Last login: Mon Sep 18 00:51:26 2023 from 192.168.3.6
    ➜ ~ cd media
    ➜ media ls
    test.txt

    ```

### Готово!

## Задание со звездочкой

1.  На ПК-3 (win) создаем пару ключей:

    ```
     ssh-keygen
     Generating public/private rsa key pair.
     Enter file in which to save the key (C:\Users\79634/.ssh/id_rsa):
     Enter passphrase (empty for no passphrase):
     Enter same passphrase again:
     Your identification has been saved in C:\Users\79634/.ssh/id_rsa
     Your public key has been saved in C:\Users\79634/.ssh/id_rsa.pub
     ...
    ```

2.  Передадим ключи на ПК2 (MacOS), к сожалению, не пользуясь удобной утилитой:

    ```
    C:\Users\79634>type C:\Users\79634\.ssh\id_rsa.pub | ssh leshafatin@192.168.3.6 "cat >> .ssh/authorized_keys"
    (leshafatin@192.168.3.6) Password:
    ```

3.  Создаем ключи на ПК2 и передаем на ПК1:

    ```
    leshafatin@Lesas-MacBook-Pro /Users/leshafatin
    ⚡️ ssh-keygen
    Generating public/private rsa key pair.
    Enter file in which to save the key (/Users/leshafatin/.ssh/id_rsa):
    /Users/leshafatin/.ssh/id_rsa already exists.
    Overwrite (y/n)? n

    leshafatin@Lesas-MacBook-Pro /Users/leshafatin
    ⚡️ ssh-copy-id -i ~/.ssh/id_rsa a.fatin@192.168.3.9
    zsh: command not found: sh-copy-id

    leshafatin@Lesas-MacBook-Pro /Users/leshafatin
    ⚡️ ssh-copy-id -i ~/.ssh/id_rsa a.fatin@192.168.3.9
    /usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/Users/leshafatin/.ssh/id_rsa.pub"
    /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
    /usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
    (a.fatin@192.168.3.9) Password:

    Number of key(s) added: 1

    Now try logging into the machine, with: "ssh 'a.fatin@192.168.3.9'"
    and check to make sure that only the key(s) you wanted were added.

    ```

4.  Создаем на ПК2 файлик и передаем на ПК3:

    ```
    leshafatin@Lesas-MacBook-Pro /Users/leshafatin

    ⚡️ echo Hello from PC3 by keys > test1.txt

    leshafatin@Lesas-MacBook-Pro /Users/leshafatin
    ⚡️ scp ./test1.txt a.fatin@192.168.3.9:media/
    Enter passphrase for key '/Users/leshafatin/.ssh/id_rsa':
    test1.txt 100% 23 4.1KB/s 00:00

    leshafatin@Lesas-MacBook-Pro /Users/leshafatin
    ⚡️ ssh a.fatin@192.168.3.9
    Enter passphrase for key '/Users/leshafatin/.ssh/id_rsa':
    Last login: Mon Sep 18 17:25:23 2023 from 192.168.3.6
    ➜ ~ cd media
    ➜ media cat test1.txt
    Hello from PC3 by keys

    ```

    Стоит отметить, что видимо на ПК1 уже был сгенерирован ssh ключ (по-видимому для GitHub), поэтому нужно было вводить passpharse

5.  Готово
