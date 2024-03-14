
Для доступа к серверу используйте ssh (креды в асане)

Для перезапуска сервера

    systemctl stop engine && systemctl start engine

Логи лежат в папке проэкта

    cd ~/engine_v_2
  
  Файлы логов
  

    events.log
    details.log
    debug_full.log
    all.log
    console.log
    project_details.log
    project_events.log
    system_details.log
    system_events.log
    sql_full.log
    ./fb/logs.log
   
   Скрипты для запуска некоторых процессов

   ~/engine_v_2/start.sh
   ~/fb.sh
   ~/move.sh
   ~/update.sh
   ~/daily.sh

Крон

    0 0 * * * sudo apt update && sudo apt upgrade  
    0 8 * * * /bin/bash /root/daily.sh  
    */15 8-22 * * * /bin/bash /root/move.sh  
    * 8-21 * * * /bin/bash /root/fb.sh

Конфиги

   

    ~/engine_v_2/prod.cfex

 
 Токен FB
 

    ~/engine_v_2/tokens.json

 
