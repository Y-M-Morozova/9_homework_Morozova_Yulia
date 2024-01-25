<div align="center"><h2>Морозова Юлия, OTUS, группа Postgre-DBA-2023-11</h2></div>


<div align=center><h3>Домашнее задание №9 по теме: «Резервное копирование и восстановление»</h3></div>  

***

**<div align=center><h3>Цель: применить логический бэкап. Восстановиться из бэкапа.</h3></div>**

***

***Описание/Пошаговая инструкция выполнения домашнего задания:***
***<br>Создаем ВМ/докер c ПГ.
<br>Создаем БД, схему и в ней таблицу.
<br>Заполним таблицы автосгенерированными 100 записями.
<br>Под линукс пользователем Postgres создадим каталог для бэкапов.
<br>Сделаем логический бэкап используя утилиту COPY.
<br>Восстановим в 2 таблицу данные из бэкапа.
<br>Используя утилиту pg_dump создадим бэкап в кастомном сжатом формате двух таблиц.
<br>Используя утилиту pg_restore восстановим в новую БД только вторую таблицу!***

***

1. На моей ранее созданной ВМ в ЯО устанавливаю устанавливаю PostgreSQL 15 версии, подключаюсь к нему, для тестов создаю БД ``otus_backup``, в этой БД создаю схему ``test_backup`` :

    ![1](https://github.com/Y-M-Morozova/9_homework_Morozova_Yulia/assets/153178571/fa1b6f4a-9012-437d-a95d-4a5ba24f4b40)

    В этой схеме создаю таблицу скриптом:
   
    ```sql
        CREATE TABLE test_backup.document_template(
        ID INTEGER NOT NULL,
        NAME TEXT,
        SHORT_DESCRIPTION TEXT,
        AUTHOR TEXT,
        DESCRIPTION TEXT,
        CONTENT TEXT,
        LAST_UPDATED DATE,
        CREATED DATE);    
    ```
    
    И заполняю эту таблицу сгенерированными записями (100 строк) , для метода генерации случайных данных - использую ``random()`` и ``generate_series`` :

    ```sql
        INSERT INTO test_backup.document_template(id,name, short_description, author, description,content, last_updated,created)
        SELECT id, 'name', md5(random()::text), 'name2'
        ,md5(random()::text),md5(random()::text)
        ,NOW() - '1 day'::INTERVAL * (RANDOM()::int * 100)
        ,NOW() - '1 day'::INTERVAL * (RANDOM()::int * 100 + 100)
        FROM generate_series(1,100) id;
    ```

    Все ок:

    ![3](https://github.com/Y-M-Morozova/9_homework_Morozova_Yulia/assets/153178571/d2f9a0f4-07b4-409a-a8b9-567603d40e6d)

2. В целях повышения отказоустойчивости(хранить бекапы на дисковых системах отдельно от БД) и для красоты решения данного домашнего задания, принимаю решение, что каталог для бекапов я буду создавать на дополнительном диске, создам и подключу его к своей ВМ в ЯО , размечу и сделаю на нем файловую систему, и буду монтировать каталог бекапов на этот диск, так как я этого делала в домашнем задании по теме: «Физический уровень PostgreSQL».


   
    

    
