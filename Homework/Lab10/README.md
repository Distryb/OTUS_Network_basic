# Настройка протокола OSPFv2 для одной области

### Задачи:
#### Часть 1. Создание сети и настройка основных параметров устройства
#### Часть 2. Настройка и проверка базовой работы протокола  OSPFv2 для одной области
#### Часть 3. Оптимизация и проверка конфигурации OSPFv2 для одной области

## Топология: 
![](Topology_lab_10.png)  

### Таблица адресации:  
Устройство | Интерфейс | IP-адрес | Маска подсети 
:---: | :---: | :---: | :---: 
R1 | G0/0/1 | 10.53.0.1 | 255.255.255.0
R1 | Loopback 1  | 172.16.1.1 | 255.255.255.0
R2 | G0/0/1 | 10.53.0.2 | 255.255.255.0 
R2 | Loopback 1 | 192.168.1.1 | 255.255.255.0 

### Решение:
#### Часть 1: 
Базовые настройки маршрутизаторов и коммутаторов выполнены, IP адреса прописаны.  

#### Часть 2. Настройка и проверка базовой работы протокола  OSPFv2 для одной области  
Конфигурация OSPF:  
```
R2#sh run | begin ospf
router ospf 56
 router-id 2.2.2.2
 log-adjacency-changes
 network 10.53.0.0 0.0.0.255 area 0
 network 192.168.1.0 0.0.0.255 area 0
```
Проверка работоспособности:  
```
R2#sh ip ospf neighbor 
Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1           1   FULL/BDR        00:00:33    10.53.0.1       GigabitEthernet0/0/1
R2#
R2#sh ip ospf database 
            OSPF Router with ID (2.2.2.2) (Process ID 56)
                Router Link States (Area 0)
Link ID         ADV Router      Age         Seq#       Checksum Link count
2.2.2.2         2.2.2.2         7           0x80000003 0x0099aa 2
1.1.1.1         1.1.1.1         7           0x80000002 0x009c2e 1
```
⦁	Какой маршрутизатор является DR? Какой маршрутизатор является BDR? Каковы критерии отбора?  
*- Маршрутизатором с самым высоким приоритетом OSPF и самым высоким Router ID выбирается как DR.  
Маршрутизатор со вторым по величине приоритетом OSPF и вторым по величине Router ID выбирается как BDR.  
В нашем примере, R2 (id 2.2.2.2) является DR и R1 (id 1.1.1.1) BDR соответственно.*  


⦁	На R1 выполните команду show ip route ospf, чтобы убедиться, что сеть R2 Loopback1 присутствует в таблице маршрутизации.  
⦁	Запустите Ping до  адреса интерфейса R2 Loopback 1 из R1  

```
R1#sh ip route ospf 
     192.168.1.0/32 is subnetted, 1 subnets
O       192.168.1.1 [110/2] via 10.53.0.2, 00:01:00, GigabitEthernet0/0/1


R1#ping 192.168.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/1 ms
```

#### Часть 3. Оптимизация и проверка конфигурации OSPFv2 для одной области  
⦁	На R1 настройте приоритет OSPF интерфейса G0/0/1 на 50  
```
R1(config)#interface gigabitEthernet 0/0/1
R1(config-if)#ip ospf priority 50
```

⦁	Настройте таймеры OSPF на G0/0/1 каждого маршрутизатора для таймера приветствия, составляющего 30 секунд.  
```
R1(config)#interface gigabitEthernet 0/0/1
R1(config-if)#ip ospf hello-interval 30
R1(config-if)#ip ospf dead-interval 120
```

⦁	На R1 настройте статический маршрут по умолчанию, который использует интерфейс Loopback 1 в качестве интерфейса выхода. Затем распространите маршрут по умолчанию в OSPF. Обратите внимание на сообщение консоли после установки маршрута по умолчанию.  
```
R1(config)#ip route 0.0.0.0 0.0.0.0 loopback 1
%Default route without gateway, if not a point-to-point interface, may impact performance
R1(config)#
R1(config)#router ospf 56
R1(config-router)#default-information originate 
```

⦁	Измените базовую пропускную способность для маршрутизаторов. После этой настройки перезапустите OSPF с помощью команды clear ip ospf process.   
```
R1(config)#router ospf 56
R1(config-router)#auto-cost reference-bandwidth 1000
R1#
R1#clear ip ospf process 
Reset ALL OSPF processes? [no]: yes
R1#
00:32:31: %OSPF-5-ADJCHG: Process 56, Nbr 2.2.2.2 on GigabitEthernet0/0/1 from FULL to DOWN, Neighbor Down: Adjacency forced to reset
00:32:31: %OSPF-5-ADJCHG: Process 56, Nbr 2.2.2.2 on GigabitEthernet0/0/1 from FULL to DOWN, Neighbor Down: Interface down or detached
R1#
00:32:45: %OSPF-5-ADJCHG: Process 56, Nbr 2.2.2.2 on GigabitEthernet0/0/1 from LOADING to FULL, Loading Done
R1#
```
⦁	Добавьте конфигурацию, необходимую для OSPF для обработки R2 Loopback 1 как сети точка-точка  
⦁	Только на R2 добавьте конфигурацию, необходимую для предотвращения отправки объявлений OSPF в сеть Loopback 1.  
```
R2(config)#interface loopback 1
R2(config-if)#ip ospf network point-to-point
R2(config)#
R2(config)#router ospf 56
R2(config-router)#passive-interface loopback 1
```

### ⦁	Убедитесь, что оптимизация OSPFv2 реализовалась.  
⦁	Выполните команду show ip ospf interface g0/0/1 на R1 и убедитесь, что приоритет интерфейса установлен равным 50, а временные интервалы — Hello 30, Dead 120, а тип сети по умолчанию — Broadcast  
```
R1#sh ip ospf interface gigabitEthernet 0/0/1

GigabitEthernet0/0/1 is up, line protocol is up
  Internet address is 10.53.0.1/24, Area 0
  Process ID 56, Router ID 1.1.1.1, Network Type BROADCAST, Cost: 10
  Transmit Delay is 1 sec, State DR, Priority 50
  Designated Router (ID) 1.1.1.1, Interface address 10.53.0.1
  Backup Designated Router (ID) 2.2.2.2, Interface address 10.53.0.2
  Timer intervals configured, Hello 30, Dead 120, Wait 120, Retransmit 5
    Hello due in 00:00:18
  Index 1/1, flood queue length 0
  Next 0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 1
```

⦁	На R1 выполните команду show ip route ospf, чтобы убедиться, что сеть R2 Loopback1 присутствует в таблице маршрутизации  

```
R1#sh ip route 
Gateway of last resort is 0.0.0.0 to network 0.0.0.0
O    192.168.1.0/24 [110/10] via 10.53.0.2, 00:03:36, GigabitEthernet0/0/1
```

⦁	Введите команду show ip route ospf на маршрутизаторе R2. Единственная информация о маршруте OSPF должна быть распространяемый по умолчанию маршрут R1.  
```
R2#sh ip ro
Gateway of last resort is 10.53.0.1 to network 0.0.0.0
O*E2 0.0.0.0/0 [110/1] via 10.53.0.1, 00:09:45, GigabitEthernet0/0/1
```

⦁	Запустите Ping до адреса интерфейса R1 Loopback 1 из R2. Выполнение команды ping должно быть успешным.   
```
R2#ping 172.16.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 172.16.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/1 ms
```  

Почему стоимость OSPF для маршрута по умолчанию отличается от стоимости OSPF в R1 для сети 192.168.1.0/24?
*- 
