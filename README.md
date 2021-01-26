# eBGP и iBGP

# Домашняя работа: eBGP и iBGP.
# Задание eBGP:
1. Настроить eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас
2. Настроить eBGP между провайдерами Киторн и Ламас
3. Настроить eBGP между Ламас и Триада
4. Настроить eBGP между офисом С.-Петербург и провайдером Триада
5. Организовать IP доступность между офисами Москва и С.-Петербург

 Задание iBGP:
1. Настроить iBGP в офисе Москва между маршрутизаторами R14 и R15
2. Настроить iBGP в провайдере Триада
3. Настройть офис Москва так, чтобы приоритетным провайдером стал Ламас.
4. Настройть офис С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно
5. Все сети в лабораторной работе должны иметь IP связность.


# Сначала настроим eBGP..

# 1. Настроика eBGP между офисом Москва и двумя провайдерами. Конфигурация R14, R15, R21, R22.
Для примера приведу конфигурацию настройки eBGP на R15. Для начала необходимо запустить процесс BGP, затем настроить соседей. 
Конфиг для R15:
```
router bgp 1001
 bgp log-neighbor-changes
 neighbor 200C:C0FE:1111:20::2 remote-as 301
 neighbor 194.14.123.6 remote-as 301
 !
  address-family ipv4
  redistribute ospf 1
  neighbor 194.14.123.6 activate
 exit-address-family
 !
 address-family ipv6
  neighbor 200C:C0FE:1111:20::2 activate
 exit-address-family
!
```
Конфиг для R21:
```
!
router bgp 301
 bgp log-neighbor-changes
 neighbor 200C:C0FE:1111:20::1 remote-as 1001
 neighbor 194.14.123.5 remote-as 1001
 !
 address-family ipv4
  neighbor 194.14.123.5 activate
 exit-address-family
 !
 address-family ipv6
  neighbor 200C:C0FE:1111:20::1 activate
 exit-address-family
!
```
На R14, R22 настройки аналогичные.

# 2. Настроика eBGP между провайдерами Киторн и Ламас. Конфигурация R21, R22.
```
!
router bgp 301
 bgp log-neighbor-changes
 neighbor 200C:C0FE:1111:A0::2 remote-as 101
 neighbor 194.14.123.33 remote-as 101
 !
 address-family ipv4
  neighbor 194.14.123.33 activate
 exit-address-family
 !
  address-family ipv6
  neighbor 200C:C0FE:1111:A0::2 activate
 exit-address-family
!
```
На R22 настройки аналогичные.

# 3. Настроика eBGP между Ламас и Триада. Конфигурация R21, R24.
```
!
router bgp 301
 bgp log-neighbor-changes
 neighbor 200C:C0FE:1111:40::2 remote-as 520
 neighbor 194.14.123.21 remote-as 520
 !
 address-family ipv4
  neighbor 194.14.123.21 activate
 exit-address-family
 !
 address-family ipv6
  neighbor 200C:C0FE:1111:40::2 activate
 exit-address-family
!
```
На R24 настройки аналогичные.

# 4. Настроика eBGP между офисом С.-Петербург и провайдером Триада. Конфигурация R18, R24, R26.
```
!
router bgp 2042
 bgp log-neighbor-changes
 neighbor 200C:C0FE:1111:50::2 remote-as 520
 neighbor 200C:C0FE:1111:60::2 remote-as 520
 neighbor 194.14.123.10 remote-as 520
 neighbor 194.14.123.14 remote-as 520
 !
 address-family ipv4
  redistribute eigrp 4
  no neighbor 200C:C0FE:1111:50::2 activate
  no neighbor 200C:C0FE:1111:60::2 activate
  neighbor 194.14.123.10 activate
  neighbor 194.14.123.14 activate
 exit-address-family
 !
 address-family ipv6
  neighbor 200C:C0FE:1111:50::2 activate
  neighbor 200C:C0FE:1111:60::2 activate
 exit-address-family
!
```

# Далее настроим iBGP.

# 1. Настроить iBGP в офисе Москва между маршрутизаторами R14 и R15.
Конфиг для R15:
```
router bgp 1001
 bgp log-neighbor-changes
 neighbor 152.95.25.1 remote-as 1001
 neighbor 152.95.25.1 update-source loopback0
 !
 address-family ipv4
  neighbor 152.95.25.1 activate
  neighbor 152.95.25.1 next-hop-self
 exit-address-family
 !
```
На R14 настройки аналогичные.

# 2. Настроить iBGP в провайдере Триада.
Для удобства настроим на R23 peer-group
Конфиг для R23:
```
router bgp 520
 bgp log-neighbor-changes
 redistribute isis level-1-2
 neighbor as520 peer-group
 neighbor as520 remote-as 520
 neighbor as520 update-source Loopback0
 neighbor 90.47.74.18 peer-group as520
 neighbor 90.47.74.19 peer-group as520
 neighbor 90.47.74.20 peer-group as520
!
```
Конфиг для R25:
```
!
router bgp 520
 bgp log-neighbor-changes
 redistribute isis level-1-2
 neighbor as520 peer-group
 neighbor as520 remote-as 520
 neighbor as520 update-source Loopback0
 neighbor 90.47.74.17 peer-group as520
 neighbor 90.47.74.18 peer-group as520
 neighbor 90.47.74.20 peer-group as520
!
```
На R24, R26 настройки аналогичные.

# 3. Настройть офис Москва так, чтобы приоритетным провайдером стал Ламас.
В данном случае, я на R15 прописал в сторону R14: neighbor 152.95.25.1 next-hop-self, через route-map увеличил атрибут local-preference до 150 в сторону R14. Так же на R22 через route-map увеличил AS-Path.
Конфиг на R22:
```
router bgp 101
 bgp log-neighbor-changes
 neighbor 200C:C0FE:1111:10::1 remote-as 1001
 neighbor 194.14.123.1 remote-as 1001
 !
 address-family ipv4
  neighbor 194.14.123.1 activate
  neighbor 194.14.123.1 route-map as1001 out
 exit-address-family
 !
 address-family ipv6
  neighbor 200C:C0FE:1111:10::1 activate
 exit-address-family
!
!
route-map as1001 permit 10
 match ip address prefix-list as1001
 set as-path prepend 101 101
!
```
Конфиг на R15:
```
router bgp 1001
 bgp log-neighbor-changes
 neighbor 152.95.25.1 remote-as 1001
 neighbor 152.95.25.1 update-source loopback0
 !
 address-family ipv4
  redistribute ospf 1
  neighbor 152.95.25.1 activate
  neighbor 152.95.25.1 next-hop-self
  neighbor 152.95.25.1 route-map BGP_TRAF_IN out
 exit-address-family
 !
ip prefix-list TRAF_IN seq 10 permit 0.0.0.0/0 le 32
!
route-map BGP_TRAF_IN permit 10
 match ip address prefix-list TRAF_IN
 set local-preference 150
```

# 4. Настройть офис С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно.
 Здесь для балансировки использовал команду maximum-paths 2 на R18.
 ```
 !
router bgp 2042
 bgp log-neighbor-changes
 neighbor 200C:C0FE:1111:50::2 remote-as 520
 neighbor 200C:C0FE:1111:60::2 remote-as 520
 neighbor 194.14.123.10 remote-as 520
 neighbor 194.14.123.14 remote-as 520
 !
 address-family ipv4
  redistribute eigrp 4
  no neighbor 200C:C0FE:1111:50::2 activate
  no neighbor 200C:C0FE:1111:60::2 activate
  neighbor 194.14.123.10 activate
  neighbor 194.14.123.14 activate
  maximum-paths 2
 exit-address-family
 !
 address-family ipv6
  neighbor 200C:C0FE:1111:50::2 activate
  neighbor 200C:C0FE:1111:60::2 activate
 exit-address-family
!
 ```

# 5. Для проверки связности  запустим пинг от VPC1 из Москвы до VPC8 в Питере.

![](https://github.com/dmitriyklimenkov/eBGP/blob/main/%D0%BF%D0%B8%D0%BD%D0%B3.PNG)

Связность есть.

Для примера, приведу таблицу BGP и таблицу маршрутизации R14:

![](https://github.com/dmitriyklimenkov/eBGP/blob/main/route%20table.PNG)

![](https://github.com/dmitriyklimenkov/eBGP/blob/main/bgp%20table.PNG)

Таблица BGP полностью на снимок не влезла.

Полные конфигурации:


Файл конфигурации R14 [здесь](https://github.com/dmitriyklimenkov/eBGP/blob/main/R14.txt).

Файл конфигурации R15 [здесь](https://github.com/dmitriyklimenkov/eBGP/blob/main/R15.txt).

Файл конфигурации R18 [здесь](https://github.com/dmitriyklimenkov/eBGP/blob/main/R18.txt).

Файл конфигурации R21 [здесь](https://github.com/dmitriyklimenkov/eBGP/blob/main/R21.txt).

Файл конфигурации R22 [здесь](https://github.com/dmitriyklimenkov/eBGP/blob/main/R22.txt).

Файл конфигурации R23 [здесь](https://github.com/dmitriyklimenkov/eBGP/blob/main/R23.txt).

Файл конфигурации R24 [здесь](https://github.com/dmitriyklimenkov/eBGP/blob/main/R24.txt).

Файл конфигурации R25 [здесь](https://github.com/dmitriyklimenkov/eBGP/blob/main/R25.txt).

Файл конфигурации R26 [здесь](https://github.com/dmitriyklimenkov/eBGP/blob/main/R26.txt).
