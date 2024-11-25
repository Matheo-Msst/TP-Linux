# TP Avancé : "Mission Ultime : Sauvegarde et Sécurisation"

## Étape 1 : Analyse et nettoyage du serveur

### 1.Lister les tâches cron pour détecter des backdoors :
>#### - Analysez les tâches cron de tous les utilisateurs pour identifier celles qui semblent malveillantes.

```powershell 
[root@localhost /]# cat etc/passwd
```
### 2.Identifier et supprimer les fichiers cachés :
>#### Recherchez les fichiers cachés dans les répertoires ```/tmp```, ```/var/tmp``` et ```/home``` :
```powershell
[root@localhost /]#ls -a tmp/

[root@localhost /]#ls -a var/tmp/

[root@localhost /]#ls -a home/
```
>#### Supprimez tout fichier suspect ou inconnu :
```powershell
[root@localhost tmp]# ls -alh

[root@localhost tmp]# rm .hidden_file ; rm .hidden_script ; rm malicious.sh
```
### 3.Analyser les connexions réseau actives :
>#### Listez les connexions actives pour repérer d'éventuelles communications malveillantes :

```powershell
[root@localhost tmp]# netstat -a
```

## Étape 2 : Configuration avancée de LVM
### 1.Créer un snapshot de sécurité pour ```/mnt/secure_data``` :
>#### Prenez un snapshot du volume logique secure_data :
```powershell
[root@localhost tmp]# lvcreate -L 5G -s -n rootsnap /dev/vg_secure/secure_data 
```
### 2.Tester la restauration du snapshot :
>#### Supprimez un fichier dans /mnt/secure_data :
```powershell
[root@localhost tmp]# rm /mnt/secure_data/sensitive1.txt
```
>#### Montez le snapshot et restaurez le fichier supprimé :
```powershell
[root@localhost tmp]# umount /mnt/secure_data/

[root@localhost tmp]# lvconvert --merge /dev/vg_secure/rootsnap
  Merging of volume rootvg/datasnap started.
  rootvg/datalv: Merged: 100,00%

[root@localhost tmp]mount /mnt/secure_data/
```
### 3.Optimiser l’espace disque :
>#### Si le volume logique secure_data est plein, étendez-le en ajoutant de l’espace à partir du groupe de volumes existant : 
```powershell
[root@localhost tmp]# lvextend -L +200M /dev/vg_secure/secure_data
  Size of logical volume vg_secure/secure_data changed from 500.00 MiB (125 extents) to 700.00 MiB (175 extents).
  Logical volume vg_secure/secure_data successfully resized.
```

## Étape 3 : Automatisation avec un script de sauvegarde
### 1.Créer un script secure_backup.sh :
>#### Archive le contenu de ```/mnt/secure_data``` dans ```/backup/secure_data_YYYYMMDD.tar.gz```:
```powershell
[root@localhost tmp]# cd /mnt/

[root@localhost tmp]# mkdir backup
```
>#### Exclut les fichiers temporaires (.tmp, .log) et les fichiers cachés :
### 2.Ajoutez une fonction de rotation des sauvegardes :
>#### Conservez uniquement les 7 dernières sauvegardes pour économiser de l’espace :
### 3.Testez le script :
>#### Exécutez le script manuellement et vérifiez que les archives sont créées correctement :
```powershell
#!/bin/bash

BACKUP_FILENAME="/backup/secure_data_$(date +%F).tar.gz"
REPO="/mnt/secure_data"

tar --exclude='*.tmp' --exclude='*.log' --exclude='.*' -czvf $BACKUP_FILENAME $REPO

rotate_backups() {
  BACKUP_COUNT=$(ls $BACKUP_DIR/secure_data_*.tar.gz | wc -l)
  if [ $BACKUP_COUNT -gt 7 ]; then
    OLDEST_BACKUP=$(ls -t $BACKUP_DIR/secure_data_*.tar.gz | tail -n 1)
    rm -f $OLDEST_BACKUP
    echo "Rotation des sauvegardes : suppression de $OLDEST_BACKUP"
  fi
}

rotate_backups
```

### 4.Automatisez avec une tâche cron :
>#### Planifiez le script pour qu’il s’exécute tous les jours à 3h du matin :
```powershell
[root@localhost /]# crontab -e
crontab: installing new crontab
[root@localhost /]# crontab -l
0 3 * * * /mnt/secure_backup.sh
```
## Étape 4 : Surveillance avancée avec ```auditd```
### 1.Configurer auditd pour surveiller ```/etc``` :
>#### Ajoutez une règle avec auditctl pour surveiller toutes les modifications dans /etc :
```powershell 
[root@localhost /]# sudo yum install audit

[root@localhost /]# sudo auditctl -w /etc -p wa -k etc_changes
Old style watch rules are slower

[root@localhost /]# sudo auditctl -l
-w /etc -p wa -k etc_changes

[root@localhost /]# sudo touch /etc/testfile

[root@localhost /]# sudo nano /etc/hostname

[root@localhost /]# sudo ausearch -k etc_changes > /var/log/audit_etc.log
```

## Étape 5 : Sécurisation avec Firewalld
### 1.Configurer un pare-feu pour SSH et HTTP/HTTPS uniquement :
>#### Autorisez uniquement les ports nécessaires pour SSH et HTTP/HTTPS :
```powershell
[root@localhost matheo]# sudo firewall-cmd --permanent --remove-service dhcpv6-client
success

[root@localhost matheo]# sudo firewall-cmd --permanent --remove-service cockpit
success

[root@localhost matheo]# sudo firewall-cmd --reload
success

[root@localhost matheo]# sudo firewall-cmd --permanent --add-port=22/tcp
success

[root@localhost matheo]# sudo firewall-cmd --permanent --add-port=443/tcp
success

[root@localhost matheo]# sudo firewall-cmd --permanent --add-port=80/tcp
success

[root@localhost matheo]# sudo firewall-cmd --reload
success

```
>#### Bloquez toutes les autres connexions :
```powershell
[root@localhost matheo]# sudo firewall-cmd --set-default-zone=drop
```
### 2.Bloquer des IP suspectes :
>#### À l’aide des logs d’audit et des connexions réseau, bloquez les adresses IP malveillantes identifiée :
```powershell

```
### 3.Restreindre SSH à un sous-réseau spécifique :
>#### Limitez l’accès SSH à votre réseau local uniquement (par exemple, 192.168.x.x) :
```powershell

```
