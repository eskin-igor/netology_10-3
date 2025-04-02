# netology_10-3
# Домашнее задание к занятию «Резервное копирование»

## Задание 1
* Составьте команду rsync, которая позволяет создавать зеркальную копию домашней директории пользователя в директорию /tmp/backup
* Необходимо исключить из синхронизации все директории, начинающиеся с точки (скрытые)
* Необходимо сделать так, чтобы rsync подсчитывал хэш-суммы для всех файлов, даже если их время модификации и размер идентичны в источнике и приемнике.
* На проверку направить скриншот с командой и результатом ее выполнения

#3 Решение 1

Составленная команда.
```
rsync -a --checksum --verbose --delete --progress --exclude '.*' /home/eskin/ /tmp/backup
```
Демонстрация выполнения команды.

![](https://github.com/eskin-igor/netology_10-3/blob/main/10-3/10-3-1-1.PNG)

Содержимое домашнего каталога /home/eskin/.

![](https://github.com/eskin-igor/netology_10-3/blob/main/10-3/10-3-1-2.PNG)

Содержимое каталога /tmp/backup/ после выполнения команды.

![](https://github.com/eskin-igor/netology_10-3/blob/main/10-3/10-3-1-3.PNG)

## Задание 2
* Написать скрипт и настроить задачу на регулярное резервное копирование домашней директории пользователя с помощью rsync и cron.
* Резервная копия должна быть полностью зеркальной
* Резервная копия должна создаваться раз в день, в системном логе должна появляться запись об успешном или неуспешном выполнении операции
* Резервная копия размещается локально, в директории /tmp/backup
* На проверку направить файл crontab и скриншот с результатом работы утилиты.

## Решение 2

Скрипт резервного копирования [backup_rsync.sh.](https://github.com/eskin-igor/netology_10-3/blob/main/10-3/backup_rsync.sh)
```
#!/bin/bash
copy=$1
paste=$2

if [ ! -d $copy ]; then
	echo -e "\033[31m----- | $(date +"%Y-%m-%d | %H:%M:%S") | Backup completed Unsuccessfully! | Source directory does not exist. | -----\033[0m" >> /var/log/backup_rsync-$(date +"%Y-%m-%d").log
elif [ ! -d $paste ]; then
	echo -e "\033[31m----- | $(date +"%Y-%m-%d | %H:%M:%S") | Backup completed Unsuccessfully! | Destination directory does not exist. | -----\033[0m" >> /var/log/backup_rsync-$(date +"%Y-%m-%d").log
else
	rsync -a --checksum --verbose --delete --progress --exclude '.*' $copy $paste >> /var/log/backup_rsync-$(date +"%Y-%m-%d").log
  echo -e "\033[32m----- | $(date +"%Y-%m-%d | %H:%M:%S") | Backup completed Successfully! | -----\033[0m" >> /var/log/backup_rsync-$(date +"%Y-%m-%d").log
fi

```
Скрипт содержит:  
* проверку наличия директории с источником данных, с выводом результата проверки в лог;  
* проверку наличия директории назначения, с выводом результата проверки в лог;  
* команду резервного копирования, с выводом результата копирова в лог.

Задача [crontab.](https://github.com/eskin-igor/netology_10-3/blob/main/10-3/crontab.txt)
``` 
# m h  dom mon dow   command
57 22 * * * /bin/bash /home/eskin/backup_rsync.sh /home/eskin/ /tmp/backup
```
Содержимое домашнего каталога /home/eskin/.
![](https://github.com/eskin-igor/netology_10-3/blob/main/10-3/10-3-2-1.PNG)

Содержимое каталога /tmp/backup/ после выполнения команды.
![](https://github.com/eskin-igor/netology_10-3/blob/main/10-3/10-3-2-2.PNG)

Содержимое [backup_rsync-2025-04-01.log] в результате выполнения задачи crontab скрипта backup_rsync.sh.
```
sending incremental file list
./
backup_rsync.sh

            762 100%    0.00kB/s    0:00:00  
            762 100%    0.00kB/s    0:00:00 (xfr#1, to-chk=9/11)
index.html

             64 100%   62.50kB/s    0:00:00  
             64 100%   62.50kB/s    0:00:00 (xfr#2, to-chk=8/11)
Desktop/
Documents/
Downloads/
Music/
Pictures/
Public/
Templates/
Videos/

sent 1,323 bytes  received 93 bytes  2,832.00 bytes/sec
total size is 826  speedup is 0.58
␛[32m----- | 2025-04-01 | 22:57:01 | Backup completed Successfully! | -----␛[0m
``` 
