# RIP Homework
##### Новиков Василий. КН-202. Вариант 18.

### Задание

Собрать сеть и запустить в ней протокол OSPF. В сети должно быть две области (area): Area 0 и Area 1. Изучить типы LSA и строение баз данных маршрутизаторов.

### Решение

1. Настраиваем интерфейсы.
    ```
    R1AR0:
        R1AR0(config)#interface FastEthernet0/0
        R1AR0(config-if)#ip address 10.2.2.1 255.255.255.0
        R1AR0(config-if)#no shutdown
        	
        R1AR0(config)#interface Serial2/0
        R1AR0(config-if)#ip address 10.1.1.1 255.255.255.0
        R1AR0(config-if)#no shutdown
    R2AR0:
    	R2AR0(config)#interface FastEthernet0/0
        R2AR0(config-if)#ip address 10.2.2.2 255.255.255.0
        R2AR0(config-if)#no shutdown
        
        R2AR0(config)#interface Serial2/0
        R2AR0(config-if)#ip address 10.3.3.1 255.255.255.0
        R2AR0(config-if)#no shutdown
        
        R2AR0(config)#interface loopback 0
        R2AR0(config-if)#ip address 10.28.0.1 255.255.255.255
        	
        R2AR0(config)#interface loopback 1
        R2AR0(config-if)#ip address 10.28.1.1 255.255.255.255
        	
        R2AR0(config)#interface loopback 2
        R2AR0(config-if)#ip address 10.28.2.1 255.255.255.255
        
        R2AR0(config)#int loopback 3
        R2AR0(config-if)#ip address 10.28.3.1 255.255.255.255
        
    ABR:
        ABR(config)#interface Serial2/0
        ABR(config-if)#ip address 10.1.1.2 255.255.255.0
        ABR(config-if)#no shutdown
        
        ABR(config)#interface Serial3/0
        ABR(config-if)#ip address 192.168.1.1 255.255.255.0
        ABR(config-if)#no shutdown
    ```
2. Запускаем OSPF на роутерах.
    ```
    R1AR0:
        R1AR0(config)#router ospf 1
        R1AR0(config-router)#router-id 1.1.1.1
        R1AR0(config-router)#network 10.1.1.0 0.0.0.255 area 0
        R1AR0(config-router)#network 10.2.2.0 0.0.0.255 area 0
        
    R2AR0:
        R2AR0(config)#router ospf 1
        R2AR0(config-router)#router-id 2.2.2.2
        R2AR0(config-router)#network 10.2.2.0 0.0.0.255 area 0
        R2AR0(config-router)#network 10.28.0.0 0.0.255.255 area 0
        
    ABR:
        ABR(config)#router ospf 1
        ABR(config-router)#router-id 1.1.2.2
        ABR(config-router)#network 10.0.0.0 0.255.255.255 area 0
    ```
3. Проверяем, что в области 0 все настроено.
![N|Solid](https://i.imgur.com/sLJaV48.png)
![N|Solid](https://i.imgur.com/sJkHdEZ.png)
![N|Solid](https://i.imgur.com/Xu10JtU.png)
![N|Solid](https://i.imgur.com/Ox8YWOx.png)
![N|Solid](https://i.imgur.com/vQ9ZhzE.png)
![N|Solid](https://i.imgur.com/vw1zYyD.png)

4. Настраиваем интерфейсы в области 1.
    ```
    R1AR1:
        R1AR1(config)#interface Serial2/0
        R1AR1(config-if)#ip address 192.168.1.2 255.255.255.0
        R1AR1(config-if)#no shutdown
        R1AR1(config-if)#exit
        
        R1AR1(config)#interface Serial3/0
        R1AR1(config-if)#ip address 192.168.2.1 255.255.255.0
        R1AR1(config-if)#no shutdown
    
    ASBR:
        ASBR(config)#interface Serial2/0
        ASBR(config-if)#ip address 192.168.2.2 255.255.255.0
        ASBR(config-if)#no shutdown
    ```
5. Настраиваем ospf в области 1.
    ```
    R1AR1:
        R1AR1(config)#router ospf 1
        R1AR1(config-router)#router-id 3.3.3.3
        R1AR1(config-router)#network 192.168.0.0 0.0.255.255 area 1
        
    ASBR:
        ASBR(config)#router ospf 1
        ASBR(config-router)#router-id 4.4.4.4
        ASBR(config-router)#network 192.168.2.0 0.0.0.255 area 1
        
    ABR:
        ABR(config)#router ospf 1
        ABR(config-router)#network 192.168.1.0 0.0.0.255 area 1
    ```
6. Убеждаемся, что работает. (В таблице маршрутизации для роутеров из 0 области появились маршруты до роутеров из 1 области)
![N|Solid](https://i.imgur.com/NFEJJLU.png)

7. Подключаем к области 0 роутер, на котором организуем loopback интерфейсы. Настраиваем на этом роутере работу протокола RIP. На соединяющем роутере запускаем RIP, синхронизируя его с уже настроенным OSPF.
    ```
    RTRRIP:
        RTRRIP(config)#int loopback 0
        RTRRIP(config-if)#ip address 172.20.0.1 255.255.255.0
        RTRRIP(config-if)#exit
        
        RTRRIP(config)#int loopback 1
        RTRRIP(config-if)#ip address 172.20.1.1 255.255.255.0
        RTRRIP(config-if)#exit
        
        RTRRIP(config)#int loopback 2
        RTRRIP(config-if)#ip address 172.20.2.1 255.255.255.0
        RTRRIP(config-if)#exit
        
        RTRRIP(config)#router rip
        RTRRIP(config-router)#network 10.0.0.0
        RTRRIP(config-router)#network 172.20.0.0
    R2AR0:
        
        R2AR0(config)#router rip
        R2AR0(config-router)#network 10.0.0.0
        R2AR0(config-router)#exit
        
        R2AR0(config)#router ospf 1
        R2AR0(config-router)#redistribute rip subnets
    ```
8. Проверяем что из области 0 можно проложить маршрут через маршрутизатор, который соединен с областью 0 с помощью протокола RIP.
![N|Solid](https://i.imgur.com/0vTX1fR.png)

9. Делаем область 1 тупиковой и проверяем результат.
![N|Solid](https://i.imgur.com/XYLbRRT.png)

### Результат
Получилась сеть с настроенным протоколом OSPF. Эта сеть разделенна на 2 области, соединяются которые при помощи маршрутизатора ABR. Область 1 является тупиковой, т.е ABR-роутер конвертирует все External LSA в единственный Summary LSA 0.0.0.0/0. Так же к этой сети подключен внешний домен, работающий по протоколу RIP. При помощи роутера R2AR0 удалось настроить передачу маршрутов из RIP в OSPF => получилось наладить связь между доменами, использующими разные внутридоменные протоклы маршрутизации.

### Промежуточные результаты
Настроеная область 0 -- ospf_area_0.pkt
Настроены области 1 и 0, установлены маршруты из одной в другую -- ospf_area_0_and_1.pkt
Настроен домен, в котором работает протокол RIP, установлено соединение с доменом, где работает OSPF -- ospf_with_rip.pkt
Поменен тип области 1 на тупиковый (конечная сборка) -- ospf_with_stub.pkt
