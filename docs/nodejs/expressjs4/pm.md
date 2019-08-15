# Диспетчеры процессов для приложений Express

При запуске приложения Express для рабочей среды использование *диспетчера процессов* позволяет выполнять следующие задачи:

- Автоматически перезапускать приложение в случае сбоя.
- Получать аналитическую информацию о производительности среды выполнения и потреблении ресурсов.
- Изменять параметры в динамическом режиме в целях повышения производительности.
- Управлять кластеризацией.

Диспетчер процессов можно сравнить с сервером приложений: это "контейнер" для приложений, обеспечивающий развертывание и высокую готовность и позволяющий управлять приложением в среде выполнения.

Наиболее популярные диспетчеры процессов для Express и других приложений Node.js перечислены ниже:

- StrongLoop Process Manager
- PM2
- Forever


Использование любого из этих трех инструментов может быть исключительно полезным, тем не менее StrongLoop Process Manager является единственным инструментом, предоставляющим универсальное решение для среды выполнения и развертывания в течение всего жизненного цикла приложения Node.js и включающим в себя, в виде единого интерфейса, все средства для каждого этапа, как до перехода в рабочий режим, так и в рабочей среде.

Ниже приводится краткий обзор каждого из этих инструментов.
Подробное сравнение можно найти в разделе [http://strong-pm.io/compare/](http://strong-pm.io/compare/).

## StrongLoop Process Manager

StrongLoop Process Manager (StrongLoop PM) - это рабочий диспетчер процессов для приложений Node.js. В StrongLoop PM предусмотрено встроенное распределение нагрузки, мониторинг, развертывание на нескольких хостах и графическая консоль.
С помощью StrongLoop PM можно выполнять следующие задачи:

- Компоновать, объединять в пакет и развертывать приложение Node.js в локальной или удаленной системе.
- Просматривать профайлы CPU и моментальные снимки кучи в целях оптимизации производительности и диагностирования утечек памяти.
- Поддерживать постоянную активность процессов и кластеров.
- Просматривать показатели производительности приложения.
- Без затруднений управлять развертываниями на нескольких хостах с интеграцией Nginx.
- Объединять несколько диспетчеров StrongLoop PM в распределенную среду выполнения микрослужб, управляемую из Arc.

Работать с StrongLoop PM можно посредством многофункционального интерфейса командной строки, называемого `slc`, или графического инструмента Arc. Arc имеет открытый исходный код и обеспечен экспертной поддержкой StrongLoop.

Дополнительная информация содержится на странице [http://strong-pm.io/](http://strong-pm.io/).

Полная версия документации:

- [Управляющие приложения Node (Документация StrongLoop)](http://docs.strongloop.com/display/SLC)
- [Использование StrongLoop Process Manager](http://docs.strongloop.com/display/SLC/Using+Process+Manager).

### Установка

```
$ [sudo] npm install -g strongloop
```

### Основы использования

```
$ cd my-app
$ slc start
```

Просмотр состояния диспетчера процессов и всех развернутых приложений:

```
$ slc ctl
Service ID: 1
Service Name: my-app
Environment variables:
  No environment variables defined
Instances:
    Version  Agent version  Cluster size
     4.1.13      1.5.14           4
Processes:
        ID      PID   WID  Listening Ports  Tracking objects?  CPU profiling?
    1.1.57692  57692   0
    1.1.57693  57693   1     0.0.0.0:3001
    1.1.57694  57694   2     0.0.0.0:3001
    1.1.57695  57695   3     0.0.0.0:3001
    1.1.57696  57696   4     0.0.0.0:3001
```

Вывод списка всех приложений (служб), подлежащих управлению:

```
$ slc ctl ls
Id          Name         Scale
 1          my-app       1
```

Остановка приложения:

```
$ slc ctl stop my-app
```

Перезапуск приложения:

```
$ slc ctl restart my-app
```

Также можно выполнить "мягкий перезапуск", при котором рабочим процессам выделяется определенное время на закрытие существующих соединений, после чего текущее приложение перезапускается:

```
$ slc ctl soft-restart my-app
```

Удаление приложения из числа подлежащих управлению:

```
$ slc ctl remove my-app
```

## PM2

PM2 - это диспетчер процессов рабочей среды для приложений Node.js, в котором предусмотрен встроенный инструмент распределения нагрузки. PM2 позволяет всегда поддерживать приложения в рабочем состоянии и перезагружать их без простоев, а также обеспечивает выполнение общих задач системного администрирования.  PM2 также позволяет управлять ведением протоколов, мониторингом и кластеризацией приложений.

Дополнительная информация содержится на странице [https://github.com/Unitech/pm2/](https://github.com/Unitech/pm2).

### Установка

```
$ [sudo] npm install pm2 -g
```

### Основы использования

При запуске приложения с помощью команды `pm2` необходимо указать путь к приложению. Тем не менее, при остановке, перезапуске или удалении приложения можно указать только имя или ИД приложения.

```
$ pm2 start app.js
[PM2] restartProcessId process id 0
┌──────────┬────┬──────┬───────┬────────┬─────────┬────────┬─────────────┬──────────┐
│ App name │ id │ mode │ pid   │ status │ restart │ uptime │ memory      │ watching │
├──────────┼────┼──────┼───────┼────────┼─────────┼────────┼─────────────┼──────────┤
│ my-app   │ 0  │ fork │ 64029 │ online │ 1       │ 0s     │ 17.816 MB   │ disabled │
└──────────┴────┴──────┴───────┴────────┴─────────┴────────┴─────────────┴──────────┘
 Use the `pm2 show <id|name>` command to get more details about an app.
```

При запуске приложения с помощью команды `pm2` оно немедленно переводится в фоновый режим. Фоновым приложением можно управлять из командной строки с помощью различных команд `pm2`.

После запуска приложения с помощью команды `pm2` оно регистрируется в списке процессов PM2 с указанием ИД. Затем можно управлять приложениями с тем же именем из различных каталогов в системе, используя только их ИД.

Обратите внимание на то, что если запущено несколько приложений с одинаковым именем, команды `pm2` применяются ко всем таким приложениям. Поэтому для управления отдельными приложениями используйте не имена, а ИД.

Список всех запущенных процессов:

```
$ pm2 list
```

Остановка приложения:

```
$ pm2 stop 0
```

Перезапуск приложения:

```
$ pm2 restart 0
```

Просмотр подробной информации о приложении:

```
$ pm2 show 0
```

Удаление приложения из реестра PM2:

```
$ pm2 delete 0
```


## Forever

Forever представляет собой простой инструмент интерфейса командной строки, обеспечивающий непрерывное (бесконечное) выполнение определенного сценария. Благодаря простому интерфейсу Forever является идеальным инструментом для выполнения не столь масштабных развертываний приложений и сценариев Node.js.

Дополнительная информация содержится на странице [https://github.com/foreverjs/forever/](https://github.com/foreverjs/forever).

### Установка

```
$ [sudo] npm install forever -g
```

### Основы использования

Для запуска сценария воспользуйтесь командой `forever start` и укажите путь к сценарию:

```
$ forever start script.js
```

Эта команда запустит сценарий в режиме демона (в фоновом режиме).

Для запуска сценария таким образом, чтобы он был соединен с терминалом, не указывайте `start`:

```
$ forever script.js
```

Целесообразно регистрировать вывод инструмента Forever и сценария в протоколе, используя для этого опции ведения протоколов `-l`, `-o` и `-e`, как показано в примере ниже:

```
$ forever start -l forever.log -o out.log -e err.log script.js
```

Для просмотра списка сценариев, запущенных с помощью Forever:

```
$ forever list
```

Для остановки сценария, запущенного с помощью Forever, воспользуйтесь командой `forever stop` и укажите индекс процесса (согласно списку, выведенному с помощью команды `forever list`).

```
$ forever stop 1
```

В качестве альтернативы, укажите путь к файлу:

```
$ forever stop script.js
```

Для остановки всех сценариев, запущенных с помощью Forever:

```
$ forever stopall
```

В Forever предусмотрено множество других опций, и, кроме того, данным инструментом предоставляется программный API.