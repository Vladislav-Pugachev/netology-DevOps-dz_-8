### 1.
> Системный вызов chdir
```buildoutcfg
root@vagrant:/home/vagrant# strace /bin/bash -c 'cd /tmp' 2>&1 | grep "/tmp"
execve("/bin/bash", ["/bin/bash", "-c", "cd /tmp"], 0x7ffd37ffebf0 /* 19 vars */) = 0
stat("/tmp", {st_mode=S_IFDIR|S_ISVTX|0777, st_size=4096, ...}) = 0
chdir("/tmp")                           = 0
```

### 2.
> Поиск осуществляется в базе данных /usr/share/misc/magic.mgc
```buildoutcfg
root@vagrant:/home/vagrant# strace -e openat file /dev/tty
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libmagic.so.1", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/liblzma.so.5", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libbz2.so.1.0", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libz.so.1", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libpthread.so.0", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/etc/magic.mgc", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/magic", O_RDONLY) = 3
openat(AT_FDCWD, "/usr/share/misc/magic.mgc", O_RDONLY) = 3
openat(AT_FDCWD, "/usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache", O_RDONLY) = 3
/dev/tty: character special (5/0)
+++ exited with 0 +++
```
### 3.
> Нужно выяснить какой файл без ссылки
```buildoutcfg
root@vagrant:/home/vagrant# lsof +L1
COMMAND    PID USER   FD   TYPE DEVICE SIZE/OFF NLINK   NODE NAME
script.py 1579 root    3w   REG  253,0        0     0 131085 /home/vagrant/test (deleted)
```
> Нужно выяснить какой номер дескриптора производящему запись в файл
```buildoutcfg
root@vagrant:/home/vagrant# ls -l /proc/1579/fd/
total 0
lrwx------ 1 root root 64 Oct 26 16:44 0 -> /dev/pts/0
lrwx------ 1 root root 64 Oct 26 16:44 1 -> /dev/pts/0
lrwx------ 1 root root 64 Oct 26 16:44 2 -> /dev/pts/0
l-wx------ 1 root root 64 Oct 26 16:44 3 -> '/home/vagrant/test (deleted)'
```
>Обнулить ссылку
```
> /proc/1579/fd/3
```

### 4.
> Опасность зомби процессов состоит в том, что они блокируют записи в таблице процессов, из за чего могут не создаться новые дочерние процессы.

### 5.

```buildoutcfg
root@vagrant:/home/vagrant# opensnoop-bpfcc
PID    COMM               FD ERR PATH
575    dbus-daemon        -1   2 /usr/local/share/dbus-1/system-services
575    dbus-daemon        18   0 /usr/share/dbus-1/system-services
816    vminfo              4   0 /var/run/utmp
575    dbus-daemon        -1   2 /lib/dbus-1/system-services
575    dbus-daemon        18   0 /var/lib/snapd/dbus-1/system-services/
588    irqbalance          6   0 /proc/interrupts
588    irqbalance          6   0 /proc/stat
588    irqbalance          6   0 /proc/irq/20/smp_affinity
588    irqbalance          6   0 /proc/irq/0/smp_affinity
588    irqbalance          6   0 /proc/irq/1/smp_affinity
588    irqbalance          6   0 /proc/irq/8/smp_affinity
588    irqbalance          6   0 /proc/irq/12/smp_affinity
588    irqbalance          6   0 /proc/irq/14/smp_affinity
588    irqbalance          6   0 /proc/irq/15/smp_affinity
```
### 6.
>системный вызов uname

```buildoutcfg
       Part of the utsname information is also accessible via /proc/sys/kernel/{ostype, hostname,  osrelease,  ver‐
       sion, domainname}.
```

### 7.

> В первом случае выполняться обе команды вне зависимости от того удачно ли завершиться первая команда
> Во втором случае вторая команда не выполнеться если проверка провалиться

> Я думаю что не имеет, так как логика скрипта может учитывать не выполнение какого то куска скрипта, к примеру
> Да и из консоли выкинет при неудачном выполнении команды

### 8
>даный набор ключей хорошо использовать при отладке скриптов, так как при неудачном  выполнении какой то части будут выведены переменные и команды в которых есть ошибки

### 9
> запущенный  или ожидающий запуска статус R, ключ + указыает на то что запущенные процессы находятся на переднем плане.
```buildoutcfg
root@vagrant:/home/vagrant# ps -o stat
STAT
S
S
S
R+
```