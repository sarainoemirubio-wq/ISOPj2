En este sprint nuestros obejetivos son:


 Crear i configurar comptes d’usuari i grups, establint permisos d’accés adequats.

 Administrar el sistema d’emmagatzematge, incloent la creació i gestió de particions, unitats
lògiques i volums.

 Documentar l'estructura de directoris, sistemes de fitxers i mètodes de gestió de discos,
incloent captures de pantalla i notes explicatives.

Para lograr nuestros objetivos lo primero que debemos hacer es:

La Preparación del Sistema:


Escogemos una maquina virtual UBUNTU/WINDOWS y le añadimos un disco nuevo:


<img width="830" height="548" alt="image" src="https://github.com/user-attachments/assets/42cfc2d0-26c6-4d28-93b8-aeb56797266e" />

Aquí vemos la configuración de VirtualBox donde se ven los dos discos duros (el principal y el nuevo) conectados al controlador SATA.


Luego veremos las gestiones de los discos iniciando con WINDOWS 11:


<img width="791" height="566" alt="image" src="https://github.com/user-attachments/assets/1d49374f-8c2b-4f1c-996a-b3225a1d37c1" />

.


<img width="801" height="692" alt="image" src="https://github.com/user-attachments/assets/87d76aa3-3d0b-472c-94aa-1cd317a5153b" />


Aquí hemos creado la Partición "Dades":

<img width="486" height="388" alt="image" src="https://github.com/user-attachments/assets/f68567b9-5fb7-40ce-a47a-a8928ee94683" />



Aquí hemos creado la Partición "Portable":

<img width="490" height="381" alt="image" src="https://github.com/user-attachments/assets/15f34615-0ded-4e59-975d-4c57a8bc679b" />



<img width="489" height="94" alt="image" src="https://github.com/user-attachments/assets/7a679639-1dce-43b7-a5ae-67b6a2f65869" />

Aquí estamos viendo el Administrador de discos mostrando el disco 1 dividido en dos con sus nombres y formatos correctos.


Ahora haremos una comprobación con Diskpart:


<img width="485" height="450" alt="image" src="https://github.com/user-attachments/assets/cebefc39-8098-4042-8917-31bffba565a7" />


Estamos viendo la consola de comandos mostrando el listado de particiones de ese disco.






Hemos visto una parte de los objetivos que nos hemos propuesto, ahora seguiremos con:

Las Cuotas y Usuarios:


<img width="786" height="640" alt="image" src="https://github.com/user-attachments/assets/6cf857a0-4ef8-4d55-9080-48ebe0299108" />


En esta captura estamos viendo la ventana de configuración de cuotas con los 300 MB escritos y las casillas marcadas.








Ahora crearemos usuarios y grupos:




<img width="532" height="361" alt="image" src="https://github.com/user-attachments/assets/3e86fc1b-b0e4-445d-bb6f-702025c16c1d" />


Aquí vemos el comando 'net localgroup Limitats' mostrando que ambos alumnos son miembros.







Después haremos la prueba de fuego en Cuotas:


Iniciamos no con el usuario que tenemos si no con el que hemos creado:

.


<img width="446" height="430" alt="image" src="https://github.com/user-attachments/assets/79764fea-efc3-4045-9a43-786f75b35f0b" />


ya que hemos iniciado algo nuevo no tenemos ningun archivo con pero entonces para eso creamps un archivo gigante desde el CMD con este comando:

.

<img width="840" height="182" alt="image" src="https://github.com/user-attachments/assets/01ccc310-c0af-4f94-851a-e43105dcf5c9" />

.


resultats:

<img width="521" height="142" alt="image" src="https://github.com/user-attachments/assets/0611dae2-a7f1-4adc-9d01-abbf891f6827" />


.

<img width="892" height="632" alt="image" src="https://github.com/user-attachments/assets/2bac5aff-a867-4576-9cb3-ab5279f8f75e" />


La captura nos muestra que el mensaje de error de Windows diciendo que no hay espacio suficiente en el disco (aunque el disco sea de 5GB, la cuota lo frena).

.

Ahora veremos los Script y Automatización

.


<img width="799" height="677" alt="image" src="https://github.com/user-attachments/assets/900c3c4d-fc66-4b00-bcae-a43665c98506" />

.

<img width="496" height="400" alt="image" src="https://github.com/user-attachments/assets/3268623d-ce75-4016-b048-648e339333e7" />

.

<img width="738" height="422" alt="image" src="https://github.com/user-attachments/assets/72f3169a-6616-4953-a32b-dc0a6e89bc53" />

.


Entramos con :

<img width="660" height="222" alt="image" src="https://github.com/user-attachments/assets/9d822fc6-5cea-4276-910c-e406ac8c3b5e" />


.


<img width="458" height="178" alt="image" src="https://github.com/user-attachments/assets/f8de1e67-63a7-4888-b1f1-4e414f9fee46" />

.

<img width="378" height="158" alt="image" src="https://github.com/user-attachments/assets/425b61c1-0a81-4933-8f01-e691f9702e67" />


.

me falta hacer: copies de seguretat i gestor de procesos






























