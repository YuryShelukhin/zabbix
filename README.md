# Домашнее задание к занятию «Zabbix» Шелухин Юрий

### Задание 1

Установите Zabbix Server с веб-интерфейсом.

Процесс выполнения:
1. Выполняя ДЗ, сверяйтесь с процессом отражённым в записи лекции.
2. Установите PostgreSQL. Для установки достаточна та версия, что есть в системном репозитороии Debian 11.
3. Пользуясь конфигуратором команд с официального сайта, составьте набор команд для установки последней версии Zabbix с поддержкой PostgreSQL и Apache.
4. Выполните все необходимые команды для установки Zabbix Server и Zabbix Web Server.  

Требования к результатам:
1. Прикрепите в файл README.md скриншот авторизации в админке.
2. Приложите в файл README.md текст использованных команд в GitHub


## Решение 1

### Для решения заданий использованы 2-е виртуальные машины в Яндекс-облаке:
  1. ВМ  "monitor1" с установленными Zabbix-сервером и  Zabbix-агентом  (ip 158.160.184.132);    
  2. ВМ  "monitor2" с установленным Zabbix-агентом  (ip 158.160.168.224).  
  <img src = "img/1-0.png" width = 60%>   
   
1.  Настроим ВМ  "monitor1"  
Установим PostgreSQL:  
sudo apt install postgresql   
sudo apt update  
Установим репозиторий Zabbix. Перечень команд возьмем на сайте https://www.zabbix.com/ru/download?zabbix=6.0&os_distribution=debian&os_version=11&components=server_frontend_agent&db=pgsql&ws=apache исходя из конфигурации нашей ВМ.  
sudo wget https://repo.zabbix.com/zabbix/6.0/debian/pool/main/z/zabbix-release/zabbix-release_latest_6.0+debian11_all.deb
sudo dpkg -i zabbix-release_latest_6.0+debian11_all.deb  
sudo apt update  
Установим Zabbix сервер, веб-интерфейс и агент.    
apt install zabbix-server-pgsql zabbix-frontend-php php7.4-pgsql zabbix-apache-conf zabbix-sql-scripts zabbix-agent  
Проверим.   
sudo systemctl status zabbix-server.service  
<img src = "img/1-1.png" width = 60%>  
Установлен, но не запущен.    
Создадим базу данных.         
Создадим пользователя zabbix.           
sudo -u postgres createuser --pwprompt zabbix    
Создадим базу zabbix для пользователя zabbix.      
sudo -u postgres createdb -O zabbix zabbix   
На хосте Zabbix сервера импортируем начальную схему и данные.     
zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix    
Отредактируем файл /etc/zabbix/zabbix_server.conf.      
sudo vim /etc/zabbix/zabbix_server.conf    
<img src = "img/1-2.png" width = 60%>        
Запустим процессы Zabbix сервера и агента, настроим их запуск при загрузке ОС.      
systemctl restart zabbix-server zabbix-agent apache2      
systemctl enable zabbix-server zabbix-agent apache2    
<img src = "img/1-3.png" width = 60%>     
<img src = "img/1-4.png" width = 60%>    
Авторизуемся по адресу http://158.160.184.132/zabbix/  (Admin/zabbix). 
<img src = "img/1-5-0.png" width = 60%> 
 проверим, все параметры в статусе - ок  
<img src = "img/1-5.png" width = 60%>     
<img src = "img/1-6.png" width = 60%>   

---

### Задание 2

Установите Zabbix Agent на два хоста.

Процесс выполнения:
1. Выполняя ДЗ, сверяйтесь с процессом отражённым в записи лекции.
2. Установите Zabbix Agent на 2 вирт.машины, одной из них может быть ваш Zabbix Server.
3. Выполняя ДЗ, сверяйтесь с процессом отражённым в записи лекции.
4. Добавьте Zabbix Server в список  разрешенных серверов ваших Zabbix Agentов.
5. Добавьте Zabbix Agentов в раздел Configuration > Hosts вашего Zabbix Servera.
6. Проверьте, что в разделе Latest Data начали появляться данные с добавленных агентов.  

Требования к результатам:
1. Приложите в файл README.md скриншот раздела Configuration > Hosts, где видно, что агенты подключены к серверу
2. Приложите в файл README.md скриншот лога zabbix agent, где видно, что он работает с сервером
3. Приложите в файл README.md скриншот раздела Monitoring > Latest data для обоих хостов, где видны поступающие от агентов данные.
4. Приложите в файл README.md текст использованных команд в GitHub  

## Решение 2  

1-3. В задании №1 был установлен Zabbix-агент на ВМ "monitor1" одновременно с установкой Zabbix-сервера.    
Установим Zabbix-агент на  ВМ  "monitor2".  
sudo apt update  
sudo  apt install zabbix-agent  
Запустим процесс Zabbix-агент и настроим его запуск при загрузке ОС.  
systemctl restart zabbix-agent   
systemc status zabbix-agent   
<img src = "img/2-1.png" width = 60%>    
Отредактируем конфигурационный файл  ВМ  "monitor2".      
sudo find / -name zabbix_agentd.conf  
sudo vim...  
<img src = "img/2-6.png" width = 60%>   
Перезапустим и проверим.   
sudo systemctl status zabbix-agent.service  
Проверим конфигурационный файл  ВМ  "monitor1".      
sudo find / -name zabbix_agentd.conf  
sudo vim...  
В этом конфигурационном файле оставим адрес по умолчанию (127.0.0.1), так как zabbix-агент установлен на одной машине с zabbix-сервером.  
<img src = "img/2-6-1.png" width = 60%>  
Добавим хосты zabbix-агентов в раздел Configuration > Hosts вашего Zabbix Servera.   
<img src = "img/2-2.png" width = 60%>    
<img src = "img/2-3.png" width = 60%>   
сохраним и вновь зайдем на хосты для начала наблюдения путем добавления метрик  
выберем стандартный шаблон для обеих ВМ 
<img src = "img/2-4.png" width = 60%>   
<img src = "img/2-5.png" width = 60%>     
Проверим данные с агентов в разделе Latest Data и в Dashbord.    
<img src = "img/2-7.png" width = 60%>     
<img src = "img/2-8.png" width = 60%>  
проверим логи  
sudo find / -name zabbix_agentd.log  
<img src = "img/2-11.png" width = 60%>  

---

## Задание 3 со звёздочкой*

Установите Zabbix Agent на Windows (компьютер) и подключите его к серверу Zabbix.

Требования к результатам
1. Приложите в файл README.md скриншот раздела Latest Data, где видно свободное место на диске C:
