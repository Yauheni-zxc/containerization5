# containerization5
# Урок 5. Docker Compose и Docker Swarm

1) создать сервис, состоящий из 2 различных контейнеров: 1 - веб, 2 - БД
2) далее необходимо создать 3 сервиса в каждом окружении (dev, prod, lab)
3) по итогу на каждой ноде должно быть по 2 работающих контейнера
4) выводы зафиксировать

### Задание 2*:
1) нужно создать 2 ДК-файла, в которых будут описываться сервисы
2) повторить задание 1 для двух окружений: lab, dev
3) обязательно проверить и зафиксировать результаты, чтобы можно было выслать преподавателю для проверки

Задание со звездочкой - повышенной сложности, это нужно учесть при выполнении (но сделать его необходимо).

### или (если нет возможности создать вторую node)
1) Создать 2 ДК-файла, в которых будут описываться сервисы состоящие из двух контейнеров: 1 - веб, 2 - БД
2) Сделать реплики контейнеров воспользовавшись сервисом Docker Swarm;
3) Проверить работоспособность контейнеров.

**Термины**

Для того чтобы пользоваться swarm, надо запомнить несколько типов сущностей:

Node - это наши виртуальные машины, на которых установлен docker. Есть manager и workers ноды. Manager нода управляет 
workers нодами. Она отвечает за создание/обновление/удаление сервисов на workers, а также за их масштабирование и 
поддержку в требуемом состоянии. Workers ноды используются только для выполнения поставленных задач и не могут управлять 
кластером.

Stack - это набор сервисов, которые логически связаны между собой. По сути это набор сервисов, которые мы описываем в 
обычном compose файле. Части stack (services) могут располагаться как на одной ноде, так и на разных.

Service - это как раз то, из чего состоит stack. Service является описанием того, какие контейнеры будут создаваться. 
Если вы пользовались docker-compose.yaml, то уже знакомы с этой сущностью. Кроме стандартных полей docker в режиме swarm 
поддерживает ряд дополнительных, большинство из которых находятся внутри секции deploy.

Task - это непосредственно созданный контейнер, который docker создал на основе той информации, которую мы указали при 
описании service. Swarm будет следить за состоянием контейнера и при необходимости его перезапускать или перемещать на 
другую ноду.

**Первый вариант решение(без использования второй node):**

Создаем два ДК-файла:

![1_1](skrins/1_1.JPG)

![1_2](skrins/1_2.JPG)

Компилируем node:

![1_3](skrins/1_3.JPG)

Проверяем:

![1_4](skrins/1_4.JPG)

Пытаемся задеплоить в Stack:

![1_5_1](skrins/1_5_1.JPG)

![1_5_2](skrins/1_5_2.JPG)

Получаем ошибку: "версия данного файла не поддерживается", продолжаем делать, как делали на семинаре.
Запускаем первый ДК-файл:

![1_7](skrins/1_7.JPG)

Проверяем:

![1_8](skrins/1_8.JPG)

![1_9](skrins/1_9.JPG)

Запускаем второй ДК-файл:

![1_10_1](skrins/1_10_1.JPG)

![1_10_1](skrins/1_10_1.JPG)

![1_11](skrins/1_11.JPG)

Воспользуемся функционалом Docker Swarm и создадим дубликат одного и того же образа:

![1_12](skrins/1_12.JPG)

![1_13](skrins/1_13.JPG)

Аналогично можем проделать с другими образами, далее повязать их сетью и получить полные дубликаты: БД + WEB.

**Второй вариант решение(с использованием второй node):**

Подключаемся по SSH ко второй VM и устанавливаем swarm соединение, предварительно инициализировав manager node:

![2_1](skrins/2_1.JPG)

Запускаем первый ДК-файл на manager node:

![2_2](skrins/2_2.JPG)

Проверяем:

![2_3](skrins/2_3.JPG)

Запускаем второй ДК-файл на worker node:

![2_4](skrins/2_4.JPG)

Проверяем:

![2_5](skrins/2_5.JPG)

Попробуем запустить первый ДК-файл на node worker:

![2_6](skrins/2_6.JPG)

Проверяем, все работает, ошибки дублирования не произошло т. к имена image разные(ДК-файлы в разных директориях 
находятся на VM)

![2_7](skrins/2_7.JPG)

Запускаем на manager node второй ДК-файл и получаем четыре работающих контейнера на одной node и на другой
(не считая hello-word):

![2_8](skrins/2_8.JPG)

![2_9](skrins/2_9.JPG)

Если бы расположение папок было идентично на manager node и worker node, пришлось бы создавать реплики образов и 
обвязывать их сетью:

![2_10](skrins/2_10.JPG)

Тем временем наблюдаем за статусом worker node, он "упал"(Отказался идти на повышение). Переподключаем swarm соединение:

![2_11](skrins/2_11.JPG)

Создаем еще реплики образов и видим как система их перераспределяет:

![2_12](skrins/2_11.JPG)

![2_13](skrins/2_13.JPG)

Для того чтобы сервисы распределять на определенные ноды необходимо применять labels
(подробнее об этом тут: https://habr.com/ru/articles/659813/)

*Подготовил студент GeekBrains* [**`Макей Евгений`**](https://gb.ru/users/19b5d41f-52eb-4dc6-8b02-5da1eb611b84), 
Seminar_5_containerization
