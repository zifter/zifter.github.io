Это небольшой мануал/история о том, как создать "идеальный" pypi пакет для python, который каждый желающий сможет установить заветной командой:
```
pip install my-perfect-package
```
Ориентирована на новичков, но призываю и профессионалов высказать свое мнение, как можно улучшить "идеальный" пакет. Поэтому прошу под кат.
<cut/>

## Что значит "идеальный" пакет?
Буду исходить из следующих требований:

* **Open source на github;**  
   Каждый должен иметь возможность внести свой вклад в развитие и поблагодарить автора.
* **Поддержка всех актуальных\популярных версий python (2.7, 3.5, 3.6, 3.7, 3.8);**  
   Питоны бывают разные, и где-то до сих пор активно пишут на 2.7.
* **100% покрытие юнит тестами;**    
   ~~Юнит тесты улучшает архитектуру, позволяет автоматизировать регрессионные проверки.~~  
   Бейдж с заветным числом повышает ЧСВ и [задает планку другим](https://cmustrudel.github.io/papers/icse18badges.pdf).
* **Использование CI:**  
   Автоматические проверки - это очень удобно! ~~А еще куча клевых бейджей~~
  * **Запуск юнит тестов на всех платформах и на всех версиях питона;**  
     Не стоит верить тем, кто утверждает, что питон и устанавливаемые пакеты - [кроссплатформенные](https://docs.python.org/2/library/os.html), ведь всегда можно натолкнуться на [баг](https://bugs.python.org/issue22587).
  * **Проверка код стайла;**  
     Единый стиль улучшает читаемость и уменьшает количество пустых дискуссий в ревью.
  * **Статический анализатора кода;**  
     Автоматический поиск багов в коде? Дайте два!
* **Актуальная документация;**  
    Примеры работы с пакетом, описание методов\классов и разбор типичных ошибок - задокументированный опыт позволят снизить порог входа для новичков.
* **Кроссплатформенность разработки;**  
   К сожалению, в проекты сложно вносить личный вклад просто потому, что разработчик заточил инструменты под Unix. К примеру, для сборки использовал bash скрипты.
* **Пакет полезен и делает мир лучше.**  
   Сложное требование, так как судя по количеству пакетов в [pypi](https://pypi.org/) *(~210к)* разработчики - дикие альтруисты и многое уже написано.

## С чего начать?
Хороших идей не было, поэтому тему выбрал избитую и очень популярную - работа со системами счисления. Первая версия должна уметь переводить числа в римские и обратно. Для мануала сложнее и не нужно. Ах, да, самое важное — это название: ~~numsys - как расшифровка numeral systems.~~ *numeral-system-py*.

## Как тестировать?
Взял *python3.7* и первым делом написал тесты с заглушками функций (мы ведь все за [TDD](https://en.wikipedia.org/wiki/Test-driven_development)) с использованием стандартного модуля [unittests](https://habr.com/ru/post/121162/).

Делаю следующую структуру проекта:
```
src/
    numeral-system/
        __init__.py
        roman.py
    tests
        __init__.py
        test_roman.py
```
Тесты в пакет класть не буду, поэтому отделяю ~~зёрна от плевел~~. Изначально папку `src/` не создавал, но дальнейшее развитие показало, что мне так удобнее оперировать. Это не обязательно, поэтому по желанию.

Для запуска решил использовать [pytest](https://habr.com/ru/post/269759/) - он умеет отлично работать с тестами из стандартного модуля. Выглядит, возможно, немного нелогично, но стандартный модуль для тестов мне ~~кажется~~ казался чуть удобнее. *Сейчас бы я советовал использовать pytest стиль написания.*

Но, поговаривают, что ставить `pytest` (как и любые другие зависимости) в системный python - не очень умная идея...

## Как управлять зависимостями?
Можно использовать только [virtualenv](https://virtualenv.pypa.io/en/latest/) и `requirements.txt`. Можно быть прогрессивным и использовать [poetry](https://poetry.eustace.io/). Я же, пожалуй, воспользуюсь [tox](https://tox.readthedocs.io/en/latest/) - средство для упрощения автоматизации и тестирования, который также позволит мне управлять зависимостями.

Создаю простой конфиг `tox.ini` и устанавливаю `pytest`:
```
[tox]
envlist = py37  ; запускать на одной предопределенной среде, а именно python3.7

[testenv]  ; секция описания тестового окружения
deps = секция `deps` описываются зависимости, которые требуется доставить.
    -r requirements.txt ; доставить зависимости самого пакета
    -r requirements-test.txt ;
commands = pytest  ; запускаем тесты
```
Изначально, я явно указывал зависимости, но практика интеграции со сторонними сервисами показала, что лучшим способом будет все-таки хранение зависимостей в `requirements.txt` файле.

Возникает очень тонкий момент. Фиксировать актуальную на момент разработки версию или всегда ставить последнюю?

Если фиксировать, то при установке могут возникнуть конфликты между пакетами из-за различных версий используемых зависимостей. Если же не фиксировать, то пакет может неожиданно перестать работать. Последняя ситуация очень неприятная для конечных продуктов, когда в одну ночь все билды могут "покраснеть" из-за минорного обновления неявной зависимости. И по [закону Мерфи](https://en.wikipedia.org/wiki/Murphy%27s_law) это произойдет в день релиза.

Поэтому для себя выработал правило:
1. Всегда фиксировать версию для конечных продуктов, так как какие версии использовать — это их ответственность.
1. Не фиксировать используемую версию для устанавливаемых пакетов. Или ограничивать диапазоном, если того требует функционал пакета.

## Что дальше?
Пишу тесты!

Заполняю тело функций, добавляю комментарии и заставляю тесты корректно выполняться.
На этом моменте обычно большинство разработчиков останавливается (я все-таки верю, что все пишут тесты =), публикуют пакет и отгребают баги. Но я иду дальше ~~и прыгаю в кроличью нору~~.

## Как работать с различными версиями python?
В конфигурации `tox` указываю запуск тестов на всех интересующих версиях python:
```
[tox]
envlist = py{27,35,36,37,38}
```
С помощью [pyenv](https://khashtamov.com/ru/pyenv-python/) доставляю нужные версии к себе локально, чтобы `tox` мог их найти и создать тестовые среды.

## Где заветные 100%?
Добавлю замер покрытия кода - для этого есть отличный пакет [coverage](https://coverage.readthedocs.io/en/v4.5.x/) и не менее прекрасная интеграция с pytest - [pytest-cov](https://github.com/pytest-dev/pytest-cov).
`requirements-test.txt` выглядит теперь так:
```
six=1.13.0
pytest=4.6.7
pytest-cov=2.8.1
parameterized=0.7.1
```
Согласно вышеуказанному правилу, фиксирую версии пакетов, которые используются для запуска тестов.

Меняю команду запуска тестов:
```
deps =
    -r requirements.txt
    -r requirements-test.txt
commands = pytest \           
    --cov=src/ \                     
    --cov-config="{toxinidir}/tox.ini" \           
    --cov-append
```
Делаю сбор статистики покрытия для всего кода из папки `src/` - самого пакета (*numeral_system/*) и обязательно для кода тестов (*tests/*) - я же не хочу, чтобы сами тесты содержали не исполняющиеся части?

Командой `--cov-append` всю собранную статистику для каждого вызова под различной версией python суммирую в одну, потому что покрытие для второго и третьего питона может быть различным (привет зависимый от версии код и модуль [six](https://pypi.org/project/six/)!), но по итогу в сумме давать 100%. Простой пример:
```python
if sys.version_info > (3, 0):
    # Python 3 code in this block
else:
    # Python 2 code in this block
```

Добавляю новую среду для создания coverage отчета.
```
[testenv:coverage_report]
deps = coverage
commands =
    coverage html  ; данные уже есть, построим отчет
    coverage report --include="src/*" --fail-under=100 -m  ; падать если не 100% покрытие
```
И добавляю в список сред после выполнения тестов на всех версиях питона.

```
[tox]
envlist =
    py{27,35,36,37,38}
    coverage_report
```
После запуска команды `tox` в корне проекта должна появится папка `htmlconv` содержащая файл `index.html` с красивым отчетом.

Для заветного бейджа в 100% интегрирую с сервисом [codecov](https://codecov.io/), который сам уже сделает интеграцию с `github` и позволит просмотреть историю изменения покрытия кода. Для этого, конечно же, придется завести там аккаунт.

Итоговая среда запуска выглядит следующим образом:
```
[testenv:coverage_report]
deps =
    coverage==5.0.2
    codecov==2.0.15
commands =
    coverage html
    coverage report --include="src/*" --fail-under=100 -m
    coverage xml
    codecov -f coverage.xml --token=2455dcfa-f9fc-4b3a-b94d-9765afe87f0f  ; Токен моего проекта в codecov, смотреть в аккаунте
```

Осталось теперь только добавить ссылку на бейдж в `README.rst`:
```
|Code Coverage|

.. |Code Coverage| image:: https://codecov.io/gh/zifter/numeral-system-py/branch/master/graph/badge.svg
    :target: https://codecov.io/gh/zifter/numeral-system-py
```

## Как форматировать и анализировать код?
Много анализаторов не бывает, потому что они, по большей части, дополняют друг друга. Поэтому буду интегрировать популярные статические анализаторы, которые проверят на соответствие [PEP8](https://www.python.org/dev/peps/pep-0008/), найдут потенциальные проблемы и ~~пофиксят все баги~~ единообразным образом отформатируют код.

Сразу следует продумать, где указывать параметры для тонкой настройки анализаторов. Для этого можно использовать файл `tox.ini`, `setup.cfg`, единый кастомный файл или же конкретные файлы для анализаторов. Я решил воспользоваться непосредственно `tox.ini`, так как можно просто копировать `tox.ini` для будущих проектов.

## isort
[isort](https://github.com/timothycrosley/isort) - утилита для форматирования импортов.

Создаю следующую среду для запуска `isort` в режиме форматирования кода.
```
[testenv:isort]
changedir = {toxinidir}/src
deps = isort==4.3.21
commands = isort -y -sp={toxinidir}/tox.ini
```
К сожалению, `isort` нельзя указать папку для форматирования. Поэтому приходится менять директорию запуска через `changedir` и указывать путь к файлу с настройками `-sp={toxinidir}/tox.ini`. Ключ `-y` нужен, чтобы отключить интерактивный режим.

Для запуска в тестах нужен режим проверки - для этого есть флаг `--check-only`:
```
[testenv:isort-check]
changedir = {toxinidir}/src
deps = isort==4.3.21
commands = isort --check-only -sp={toxinidir}/tox.ini
```

## black
Далее интегрирую с форматером кода [black](https://black.readthedocs.io/en/stable/). Делаю по аналогии с `isort`:

```
[testenv:black]
deps = black==19.10b0
commands = black src/

[testenv:black-check]
deps = black==19.10b0
commands = black --check src/
```
Все хорошо работает, но возникает конфликт с `isort` - есть различие в [форматировании импортов](https://github.com/psf/black/issues/127). 

В [одном из комментариев](https://github.com/timothycrosley/isort/issues/694) нашел минимально совместимую настройку `isort`, которой и воспользовался:
```
[isort]
multi_line_output=3
include_trailing_comma=True
force_grid_wrap=0
use_parentheses=True
line_length=88
```

## flake8
Далее интегрирую со статическими анализаторами [flake8](http://flake8.pycqa.org/en/latest/).
```
[testenv:flake8-check]
deps = flake8==3.7.9
commands = flake8 --config=tox.ini src/
```
Снова возникают [проблемы с интеграцией](https://github.com/psf/black/issues/113) с `black`. Приходится добавить тонкую настройку, которую, собственно, и [рекомендует](https://github.com/psf/black) сам `black`:
```
[flake8]
max-line-length=88
ignore=E203
```
К сожалению, с первого раза не сработало. Упало с ошибкой `E231 missing whitespace after ','`, пришлось добавить в игнор и эту ошибку:
```
[flake8]
max-line-length=88
ignore=E203,E231
```

## pylint
Интегрирую со статическими анализаторами кода [pylint](https://www.pylint.org/)
```
[testenv:pylint-check]
deps =
    {[testenv]deps}  # pylint проверят зависимости, поэтому следует их устанавливать
    pylint==2.4.4
commands = pylint --rcfile=tox.ini src/
```

Сразу же сталкиваюсь со странными ограничениями - имена функций в 30 символов (да, я пишу очень длинные имена тестовых методов) и предупреждения на наличие `TODO` в коде. 
Приходится добавить пару исключений:
```
[MESSAGES CONTROL]
disable=fixme,invalid-name
```

Так же неприятный момент в том, что разработчики `pylint` уже похоронили `python2.7` и не развивают больше пакет для него. Поэтому проверки стоит запускать на актуальном пакете для `python3.7`.
Добавляю соответствующую строчку в конфигурацию:
```
[tox]
envlist =
    isort-check
    black-check
    flake8-check
    pylint-check
    py{27,35,36,37,38}
    coverage_report
basepython = python3.7
```
Это так же важно для запуска тестов на различных платформах, так как дефолтная версия питона в системах CI различная.


## Что там с CI?
### Appveyor
Интегрирую с [appveyor](https://www.appveyor.com/) - CI под windows. Первичная настройка простая - все можно сделать в интерфейсе, затем скачать yaml файл и закоммитеть его в репозиторий.
```
version: 0.0.{build}
install:
- cmd: >-
    C:\\Python37\\python -m pip install --upgrade pip
    C:\\Python37\\pip install tox
build: off
test_script:
- cmd: C:\\Python37\\tox
```
Здесь я явно указываю версию `python3.7`, так как по умолчанию будет использован `python2.7` (и `tox` так же будет использовать эту версию, хоть я явно и указал `python3.7`).
Ссылка на бейдж, как обычно, добавляется в `README.rst`

```
|Build status Appveyor|

.. |Build status Appveyor| image:: https://ci.appveyor.com/api/projects/status/github/zifter/numeral-system-py?branch=master&svg=true
    :target: https://ci.appveyor.com/project/zifter/numeral-system-py
```

### Travis CI
После, интегрирую с [Travis CI](https://travis-ci.org/) - CI под Linux (и под MacOS c Windows, но [`Python builds are not available on the macOS and Windows environments`](https://docs.travis-ci.com/user/languages/python/#what-this-guide-covers). Настройка чуть сложнее, так как конфигурационный файл будет использоваться непосредственно из репозитория. Пару итераций проб и ошибок - конфигурация готова. Сквошу в один красивый коммит и merge request готов.
```
language: python
python: 3.8  #

dist: xenial    # required for Python 3.7 (travis-ci/travis-ci#9069)
sudo: required  # required for Python 3.7 (travis-ci/travis-ci#9069)

addons:
  apt:
    sources:
      - deadsnakes
    packages:
      - python3.5
      - python3.6
      - python3.7
      - pypy

install:
  - pip install tox

script:
  - tox
```
(*Риторический вопрос: И почему CI проектам так нравится yaml формат?*)

Указываю версию `python3.8`, так как установить ее через `addon` корректно не получилось, а `Travis CI` успешно создает `virtualenv` с указанной версии.

Люди, знакомые с `Travis CI`, могут вопросить, почему таким образом явно не указать версии python? Ведь `Travis CI` создает автоматически `virtualenv` и выполнит нужные команды в нем.

Причина в том, что нам нужно собрать данные по покрытию кода со всех версий. Но тесты будут запущены в разных джобах параллельно, из-за чего собрать общий отчет по покрытию не получится.

Конечно же, я уверен, что чуть больше разобравшись и это можно исправить.

По традиции, ссылка на бейдж так же добавляется в `README.rst`
```
|Build Status Travis CI|

.. |Build Status Travis CI| image:: https://travis-ci.org/zifter/numeral-system-py.svg?branch=master
    :target: https://travis-ci.org/zifter/numeral-system-py
```

## Документация
Думаю, каждый python разработчик хоть раз пользовался сервисом - [readthedocs.org](https://readthedocs.org/). Мне кажется, что это лучший сервис для хостига своей документации.
Воспользуюсь стандартным средством для генерации документации [Sphinx](http://www.sphinx-doc.org/en/master/). Выполняю шаги из [стартового мануала](https://docs.readthedocs.io/en/stable/intro/getting-started-with-sphinx.html) и получаю следующую структуру:
```
src/  
docs/
    build/  # здесь будет располагаться документация в html формате
    source/  # исходники для генерации документации
        _static/  # сюда положим статику, например, картинки
        _templates/  # шаблоны для генерации документации
        conf.py  # настройка генерации документов
        index.rst  # описание стартовой страницы
    make.bat  
    Makefile  # make для сборки с помощью make
```
Далее нужно проделать минимальные шаги для настройки:
1. `github` по умолчанию предлагает создать `README.md` файл в формате [Markdown](https://en.wikipedia.org/wiki/Markdown), когда как `sphinx` по умолчанию предлагает использовать [ReStructuredText](https://en.wikipedia.org/wiki/ReStructuredText).

Поэтому пришлось переписать его в формате `.rst`. ~~А если бы хоть раз дочитал до конца стартовый мануал, то понял, что `sphinx` умеет и в Markdown~~.
Включаю файл `README.rst` в `index.rst`
```
.. include:: ../../README.rst
```
1. Для автогенерации документации из комментариев в исходниках добавляю расширение [sphinx.ext.autodoc](http://www.sphinx-doc.org/en/master/usage/extensions/autodoc.html).
1. Добавляю папку с пакетом в `conf.py`. Это позволит `sphinx` делать импорты нашего кода для анализа.
```
import os
import sys
sys.path.insert(0, os.path.abspath('./../../src'))
import numeral_system
```
1. Добавляю папку `docs/source/api-docs` и закидываю туда файл описания каждого модуля. Документация должна автомагически сгенерироваться из комментариев:
```
Roman numeral system
=========================
.. automodule:: numeral_system.roman
   :members:
```

После этого проект готов явить миру свое описание. Нужно создать аккаунт (лучше через аккаунт на `github`) и импортировать свой проект, подробные шаги описаны в [инструкции](https://github.com/readthedocs/readthedocs.org/blob/master/README.rst).
По традиции создаю среду в `tox`:
```
[testenv:gen_docs]
deps = -r docs/requirements.txt
commands =
    sphinx-build -b html docs/source/ docs/build/
```
Использую команду `sphinx-build` явно, вместо `make`, так как ее под Windows нет. А я не хочу нарушать принцип о кроссплатформенной разработке.

Как только сделанные изменения замержены, `readthedocs.org` автоматически соберет документацию и опубликует.

Но... [`Build failed`](https://github.com/readthedocs/readthedocs.org/issues/2569). Я не зафиксировал версии `sphinx` и `sphinx_rtd_theme`, и ожидал что `readthedocs.org` возьмет актуальные версии. Но это не так. Фиксирую:
```
sphinx==2.3.1
sphinx_rtd_theme==0.4.3
```
И создаю специальный конфиг файл `.readthedocs.yml` для `readthedocs.org`, в котором описываю среду для запуска билда:
```
python:
   version: 3.7
   install:
      - requirements: docs/requirements.txt
      - requirements: requirements.txt
```
Вот здесь как раз и пригодился тот факт, что зависимости лежат в `requirements.txt` файлах. Дожидаюсь билда и [документация становится доступной](https://numeral-system-py.readthedocs.io/en/latest/).

Снова добавляю бейдж:
```
|Docs|
.. |Docs| image:: https://readthedocs.org/projects/numeral-system-py/badge/?version=latest&style=flat
    :target:  https://numeral-system-py.readthedocs.io/en/latest/
```

## Лицензирование
Стоит подумать о выборе лицензии для пакета.
Это очень обширная тема, поэтому ознакомился c [этой статьей](https://habr.com/ru/post/243091/). В принципе, выбор стоит между [MIT](http://directory.fsf.org/wiki/License:Expat) и [Apache 2.0](http://directory.fsf.org/wiki/License:Apache2.0). Мне понравилась удачно вырванная из контекста фраза:
```
MIT предлагают использовать лишь для небольших проектов
```
Согласен полностью, так и поступлю Если планы поменяются, можно без проблем сменить лицензию (правда, предыдущие версии будут под старой).

Опять же, добавлю бейдж ~~богу бейджей~~:
```
|License|
.. |License| image:: https://img.shields.io/badge/License-MIT-yellow.svg
    :target:  https://opensource.org/licenses/MIT
```
[Здесь](https://gist.github.com/lukas-h/2a5d00690736b4c3a7ba) можно найти бейджи для всех лицензий.

## Как залить на pypi?
Для начала нужно завести аккаунт на [pypi.org](https://pypi.org). Затем приступить к подготовке пакета.

### Создаю setup.cfg
Нужно корректно описать конфигурацию для установки\сборки пакета. Я следовал [инструкции](https://docs.python.org/3/distutils/introduction.html). Есть возможность задать данные через `setup.py`, но некоторые параметры задать возможности нет. Поэтому воспользоваться `setup.cfg` файлом, в котором можно указать все нюансы. Нашел [небольшой шаблон](https://gist.github.com/althonos/6914b896789d3f2078d1e6237642c35c) того, как заполнять этот файл. По итогу использую и тот и тот файл - так удобнее.

Этот файл также можно использовать для конфигурации `pylint`, `flake8` и прочих настроек, но я так не делал.

### Как собрать пакет?
Снова пишу среду, которая поможет мне собрать необходимый пакет:
```
[testenv:build_wheel]
skip_install = True
deps = ; зависимости для сборки проекта
    wheel  
    docutils
    pygments
commands =    
    python -c 'import shutil; (shutil.rmtree(p, ignore_errors=True) for p in ["build", "dist"]);'
    python setup.py sdist bdist_wheel
```
Зачем я удаляю папки с помощью python? Хочу соблюсти требование о кроссплатформенности разработки, удобного пути сделать это под Windows и Unix нет.

Запускаю тестовую среду:
```
tox -e build_wheel
```
В результате в папке `dist` получаю:
```
dist/
    numeral-system_py-0.1.0.tar.gz
    numeral_system-py-0.1.0-py2.py3-none-any.whl
```

### Заливаю!
Не совсем.

Для начала стоит проверить, что пакет работает корректным образом. Залью в тестовый репозиторий пакетов. Поэтому нужно завести еще один аккаунт, но уже на [test.pypi.org](https://test.pypi.org/).

Использую для этого пакет [twine](https://pypi.org/project/twine/) - инструмент для заливки артефактов в PyPi.
```
[testenv:test_upload]
skip_install = True
deps = twine ; ставлю последнюю версию
commands =    
     python -m twine upload --repository-url https://test.pypi.org/legacy/ dist/*
```
Изначально проект назывался `numsys`, но при попытке заливки столкнулся с тем, что пакет с таким именем уже есть! И что самое обидное - он тоже умеет конвертировать в римские цифры:) Сильно не расстроился и переименовал в `numeral-system-py`.

Теперь нужно установить пакет из тестового окружения. Проверку так же стоит автоматизировать:
```
[testenv:test_venv]
skip_install = True  ; не следует ставить текущий пакет в эту среду
deps = ; пустые зависимости
commands =
    pip install -i https://test.pypi.org/simple/ numeral-system-py
```

Теперь нужно только запустить:
```
tox -e test_venv
...
test_venv: commands_succeeded
congratulations :)
```
Вроде работает :)

### Вот теперь точно заливаю!
Да.

Создаю среду для заливки в production репозиторий.
```
[testenv:pypi_upload]
skip_install = True
deps =
    twine
commands =
    python -m twine upload dist/*
```

И среду для production проверки.
```
[testenv:pypi_venv]
skip_install = True
deps = ; не ставим зависимости
commands =
    pip install numeral-system-py
```

### А все точно работает?
Проверяю простыми командами:
```
> virtualenv venv
> source venv/bin/activate
(venv) > pip install numeral-system-py
(venv) > python
>>> import numeral_system
>>> numeral_system.roman.encode(7)
'VII'
```
Все отлично!

[Срезаю релиз на github](https://help.github.com/en/articles/creating-releases), собираю пакет и заливаю в продовский pypi.

## Замечания
### Обновления зависимостей
За время подготовки этой статьи была выпущена новая версия pytest, в которой, по факту, дропнули поддержку python 3.4 ([на самом деле](https://github.com/tox-dev/tox/issues/1483) в пакете [colorama](https://github.com/tartley/colorama)). Вариантов было два:
1. Зафиксировать версию colorama, совместимую с 3.4;
1. Дропнуть поддержку 3.4 :)

В пользу второго варианта последним аргументом стало то, что и `pip` дропнул поддержку 3.4 в версии 19.1.

Так же есть зафиксированные зависимости в виде анализаторов, форматеров и прочих сервисов. Эти зависимости можно обновлять одновременно. Если повезет, то отделаетесь только подъёмом версией, если нет - то придется подправить код или даже дописать настройки.

### TravisCI
Не поддерживает python для MacOS и Windows. Есть сложности с запуском `tox` под все версии питона в рамках одной джобы.

### Версия пакета
Нужно придерживаться [семантического версионирования](https://semver.org/), а именно формата:
```
MAJOR.MINOR.PATCH
```

### Дублированиe мета информации
Версию пакета и некоторые другие параметры требуется указывать для установки пакета (в `setup.cfg` или `setup.py`) и в документации. Чтобы избежать дублирования, сделал указание только в пакете `numeral_system/__init__.py`:
```
__version__ = '0.2.0'
```
А затем в `setup.py` явно использую эту переменную
```
setup(version=numeral_system.__version__)
```
Тоже верно и для в `docs/source/conf.py`
```
release = numeral_system.__version__
```
Вышеописанное справедливо для любой мета информации - `REAMDE.rst`, описания проекта, лицензии, имена автора и прочего.

Правда, это приводит к тому, что происходит импорт пакета в момент сборки, что может быть нежелательным.

### Дублирование зависимостей
Вначале работы меня смущал тот факт, что мне нужно указывать зависимости для пакета в `requirements.txt` и `setup.cfg`.
Затем почитал [отличную статью](https://caremad.io/posts/2013/07/setup-vs-requirement/), которая разъяснила - указывать нужно только в `setup.cfg`.
Есть так же дублирование версий анализаторов. Думаю, это можно исправить созданием файла `requirements-dev.txt` в будущем.

### Организация исходников
Может быть спорным то, что я выделил исходники в папку `src/`. Я исходил из следующих соображений:
* Легче указывать только одну папку различным инструментам;
* Не раздувать корень проекта, так как со временем количество файлов и папок конфигурации в корне только увеличивается, что затруднит ориентирование;

Но столкнулся с некоторыми неудобствами:
* Потребуется дополнительная настройка `PyCharm` (или что вы там используете?) - указать папку `src`, как папку исходников. 

### Коммиты и история изменений
Во время разработки пакета совершенно забыл про очень важную вещь - адекватная история изменений.
Сейчас [в истории](https://github.com/zifter/numeral-system-py/commits/master) есть комментарии, за которые не сильно стыдно:
```
Add badge with supported version
Support for py38
```
Но есть и совсем треш:
```
Try fix py38 env creating
Try fix py38 env creating
Try fix py38 env creating
Fix check
```
Да, 3 коммита с одним и тем же комментарием! Уж очень хотел пофиксить запуск тестов на Travis CI.

Есть [отличная статья](https://habr.com/ru/post/416887/) по тому, как правильно оформлять комментарии. От себя хочу добавить - часто имеет смысл сквошить изменения, чтобы не было кучи однотипных коммитов без смысловой нагрузки.

Стоит пристально следить за историей изменений и за коммитами, так как это очень важно для дальнейшей поддержки проекта. Для удобства можно так же вести `CHANGELOG`.

### Документация
Многие сервисы по умолчанию предполагают, что используется `Markdown`. Поэтому советовал бы использовать его, если нет явной необходимости в `rst`.

### Бейджи
Бейджи помогают быстрее оценить проект - поддерживаемые версии, статус тестов, номер стабильной версии, контакты для связи и многое другое. Это мощный социальный инструмент, который способствует более высокому [качеству проекта](https://cmustrudel.github.io/papers/icse18badges.pdf).
Советую взглянуть на описание [coverage](https://pypi.org/project/coverage/).

### Ограничения из-за использования разных версий
Разница в функционале, синтаксисе и прочем между версиями, к сожалению, велика. Если модуль `six` поможет в общих конструкциях, то использовать функционал из последних версий такие, как `type hint`, `asyncio` и прочих чудес - нет.
Правильным решением будет отказ от `python2.7`, учитывая что [поддержка закончилась](https://pythonclock.org/). По субъективному впечатлению, оставив поддержку только python3.5+, можно значительно упростить разработку пакета. Например, не придется собирать в одной джобе результаты покрытия кода, что ускорит запуск тестов в CI благодаря параллельному выполнению. Я уже не говорю о различных фичах в самом языке.

## Можно ли сделать лучше?
На этом останавливаться все равно не стоит, можно сделать еще улучшения:
* Добавить поддержку MacOS;
* Тестирования на python x64 и x86;
* Добавить перфоманс регрессию;
* 100% покрытие кода далеко не показатель качества, тесты должны покрывать [все ветки исполнения](https://en.wikipedia.org/wiki/Cyclomatic_complexity). Можно мерить с помощью [mccabe](https://pypi.org/project/mccabe/), но последний релиз в январе 2017 года. Как еще можно мерить?
* Зависимости обновляются и это следует отслеживать через [PyUp](https://PyUp.io) или [requires.io](https://requires.io/). Как лучше?
* Нужно больше статик анализаторов. Смотрел непосредственно в репозиториях [Python Code Quality Authority](https://github.com/PyCQA0), что-нибудь еще?
  * [mypy](http://mypy-lang.org/) - анализатор типов;
  * [bandit](https://github.com/PyCQA/bandit) - тулза для анализа поиска проблем с безопасностью;
  * [Pyflakes](https://github.com/PyCQA/pyflakes) - по всей видимости уже устаревший анализатор;
  * [prospector](https://github.com/PyCQA/prospector) - инструмент, интегрирующий `pylint`, `pep8`, `mccabe`. Выглядит, что так же устарел;
* Добавить запуск форматирования\анализ кода на прекоммит?

## Заключение
Придется написать не одну библиотеку и поконтрибьютить не в один open source проект, чтобы разобраться со всеми тонкостями.
Во время подготовки к статье я прочитал не один десяток мануалов, посмотрел еще больше python проектов на `github`, протестировал разные сервисы и их варианты конфигурации.
Но больше всего времени я провел на `stackoverflow.com` и issues используемых проектов на `github` так все эти сервисы так и норовят упасть с очередной непонятной ошибкой. 
Добавление новых зависимостей в проект требует время на интеграцию и поддержку. Поэтому стоит рационально оценивать необходимость интеграций. А когда следует остановиться с интеграциями?..

Тем, кто заинтересовался, советую так же прочитать [статью с похожей тематикой](https://habr.com/ru/company/ruvds/blog/444344/).
Перед завершением своей статьи нашел очень похожую, которую так же [рекомендую посмотреть](https://sourcery.ai/blog/python-best-practices/).

[Проект можно потрогать на github](https://github.com/zifter/numeral-system-py). 

Спасибо!