### Содержание
- [Итоговый проект](#итоговый-проект)
    - [Комментарии к условию](#комментарии-к-условию)
      - [Здание](#здание)
      - [Лифты](#лифты)
        - [Генератор толстых и тонких людей](#генератор-толстых-и-тонких-людей)
      - [Что делать?](#что-делать?)
      - [Сбор статистики](#сбор-статистики.)
      - [Лифт-зомби](#лифт-зомби)
      - [Контроллер](#что-делать?)
- [Реализация](#реализация)
  - [Класс Call](#класс-call:)
  - [Enum Direction](#enum-direction)
  - [Класс Passenger](#класс-Passenger:)
  - [Класс Floor](#класс-Floor:)
  - [Класс Elevator](#класс-Elevator-implements-Runnable:)
  - [Класс Building](#класс-building:)
  - [Класс ElevatorPicker(алгоритм подбора лифта)](#класс-elevatorPicker:)
  - [Класс Randomizer и его наследники PassengerRandomizer и ElevatorRandomizer](#класс-Randomizer-и-его-наследники-PassengerRandomizer-и-ElevatorRandomizer:)
  - [Класс ElevatorSpawn](#класс-ElevatorSpawn:)
  - [Класс PassengerSpawn](#класс-PassengerSpawn:)
  - [Потоки](#потоки:)

# Итоговый проект
1. Есть многоэтажное здание (этажность конфигурируема). В здании есть лифты (количество конфигурируемо). На каждом этаже есть кнопки вызова “вверх” и “вниз” (общие для всех лифтов) На каждом этаже появляются люди (рандомной массы), которые хотят ехать на другой этаж (рандомный). Интенсивность генерации людей конфигурируема
2. У каждого лифта есть грузоподъемность, скорость и скорость открытия/закрытия дверей.
3. У человека есть масса и этаж, на который ему нужно.
4. Люди стоят в очереди на засадку в лифты (одна очередь вверх, одна вниз) не нарушая её. Приехав на нужный этаж, человек исчезает.
**Задание.** Необходимо реализовать непрерывно работающее приложение (люди появляются, вызывают лифт и едут на нужный этаж) используя многопоточность (Thread, wait, notify, sleep).
По желанию можно использовать `java.util.concurrent`. Также описать выбранный алгоритм текстом (кратко).
- тесты, maven, логгирование;
- реализовать сбор статистики (сколько людей перевезено каждым лифтом, с каких этажей и на какие этажи);
- логировать основные события системы (чтобы по логам можно было следить за тем, что происходит);
- рекомендация: реализовать логику управления лифтами и покрыть её тестами без потоков, а потом подключить многопоточность. 
## Комментарии к условию
### Здание
В условии везде в местах, где указано конфигурируемо, не имеется в виду, что должно быть конфигурируемо обязательно через `property file` или еще как-нибудь. Просто, например, `Building` будет принимать параметр количество этажей, например, в здании есть (а это многоэтажное здание) сорок этажей. 
### Лифты
Чаще всего такие лифты присутствуют в офисных зданиях, не в жилых. Когда на этаже есть кнопки вызова вверх или вниз. Как мы понимаем вызов лифта может происходить по разному. В жилых домах обычно ставят, что лифт когда едет наверх, он не останавливается, а когда едет вниз он забирает всех на своем пути. Но у нас будут кнопочки “вверх” и “вниз”. И на каждом этаже есть только две кнопки. Эти кнопки являются общими для всех лифтов. Думаю вы замечали, что бывает так: приходишь, а там четыре лифта и кнопки у каждого свои “вверх” и “вниз”. Это не наш случай. У нас приходишь и нажимаешь кнопочку “вверх” если тебе нужно вверх и приезжает какой-то лифт, один из четырех, и ты в него заходишь и едешь вверх.
#### Параметры
У каждого лифта есть следующие параметры: 
- грузоподъемность, которая будет измеряйся в килограммах.
- скорость, например он проезжает два этажа в секунду, или один этаж в секунду, или один этаж за две секунды. Эта скорость может быть конфигурируема. Вы можете сделать в своём здании пять лифтов и у каждого будет своя скорость.
- скорость открытия/закрытия дверей, этот параметр тоже может быть конфигурируемый, но не обязательно задавать его разным для разных лифтов. Один лифт быстро двери открывает, а второй медленно. Это перебор. Просто нужно сделать так, чтобы этот параметр было удобно менять.
### Люди
На каждом этаже случайным образом появляются люди, которые хотят куда-то ехать. Люди эти случайной массы.
Приехав на нужных этаж человек исчезает, т.е. он приехал на пятый и куда-то исчезает. Зашел к себе в кабинет и всё, больше его нет. Ну и люди честные. Они когда приходят и видят очередь, то спрашивают: “А кто тут наверх сейчас крайний в очереди?”. На первом и на последнем этажах, понятно, будет только одна очередь. А на всех промежуточных этажах будет по две очереди.
#### Генератор толстых и тонких людей
Интенсивность генерации людей также конфигурируема, как и все остальное. Вам нужно сделать какой-то класс, объект, который будет называться `PeopleSpawn` и будет генерировать людей. Т.е. он сгенерирвал толстого человека на текущем этаже (на каждом этаже свой генератор людей). На первом этаже генератор, который много людей генерирует. На третьем этаже возможно мало или на втором мало. Тут нет строгих требований. Вы можете сделать генераторы для очереди вверх и очереди вниз отдельно. Либо сделать общий генератор, который будет рандомно добавлять людей в обе очереди.
По поводу использования `java.util.concurrent`. В вашем задании было прочитать, там в статье это есть про `java.util.concurrent`. Это более высокоуровневое понятие. Мы про что-то поговорим, но опять же не про все. Поэтому лучше прочитайте дополнительно.
### Что делать?
Вот и все задание. Нужно это дело сделать, чтобы работало непрерывно. Вы инициализируете здание, вы инициализируете лифты, вы инициализируете рандомное появление людей.
### Сбор статистики.
Про сбор статистики возникают обычно вопросы у людей, если что-то такое написать. Нужно сделать какой-то `Storage`, в котором будет известна статистика. Что на таком-то этаже, столько-то приехало людей, на таких-то лифтах. Чтобы можно было потом посмотреть посчитать.
Логирование. Тут можно было бы сделать визуализацию, консольную визуализацию. Просто это не нужно. Можно было бы нарисовать в консоли этажи и по ним бы ехали лифты. Или не консольную, а другую какую-то визуализацию. Но наше задание не про то совсем. У нас акцент на многопоточность плюс, и даже больше чем многопоточность, на дизайн классов, на организацию кода. Не забывайте писать пакеты и прочие вещи.
Также я бы хотел дать рекомендацию, про это мы еще поговорим на последнем занятии. Лучше реализовать логику управления лифтами, я тут не сильно вдаюсь в подробности, и покрыть её тестами. Получается лифты будут потоками, т.е. поток в данном случае будет выполнять задачу не какого-то распараллеливания, здесь у нас вообще получается какая-то модель, где система живет своей жизнью. 
Потоки здесь будут для того, чтобы каждый лифт, каждый подвижный товарищ здесь. Кто у нас здесь подвижный? Кто у нас живет своей жизнью? Лифты живут своей жизнью. И люди, наверное, не обязательно делать, чтобы они жили своей жизнью. Потому что они, в принципе, ничего не делают. У них нету `life-cycle`. Они появились, они сели в лифт. Заниматься такой имитацией, чтобы человек топал и садился в лифт, нам не надо. Просто можно сделать, чтобы наш лифт засасывал нужное количество людей из соответствующей очереди. Ну и причем логика может быть какая? Если там первым стоит в очереди очень толстый человек, и он уже не влазит в лифт, то понятное дело, что можно пройтись по очереди на посадку и всосать более худеньких. Которые всё еще помещаются в лифт. Т.е. как обычно. “Ой, я уже не влезу, у меня вещей много, проходите вы.” Здесь такое тоже можно реализовать.
### Лифт зомби
Но я бы хотел, что сказать. От этого не загоняйтесь. Делайте как получается. Просто в идеале лифты должны быть, я бы так рекомендовал, максимально глупыми, они должны выполнять только определенные задачи. Лифт, он такой как зомби. Ой, я проснулся. Кто я? Я лифт. Что мне делать? Мне нужно открывать двери. Ну, хорошо, я открываю двери. Всё, дальше. Так кто? Я лифт. Что со мной? Я открываю двери. У меня открыты двери. Что мне нужно делать дальше? Ну вот что-то такое. 
### Контроллер
При этом в здании, в котором есть больше одного лифта, скорее всего нужно реализовать какой-то контроллер, который будет смотреть, что если на пятом этаже нажата кнопка, то можно определить, что какой-то лифт сейчас находится без дела. Понятно, что у лифта могут быть разные состояния. И сказать, что вот ты, лифт, едь сюда, на пятый этаж и открывай двери. Что-то типа такого. 
Но эта задача почему интересная? Я пробовал эту задачку на людях с большим опытом, просто обсуждать варианты решения. У всех сразу загораются глаза, все хотят её сразу делать. Все хотят реализовывать всякие `state`-машины и прочее. Ну, т.е. очень можно по разному её реализовать. 
Что еще важно?

---

# Реализация


### Класс Call:
- floor(вызванный этаж)
- direction(направление движения)
### Enum Direction: 
- UP, DOWN, NO.
### Класс Passenger: 
- weight
- currentFloor(с какого этажа)
- requestedFloor(на какой этаж)
- direction
- -> callFromFloor() - возвращает вызов, сделанный с этажа
- -> callFromElevator() - возвращает вызов, сделанный с внутри лифта
### Класс Floor:
- queueUp(очередь людей, направляющихся вверх)
- queueDown(очередь вниз)
- -> addToWaitingQueue(passenger) - добавляет нового человека в очередь и возвращает новый вызов, если это первый человек в очереди
- -> providePassengersToPickUp(direction, availableWeight) - возвращает список людей, которые сядут в приехавший лифт. Идёт по очереди и подбирает всех, кто влазит.
- -> getRepeatedCall(direction) - генерирует повторный вызов лифта: для случая, когда лифт приехал на этаж, но забрал не всех пассажиров из соответствующей очереди.
### Класс Elevator implements Runnable:
- сортированная очередь вызовов
- список пассажиров
- elevatorDirection(в каком направлении сейчас движется лифт) 
- callDirection(направление, в котором хотят ехать пассажиры)
- availableCapacity и capacity
- passengerProvider - объект функционального интерфейса PassengerProvider, определённого в данном классе, и реализованного в классе Building для передачи между данными классами списка пассажиров, которых нужно забрать с этажа
- -> performCallsFromElevator(passenger) - когда пассажир заходит в лифт, он нажимает кнопку нужного этажа и генерируется новый вызов
- -> updateDirections(call) - если есть пассажиры в лифте, то направляемся туда, куда хотят пассажиры, иначе если лифт бездействовал, то направляемся за пассажирами; в остальных случаях направление не меняется
- -> getCurrentCall() - обслуживаем этажи в порядке возрастания, если пассажиры едут вверх; в порядке убывания - если едут вниз
```java
- жизненный цикл:
            waitForCalls();
            while (!needsStop()) {
                move();
            }
            openDoors();
            dropOffPassengers();
            pickUpPassengers();
            closeDoors();
``` 
### Класс Building:
- основной класс, который связывает все компоненты в работающую систему
- -> acceptPassenger(passenger) - передаётся в PassengerRandomizer в качестве Consumer, получает из него пассажира и вызывает addToWaitingCalls
- -> addToWaitingCalls(call) - добавляет пассажира в очередь на соответствующем этаже и, если кнопка лифта на нем не была нажата, получает новый вызов и добавляет его в очередь ожидания
- -> run() - запускает все потоки приложения и вызывает основную функцию processWaitingQueue, которая выполняется в основном потоке
- -> providePassengers(elevatorId, floorNumber, direction, availableWeight) - реализация метода интерфейса PassengerProvider из класса Elevator. Возвращает список пассажиров, которых нужно добавить в лифт, а также производит повторный вызов лифта, если не все пассажиры из очереди текущего направления поместились. Учтены граничные случаи: первый и последний этаж, на которых есть очередь только в одном направлении.
- -> handleCall(call) - получает наиболее подходящий лифт и, если такой имеется, добавляет вызов к нему в очередь и возвращает true
- -> processWaitingQueue() - ожидает вызов, и, когда вызов получен, пытается его обработать. Если удаётся - удаляет вызов из очереди.
### Класс ElevatorPicker:
Алгоритм представляет собой булеву функцию. Для её получения сначала строим таблицу истинности для 4 аргументов: направление elevatorDirection текущего движения лифта(a), направление callDirection вызовов лифта(b) лифта, направление call.direction() нового вызова (для которого ищем лифт) и направления, определяющего выше этаж нового вызова, чем текущий, или ниже. Затем строим по ней СДНФ -> abcd + ab̅c̅d̅ + ab̅c̅d + a̅bcd + a̅bcd̅ + a̅b̅c̅d̅. Минимизируем СДНФ -> ab̅c̅ + a̅bc + bcd + b̅c̅d̅. Можно заметить, что условие b==c можно вынести, получаем -> b==c && (a!=b || b!=d). К этой функции также всегда добавляется условие, что в лифте ещё есть место. Также нам подходят все лифты, которые бездействуют. 
- Что делает алгоритм:
  - получает список всех подходящих лифтов
    - лифт подходит в двух случаях: 1) лифт бездействует; 2) направление вызова лифта совпадает с направлением нового вызова, в лифте ещё есть место и (либо пассажиров в лифте нет и лифт двигается за ними, либо пассажиры есть, но этаж нового вызова ещё не проехали)
  - выбирает лифт с наименьшей стоимостью
    + стоимость вычисляется как (количество вызовов лифта * текущий вес пассажиров в лифте) и введена для более оптимального распределения вызовов между лифтами (чем меньше у лифта вызовов и пассажиров на борту, тем вероятнее этот лифт будет выбран)
  - возвращает наиболее подходящий лифт, если таковой был найден
### Класс Randomizer и его наследники PassengerRandomizer и ElevatorRandomizer:
+ случайная величина вес пассажира подчиняется нормальному распределению
+ текущий и желаемый этаж пассажира не должны совпадать
+ грузоподъёмность лифта случайно выбирается из списка возможных грузоподъёмностей
### Класс ElevatorSpawn:
+ возвращает следующий рандомный лифт
### Класс PassengerSpawn:
- отдельный поток, который генерирует пассажиров с заданной частотой
### Потоки:
  1) Отдельный для каждого лифта
  2) Один для генератора пассажиров
  3) Один для статистики