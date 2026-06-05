# 1. INSTAL·LACIÓ DEL SISTEMA OPERATIU (WINDOWS)
## 1.1 Configuració del nom de l'equip

En una xarxa corporativa, els noms per defecte (com DESKTOP-XXXXXX) impedeixen la traçabilitat. Establir una nomenclatura estandarditzada (com VM-LBPCW11) permet als administradors identificar ràpidament el propòsit del servidor/client, la seva ubicació o el departament al qual pertany des de les consoles de gestió i els inventaris (CMDB).

El nom de l'equip s'ha configurat com 'VM-LBPCW11':
wmic computersystem where name="LBPCW11" rename name="VM-LBPCW11"

<img width="343" height="151" alt="image" src="https://github.com/user-attachments/assets/571e251b-6313-4d44-8be9-6d4390fa26f5" />

.

<img width="613" height="692" alt="image" src="https://github.com/user-attachments/assets/5061f8c8-60e4-450c-9a62-398f12fac484" />

.

<img width="708" height="345" alt="image" src="https://github.com/user-attachments/assets/ed04a5ca-d66c-4718-a1a9-3d3b100ca770" />




## 1.2 Usuari administrador

Per motius de seguretat fonamentals, l'compte d'administrador per defecte de Windows ("Administrator") s'ha de deshabilitar o reanomenar per evitar atacs de força bruta. Crear un compte administrador local personalitzat amb una contrasenya robusta garanteix l'accés d'emergència a la màquina sense exposar els vectors d'atac clàssics.

S'ha creat un usuari administrador local:
net user admin Contrasenya123! /add net localgroup Administrators admin /add

<img width="667" height="261" alt="image" src="https://github.com/user-attachments/assets/17ae9ff3-fc17-4036-b8ce-172f16f39e63" />


## 1.3 Configuració de disc i controladors

Abans de passar un equip a producció, és crític verificar que el sistema operatiu reconeix correctament tot el programari físic (hardware) i que no hi ha conflictes de controladors (drivers). Validar l'espai en disc disponible prevé fallades d'instal·lació posteriors a causa d'un emmagatzematge insuficient.

S'ha verificat la configuració de disc i dels controladors principals:
wmic logicaldisk get name,size,freespace devmgmt.msc  # Device Manager para verificar controladores

<img width="196" height="498" alt="image" src="https://github.com/user-attachments/assets/dc2665c7-e9cd-4d1b-bc54-8be339be6fca" />


## 1.4 Punts de restauració

Actua com una xarxa de seguretat immediata. Crear un punt de restauració abans de realitzar canvis estructurals grossos (com instal·lar rols de servidor o programari crític) permet revertir el sistema a un estat funcional conegut si es produeix una fallada greu o una pantalla blava (BSOD).

S'han configurat punts de restauració del sistema:
Enable-ComputerRestore -Drive "C:\\"  # PowerShell Create-SystemRestorePoint -Description "Pre-Deployment Backup"

<img width="430" height="150" alt="image" src="https://github.com/user-attachments/assets/b887598d-7e0f-4d32-92a7-5893c40e7f7f" />


## 1.5 Estat de llicència

Per conformitat legal (compliance) i operativitat. Un Windows no activat perd funcionalitats de personalització, deixa de rebre certes actualitzacions de seguretat crítiques passat un temps i pot provocar reinicis automàtics, a més de bloquejar auditories de programari.

Verificació de l'activació i llicència de Windows:
slmgr /xpr              # Comprovar estat de llicència

<img width="314" height="290" alt="image" src="https://github.com/user-attachments/assets/ba631a5a-0f00-4f0f-8b4d-5acf1a9f8ba7" />

.


<img width="768" height="541" alt="image" src="https://github.com/user-attachments/assets/548c6cd4-49fd-4a55-8bba-131d430fcd85" />


## 1.6 Instal·lació d'eines d'administració

Les RSAT (Remote Server Administration Tools) permeten als administradors gestionar de forma remota rols de Windows Server (com Active Directory, DNS, DHCP) des d'aquest equip sense necessitat d'iniciar sessió directament per escriptori remot (RDP) al servidor central, millorant la seguretat i l'eficiència.

S'han instal·lat les RSAT (Remote Server Administration Tools):
winget install Microsoft.RemoteServerAdministrationTools

<img width="381" height="299" alt="image" src="https://github.com/user-attachments/assets/fa5ee057-c2e5-47d7-b023-36cf29d63239" />



# 2. VIRTUALITZACIÓ I GESTIÓ D'ARRENCADA (WINDOWS)
## 2.1 Instal·lació de Hyper-V

Hyper-V és l'hipervisor natiu de tipus 1 de Microsoft. La seva instal·lació és indispensable per a la consolidació de servidors, permetent executar múltiples sistemes operatius aïllats en una sola màquina física, maximitzant l'ús del hardware i estalviant costos energètics i d'infraestructura.

S'ha instal·lat l'hipervisor Hyper-V:
Enable-WindowsOptionalFeature -FeatureName Microsoft-Hyper-V -Online -All

<img width="413" height="362" alt="image" src="https://github.com/user-attachments/assets/a0d94939-9898-4b65-95c1-1cdddde1d909" />


## 2.2 Creació de màquina virtual

L'aïllament de serveis és una bona pràctica de seguretat (per exemple, no barrejar un servidor web públic amb la base de dades interna). Assignar recursos controlats (2 vCPUs, 2GB RAM) garanteix que la VM tingui el rendiment necessari sense canibalitzar els recursos de la màquina amfitriona (host).

S'ha creat una VM funcionant amb 2 vCPUs i 2GB RAM:
New-VM -Name "Windows-VM-01" -MemoryStartupBytes 2GB -SwitchName "Default Switch"

<img width="809" height="540" alt="image" src="https://github.com/user-attachments/assets/9cf7b797-b58a-437b-92fd-fe7469aff0bb" />


## 2.3 Configuració del gestor d'arrencada

El BCD (Boot Configuration Data) controla com arranca l'equip. Revisar-lo i configurar-lo permet gestionar escenaris de doble arrencada (dual-boot), entorns de recuperació o assegurar que l'ordre de càrrega dels sistemes operatius sigui el correcte després d'una actualització o clonació.

S'ha revisat i configurat el Boot Manager:
bcdedit /enum            # Llistar opcions d'arrencada

<img width="512" height="642" alt="image" src="https://github.com/user-attachments/assets/695d4d81-7c24-49c3-b8b1-ef4528ff2ee8" />



# 3. CONFIGURACIÓ DE XARXA I ADMINISTRACIÓ BÀSICA (WINDOWS)
## 3.1 Configuració IP

Els servidors i els equips d'infraestructura crítica requereixen adreces IP estàtiques (fixes). Si s'utilitzés DHCP (IP dinàmica), l'adreça podria canviar en reiniciar, trencant les connexions de xarxa dels clients, les regles del tallafocs (firewall) i les configuracions de rutes.

S'ha configurat una adreça IP estàtica:
New-NetIPAddress -IPAddress 192.168.1.20 -DefaultGateway 192.168.1.1 -PrefixLength 24

<img width="670" height="668" alt="image" src="https://github.com/user-attachments/assets/8369f43d-3e76-4d9f-8ebc-47d2231c369d" />


## 3.2 Configuració de DNS

El servei de noms (DNS) és el cor de la navegació i de la resolució de dominis a Windows (especialment en entorns d'Active Directory). Configurar servidors DNS fiables (com els de Google o els interns del domini) garanteix que l'equip es pugui comunicar amb l'exterior i trobar els controladors de domini.

S'han configurat els servidors DNS:
Set-DnsClientServerAddress -InterfaceIndex 2 -ServerAddresses 8.8.8.8,8.8.4.4

<img width="971" height="307" alt="image" src="https://github.com/user-attachments/assets/dc157130-f811-4eba-9d2f-900d34d9ded9" />


## 3.3 Nom de l'equip i grup de treball

Unir l'equip a un grup de treball específic (TECHSOLUTIONS) abans de la migració a un domini permet una organització bàsica en xarxes locals (LAN) i facilita la compartició de recursos inicials (fitxers, impressores) entre equips del mateix segment.

El nom de l'equip es va configurar anteriorment i s'ha afegit al grup de treball:
Add-Computer -WorkgroupName "TECHSOLUTIONS"

<img width="428" height="342" alt="image" src="https://github.com/user-attachments/assets/c82594c3-c56d-4a22-a029-1539401c4d31" />



# 4. GESTIÓ DE PROCESSOS I ADMINISTRACIÓ D'USUARIS (WINDOWS)
## 4.1 Creació d'usuaris i grups locals

Principi de menor privilegi. No tots els usuaris han de tenir permisos d'administrador. Crear grups específics (com developers) permet assignar permisos col·lectius de forma eficient, facilitant l'alta i baixa d'usuaris sense haver de modificar les carpetes una per una.

S'han creat usuaris i grups locals:
net user developer1 Passwrd123! /add net localgroup developers developer1 /add

<img width="665" height="406" alt="image" src="https://github.com/user-attachments/assets/cdfbdd12-263d-4cb6-9eb6-318a297e8dc8" />


## 4.2 Configuració de permisos

L'ús de comandes com icacls amb herència (OI - Object Inherit, CI - Container Inherit) i permisos de modificació (M) assegura que només el personal autoritzat pugui llegir o alterar dades sensibles, evitant fugues d'informació o modificacions accidentals.

S'han configurat permisos adequats sobre carpetes:
icacls C:\shared /grant developers:(OI)(CI)M /T

<img width="495" height="176" alt="image" src="https://github.com/user-attachments/assets/8d46d3cd-55b5-45e3-86c5-e55318b27cbe" />


## 4.3 Gestió de processos

El control de processos és vital per al manteniment del sistema. Permet identificar aplicacions que consumeixen massa CPU/RAM, tancar aplicacions penjades (com forçar el reinici d' explorer.exe si es bloqueja la interfície gràfica) i garantir l'estabilitat del sistema sense reiniciar tot el servidor.

Gestió de processos del sistema:
Get-Process              # Llistar processos (PowerShell) Stop-Process -Name explorer.exe -Force  # Finalitzar procés

<img width="391" height="131" alt="image" src="https://github.com/user-attachments/assets/ffebab4a-91b4-415f-a836-61781def8510" />



# 5. SISTEMES DE FITXERS, PARTICIONS I QUOTES (WINDOWS)
## 5.1 Configuració de particions

Per seguretat de dades i organització. Sempre s'ha de separar el sistema operatiu (Unitat C:) de les dades de l'usuari o de les aplicacions (Unitat D:, E:, etc.). Si el sistema operatiu falla o s'omple, les dades continuen intactes i protegides en una altra partició amb format modern (GPT).

S'ha afegit un volum addicional:
New-Partition -DiskNumber 1 -Size 50GB -GptType "{ebd0a0a2-b9e5-4433-a960-d260d5e38e02}"

<img width="742" height="537" alt="image" src="https://github.com/user-attachments/assets/2ee4461e-cfce-4d6e-bb15-0f53db6fdd0d" />

.

<img width="745" height="577" alt="image" src="https://github.com/user-attachments/assets/07aaf43a-d607-4034-8dc3-547de899aca2" />

.

<img width="883" height="444" alt="image" src="https://github.com/user-attachments/assets/45b3fc04-f808-4407-8995-16ae35f3ef37" />




## 5.2 Quotes de disc

Evita la denegació de servei per falta d'espai. Si un únic usuari omple tot el disc dur del servidor (per exemple, pujant fitxers personals), pot tombar el sistema operatiu per complet. Les quotes limiten l'espai màxim que cada usuari pot consumir.

S'han establert quotes de disc per usuari:
fsutil quota enforce C:\ fsutil quota modify C:\ 1000000000 900000000 user@domain.com

<img width="505" height="299" alt="image" src="https://github.com/user-attachments/assets/9bc3752f-ccda-4ec2-99c8-1ce008e6a8bf" />



# 6. CÒPIES DE SEGURETAT I AUTOMATITZACIÓ DE TASQUES (WINDOWS)
## 6.1 Còpies de seguretat automàtiques

La pèrdua de dades és un dels riscos més grossos d'una empresa (per fallades de disc, error humà o ransomware). Crear scripts automatitzats que empaquetin i comprimeixin les dades crítiques amb la data actualitzada garanteix tenir històrics de recuperació (RPO).

S'ha creat un script de backup:
@echo off set BACKUP_DIR=C:\\Backups for /f "tokens=2-4 delims=/ " %%a in ('date /t') do (set mydate=%%c%%a%%b) tar.exe -a -c -f %BACKUP_DIR%\\backup_%mydate%.tar.gz C:\\Data\\Important\\

<img width="646" height="551" alt="image" src="https://github.com/user-attachments/assets/b392e18a-9240-4e65-b46e-b21b3c7cc8c1" />

.

<img width="449" height="281" alt="image" src="https://github.com/user-attachments/assets/ec22a695-b0af-4198-8095-28faac543536" />

.

<img width="737" height="564" alt="image" src="https://github.com/user-attachments/assets/e0b91f77-4df7-4312-a9f1-b1e598d008b3" />

.

<img width="509" height="184" alt="image" src="https://github.com/user-attachments/assets/3ef3b016-b2e6-4665-a8ef-80d36e5e7490" />




## 6.2 Programació de tasques

El factor humà falla; els administradors poden oblidar-se de fer les còpies manualment. Executar la tasca automàticament a la matinada (02:00) garanteix que el procés es realitzi en hores de baixa activitat laboral, evitant l'alentiment del sistema durant la jornada de treball.

S'ha programat l'execució automàtica:
schtasks /create /tn "DailyBackup" /tr "C:\\Scripts\\backup.bat" /sc daily /st 02:00

<img width="362" height="376" alt="image" src="https://github.com/user-attachments/assets/17b61325-0b16-4217-ab4e-99fd8ca18a68" />



# 7. INSTAL·LACIÓ I CONFIGURACIÓ D'ACTIVE DIRECTORY (WINDOWS)
## 7.1 Instal·lació de AD DS

Active Directory Domain Services és la base de la gestió d'identitats a l'empresa. Transforma un servidor aïllat en un Controlador de Domini, el qual centralitza l'autenticació d'usuaris, equips i polítiques de seguretat de tota la xarxa corporativa.

S'ha instal·lat Active Directory Domain Services:
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

<img width="500" height="122" alt="image" src="https://github.com/user-attachments/assets/173f168f-4963-4c12-9ec0-41adff13c8b1" />


## 7.2 Configuració del domini

Crear un bosc i un domini propi (techsolutions.local) defineix la frontera lògica de seguretat de l'empresa. Configurar el mode de bosc a Win2019 assegura que s'utilitzin les característiques de seguretat i replicació més modernes i optimitzades de Microsoft.

S'ha configurat el domani 'techsolutions.local':
Install-ADDSForest -DomainName "techsolutions.local" -ForestMode Win2019

<img width="628" height="269" alt="image" src="https://github.com/user-attachments/assets/104a0c6b-173e-48c2-82ae-419a229c7c8b" />


## 7.3 Creació d'usuaris i grups

Un cop creat el domini, els usuaris ja no s'autentiquen en local a cada màquina, sinó contra el domini central. Això permet que un empleat pugui iniciar sessió de forma segura en qualsevol ordinador de l'empresa utilitzant les seves credencials úniques de l'AD.

S'han creat usuaris dins del domani AD:
New-ADUser -Name "domainuser1" -UserPrincipalName "domainuser1@techsolutions.local"

<img width="415" height="102" alt="image" src="https://github.com/user-attachments/assets/87540639-7738-44b1-9152-b5d3a2f0316d" />



# 8. UNIR EQUIPS AL DOMINI (WINDOWS)
## 8.1 Unir equip al domini

En unir un equip client al domini, aquest passa a estar sota el control de l'organització. A partir d'aquest moment, se li poden aplicar Polítiques de Grup (GPOs) de forma centralitzada (com fons de pantalla, restriccions de programari, configuracions de seguretat, etc.).

S'ha unut el client al domani AD:
Add-Computer -DomainName "techsolutions.local" -Credential "techsolutions\\administrator"

<img width="384" height="436" alt="image" src="https://github.com/user-attachments/assets/71c9049b-13ed-490d-b9fe-368d92f86c55" />


## 8.2 Verificació de connectivitat

Unir l'equip pot semblar correcte visualment, però utilitzar nltest verifica que el canal de seguretat intern entre el client i el Controlador de Domini està realment actiu i responent, evitant errors posteriors d'inici de sessió ("No hi ha servidors de logon disponibles").

Comprovació de la connexió amb el controlador:
nltest /query_preferred_dc:techsolutions.local

<img width="661" height="290" alt="image" src="https://github.com/user-attachments/assets/a5050727-271e-4bdb-918b-62f5972534db" />


# 9. RAIDs I TOLERÀNCIA A FALLADES (WINDOWS)
## 9.1 Configuració de RAID 1

Tolerància a fallades de hardware. El RAID 1 (Mirall / Mirror) duplica la informació exactament en dos discs en temps real. Si un dels discs durs es trenca físicament, el servidor continua funcionant sense aturar el servei i sense pèrdua de dades mentre se substitueix el disc danyat.

S'ha configurat un RAID 1 per a tolerància a fallades:
diskpart.exe list disk create volume mirror disk=1 disk=2

<img width="792" height="684" alt="image" src="https://github.com/user-attachments/assets/a917404d-be50-42a1-bbd2-a952739e98ac" />

.

<img width="1006" height="837" alt="image" src="https://github.com/user-attachments/assets/6ed5086d-25cd-42dd-b02c-86f2c9d5f083" />



## 9.2 Verificació d'estat

Un RAID pot estar funcionant en mode "degradat" (amb un disc trencat) sense que l'usuari ho noti. Monitoritzar l'estat operacional (HealthStatus) de forma periòdica permet detectar fallades a temps abans que es trenqui el segon disc, cosa que provocaria la pèrdua total de les dades.

Verificació de l'estat del RAID:
Get-PhysicalDisk | Select FriendlyName, HealthStatus, OperationalStatus

<img width="703" height="497" alt="image" src="https://github.com/user-attachments/assets/045cc415-ac0d-4aeb-b45d-142af7ca185a" />

.

<img width="855" height="700" alt="image" src="https://github.com/user-attachments/assets/27d9931e-74b5-4b2c-a01f-5db998a99635" />

.

<img width="749" height="450" alt="image" src="https://github.com/user-attachments/assets/edc302a8-2044-4355-84ca-84dd1d3e6d05" />

.

<img width="1016" height="810" alt="image" src="https://github.com/user-attachments/assets/2b4b7d9e-bcf0-44cd-9fe7-4283130a1ea3" />

.

<img width="958" height="857" alt="image" src="https://github.com/user-attachments/assets/102da934-5004-4ffe-a437-a20e9ca9911a" />

.

<img width="821" height="770" alt="image" src="https://github.com/user-attachments/assets/0b9b7df8-26df-4957-95e6-4653c3ad9d42" />

.

<img width="759" height="869" alt="image" src="https://github.com/user-attachments/assets/7662bfd2-b62d-4e5a-9fd1-21ded5e57114" />

.

<img width="680" height="696" alt="image" src="https://github.com/user-attachments/assets/7c726b44-5140-43cb-a3d0-c2a35eec965a" />



# 10. MONITORITZACIÓ, RENDIMENT I AUDITORIES (WINDOWS)
## 10.1 Monitorització de recursos

Permet realitzar una gestió proactiva de la infraestructura. Obtenir mètriques de l'ús de la CPU ajuda a detectar colls d'ampolla, dimensionar correctament el hardware en el futur (saber si cal ampliar el servidor) i auditar aplicacions que fan un ús anòmal del processador.

S'ha instal·lat el monitor de rendiment:
Get-Counter -Counter '\\Processor(_Total)\\% Processor Time'

<img width="1102" height="707" alt="image" src="https://github.com/user-attachments/assets/7dbcd97a-c8cd-431b-a6d1-e4b4b12e0914" />

.

<img width="1387" height="704" alt="image" src="https://github.com/user-attachments/assets/14a43f70-a4ea-43cf-861f-4c4386d1d5d2" />

.

<img width="782" height="585" alt="image" src="https://github.com/user-attachments/assets/c123ce49-9328-41f7-bdfe-1e23ab73d282" />

.

<img width="1036" height="603" alt="image" src="https://github.com/user-attachments/assets/1d22fb2b-58ba-4264-95aa-ff335b1e868e" />





## 10.2 Registres del sistema

Seguretat i Forense. Auditar els inicis de sessió (Account Logon) permet registrar qualsevol intent (correcte o fallit) d'accés al sistema. En cas d'un incident de seguretat (atac informàtic o accés no autoritzat), els logs de seguretat revisats amb Get-EventLog són l'única evidència legal per saber què va passar, qui ho va fer i quan.

Configuració d'auditories i revisió de logs:
auditpol.exe /set /category:"Account Logon" /success:enable
Get-EventLog -LogName Security -Newest 10


<img width="775" height="581" alt="image" src="https://github.com/user-attachments/assets/d5243f8e-b903-4e85-a12d-953df6626759" />

.

<img width="1041" height="621" alt="image" src="https://github.com/user-attachments/assets/cb99a262-f9e5-47df-92db-829e264b5871" />

.

<img width="1162" height="283" alt="image" src="https://github.com/user-attachments/assets/cce33df8-1496-4a4d-b73c-6970b874f97a" />

.

<img width="849" height="346" alt="image" src="https://github.com/user-attachments/assets/ee2265ff-63d8-4252-9f1b-8f21df621ff8" />


.

<img width="862" height="655" alt="image" src="https://github.com/user-attachments/assets/15f554e2-96e6-4760-98df-e99eb1bb5b70" />

.

<img width="1401" height="712" alt="image" src="https://github.com/user-attachments/assets/b3b67bc2-a828-4f56-97f7-68185ff0c8ed" />


.

<img width="1155" height="277" alt="image" src="https://github.com/user-attachments/assets/70dd52d9-bad2-4034-befd-b9ac76cefa54" />

