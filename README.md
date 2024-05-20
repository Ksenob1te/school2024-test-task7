# Тестовое задание для отбора на Летнюю ИТ-школу КРОК по разработке

## Условие задания
Будучи тимлидом команды разработки, вы получили от менеджера проекта задачу повысить скорость разработки. Звучит, как начало плохого анекдота, но, тем не менее, решение вам все же нужно найти. В ходе размышлений и изучений различного внешнего опыта других команд разработки вы решили попробовать инструменты геймификации. То есть применить техники и подходы игрового характера с целью повышения вовлеченности команды в решение задач.

Вами была придумана рейтинговая таблица самых активных контрибьютеров за спринт. Что это значит в теории: по окончании итерации (4 рабочие недели) выгружается список коммитов, сделанных в релизную ветку продукта, и на его основе вычисляются трое самых активных разработчиков, сделавших наибольшее количество коммитов. В зависимости от занятого места, разработчик получает определенное количество внутренней валюты вашей компании, которую он впоследствии может обменять на какие-то товары из внутреннего магазина.

На практике вы видите решение следующим образом: на следующий день после окончания спринта в 00:00 запускается автоматическая процедура, которая забирает файл с данными о коммитах в релизную ветку, сделанных в период спринта, после чего выполняется поиск 3-х самых активных контрибьютеров. Имена найденных разработчиков записываются в файл, который впоследствии отправляется вам на почту.

В рамках практической реализации данной задачи вам необходимо разработать процедуру формирование отчета “Топ-3 контрибьютера”. Данная процедура принимает на вход текстовый файл (commits.txt), содержащий данные о коммитах (построчно). Каждая строка содержит сведения о коммите в релизную ветку в формате: “_<Имя пользователя> <Сокращенный хэш коммита> <Дата и время коммита>_”.
Например: AIvanov 25ec001 2024-04-24T13:56:39.492

К данным предъявляются следующие требования:
- имя пользователя может содержать латинские символы в любом регистре, цифры (но не начинаться с них), а также символ "_";
- сокращенный хэш коммита представляет из себя строку в нижнем регистре, состояющую из 7 символов: букв латинского алфавита, а также цифр;
- дата и время коммита в формате YYYY-MM-ddTHH:mm:ss.

В результате работы процедура формирует новый файл (result.txt), содержащий информацию об именах 3-х самых активных пользователей по одному в каждой строке в порядке убывания места в рейтинге. Пример содержимого файла:
AIvanov
AKalinina
CodeKiller777

Ручной ввод пути к файлу (через консоль, через правку переменной в коде и т.д.) недопустим. Необходимость любых ручных действий с файлами в процессе работы программы будут обнулять решение.

## Автор решения
Скалин Иван

## Описание реализации
### Проект структурирован соответственно:

    .
    ├── ...
    ├── app                     # main app dir, also a module for running the code
    │   ├── file_manager.py     # functions for the file manipulation 
    │   ├── task.py             # ContributorList class is placed here, also main task func is placed here
    │   └── classes
    │       ├── commits.py      # Commit class is here
    │       └── contributor.py  # Contributor class is here
    ├── test
    │   ├── generator.py        # script for sample generation
    │   └── tests.py            # unit tests for the classes
    └── main.py                 # script for running the code

### Основные моменты
В основу для реализации легли 3 класса:
  - Commit
  - Contributor
  - ContributorList

Их свойства ясны из названий, соответственно Commit класс используется для унификации и проверки данных о коммите, Contributor - для проверки кортрибутора, а так же для сохранения его коммитов, а ContributorList уже специфичный для задачи класс, который обрабатывает конкретный формат файла

Для валидации данных были использованны библиотеки 'regex' и 'datetime', для валидации (имен и хэшей) и (даты) соответственно, в случае попадания ошибочной строки во входной файл, она будет пропущена и будет зафиксированна ошибка, на работоспособность программы это не повлияет

Также вспомогательно были реализованы методы для взаимодействия с файлами, базовые алгоритмы тестирования и генерации тестовых строк, реализованна система логирования для основных процедур

### Описание классов

- class Commit
    * при создании обязан иметь валидный `hash` и `date`, иначе будет вызван exception ValueError при создании объекта
    * имеет метод `validate()` который возвращает `bool`, означающий валиден ли данный коммит

- class Contributor
    * аналогично с классом коммита, необходимо при создании указать валидное имя пользователя `name`, иначе аналогично будет вызван exception ValueError
    * имеет 2 основных метода `add_commit_raw(c_hash: str, dt_str: str)` и `add_commit(commit: Commit)` для добавления коммита конкретному пользователю, которые возвращают `Tuple[bool, str]`, где bool - флаг, обозначающий, добавлена ли строка, и str - строка с сообщением об ошибке
    * ну и метод для получения количества коммитов соответственно `get_commits_amount()`
 
### Генератор строк
Расположенный в директории `test/generator.py`, позволяет генерировать файл с корректными строками для тестирования

### Тестирование
Небольшое количество юнит тестов присутствуют в `test/tests.py`

### Работа с файлами
При работе с файлами было принято решение сделать поисковик входного файла, который ищет его внутри рабочей директории, т.к в задании не было указано, где расположен файл с данными: если программа не сможет найти файл в корневой директории, она будет пытаться найти его внутри всех вложенных директорий, и завершит работу, если файл так и не будет найден

Результирующий файл будет записан под соответствующим названием в корневую директорию
Также будет создан файл с логами, в корневой директории, внутрь которого будут заноситься сообщения об ошибках и информационные сообщения

## Инструкция по сборке и запуску решения
Для запуска решения необходимо иметь `python 3.10.11` (иначе могут возникнуть конфликты) или выше

В корневой директории запуска решения необходимо иметь файл `commits.txt` (файл может быть расположен и во вложенных директориях решения)

После завершения выполнения решения в корневую директорию будут записаны файлы `result.txt` и `log.txt`

Запуск может происходить в двух вариантах:
- Находясь в корневой директории проекта можно запустить скрипт `main.py` соответственно командой `python ./main.py`
- Альтернативный вариант запуска - запустить модуль `app`, соответственно командой `python -m app`

Также можно запустить юнит тестирование:
- запустить модуль `test`: `python -m test`
