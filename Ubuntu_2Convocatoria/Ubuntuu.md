# 1. INSTAL·LACIÓ DEL SISTEMA OPERATIU

## 1.1 Configuració del nom de l'equip
En una xarxa corporativa com la de TechSolutions, és crític identificar unívocament cada host per evitar col·lisions, facilitar auditories de seguretat i gestionar inventaris. Seguir una convenció de nomenclatura (empresa-os-ordre) garanteix l'escalabilitat.

El nom de l'equip s'ha configurat com 'techsolutions-ubuntu-01' per seguir una convenció clara de nomenclatura de hosts.

Evidència/Comprovació:
.

<img width="714" height="377" alt="image" src="https://github.com/user-attachments/assets/9fb7cc13-0e43-429e-a6c9-8f632139f093" />


## 1.2 Configuració de la xarxa

Els servidors i equips d'infraestructura central han de tenir adreces IP estàtiques i predictibles. Si canviessin per DHCP, la resta de serveis dependents (com LDAP o carpetes compartides) fallarien de forma immediata.

S'ha configurat una adreça IP estàtica per assegurar connectivitat consistent:
sudo nano /etc/netplan/01-netcfg.yaml
Configuració aplicada:
network:   version: 2   ethernets:     eth0:       dhcp4: no       addresses: [192.168.1.10/24]       gateway4: 192.168.1.1       nameservers:         addresses: [8.8.8.8, 8.8.4.4]
sudo netplan apply

<img width="1817" height="162" alt="image" src="https://github.com/user-attachments/assets/b630aeda-2325-4595-807b-14c5ca0015db" />


## 1.3 Usuari administrador

Per motius de seguretat basats en el principi de menor privilegi, mai s'ha d'utilitzar l'usuari root directament per a tasques diàries. Es crea un usuari administrador dedicat (admin) inserit al grup sudo per a l'escalada temporal de privilegis auditable.

S'ha creat l'usuari administrador 'admin' amb permisos sudo complerts:
sudo useradd -m -s /bin/bash -G sudo admin sudo passwd admin

<img width="1114" height="899" alt="image" src="https://github.com/user-attachments/assets/8bd4a83d-23b4-4606-a606-3e1c2884961a" />


## 1.4 Configuració del disc i entorn gràfic

Separar el directori arrel (/) del directori dels usuaris (/home) impedeix que un desbordament de dades d'un usuari bloquegi i saturi completament el sistema operatiu. L'entorn gràfic garanteix un accés amigable si és requerit pel departament tècnic.

El disc s'ha configurat amb particions dedicades per a / (50GB) i /home (100GB). L'entorn gràfic GNOME 3.38 s'ha verificat correctament instal·lat.
lsblk  # Verificar particions df -h   # Verificar espai disponible

<img width="1023" height="952" alt="image" src="https://github.com/user-attachments/assets/6b440c71-5377-4bd1-b81b-8ab8e709cf8e" />


## 1.5 Punts de restauració

Abans de qualsevol canvi crític o actualització a TechSolutions, cal tenir un estat de retorn ràpid. Utilitzar snapshots de LVM (Logical Volume Manager) permet congelar l'estat del disc sense aturar completament el sistema.

S'ha configurat un sistema de snapshots amb LVM per a recuperació:
sudo lvcreate -L 5G -s -n backup_snapshot /dev/vg0/root

<img width="923" height="223" alt="image" src="https://github.com/user-attachments/assets/179b8c65-b7d7-4f03-a101-e222fa7ae834" />


## 1.6 Instal·lació d'eines bàsiques

L'ecosistema Ubuntu utilitza llicències de codi obert (GPL/MIT) de lliure ús per a empreses. Instal·lar eines bàsiques (curl, htop, openssh-server) és essencial per permetre el diagnòstic local i la gestió remota xifrada des del primer minut.

S'han instal·lat les eines essencials d'administració:
sudo apt update sudo apt install -y curl wget git htop net-tools openssh-server

<img width="920" height="580" alt="image" src="https://github.com/user-attachments/assets/a9512545-a4a3-4734-b6e0-26989223578a" />



# 2. VIRTUALITZACIÓ I GESTIÓ D'ARRENCADA

## 2.1 Instal·lació de KVM/QEMU

La virtualització de tipus 1 (bare-metal subjacent) amb KVM optimitza els recursos físics de TechSolutions, permetent aïllar serveis en entorns completament independents sense la pèrdua de rendiment d'altres hipervisors comercials.

S'ha instal·lat l'hipervisor KVM per a virtualització de baix nivell:
sudo apt install -y qemu-kvm libvirt-daemon libvirt-daemon-system bridge-utils
sudo systemctl enable libvirtd sudo systemctl start libvirtd

<img width="1570" height="577" alt="image" src="https://github.com/user-attachments/assets/e379f424-d4bf-4d55-8e85-c60a7b530656" />


## 2.2 Creació de màquina virtual

Permet desplegar un servidor secundari o un entorn de proves de sandbox sense necessitat d'adquirir nou maquinari físic, aprofitant l'emulació per línia de comandes per a una automatització futura.

S'ha creat una màquina virtual Ubuntu funcionant amb 2 vCPUs i 2GB RAM:
sudo virt-install --name ubuntu-vm01 --memory 2048 --vcpus 2 \   --disk size=20 --os-type linux --os-variant ubuntu20.04 \   --network bridge=virbr0 --cdrom /path/to/ubuntu.iso
sudo virsh list  # Verificar VM activa

<img width="1858" height="477" alt="image" src="https://github.com/user-attachments/assets/e9be6bc5-9135-4137-b8e1-0816826cd56d" />


## 2.3 Configuració del gestor d'arrencada

Ajustar els paràmetres del GRUB és vital per garantir que, en cas de fallada elèctrica o reinici, el sistema operatiu de producció s'iniciï immediatament de forma predeterminada i s'exposin les eines d'emergència en mode rescat si fos necessari.

S'ha verificat la configuració de GRUB 2 i s'han activat opcions de recuperació:
sudo nano /etc/default/grub
Paràmetres configurats:
GRUB_DEFAULT=0 GRUB_TIMEOUT=10 GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
sudo update-grub

<img width="786" height="591" alt="image" src="https://github.com/user-attachments/assets/6260dcdb-1de4-4c5f-9357-40401896e799" />



# 3. CONFIGURACIÓ DE XARXA I ADMINISTRACIÓ BÀSICA

## 3.1 Configuració IP

Després del desplegament inicial, cal certificar que la capa física i d'enllaç de la xarxa funciona i que l'equip disposa de sortida cap a passarel·les externes.

Configuració de xarxa estàtica amb verificació de connectivitat:
ip addr show              # Verificar direccions IP ping -c 3 8.8.8.8        # Test de connectivitat ip route show            # Verificar rutes

<img width="1094" height="614" alt="image" src="https://github.com/user-attachments/assets/866d0a61-e443-4571-9aa4-ee6d101ddc3a" />

.

<img width="1061" height="955" alt="image" src="https://github.com/user-attachments/assets/bf2e4c26-3926-47f1-b91b-b2f9b65f6106" />



## 3.2 Resolució DNS

Sense una resolució DNS configurada localment via systemd-resolved, l'equip seria incapaç de traduir noms de domini (com els repositoris d'actualització d'Ubuntu) a adreces IP, inhabilitant les descàrregues del sistema.

S'ha configurat resolució DNS utilitzant systemd-resolved:
sudo nano /etc/systemd/resolved.conf
DNS=8.8.8.8 8.8.4.4 FallbackDNS=1.1.1.1
sudo systemctl restart systemd-resolved

<img width="800" height="505" alt="image" src="https://github.com/user-attachments/assets/22f591ec-e5cf-4977-8c58-b98954066c2b" />

## 3.3 Instal·lació d'aplicacions

Per dotar el servidor de les capacitats operatives requerides per TechSolutions (editors de text avançats, control de versions o motors de contenidors/web corporatius).

S'han instal·lat aplicacions via apt (gestor de paquets):
sudo apt install -y vim nano git docker.io nginx
Verificació de les instal·lacions:
git --version docker --version nginx -v

<img width="962" height="551" alt="image" src="https://github.com/user-attachments/assets/924e1ffa-f941-46fb-b2fc-f36d73d932d3" />



# 4. GESTIÓ DE PROCESSOS I USUARIS

## 4.1 Creació de usuaris i grups

Organitzar els treballadors en grups (ex. developers i sysadmins) permet aplicar polítiques de seguretat massives basades en rols legals d'empresa, evitant assignar privilegis de forma individualitzada.

S'han creat diversos usuaris amb grups específics:
sudo useradd -m -s /bin/bash -G developers developer1 sudo useradd -m -s /bin/bash -G sysadmins sysadmin1 sudo groupadd developers sudo groupadd sysadmins

<img width="829" height="180" alt="image" src="https://github.com/user-attachments/assets/df5bd8a8-dc1c-48b7-a21c-3cdd97a9a583" />


## 4.2 Configuració de permisos

Restringir els permisos d'accés (Lectura, Escriptura, Execució) protegeix la propietat intel·lectual i dades confidencials. Els permisos especials tenen objectius clars:

SUID (u+s): Permet executar un script amb permisos de l'amo (administrador).

Sticky Bit (o+t): Garanteix que en un directori compartit (com /tmp/shared), els usuaris només puguin esborrar els fitxers que hagin creat ells mateixos.

S'han establert permisos adequats sobre fitxers i carpetes:
sudo chmod 750 /home/developer1 sudo chown developer1:developers /home/developer1
Permisos especials per a scripts sudo chmod u+s /usr/local/bin/admin_script sudo chmod o+t /tmp/shared

<img width="981" height="357" alt="image" src="https://github.com/user-attachments/assets/635a7b7d-b0f5-493e-adca-a6990416c1b4" />


## 4.3 Gestió de processos

Quan una aplicació es bloqueja o consumeix un percentatge anòmal de CPU/RAM, els administradors han de saber auditar el sistema en temps real i finalitzar el procés de forma controlada (SIGKILL o SIGTERM) per evitar la caiguda global del servidor.

Monitoritzar i finalitzar processos del sistema:
ps aux                    # Llistar tots els processos top                      # Monitorització en temps real kill -9 <PID>           # Finalitzar procés

<img width="965" height="309" alt="image" src="https://github.com/user-attachments/assets/be4bd983-bfd2-418b-b9db-e3915ba5b0e9" />


# 5. SISTEMES DE FITXERS, PARTICIONS I QUOTES

## 5.1 Configuració de particions

L'alta disponibilitat requereix connectar i preparar nous discos físics o cabines de emmagatzematge quan el volum de negoci creix. S'utilitza el format modern corporatiu ext4 per les seves funcionalitats de journaling (evita pèrdues de dades davant apagades sobtades).

S'ha afegit un disc addicional i creat una partició nova:
sudo fdisk -l              # Llistar discos sudo fdisk /dev/sdb      # Particionar sudo mkfs.ext4 /dev/sdb1  # Format

## 5.2 Muntatge automàtic
S'ha configurat muntatge automàtic al fitxer fstab:
sudo blkid                 # Obtenir UUID sudo nano /etc/fstab
UUID=abc123 /mnt/storage ext4 defaults 0 2
sudo mount -a             # Verificar muntatge

## 5.3 Quotes de disc
S'han establert quotes de disc per limitar ús per usuari:
sudo apt install quotatool sudo setquota -u developer1 1000000 1200000 0 0 /dev/sdb1
sudo repquota -a          # Verificar quotes


# 6. CÒPIES DE SEGURETAT I AUTOMATITZACIÓ

## 6.1 Còpies de seguretat automàtiques
S'ha creat un script de còpia de seguretat per a carpetes importants:
#!/bin/bash BACKUP_DIR=/backups DATE=$(date +%Y%m%d_%H%M%S) tar -czf $BACKUP_DIR/data_$DATE.tar.gz /home/*/important/

## 6.2 Automatització amb cron
S'ha programat l'execució diària del backup:
sudo crontab -e
0 2 * * * /usr/local/bin/backup.sh  # Execució a les 2:00 AM

## 6.3 Verificació de còpies
Verificació del funcionament correcte:
ls -lah /backups/         # Verificar fitxers creats tar -tzf /backups/data_*.tar.gz | head  # Verificar contingut


# 7. SERVEI LDAP I INTEGRACIÓ DE DOMINI

## 7.1 Instal·lació d'LDAP
S'ha instal·lat i configurat el servidor OpenLDAP:
sudo apt install -y slapd ldap-utils
sudo dpkg-reconfigure slapd

## 7.2 Creació d'usuaris en el domini
S'han creat usuaris dins del domani LDAP:
cat > user.ldif << EOF dn: uid=ldapuser1,ou=people,dc=techsolutions,dc=local objectClass: inetOrgPerson uid: ldapuser1 sn: User cn: LDAP User One userPassword: {SSHA}hash_password_here EOF  ldapadd -x -D cn=admin,dc=techsolutions,dc=local -W -f user.ldif

## 7.3 Configuració del client LDAP
S'ha configurat una màquina client per unir-se al domini:
sudo apt install -y libnss-ldap libpam-ldap
sudo auth-client-config -t nss -p lac_ldap

## 7.4 Verificació d'inici de sessió
Comprovació que els usuaris del domini poden iniciar sessió:
getent passwd ldapuser1  # Verificar usuari disponible su - ldapuser1           # Provar inici de sessió


# 8. COMPARTICIÓ DE RECURSOS

## 8.1 Servidor SAMBA
S'ha instal·lat i configurat Samba per a compartició SMB:
sudo apt install -y samba samba-common-bin
sudo nano /etc/samba/smb.conf
Configuració de compartició:
[documents] path = /home/shared/documents valid users = developer1, developer2 writable = yes create mask = 0775 directory mask = 0775
sudo systemctl restart smbd
 
## 8.2 Servidor NFS
S'ha configurat NFS per a compartició entre equips Linux:
sudo apt install -y nfs-kernel-server
sudo nano /etc/exports
/home/shared 192.168.1.0/24(rw,sync,no_subtree_check)
sudo exportfs -ra

## 8.3 Muntatge remot NFS
Client NFS montat correctament:
sudo mount -t nfs 192.168.1.10:/home/shared /mnt/nfs
mount | grep nfs         # Verificar muntatge


# 9. RAIDs I TOLERÀNCIA A FALLADES

## 9.1 Configuració de RAID 1
S'ha configurat un RAID 1 per a redundància:
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
sudo mkfs.ext4 /dev/md0 sudo mount /dev/md0 /mnt/raid

## 9.2 Verificació d'estat
Verificació del RAID i status de discos:
cat /proc/mdstat
sudo mdadm --detail /dev/md0

## 9.3 Comportament davant d'errors
Test de comportament en cas de fallada:
sudo mdadm --manage /dev/md0 --set-faulty /dev/sdb
El disc es marca com a fallat, RAID segueix funcionant sudo mdadm --manage /dev/md0 --remove /dev/sdb


# 10. MONITORITZACIÓ, RENDIMENT I AUDITORIES

## 10.1 Monitorització de recursos
S'han instal·lat eines de monitorització del sistema:
sudo apt install -y monitoring-plugins nagios-plugins
free -h              # Memòria disponible df -h              # Espai en disc vcstat             # Estadístiques de CPU

## 10.2 Auditories i registres
Configuració d'auditories del sistema:
sudo apt install -y auditd
sudo auditctl -w /etc/passwd -p wa -k passwd_changes
sudo ausearch -k passwd_changes  # Revisar auditories

## 10.3 Revisió de logs
Monitoritzar logs del sistema:
sudo tail -f /var/log/syslog journalctl -f                    # Últims eventos grep -i error /var/log/syslog   # Buscar errors


# 11. GESTIÓ D'ACTUALITZACIONS

## 11.1 Sistema de gestió d'actualitzacions
S'ha configurat l'actualització automàtica de paquets:
sudo apt install -y unattended-upgrades
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades

## 11.2 Actualitzacions automàtiques
Verificació del funcionament de les actualitzacions:
sudo systemctl status unattended-upgrades sudo tail -f /var/log/unattended-upgrades/unattended-upgrades.log

## 11.3 Manteniment del sistema
Verificació i manteniment del sistema:
sudo apt autoremove      # Eliminar paquets innecessaris sudo apt autoclean      # Netejar cache
