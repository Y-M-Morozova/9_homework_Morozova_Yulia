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

    В этой схеме создаю таблицу ``test_backup.document_template`` скриптом:
   
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

   Смотрю содержимое таблицы селектом:

    ![4](https://github.com/Y-M-Morozova/9_homework_Morozova_Yulia/assets/153178571/15a1541e-34de-41a5-b970-f2dba27c79f2)


2. В целях повышения отказоустойчивости(хранить бекапы на дисковых системах отдельно от БД) и для красоты решения данного домашнего задания, принимаю решение, что каталог для бекапов я буду создавать на дополнительном диске, создам и подключу его к своей ВМ в ЯО , размечу и сделаю на нем файловую систему, и буду монтировать каталог бекапов на этот диск, так как я этого делала в домашнем задании по теме: «Физический уровень PostgreSQL»:

    Создаю диск ``disk1-otus-dz9`` размером 20 ГБ в ЯО и монтbрую его в ЯО к своей ВМ ``otus-db-pg-vm-1``:

    ![10_1](https://github.com/Y-M-Morozova/9_homework_Morozova_Yulia/assets/153178571/ef533fcf-964b-49f4-b42b-e4f328fbe8c9)

    Определяю новый диск в системе, создаю раздел, файловую систему, создаю каталог ``/mnt/backup_otus/``и монтирую в него новый диск, делаю владельцем этого каталога ``/mnt/backup_otus/`` пользователя и группу     пользователя ``postgres`` и все ок - каталог на внешнем диске для бекапов готов:

    ![10_5](https://github.com/Y-M-Morozova/9_homework_Morozova_Yulia/assets/153178571/4cc9c258-7102-4bb0-94a7-d618e1540a7d)

3. Делае логический бэкап таблицы ``test_backup.document_template`` , используя утилиту COPY, командой:

     ```sql
        \copy test_backup.document_template to '/mnt/backup_otus/backup_copy_test_backup.document_template.sql';
    ```  

    все ок, 100 строк скопировано:

    ![11_1](https://github.com/Y-M-Morozova/9_homework_Morozova_Yulia/assets/153178571/29cb508e-bfa3-42c2-863b-ec071647d96e)

    смотрю файл:

    ![11_2](https://github.com/Y-M-Morozova/9_homework_Morozova_Yulia/assets/153178571/cf15da56-fe46-4972-803e-874284829db5)

4. Для восстановления данных из файла ``'/mnt/backup_otus/backup_copy_test_backup.document_template.sql'`` в другую таблицу,

    я сначала создаю эту новую таблицу командой:

    ```sql
        CREATE TABLE test_backup.document_template_for_copy(
        ID INTEGER NOT NULL,
        NAME TEXT,
        SHORT_DESCRIPTION TEXT,
        AUTHOR TEXT,
        DESCRIPTION TEXT,
        CONTENT TEXT,
        LAST_UPDATED DATE,
        CREATED DATE
        );
    ```

    все ок:

    ![11_3](https://github.com/Y-M-Morozova/9_homework_Morozova_Yulia/assets/153178571/d3802fd0-e42a-4a5e-b62f-db806ee992db)


    а потом восстанавливаю данные командой:

    ```sql
        \copy test_backup.document_template_for_copy from '/mnt/backup_otus/backup_copy_test_backup.document_template.sql';
    ```

    100 строк скопировано:

    ![11_4](https://github.com/Y-M-Morozova/9_homework_Morozova_Yulia/assets/153178571/d59ec6a9-bc0d-4ea9-88f0-a1c3a8e2847a)

    проверяю селектом таблицу ``test_backup.document_template_for_copy`` :

    ![11_5](https://github.com/Y-M-Morozova/9_homework_Morozova_Yulia/assets/153178571/c134f65d-9149-4baa-88e4-ff5cf483a8da)

   <br/><br/>
   
6. Для выполнения задания по созданию бекапов с помощью ``pg_dump`` беру две таблицы , которые созданы в ДЗ пунктами выше в БД ``otus_backup``,схема ``test_backup``, это таблицы:  ``test_backup.document_template`` (в эту вставила еще 100 строк, итого 200 строк) и ``test_backup.document_template_for_copy``(в этой оставила 100 строк):

    ![17_1](https://github.com/Y-M-Morozova/9_homework_Morozova_Yulia/assets/153178571/74a010ad-2a38-4d39-ba2b-f03889691501)

7. Согласно заданию, используя утилиту ``pg_dump``, создаю бэкап в кастомном сжатом формате двух таблиц, командой:

    ``pg_dump -d otus_backup --compress=9 --table=test_backup.document_template --table=test_backup.document_template_for_copy -Fc > /mnt/backup_otus/backup_2_tables.gz``

    далее смотрю размер файла в каталоге , а потом  проверяю содержимое бекапа командой: 

    ``pg_restore --list /mnt/backup_otus/backup_2_tables.gz``
 
    все ок:
    
    ![17_2](https://github.com/Y-M-Morozova/9_homework_Morozova_Yulia/assets/153178571/cd88ac63-3b54-4ba3-bce2-b130f81f9211)

8. Так как ,согласно заданию, надо используя утилиту ``pg_restore`` восстановить в новую БД только одну таблицу, то сначала создаю новую БД(назову её ``otus_restore`` ), потом создаю в ней схему (``test_backup``):

   ![17_3](https://github.com/Y-M-Morozova/9_homework_Morozova_Yulia/assets/153178571/75725c29-9f3d-4b29-b3e0-1ba1b5db641a)

9. Восстанавливаю только одну таблицу по заданию, (беру ``test_backup.document_template_for_copy``, в которой 100 строк), команда для восстановления:

    ```sql
        pg_restore -d otus_restore --table=document_template_for_copy /mnt/backup_otus/backup_2_tables.gz
    ```
    
    подключаюсь к базе ``otus_restore``, проверяю таблицу ``test_backup.document_template_for_copy``, все ок, восстановилась, 100 строк:

   ![17_7](https://github.com/Y-M-Morozova/9_homework_Morozova_Yulia/assets/153178571/5dc8c147-030a-4a95-b4ca-d3e9e9c5f7f5)

***

**<div align=center><h3>Таким образом я в рамках этого домашнего задания тестирую два инструмента логического бекапа в postgres: команды COPY и PG_DUMP.</h3></div>**

***   


   


    
   

    

    

    


    


   



   

   


  
   
   

    




    

    


        

     

    


   

    



    

    
