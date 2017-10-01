Протоколирование
=======

.. image:: https://farm5.staticflickr.com/4246/35254379756_c9fe23f843_k_d.jpg

Модуль протоколирования был частью стандартной библиотеки Python версии 2.3. Он кратко описан в PEP 282. Документацию, как известно, трудно читать, за исключением основного учебника по ведению журнала.
Ведение журнала необходим для  двух целей:

- **Журнал диагностики** записывает события, связанные с работой приложения. Например, если пользователь вызывает сообщение об ошибке, журналы можно искать в контексте.
- ** Аудит ведения журнала ** записывает события для бизнес-анализа. Транзакции пользователя могут быть извлечены и объединены с другими данными пользователя для отчетов или для оптимизации бизнес-целей.

... или Печать?
-------------

Единственный раз, когда ``печать`` является лучшим вариантом, чем протоколирование, когда целью является показать справочную информацию для приложения командной строки. Другие причины, почему ведение журнала лучше, чем ``печать``:


- `Запись журнала`_, которая создается с каждым журналом события, содержит легкодоступные диагностические сведения, такие как имя файла, полный курс, функция и номер строки события протоколирования.

- События, зарегистрированные в включенных модулях, автоматически доступны через корневой регистратор в поток ведения журнала приложения, если их не отфильтровать.
- Ведение журнала можно выборочно заглушить с помощью метода 
  `logging.Logger.setLevel` или отключить, установив атрибут 
  `logging.Logger.disabled` в значение  ``True``.


Регистрация в библиотеке 
--------------------

Примечания по `настройке протоколирования для библиотеки`_ приведены в 
`учебном пособии`_.  Поскольку *пользователь*, а не библиотека, должен диктовать, что происходит, когда случается журнал событий, одно замечание носит повторяющийся характер:

Внимание
    Настоятельно рекомендуется  не добавлять никаких обработчиков,  кроме NullHandler, в регистраторы вашей библиотеки.


Рекомендуется создавать экземпляры регистраторов в библиотеке только с помощью ``__name__`` общая переменная:  модуль ведения журнала создает иерархию регистраторов, используя точечную нотацию, так, что используя ``__name__``, гарантирует отсутствие конфликтов имен. 

Вот пример наилучшего упражнения из `источника запросов`_ -- поместите это в свой  ``__init__.py``

.. code-block:: python

    import logging
    logging.getLogger(__name__).addHandler(logging.NullHandler())


Вход в приложение
-------------------------

`12-факторное приложение <http://12factor.net>`_, авторитетный справочник для эффективной практики в разработке приложений, содержит раздел по `ведению журнала лучшей практики <http://12factor.net/logs>`_. Оно решительно выступает за обработку событий журнала как потока событий и за отправку этого потока событий в стандартные выходные данные, которые будут обрабатываться средой приложения. 


Существует, по крайней мере, три способа настройки регистратора:


- Использование Using an INI-formatted файла:
    - ** За**: возможно обновление конфигурации во время работы с помощью функции :func:`logging.config.listen` чтобы воспроизводить на сокете.
    - ** Против**: меньше управления (например, пользовательские подклассовые фильтры или регистраторы), чем это возможно при настройке регистратора в коде.
- Использование словаря или файла в формате JSON:
    - **За**: в дополнение к обновлению во время работы, можно загрузить из файла с помощью модуля `json`, в стандартной библиотеке начиная с Python 2.6. 
    - **Против**: меньше управления, чем при настройке регистратора в коде.
- Использование кода:
    - **За**: полный контроль над конфигурацией.
    - **Против**: модификации требуют изменения исходного кода.


Пример конфигурации через INI-файл
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Скажем, файл называется ``logging_config.ini``.
Более подробная информация о формате файла содержится в ` разделе конфигурации `_
ведения журнала в ` учебном пособии `_.

.. code-block:: ini

    [loggers]
    keys=root
    
    [handlers]
    keys=stream_handler
    
    [formatters]
    keys=formatter
    
    [logger_root]
    level=DEBUG
    handlers=stream_handler
    
    [handler_stream_handler]
    class=StreamHandler
    level=DEBUG
    formatter=formatter
    args=(sys.stderr,)
    
    [formatter_formatter]
    format=%(asctime)s %(name)-12s %(levelname)-8s %(message)s


Затем используется :meth:`logging.config.fileConfig` в коде:

.. code-block:: python

    import logging
    from logging.config import fileConfig

    fileConfig('logging_config.ini')
    logger = logging.getLogger()
    logger.debug('often makes a very good meal of %s', 'visiting tourists')
    

Пример конфигурации через словарь 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Начиная с Python 2.7, вы можете использовать словарь с подробностями конфигурации. :pep:`391` содержит список обязательных и необязательных элементов в словаре конфигурации. 

.. code-block:: python

    import logging
    from logging.config import dictConfig

    logging_config = dict(
        version = 1,
        formatters = {
            'f': {'format':
                  '%(asctime)s %(name)-12s %(levelname)-8s %(message)s'}
            },
        handlers = {
            'h': {'class': 'logging.StreamHandler',
                  'formatter': 'f',
                  'level': logging.DEBUG}
            },
        root = {
            'handlers': ['h'],
            'level': logging.DEBUG,
            },
    )

    dictConfig(logging_config)

    logger = logging.getLogger()
    logger.debug('often makes a very good meal of %s', 'visiting tourists')


Пример конфигурации непосредственно в коде
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

    import logging

    logger = logging.getLogger()
    handler = logging.StreamHandler()
    formatter = logging.Formatter(
            '%(asctime)s %(name)-12s %(levelname)-8s %(message)s')
    handler.setFormatter(formatter)
    logger.addHandler(handler)
    logger.setLevel(logging.DEBUG)

    logger.debug('often makes a very good meal of %s', 'visiting tourists')


.. _basic logging tutorial: http://docs.python.org/howto/logging.html#logging-basic-tutorial
.. _logging configuration: https://docs.python.org/howto/logging.html#configuring-logging
.. _logging tutorial: http://docs.python.org/howto/logging.html
.. _configuring logging for a library: https://docs.python.org/howto/logging.html#configuring-logging-for-a-library
.. _log record: https://docs.python.org/library/logging.html#logrecord-attributes
.. _requests source: https://github.com/kennethreitz/requests
