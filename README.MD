# !!!WIP Сервер платформы 1С:Предприятие в windows контейнере

Проект в процессе разработки

Основной репозиторий доступен в организации [VanessaDockers](https://github.com/VanessaDockers/onec-win-docker)

## Запуск сервиса хранилища конфигураций

```cmd
docker-compose up -d onec-storage
```

В каталоге `./storagedata` создать папку (`path`) для хранения файлов хранилища.

Для подклчюения по `tcp` использовать `tcp://hostname:[port]/path`

Порт по-умолчанию `1542` может быть переопределен в файле [docker-compose.yml](./docker-compose.yml)

## Вопросы для исследования

- [ ] Проброс аппаратных ключей в контейнер
- [ ] Сетевое взаимодействие (будет ли как linux контейнером)?
- [*] Копировать уже установленный дистрибутив 1с (не нужно,)
- [*] Выяснить синтаксис команды инсталляции мси с помощью простого пакета

## TODO

- [+] Открыть порты для доступа к запущенному сервису.
- [ ] Логи работы запускаемого приложения на консоль
- [+] Монтировать папки сервера приложений и сервера хранилища конфигураци
- [ ] Очистка каталога дистрибутива
- [+] Задать имя запускаемого контейнера

### Рабочие заметки

1. В папке distr нужна распакованная платформа (./distr/8.3.10.2650/windows64full/)
2. Создание службы для сервера

    - в compose переопределить исполняемый файл на `ping -t localhost`
    - В интерактивном режиме выполнить

    ```cmd
    sc create "onec-srv" binpath= "C:\onec\bin\ragent.exe -srvc -agent -regport 1541 -port 1540 -d C:\serverdata" displayname= "onec-srv" start= auto
    ```

3. Ссылки на документацию по PG
  [Запускаем PostgreSQL без инсталляции](https://habr.com/post/101678/)
  [16.2.3. Установка в командной строке (PostgresPRO)](https://postgrespro.ru/docs/postgrespro/10/binary-installation-on-windows#WIN-CLI-INSTALLATION)

4. Ссылки для чтения:

    [Docker on Windows: Windows-containers-specific](https://26thcentury.com/reading-list-devops-build-test-deploy-automation-monitoring/#dockerWindowsContainersSpecific)

5. Заполнение файлов на основе переменных:
    [GREP and SED with PowerShell](https://blogs.msdn.microsoft.com/zainnab/2007/07/08/grep-and-sed-with-powershell/)
    ```powershell
    cat .\default.vrd | % { $_ -replace "127.0.0.1","$Env:COMPUTERNAME"} > default.vrd.new
    ```

#### Включить telnet

Install-WindowsFeature -name Telnet-Client

VagrantFile - только запуск виртуальной машины windowsservercore

Быстрое подключение к одному работающему контейнеру в процессе сборки:
  docker exec -it $(docker ps -q) powershell

#### В PS

c интерфейсом

``` powershell
Start-Process msiexec.exe -Wait -ArgumentList '/i "distr\8.3.10.2650\windows64full\1CEnterprise 8 (x86-64).msi" /qb TRANSFORMS="1049.mst" /l* install.log INSTALLSRVRASSRVC=0 DESIGNERALLCLIENTS=1 THICKCLIENT=1 THINCLIENTFILE=1 THINCLIENT=1 WEBSERVEREXT=1 SERVER=1 CONFREPOSSERVER=1 SERVERCLIENT=1 LANGUAGES=RU'
```

Удаление

```powershell
Start-Process msiexec.exe -Wait -ArgumentList '/x "C:\Users\ebessonov\Documents\repo\gitlab\onecdocker\onec-win-docker\distr\8.3.10.2650\windows64full\1CEnterprise 8 (x86-64).msi" /qn /l*v uninstall.log'
```

Проверяем работу команды инсталляции пакетов

В cmd от имени администратора выполнить

```cmd
  msiexec /qn /l* node-log.txt /i node.msi
```

В Powershell

после инсталляции
  node --version