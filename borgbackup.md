## BorgBackup

### SERVER
```
sudo wget https://github.com/borgbackup/borg/releases/download/1.1.6/borg-linux64 -O /usr/local/bin/borg
sudo chmod +x /usr/local/bin/borg
sudo useradd -m borg
sudo passwd borg
mkdir -p /opt/storage/backup/borg/hshp
chown -R borg /opt/storage/backup/borg
```
### CLIENT
```
ssh-keygen -t ecdsa
ssh-copy-id -p '22123' 'borg@sa.on-dev.ru'
```
### SERVER
```
sudo su borg
nano .ssh/authorized_keys
```
> добавить в начало строки command: 'command="/usr/local/bin/borg serve" ssh-rsa AAAAB3... '

```
exit
```
### CLIENT
```
wget https://github.com/borgbackup/borg/releases/download/1.1.6/borg-linux64 -O /usr/local/bin/borg
chmod +x /usr/local/bin/borg
```
#### Иннициализация зашифрованого хранилища
```
borg init -e repokey ssh://borg@sa.on-dev.ru:22123/opt/storage/backup/borg/hshp
```
#### Иннициализация не зашифрованого хранилища
```
borg init -e none ssh://borg@sa.on-dev.ru:22123/opt/storage/backup/borg/hshp
```
#### Тестовый запуск
```
borg create --stats --list  ssh://borg@sa.on-dev.ru:22123/opt/storage/backup/borg/hshp::"opt-{now:%Y-%m-%d_%H:%M:%S}" /opt
```
#### Тестовый беспарольный запуск
```
umask 0077; echo "password" | base64 -w 0 > /root/.borg-passphrase
BORG_PASSPHRASE=$(cat /root/.borg-passphrase | base64 --decode) borg create --stats --list \
  ssh://borg@sa.on-dev.ru:22123/opt/storage/backup/borg/hshp::"opt-{now:%Y-%m-%d_%H:%M:%S}" /opt
```
#### Настройка запуска по рассписанию
```
mkdir /opt/borg
cat << 'EOF' > /opt/borg/start.sh
#!/bin/bash
#
BORG_PASSPHRASE=$(cat /root/.borg-passphrase | base64 --decode) borg create  \
  ssh://borg@sa.on-dev.ru:22123/opt/storage/backup/borg/hshp::opt-'{now:%Y-%m-%d_%H:%M:%S}' /opt
sleep 3
BORG_PASSPHRASE=$(cat /root/.borg-passphrase | base64 --decode) borg prune -d 7 -w 4 -m 12 -y 3 \
  ssh://borg@sa.on-dev.ru:22123/opt/storage/backup/borg/hshp/
EOF
chmod +x /opt/borg/start.sh
crontab -e
```
> добавить строку: 0 3 * * * /opt/borg/start.sh

### Источники
- https://habr.com/ru/company/flant/blog/420055/
- https://borgbackup.readthedocs.io/en/1.1.6/
