# 1. INSTAL·LACIÓ DEL SISTEMA OPERATIU (WINDOWS)
## 1.1 Configuració del nom de l'equip

En una xarxa corporativa, els noms per defecte (com DESKTOP-XXXXXX) impedeixen la traçabilitat. Establir una nomenclatura estandarditzada (com TECHSOLUTIONS-WIN-01) permet als administradors identificar ràpidament el propòsit del servidor/client, la seva ubicació o el departament al qual pertany des de les consoles de gestió i els inventaris (CMDB).

El nom de l'equip s'ha configurat com 'TECHSOLUTIONS-WIN-01':
wmic computersystem where name="%COMPUTERNAME%" rename name="TECHSOLUTIONS-WIN-01"

## 1.2 Usuari administrador

Per motius de seguretat fonamentals, l'compte d'administrador per defecte de Windows ("Administrator") s'ha de deshabilitar o reanomenar per evitar atacs de força bruta. Crear un compte administrador local personalitzat amb una contrasenya robusta garanteix l'accés d'emergència a la màquina sense exposar els vectors d'atac clàssics.

S'ha creat un usuari administrador local:
net user admin Contrasenya123! /add net localgroup Administrators admin /add

## 1.3 Configuració de disc i controladors

Abans de passar un equip a producció, és crític verificar que el sistema operatiu reconeix correctament tot el programari físic (hardware) i que no hi ha conflictes de controladors (drivers). Validar l'espai en disc disponible prevé fallades d'instal·lació posteriors a causa d'un emmagatzematge insuficient.

S'ha verificat la configuració de disc i dels controladors principals:
wmic logicaldisk get name,size,freespace devmgmt.msc  # Device Manager para verificar controladores

## 1.4 Punts de restauració

Actua com una xarxa de seguretat immediata. Crear un punt de restauració abans de realitzar canvis estructurals grossos (com instal·lar rols de servidor o programari crític) permet revertir el sistema a un estat funcional conegut si es produeix una fallada greu o una pantalla blava (BSOD).

S'han configurat punts de restauració del sistema:
Enable-ComputerRestore -Drive "C:\\"  # PowerShell Create-SystemRestorePoint -Description "Pre-Deployment Backup"

## 1.5 Estat de llicència

Per conformitat legal (compliance) i operativitat. Un Windows no activat perd funcionalitats de personalització, deixa de rebre certes actualitzacions de seguretat crítiques passat un temps i pot provocar reinicis automàtics, a més de bloquejar auditories de programari.

Verificació de l'activació i llicència de Windows:
slmgr /xpr              # Comprovar estat de llicència

## 1.6 Instal·lació d'eines d'administració

Les RSAT (Remote Server Administration Tools) permeten als administradors gestionar de forma remota rols de Windows Server (com Active Directory, DNS, DHCP) des d'aquest equip sense necessitat d'iniciar sessió directament per escriptori remot (RDP) al servidor central, millorant la seguretat i l'eficiència.

S'han instal·lat les RSAT (Remote Server Administration Tools):
winget install Microsoft.RemoteServerAdministrationTools


# 2. VIRTUALITZACIÓ I GESTIÓ D'ARRENCADA (WINDOWS)
## 2.1 Instal·lació de Hyper-V

Hyper-V és l'hipervisor natiu de tipus 1 de Microsoft. La seva instal·lació és indispensable per a la consolidació de servidors, permetent executar múltiples sistemes operatius aïllats en una sola màquina física, maximitzant l'ús del hardware i estalviant costos energètics i d'infraestructura.

S'ha instal·lat l'hipervisor Hyper-V:
Enable-WindowsOptionalFeature -FeatureName Microsoft-Hyper-V -Online -All

## 2.2 Creació de màquina virtual

L'aïllament de serveis és una bona pràctica de seguretat (per exemple, no barrejar un servidor web públic amb la base de dades interna). Assignar recursos controlats (2 vCPUs, 2GB RAM) garanteix que la VM tingui el rendiment necessari sense canibalitzar els recursos de la màquina amfitriona (host).

S'ha creat una VM funcionant amb 2 vCPUs i 2GB RAM:
New-VM -Name "Windows-VM-01" -MemoryStartupBytes 2GB -SwitchName "Default Switch"

## 2.3 Configuració del gestor d'arrencada

El BCD (Boot Configuration Data) controla com arranca l'equip. Revisar-lo i configurar-lo permet gestionar escenaris de doble arrencada (dual-boot), entorns de recuperació o assegurar que l'ordre de càrrega dels sistemes operatius sigui el correcte després d'una actualització o clonació.

S'ha revisat i configurat el Boot Manager:
bcdedit /enum            # Llistar opcions d'arrencada


# 3. CONFIGURACIÓ DE XARXA I ADMINISTRACIÓ BÀSICA (WINDOWS)
## 3.1 Configuració IP

Els servidors i els equips d'infraestructura crítica requereixen adreces IP estàtiques (fixes). Si s'utilitzés DHCP (IP dinàmica), l'adreça podria canviar en reiniciar, trencant les connexions de xarxa dels clients, les regles del tallafocs (firewall) i les configuracions de rutes.

S'ha configurat una adreça IP estàtica:
New-NetIPAddress -IPAddress 192.168.1.20 -DefaultGateway 192.168.1.1 -PrefixLength 24

## 3.2 Configuració de DNS

El servei de noms (DNS) és el cor de la navegació i de la resolució de dominis a Windows (especialment en entorns d'Active Directory). Configurar servidors DNS fiables (com els de Google o els interns del domini) garanteix que l'equip es pugui comunicar amb l'exterior i trobar els controladors de domini.

S'han configurat els servidors DNS:
Set-DnsClientServerAddress -InterfaceIndex 2 -ServerAddresses 8.8.8.8,8.8.4.4

## 3.3 Nom de l'equip i grup de treball

Unir l'equip a un grup de treball específic (TECHSOLUTIONS) abans de la migració a un domini permet una organització bàsica en xarxes locals (LAN) i facilita la compartició de recursos inicials (fitxers, impressores) entre equips del mateix segment.

El nom de l'equip es va configurar anteriorment i s'ha afegit al grup de treball:
Add-Computer -WorkgroupName "TECHSOLUTIONS"


# 4. GESTIÓ DE PROCESSOS I ADMINISTRACIÓ D'USUARIS (WINDOWS)
## 4.1 Creació d'usuaris i grups locals

Principi de menor privilegi. No tots els usuaris han de tenir permisos d'administrador. Crear grups específics (com developers) permet assignar permisos col·lectius de forma eficient, facilitant l'alta i baixa d'usuaris sense haver de modificar les carpetes una per una.

S'han creat usuaris i grups locals:
net user developer1 Passwrd123! /add net localgroup developers developer1 /add

## 4.2 Configuració de permisos

L'ús de comandes com icacls amb herència (OI - Object Inherit, CI - Container Inherit) i permisos de modificació (M) assegura que només el personal autoritzat pugui llegir o alterar dades sensibles, evitant fugues d'informació o modificacions accidentals.

S'han configurat permisos adequats sobre carpetes:
icacls C:\shared /grant developers:(OI)(CI)M /T

## 4.3 Gestió de processos

El control de processos és vital per al manteniment del sistema. Permet identificar aplicacions que consumeixen massa CPU/RAM, tancar aplicacions penjades (com forçar el reinici d' explorer.exe si es bloqueja la interfície gràfica) i garantir l'estabilitat del sistema sense reiniciar tot el servidor.

Gestió de processos del sistema:
Get-Process              # Llistar processos (PowerShell) Stop-Process -Name explorer.exe -Force  # Finalitzar procés


# 5. SISTEMES DE FITXERS, PARTICIONS I QUOTES (WINDOWS)
## 5.1 Configuració de particions

Per seguretat de dades i organització. Sempre s'ha de separar el sistema operatiu (Unitat C:) de les dades de l'usuari o de les aplicacions (Unitat D:, E:, etc.). Si el sistema operatiu falla o s'omple, les dades continuen intactes i protegides en una altra partició amb format modern (GPT).

S'ha afegit un volum addicional:
New-Partition -DiskNumber 1 -Size 50GB -GptType "{ebd0a0a2-b9e5-4433-a960-d260d5e38e02}"

## 5.2 Quotes de disc

Evita la denegació de servei per falta d'espai. Si un únic usuari omple tot el disc dur del servidor (per exemple, pujant fitxers personals), pot tombar el sistema operatiu per complet. Les quotes limiten l'espai màxim que cada usuari pot consumir.

S'han establert quotes de disc per usuari:
fsutil quota enforce C:\ fsutil quota modify C:\ 1000000000 900000000 user@domain.com


# 6. CÒPIES DE SEGURETAT I AUTOMATITZACIÓ DE TASQUES (WINDOWS)
## 6.1 Còpies de seguretat automàtiques

La pèrdua de dades és un dels riscos més grossos d'una empresa (per fallades de disc, error humà o ransomware). Crear scripts automatitzats que empaquetin i comprimeixin les dades crítiques amb la data actualitzada garanteix tenir històrics de recuperació (RPO).

S'ha creat un script de backup:
@echo off set BACKUP_DIR=C:\\Backups for /f "tokens=2-4 delims=/ " %%a in ('date /t') do (set mydate=%%c%%a%%b) tar.exe -a -c -f %BACKUP_DIR%\\backup_%mydate%.tar.gz C:\\Data\\Important\\

## 6.2 Programació de tasques

El factor humà falla; els administradors poden oblidar-se de fer les còpies manualment. Executar la tasca automàticament a la matinada (02:00) garanteix que el procés es realitzi en hores de baixa activitat laboral, evitant l'alentiment del sistema durant la jornada de treball.

S'ha programat l'execució automàtica:
schtasks /create /tn "DailyBackup" /tr "C:\\Scripts\\backup.bat" /sc daily /st 02:00

# 7. INSTAL·LACIÓ I CONFIGURACIÓ D'ACTIVE DIRECTORY (WINDOWS)
## 7.1 Instal·lació de AD DS

Active Directory Domain Services és la base de la gestió d'identitats a l'empresa. Transforma un servidor aïllat en un Controlador de Domini, el qual centralitza l'autenticació d'usuaris, equips i polítiques de seguretat de tota la xarxa corporativa.

S'ha instal·lat Active Directory Domain Services:
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

## 7.2 Configuració del domini

Crear un bosc i un domini propi (techsolutions.local) defineix la frontera lògica de seguretat de l'empresa. Configurar el mode de bosc a Win2019 assegura que s'utilitzin les característiques de seguretat i replicació més modernes i optimitzades de Microsoft.

S'ha configurat el domani 'techsolutions.local':
Install-ADDSForest -DomainName "techsolutions.local" -ForestMode Win2019

## 7.3 Creació d'usuaris i grups

Un cop creat el domini, els usuaris ja no s'autentiquen en local a cada màquina, sinó contra el domini central. Això permet que un empleat pugui iniciar sessió de forma segura en qualsevol ordinador de l'empresa utilitzant les seves credencials úniques de l'AD.

S'han creat usuaris dins del domani AD:
New-ADUser -Name "domainuser1" -UserPrincipalName "domainuser1@techsolutions.local"


# 8. UNIR EQUIPS AL DOMINI (WINDOWS)
## 8.1 Unir equip al domini

En unir un equip client al domini, aquest passa a estar sota el control de l'organització. A partir d'aquest moment, se li poden aplicar Polítiques de Grup (GPOs) de forma centralitzada (com fons de pantalla, restriccions de programari, configuracions de seguretat, etc.).

S'ha unut el client al domani AD:
Add-Computer -DomainName "techsolutions.local" -Credential "techsolutions\\administrator"

## 8.2 Verificació de connectivitat

Unir l'equip pot semblar correcte visualment, però utilitzar nltest verifica que el canal de seguretat intern entre el client i el Controlador de Domini està realment actiu i responent, evitant errors posteriors d'inici de sessió ("No hi ha servidors de logon disponibles").

Comprovació de la connexió amb el controlador:
nltest /query_preferred_dc:techsolutions.local


# 9. RAIDs I TOLERÀNCIA A FALLADES (WINDOWS)
## 9.1 Configuració de RAID 1

Tolerància a fallades de hardware. El RAID 1 (Mirall / Mirror) duplica la informació exactament en dos discs en temps real. Si un dels discs durs es trenca físicament, el servidor continua funcionant sense aturar el servei i sense pèrdua de dades mentre se substitueix el disc danyat.

S'ha configurat un RAID 1 per a tolerància a fallades:
diskpart.exe list disk create volume mirror disk=1 disk=2

## 9.2 Verificació d'estat

Un RAID pot estar funcionant en mode "degradat" (amb un disc trencat) sense que l'usuari ho noti. Monitoritzar l'estat operacional (HealthStatus) de forma periòdica permet detectar fallades a temps abans que es trenqui el segon disc, cosa que provocaria la pèrdua total de les dades.

Verificació de l'estat del RAID:
Get-PhysicalDisk | Select FriendlyName, HealthStatus, OperationalStatus


# 10. MONITORITZACIÓ, RENDIMENT I AUDITORIES (WINDOWS)
## 10.1 Monitorització de recursos

Permet realitzar una gestió proactiva de la infraestructura. Obtenir mètriques de l'ús de la CPU ajuda a detectar colls d'ampolla, dimensionar correctament el hardware en el futur (saber si cal ampliar el servidor) i auditar aplicacions que fan un ús anòmal del processador.

S'ha instal·lat el monitor de rendiment:
Get-Counter -Counter '\\Processor(_Total)\\% Processor Time'

## 10.2 Registres del sistema

Seguretat i Forense. Auditar els inicis de sessió (Account Logon) permet registrar qualsevol intent (correcte o fallit) d'accés al sistema. En cas d'un incident de seguretat (atac informàtic o accés no autoritzat), els logs de seguretat revisats amb Get-EventLog són l'única evidència legal per saber què va passar, qui ho va fer i quan.

Configuració d'auditories i revisió de logs:
auditpol.exe /set /category:"Account Logon" /success:enable
Get-EventLog -LogName Security -Newest 10
