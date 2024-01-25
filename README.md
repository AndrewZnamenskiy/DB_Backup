### Задание 1. Резервное копирование

### Кейс
Финансовая компания решила увеличить надёжность работы баз данных и их резервного копирования. 

Необходимо описать, какие варианты резервного копирования подходят в случаях: 

1.1. Необходимо восстанавливать данные в полном объёме за предыдущий день.

1.2. Необходимо восстанавливать данные за час до предполагаемой поломки.

1.3.* Возможен ли кейс, когда при поломке базы происходило моментальное переключение на работающую или починенную базу данных.

*Приведите ответ в свободной форме.*

---

### Задание 2. PostgreSQL

2.1. С помощью официальной документации приведите пример команды резервирования данных и восстановления БД (pgdump/pgrestore).

2.1.* Возможно ли автоматизировать этот процесс? Если да, то как?

*Приведите ответ в свободной форме.*

---

### Задание 3. MySQL

3.1. С помощью официальной документации приведите пример команды инкрементного резервного копирования базы данных MySQL. 

3.1.* В каких случаях использование реплики будет давать преимущество по сравнению с обычным резервным копированием?

*Приведите ответ в свободной форме.*

----


### Решение 1

1.1. Необходимо восстанавливать данные в полном объёме за предыдущий день.

- Для решения это задачи предлагается выполнять полный бекап раз в неделю, а так же дифференциальный бекап раз в сутки.
  В этом случае, нам достаточно будет установки полного бекап начала недели, когда произошла авария и дифференциальный бекап
  сделанный в день, предшествующий аварии. 

1.2. Необходимо восстанавливать данные за час до предполагаемой поломки.

- Для решения это задачи предлагается выполнять полный бекап раз в неделю, дифференциальный бекап раз в сутки, а также
  инкрементный бекап раз в час. 
  В этом случае, нам достаточно будет установки полного бекап начала недели, когда произошла авария, дифференциальный бекап
  сделанный в день, предшествующий аварии и необходимое количество инкрементных бекап до времени, когда случилась авария.

1.3.* Возможен ли кейс, когда при поломке базы происходило моментальное переключение на работающую или починенную базу данных.

- В данном кейсе рассматриваю вариант с двумя нодами с установленными на них DRBD и Corosync + Pacemaker. 
  С помощью распределённого блочного устройсва (DRBD) мы осуществляем копирования раздела или диска с устновленной БД с одной ноды на другую.
  А с помощью утилиты кластеризации и менеджера ресурсов (Corosync+Pacemaker) монтируем раздел или диск на второй ноде и запускаем БД. 
  Таким образом база будет доступна с актуальными данными и под прежним IP кластера.


### Решение 2  PostgreSQL

2.1. С помощью официальной документации приведите пример команды резервирования данных и восстановления БД (pgdump/pgrestore).

- Примеры бекапирования  и восстановления pgsql
  Backup: 

```
	$ pg_dump mydb > db.sql       		# Выгрузка базы данных mydb в файл SQL-скрипта
	$ pg_dump -Fd mydb -f dumpdir 	 	# Выгрузка базы данных в формате каталога
	$ pg_dump -Fc mydb > db.dump    	# Выгрузка базы данных в специальном формате
	$ pg_dump -t mytab mydb > db.sql 	# Выгрузка отдельной таблицы mytab
```

  Restore: 

```
	$ pg_restore -d postgres --clean --create db.dump  	   # Восстановление архива в ту же базу, с предварительным удалением текущего содержимого
	$ pg_restore -d newdb db.dump				   # Восстановление из архива в чистую новую базу данных newdb
	$ pg_restore -d database_name -U username -C backup.dump   # Восстановление из архива с авторизацией и предварительым созданием БД (-C)
	$ psql -U username -d dbname < filename.sql		   # Восстановление из файла SQL-скрипта
```	
	
2.1.* Возможно ли автоматизировать этот процесс? Если да, то как?

- Резервирование можно автоматизировать написав Bash script с командой pg_dump и необходимыми
  параметрами, и добавив его на исполнение в планировщик linux - cron.


### Решение 3  MySQL

3.1. С помощью официальной документации приведите пример команды инкрементного резервного копирования базы данных MySQL.

* Для инкрементного бекапа требуется устаноки утилиты  mysqlbackup.

Для инкрементного бекапа выполянется следующая команда:

	mysqlbackup --defaults-file=/home/dbadmin/my.cnf \
 	 --incremental --incremental-base=history:last_backup \
	  --backup-dir=/home/dbadmin/temp_dir \
	  --backup-image=incremental_image1.bi \
 	  backup-to-image

Значение ключей:

	--defaults-file=/home/dbadmin/my.cnf 				  # указание на конфигурационный файл СУБД, в котором хранятся данные о размещении БД.	
	--incremental --incremental-base=history:last_backup  		  # указание места выполенния предыдущего бекапа для определения точки начала бекапа из метаданных.
	--backup-dir=/home/dbadmin/temp_dir 				  # директория для хранения метаданных бекапа и временных файлов.
	--backup-image=incremental_image1.bi				  # если не указан путь, то backup image соханится по backup-dir пути. 


3.1.* В каких случаях использование реплики будет давать преимущество по сравнению с обычным резервным копированием?

В случаях когда база высоко нагружена, мы можем считывать данные с Slave БД снижая нагрузку с основной БД. 
Кроме того можно осуществлять бекапирование с Slave БД. Также в случае отказа Master БД, роль Slave БД можно
оперативно сменить на Master сократив время простоя.
