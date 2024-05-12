# Домашнее задание №1
## Основы проектирования сети

### Задание:
- Собрать схему CLOS;
- Распределить адресное пространство.

## Решение:

### Схема сети
![](img/Schema.png)

### Таблица адресов

| hostname | interface |   IP/MASK   | Description |
| :------: | :-------: | :----------: | :---------: |
|  leaf-1  | Loopback2 | 10.1.0.1 /32 |            |
|  leaf-1  |  eth 1/1  | 10.2.1.1 /31 | to-spine-1 |
|  leaf-1  |  eth 1/2  | 10.2.2.1 /31 | to-spine-2 |
|          |          |              |            |
|  leaf-2  | Loopback2 | 10.1.0.2 /32 |            |
|  leaf-2  |  eth 1/1  | 10.2.1.3 /31 | to-spine-1 |
|  leaf-2  |  eth 1/2  | 10.2.2.3 /31 | to-spine-2 |
|          |          |              |            |
|  leaf-3  | Loopback2 | 10.1.0.3 /32 |            |
|  leaf-3  |  eth 1/1  | 10.2.1.5 /31 | to-spine-1 |
|  leaf-3  |  eth 1/2  | 10.2.2.5 /31 | to-spine-2 |
|          |          |              |            |
| spine-1 | Loopback1 | 10.0.1.0/32 |            |
| spine-1 |  eth 1/1  | 10.2.1.0/31 |  to-leaf-1  |
| spine-1 |  eth 1/2  | 10.2.1.2/31 |  to-leaf-2  |
| spine-1 |  eth 1/3  | 10.2.1.4/31 |  to-leaf-3  |
|          |          |              |            |
| spine-2 | Loopback1 | 10.0.2.0/32 |            |
| spine-2 |  eth 1/1  | 10.2.2.0/31 |  to-leaf-1  |
| spine-2 |  eth 1/2  | 10.2.2.2/31 |  to-leaf-2  |
| spine-2 |  eth 1/3  | 10.2.2.2/31 |  to-leaf-3  |

## Конфигурации устройств: