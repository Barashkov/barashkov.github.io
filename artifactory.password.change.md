## Сменить домен для artifactory v5.6 не зная пароля admin
### Сменить текущий пароль admin на password
```bash
cp /var/opt/jfrog/artifactory/backup/backup-daily/current/security.xml /etc/opt/jfrog/artifactory/security.import.xml
cp /var/opt/jfrog/artifactory/backup/backup-daily/current/security.xml ~/artifactory.security.xml.back
nano /etc/opt/jfrog/artifactory/security.import.xml
```
> Для пользователя admin установить хэш пароля:
<code>\<password>1f70548d73baca61aab8660733c7de81\</password> </code>

```bash
chown artifactory:artifactory /etc/opt/jfrog/artifactory/security.import.xml
systemctl restart artifactory
```

### Выполнить настройку имени домена

https://example.com/artifactory/webapp/#/admin/configuration/general

![alt test](https://github.com/Barashkov/barashkov.github.io/raw/main/images/2023-01-12_09-59.png)

### Вернуть пароль для admin на место
```bash
cp ~/artifactory.security.xml.back /etc/opt/jfrog/artifactory/security.import.xml
chown artifactory:artifactory /etc/opt/jfrog/artifactory/security.import.xml
systemctl restart artifactory
```

### Источники
- https://www.jfrog.com/confluence/display/RTF5X/Configuring+Artifactory
- https://www.jfrog.com/confluence/display/RTF5X/Managing+Users#ManagingUsers-ResettingtheAdminPassword
