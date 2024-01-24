<div align="center"><h2>Морозова Юлия, OTUS, группа Postgre-DBA-2023-11</h2></div>


<div align=center><h3>Домашнее задание №9 по теме: «Резервное копирование и восстановление»</h3></div>  

***
<br/><br/>

**Цель: применить логический бэкап. Восстановиться из бэкапа.**

1. На моей ранее созданной ВМ в ЯО устанавливаю устанавливаю PostgreSQL 15 версии, подключаюсь к нему, для тестов создаю БД ``otus_backup``, в этой БД создаю схему ``test_backup``:

    ![1](https://github.com/Y-M-Morozova/9_homework_Morozova_Yulia/assets/153178571/fa1b6f4a-9012-437d-a95d-4a5ba24f4b40)







    ```sql
    alter system set log_lock_waits = on;
    alter system set deadlock_timeout = 200;
    ```
