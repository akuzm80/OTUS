#  Лабораторная работа 1.3.
## Развертывание коммутируемой сети с резервными каналами

----------

###  Цели:

[1. Создание сети и настройка основных параметров устройства;](#this-is-a-title)   
[2. Выбор корневого моста;](#this-is-a-title)  
[3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов;](#this-is-a-title)  
[4. Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов;](#this-is-a-title)  


###  Решение:

  


A [link](#this-is-a-title) to jump towards target header


###  1. Создание сети и настройка основных параметров устройства.

Настроить топологию сети и основные параметры маршрутизаторов  
    
__1.1 	Создайте сеть согласно топологии.__

- Подключите устройства, как показано в топологии, и подсоедините необходимые кабели. 

__1.2.	Выполните инициализацию и перезагрузку коммутаторов.__


__1.3.	Настройте базовые параметры каждого коммутатора.__

-	Отключите поиск DNS.  
no ip domain-lookup
-	Присвойте имена устройствам в соответствии с топологией.   
Hostname s1  
Hostname s2  
Hostname s3  
-	Назначьте class в качестве зашифрованного пароля доступа к привилегированному режиму.  
enable secret class
-	Назначьте cisco в качестве паролей консоли и VTY и активируйте вход для консоли и VTY каналов.  
line con 0  
password cisco  
login  
line vty 0 4  
password cisco  
login   
-	Настройте logging synchronous для консольного канала.  
line con 0
logging synchronous  
-	Настройте баннерное сообщение дня (MOTD) для предупреждения пользователей о запрете несанкционированного доступа.
banner motd "This is a secure system. Authorized Access Only!"
-	Задайте IP-адрес, указанный в таблице адресации для VLAN 1 на всех коммутаторах.  
Conf t  
Int vlan 1  
Ip address 192.168.1.x 255.255.255.0  
No sh
-	Скопируйте текущую конфигурацию в файл загрузочной конфигурации.  
copy running-config startup-config
 
__1.4.	Проверьте связь.__

Проверьте способность компьютеров обмениваться эхо-запросами.
  
- Успешно ли выполняется эхо-запрос от коммутатора S1 на коммутатор S2?	__да__  
- Успешно ли выполняется эхо-запрос от коммутатора S1 на коммутатор S3?	__да__  
- Успешно ли выполняется эхо-запрос от коммутатора S2 на коммутатор S3?	__да__  
Выполняйте отладку до тех пор, пока ответы на все вопросы не будут положительными.  


###  2. Выбор корневого моста.

Для каждого экземпляра протокола spanning-tree (коммутируемая сеть LAN или широковещательный домен) существует коммутатор, выделенный в качестве корневого моста. Корневой мост служит точкой привязки для всех расчётов протокола spanning-tree, позволяя определить избыточные пути, которые следует заблокировать.
Процесс выбора определяет, какой из коммутаторов станет корневым мостом. Коммутатор с наименьшим значением идентификатора моста (BID) становится корневым мостом. Идентификатор BID состоит из значения приоритета моста, расширенного идентификатора системы и MAC-адреса коммутатора. Значение приоритета может находиться в диапазоне от 0 до 65535 с шагом 4096. По умолчанию используется значение 32768.

__2.1. 	Отключите все порты на коммутаторах.__  
int range Et0/0-3  
shutdown  
__2.2.	Настройте подключенные порты в качестве транковых.__
switchport trunk encapsulation dot1q  
switchport mode trunk  
__2.3.	Включите порты Et0/1 и Et0/3 на всех коммутаторах.__  
int et0/1  
no sh    
__2.4.	Отобразите данные протокола spanning-tree.__  
Введите команду show spanning-tree на всех трех коммутаторах. Приоритет идентификатора моста рассчитывается путем сложения значений приоритета и расширенного идентификатора системы. Расширенным идентификатором системы всегда является номер сети VLAN. В примере ниже все три коммутатора имеют равные значения приоритета идентификатора моста (32769 = 32768 + 1, где приоритет по умолчанию = 32768, номер сети VLAN = 1); следовательно, коммутатор с самым низким значением MAC-адреса становится корневым мостом (в примере — S2).

__S1(config-if)#do show spanning-tree__

VLAN0001  
  Spanning tree enabled protocol ieee  
  Root ID    Priority    32769  
             Address     aabb.cc00.1000  
             This bridge is the root  
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec  

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)  
             Address     aabb.cc00.1000  
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec  
             Aging Time  15  sec  

Interface           Role Sts Cost      Prio.Nbr Type  
Et0/1               Desg FWD 100       128.2    Shr  
Et0/3               Desg FWD 100       128.4    Shr
  
__S2(config-if)#do show spanning-tree__

VLAN0001  
  Spanning tree enabled protocol ieee  
  Root ID    Priority    32769  
             Address     aabb.cc00.1000  
             Cost        100  
             Port        2 (Ethernet0/1)  
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec  

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)  
             Address     aabb.cc00.2000  
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec    
             Aging Time  300 sec    

Interface           Role Sts Cost      Prio.Nbr Type    
Et0/1               Root FWD 100       128.2    Shr  
Et0/3               Desg FWD 100       128.4    Shr  

__S3(config-if)#do show spanning-tree__

VLAN0001  
  Spanning tree enabled protocol ieee  
  Root ID    Priority    32769  
             Address     aabb.cc00.1000  
             Cost        100  
             Port        4 (Ethernet0/3)  
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec  

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)  
             Address     aabb.cc00.3000  
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec  
             Aging Time  15  sec  
  
Interface           Role Sts Cost      Prio.Nbr Type    
Et0/1               Altn BLK 100       128.2    Shr  
Et0/3               Root FWD 100       128.4    Shr  

__Примечание.__ Режим STP по умолчанию на коммутаторе 2960 — протокол STP для каждой сети VLAN (PVST).
В схему ниже запишите роль и состояние (Sts) активных портов на каждом коммутаторе в топологии.

![](Lab1.3.jpg)




С учетом выходных данных, поступающих с коммутаторов, ответьте на следующие вопросы.  
Какой коммутатор является корневым мостом?  
- S1  
Почему этот коммутатор был выбран протоколом spanning-tree в качестве корневого моста?  
- Имеет наименьший идентификатор BID_(Приоритет по-умолчанию + Vlan у всех равный, но S1 имеет меньший номер Mac-адреса, поэтому он стал Root)  
Какие порты на коммутаторе являются корневыми портами?   
- Направленные к корневому свитчу  
Какие порты на коммутаторе являются назначенными портами?  
- На корневом все порты, а на некорневых, ближайшие к корневому порту  
Какой порт отображается в качестве альтернативного и в настоящее время заблокирован?  
- Альтернативные порты ведут к корневому порту, но не являются корневыми на схеме это все порты в состоянии Altn BLK  
Почему протокол spanning-tree выбрал этот порт в качестве невыделенного (заблокированного) порта?  
- Он не является корневым или назначенным портом и чтобы не получилась петля остальные порты блокируются  


###  3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов.  

Алгоритм протокола spanning-tree (STA) использует корневой мост как точку привязки, после чего определяет, какие порты будут заблокированы, исходя из стоимости пути. Порт с более низкой стоимостью пути является предпочтительным. Если стоимости портов равны, процесс сравнивает BID. Если BID равны, для определения корневого моста используются приоритеты портов. Наиболее низкие значения являются предпочтительными. В части 3 вам предстоит изменить стоимость порта, чтобы определить, какой порт будет заблокирован протоколом spanning-tree. 

__3.1.	Определите коммутатор с заблокированным портом.__  
При текущей конфигурации только один коммутатор может содержать заблокированный протоколом STP порт. Выполните команду show spanning-tree на обоих коммутаторах некорневого моста. В примере ниже протокол spanning-tree блокирует порт F0/4 на коммутаторе с самым высоким идентификатором BID (S1).
 

__S1# show spanning-tree__ 

VLAN0001  
  Spanning tree enabled protocol ieee  
  Root ID    Priority    32769  
             Address     0cd9.96d2.4000  
             Cost        19  
             Port        2 (FastEthernet0/2)  
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec  

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)  
             Address     0cd9.96e8.8a00  
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec  
             Aging Time  300 sec  

Interface           Role Sts Cost      Prio.Nbr Type    
Fa0/2               Root FWD 19        128.2    P2p   
Fa0/4               Altn BLK 19        128.4    P2p  

__S3# show spanning-tree__

VLAN0001  
  Spanning tree enabled protocol ieee  
  Root ID    Priority    32769  
             Address     0cd9.96d2.4000  
             Cost        19  
             Port        2 (FastEthernet0/2)  
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec  

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)  
             Address     0cd9.96e8.7400  
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec  
             Aging Time  15  sec  

Interface           Role Sts Cost      Prio.Nbr Type    
Fa0/2               Root FWD 19        128.2    P2p   
Fa0/4               Desg FWD 19        128.4    P2p 
 
__Примечание.__ В конкретной топологии корневой мост может отличаться от выбора порта.  

__3.2.	Измените стоимость порта.__  
Помимо заблокированного порта, единственным активным портом на этом коммутаторе является порт, выделенный в качестве порта корневого моста. Уменьшите стоимость этого порта корневого моста до 18, выполнив команду spanning-tree cost 18 режима конфигурации интерфейса.  
S1(config)# interface f0/2  
S1(config-if)# spanning-tree cost 18

__3.3.	Просмотрите изменения протокола spanning-tree.__  
Повторно выполните команду show spanning-tree на обоих коммутаторах некорневого моста. Обратите внимание, что ранее заблокированный порт (S1 – F0/4) теперь является назначенным портом, и протокол spanning-tree теперь блокирует порт на другом коммутаторе некорневого моста (S3 – F0/4).  

__S1# show spanning-tree__  

VLAN0001  
  Spanning tree enabled protocol ieee  
  Root ID    Priority    32769  
             Address     0cd9.96d2.4000  
             Cost        18  
             Port        2 (FastEthernet0/2)  
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec  

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)  
             Address     0cd9.96e8.8a00  
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec  
             Aging Time  300 sec  

Interface           Role Sts Cost      Prio.Nbr Type    
Fa0/2               Root FWD 18        128.2    P2p   
Fa0/4               Desg FWD 19        128.4    P2p  

__S3# show spanning-tree__   

VLAN0001  
  Spanning tree enabled protocol ieee  
  Root ID    Priority    32769  
             Address     0cd9.96d2.4000  
             Cost        19  
             Port        2 (FastEthernet0/2)  
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec  

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)  
             Address     0cd9.96e8.7400  
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec  
             Aging Time  300 sec  

Interface           Role Sts Cost      Prio.Nbr Type    
Fa0/2               Root FWD 19        128.2    P2p   
Fa0/4               Altn BLK 19        128.4    P2p  
Почему протокол spanning-tree заменяет ранее заблокированный порт на назначенный порт и блокирует порт, который был назначенным портом на другом коммутаторе?
  
- Стоимость порта по направлению к корневому порту стала ниже, то есть предпочтительней и протокол переключил интефейс на него как на более приоритетный

__3.4.	Удалите изменения стоимости порта.__  
-	Выполните команду no spanning-tree cost 18 режима конфигурации интерфейса, чтобы удалить запись стоимости, созданную ранее.  
S1(config)# interface f0/2  
S1(config-if)# no spanning-tree cost 18  
-	Повторно выполните команду show spanning-tree, чтобы подтвердить, что протокол STP сбросил порт на коммутаторе некорневого моста, вернув исходные настройки порта. Протоколу STP требуется примерно 30 секунд, чтобы завершить процесс перевода порта.


###  4.	Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов.

Если стоимости портов равны, процесс сравнивает BID. Если BID равны, для определения корневого моста используются приоритеты портов. Значение приоритета по умолчанию — 128. STP объединяет приоритет порта с номером порта, чтобы разорвать связи. Наиболее низкие значения являются предпочтительными. В части 4 вам предстоит активировать избыточные пути до каждого из коммутаторов, чтобы просмотреть, каким образом протокол STP выбирает порт с учетом приоритета портов.  
-	Включите порты F0/1 и F0/3 на всех коммутаторах.  
-	Подождите 30 секунд, чтобы протокол STP завершил процесс перевода порта, после чего выполните команду show spanning-tree на коммутаторах некорневого моста. Обратите внимание, что порт корневого моста переместился на порт с меньшим номером, связанный с коммутатором корневого моста, и заблокировал предыдущий порт корневого моста.  

S1# show spanning-tree

VLAN0001  
  Spanning tree enabled protocol ieee  
  Root ID    Priority    32769  
             Address     0cd9.96d2.4000  
             Cost        19  
             Port        1 (FastEthernet0/1)  
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec  

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)  
             Address     0cd9.96e8.8a00  
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec  
             Aging Time  15  sec  

Interface           Role Sts Cost      Prio.Nbr Type  
Fa0/1               Root FWD 19        128.1    P2p   
Fa0/2               Altn BLK 19        128.2    P2p   
Fa0/3               Altn BLK 19        128.3    P2p   
Fa0/4               Altn BLK 19        128.4    P2p  

S3# show spanning-tree  

VLAN0001  
  Spanning tree enabled protocol ieee  
  Root ID    Priority    32769  
             Address     0cd9.96d2.4000  
             Cost        19  
             Port        1 (FastEthernet0/1)  
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec  

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)  
             Address     0cd9.96e8.7400  
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec  
             Aging Time  15  sec  

Interface           Role Sts Cost      Prio.Nbr Type   
Fa0/1               Root FWD 19        128.1    P2p   
Fa0/2               Altn BLK 19        128.2    P2p   
Fa0/3               Desg FWD 19        128.3    P2p   
Fa0/4               Desg FWD 19        128.4    P2p  

Какой порт выбран протоколом STP в качестве порта корневого моста на каждом коммутаторе некорневого моста?   
- Fa0/1 на обеих  
Почему протокол STP выбрал эти порты в качестве портов корневого моста на этих коммутаторах?  
- т.к. по приоритету наиболее низкие значения у Fa0/1  

__Вопросы для повторения__  
1.	Какое значение протокол STP использует первым после выбора корневого моста, чтобы определить выбор порта?  
- Стоимость портов  
2.	Если первое значение на двух портах одинаково, какое следующее значение будет использовать протокол STP при выборе порта?  
- Bridge ID  
3.	Если оба значения на двух портах равны, каким будет следующее значение, которое использует протокол STP при выборе порта?  
- Приоритеты портов (по умолчанию 128)  

__Листинги конфигурации.__  
Все файлы конфигураций приведены [здесь](https://github.com/akuzm80/OTUS/tree/master/LAB1.3/Config).
# This is a title