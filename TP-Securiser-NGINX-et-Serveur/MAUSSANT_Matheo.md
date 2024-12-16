## Sur root :
```powershell
root@debian:/etc/ssh# sudo nano sshd_config
"Dans le fichier :"
PermitRootLogin No
```
```powershell
root@debian:~# ssh-keygen -t rsa -b 4096
ssh-copy-id root@wilfart.fr -p 6219


```
```powershell
root@debian:~# useradd NginxUser

root@debian:~# passwd NginxUser
"Nginx" le mot de passe

root@debian:~# usermod -aG sudo NginxUser

root@debian:~# apt install firewalld
```
```powershell
root@debian:~#
```
## Sur mon NginxUser
```powershell
NginxUser@debian:/root$ sudo apt install Nginx
```
```powershell
NginxUser@debian:/root$ sudo firewall-cmd --list-all
public
  services: dhcpv6-client ssh
  ports:

NginxUser@debian:/root$ sudo firewall-cmd --permanent --remove-service dhcpv6-client

NginxUser@debian:/root$ sudo firewall-cmd --reload
```
```powershell
NginxUser@debian:/root$ sudo ss -lnpt | grep 80
LISTEN 0      511          0.0.0.0:80        0.0.0.0:*    users:(("nginx",pid=2627,fd=5),("nginx",pid=2626,fd=5))
LISTEN 0      511             [::]:80           [::]:*    users:(("nginx",pid=2627,fd=6),("nginx",pid=2626,fd=6))
```
```powershell
NginxUser@debian:/root$ sudo nano /etc/nginx/site-available/default
"Dans le fichier :"
server {
        listen 1608 default_server;
        listen [::]:1608 default_server;
}
NginxUser@debian:/root$ sudo systemctl restart nginx
```
```powershell
NginxUser@debian:/etc/nginx/sites-available$ ss -lnpt | grep 1608
LISTEN 0      511          0.0.0.0:1608      0.0.0.0:*
LISTEN 0      511             [::]:1608         [::]:*
```
```powershell
NginxUser@debian:/etc/nginx/sites-available$ sudo firewall-cmd --permanent --add-port=1608/tcp

NginxUser@debian:/etc/nginx/sites-available$ sudo firewall-cmd --reload

NginxUser@debian:/etc/nginx/sites-available$ sudo firewall-cmd --list-all
public
  services: ssh
  ports: 1608/tcp
```