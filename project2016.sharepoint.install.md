## Установка Project 2016 на SharePoint в изолированном режиме на одном сервере для совместной работы

1. Установить Windows Server 2012 R2 (ru_windows_server_2012_r2_x64_dvd_2708010.iso)

2. Поставить обновления на Windows 

3. Установить MS SQL Server 2014 (en_sql_server_2014_enterprise_edition_x64_dvd_3932700.iso)

4. Установить SP3 для MS SQL Server 2014 (SQLServer2014SP3-KB4022619-x64-ENU.exe)

5. Подключить образ PROJECT_SERVER_2016.img

6. Установить необходимое ПО (зависимости для Sharepoint)

7. Установить SharePoint Server (SharePoint Server 2016 Enterprise: TY6N4-K9WD3-JD2J2-VYKTJ-GVJ2J)

8. В "Командная консоль SharePoint 2016" запустить
```
New-SPConfigurationDatabase -DatabaseName "SharePoint_Config" -DatabaseServer "sharepoint" -LocalServerRole SingleServerFarm -FarmCredentials (Get-Credential) -Passphrase (ConvertTo-SecureString "password" -AsPlainText -force)
```

9. Запустить "Мастер настройки продуктов SharePoint 2016"
> указать порт 8888

10. В панели администрирования http://sharepoint:8888 в мастере настройки проверить галку напротив "Приложение службы сервера ProjectServer"

11. В "Командная консоль SharePoint 2016" запустить
```
Enable-ProjectServerLicense
```
>ввести ключ для Project Server 2016: 23CB6-N4X8Q-WWD7M-6FHCW-9TPVP
>

12. Добавить веб приложение
http://sharepoint:8888/_admin/WebApplicationList.aspx -> "Создать"
> (среди прочих параметров обязательно указать: "Простая проверка подлинности (учетные данные отправляются без шифрования)")

13. В "Командная консоль SharePoint 2016" запустить
```
# название сайта PWA
$siteName = "Project Server 2016"
# веб-приложение, где будет наш PWA
$webApp = "https://pwa.example.ru:443"
# URL-адрес PWA
$siteUrl = "https://pwa.example.ru"
# администратор PWA
$owner = "Администратор"
# название базы данных контента
$databaseName = "ProjectWebAppContent"
# имя сервере баз данных
$databaseServer = "sharepoint"
$template = "pwa#0"
New-SPContentDatabase -Name $databaseName -DatabaseServer $databaseServer -WebApplication $webApp
New-SPSite -Url $siteUrl -OwnerAlias $owner -ContentDatabase $databaseName -Template $template -Name $siteName -Language 1049
Enable-SPFeature pwasite -URL $siteUrl
```

14. Добавить самоподписанный сертификат в IIS (настоящий сертификат будет на прокси в nginx)

Меню IIS -> "Начальная страница SHAREPOINT" -> "Сертификаты сервера"

15. Подключить сертификат к сайту

Меню IIS -> "Начальная страница SHAREPOINT - Sharepoint443" -> "Привязки..." -> "Добавить..."

16. Добавить нового пользователя в систему Windows

17. Добавить нового пользователя в sharepoint. В "Командная консоль SharePoint 2016" запустить
```
New-SPUser -UserAlias 'sharepoint\vorobyov' -DisplayName 'V. Vorobyov' -Web https://pwa.example.ru
```

18. Добавить пользователя в группу "Руководители проектов для Project·Web·App"

Выполняется по ссылке: https://pwa.example.ru/_layouts/15/groups.aspx

19. Лепота.
