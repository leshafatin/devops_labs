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

    ➜ scp 22 ./test.txt a.fatin@192.168.3.9:media/

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
