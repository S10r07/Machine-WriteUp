
Vulnerabilidad [/bin/pip](https://gtfobins.github.io/gtfobins/pip/)
Enumeración para repositorios [Git Tool](https://github.com/internetwache/GitTools) - [Git Dumper](https://github.com/arthaud/git-dumper)**.

### **netdiscover (Enumeración de dispositivos en la red)**

Se utiliza para realizar un escaneo de la **red local** con el objetivo de descubrir dispositivos conectados a esa red y obtener información sobre ellos, como direcciones IP y direcciones MAC.

| Opción | Descripción                                                                                                         |
| ------ | ------------------------------------------------------------------------------------------------------------------- |
| -i     | Dispositivo de red                                                                                                  |
| -r     | Escanea un rango específico en lugar de un escaneo automático. Ejemplo: 192.168.6.0/24, /16, /8                     |
| -l     | Escanea la lista de rangos contenidos en el archivo dado                                                            |
| -p     | Modo pasivo: no envía nada, solo espiar                                                                             |
| -m     | Escanea una lista de direcciones MAC y nombres de host conocidos                                                    |
| -F     | Personaliza la expresión de filtro de pcap (por defecto: "arp")                                                     |
| -s     | Tiempo de espera entre cada solicitud ARP (milisegundos)                                                            |
| -c     | Número de veces que se enviará cada solicitud ARP (para redes con pérdida de paquetes)                              |
| -n     | Último octeto de dirección IP de origen utilizado para el escaneo (de 2 a 253)                                      |
| -d     | Ignora archivos de configuración local para escaneo automático y modo rápido                                        |
| -f     | Activa el escaneo en modo rápido, ahorra mucho tiempo, recomendado para el modo automático                          |
| -P     | Imprime resultados en un formato adecuado para ser analizado por otro programa y detiene después del escaneo activo |
| -L     | Similar a -P pero continúa escuchando después de que se complete el escaneo activo                                  |
| -N     | No imprime encabezado. Solo válido cuando -P o -L está habilitado.                                                  |
| -S     | Habilita la supresión del tiempo de espera entre cada solicitud (modo hardcore)                                     |

### **nmap**

|          | Bandera   | Descripción                                                                                                                                                                   |
| -------- | --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|          | -sS       | Escaneo de tipo SYN (Half-open)                                                                                                                                               |
|          | -sT       | Escaneo de tipo TCP Connect                                                                                                                                                   |
|          | -sU       | Escaneo de tipo UDP                                                                                                                                                           |
|          | -sF       | Escaneo de tipo FIN                                                                                                                                                           |
|          | -sN       | Escaneo de tipo NULL (sin banderas establecidas)                                                                                                                              |
|          | -sP       | Ping Scan: detecta hosts activos sin realizar un escaneo de puertos                                                                                                           |
|          | -O        | Detección de sistema operativo                                                                                                                                                |
|          | -p        | Escaneo de puertos específicos                                                                                                                                                |
|          | -A        | Habilita la detección del sistema operativo, la versión del servicio y el escaneo de scripts                                                                                  |
|          | -v        | Modo verboso: aumenta el nivel de detalle en la salida                                                                                                                        |
|          | -T        | Especifica el nivel de velocidad del escaneo (de T0 a T5)                                                                                                                     |
|          | -oN       | Guarda la salida en un archivo de texto normal                                                                                                                                |
|          | -oX       | Guarda la salida en formato XML                                                                                                                                               |
|          | -oG       | Guarda la salida en formato Grepable (para su uso con scripts)                                                                                                                |
|          | -sV       | Devuelve la versión del servicio que se ejecuta en el puerto                                                                                                                  |
|          | -p-       | Realiza un escaneo a todos los puertos existentes                                                                                                                             |
| --script | auth      | Determinación de credenciales de autenticación.                                                                                                                               |
| --script | broadcast | Scripts utilizados para la detección de hosts mediante transmisión y los hosts descubiertos pueden ser agregados automáticamente a los escaneos restantes.                    |
| --script | default   | Scripts predeterminados ejecutados utilizando la opción -sC.                                                                                                                  |
| --script | discovery | Evaluación de servicios accesibles.                                                                                                                                           |
| --script | dos       | Scripts utilizados para verificar servicios en busca de vulnerabilidades de denegación de servicio y se usan menos porque dañan los servicios.                                |
| --script | exploit   | Esta categoría de scripts intenta explotar vulnerabilidades conocidas para el puerto escaneado.                                                                               |
| --script | fuzzer    | Esta categoría utiliza scripts para identificar vulnerabilidades y el manejo inesperado de paquetes mediante el envío de diferentes campos, lo que puede llevar mucho tiempo. |
| --script | safe      | Scripts defensivos que no realizan acceso intrusivo y destructivo.                                                                                                            |
| --script | version   | Extensión para la detección de servicios.                                                                                                                                     |
| --script | vuln      | Identificación de vulnerabilidades específicas.                                                                                                                               |

### **enum4linux (Enumeración de usuarios)**

Es una herramienta de enumeración de información para sistemas Windows y Samba en entornos de red. Se utiliza para recopilar información valiosa sobre **usuarios, grupos, recursos compartidos y políticas de seguridad** en sistemas Windows y en servidores Samba que se ejecutan en sistemas Linux.

| Opción        | Descripción                                                       |
| ------------- | ----------------------------------------------------------------- |
| `-A`          | Ejecuta todas las opciones de enumeración disponibles.            |
| `-U`          | Enumera información de usuarios.                                  |
| `-G`          | Enumera información de grupos.                                    |
| `-M`          | Enumera información de máquinas.                                  |
| `-S`          | Enumera información de políticas de seguridad.                    |
| `-P`          | Enumera información de recursos compartidos.                      |
| `-N`          | No enumera el listado de recursos compartidos.                    |
| `-r`          | Enumera información de confianza de dominio.                      |
| `-i <ipaddr>` | Dirección IP del objetivo.                                        |
| `-u <user>`   | Nombre de usuario para autenticación.                             |
| `-p <pass>`   | Contraseña para autenticación.                                    |
| `-d`          | Modo depuración, imprime información detallada.                   |
| `-v`          | Modo verbose, imprime información adicional durante la ejecución. |

### **nc**

es una poderosa herramienta de línea de comandos que se utiliza para leer y escribir datos a través de conexiones de red utilizando el protocolo TCP/IP o UDP

| Opción        | Descripción                                                       |
| ------------- | ----------------------------------------------------------------- |
| -l            | Modo de escucha (listener).                                       |
| -p <port>     | Especifica el puerto en el que escuchar o conectar.               |
| -u            | Usa UDP en lugar de TCP.                                          |
| -v            | Modo verbose, muestra información detallada durante la ejecución. |
| -e <command>  | Ejecuta un comando después de que se establezca la conexión.      |
| -z            | Escanea puertos sin establecer una conexión.                      |
| -k            | Mantiene la conexión abierta para conexiones múltiples.           |
| -w <timeout>  | Establece un tiempo de espera para las conexiones.                |
| -i <interval> | Establece un intervalo entre envíos de datos.                     |
| -h            | Muestra la ayuda y la lista de opciones disponibles.              |

### **getcap**

es una herramienta utilizada en sistemas Unix y Linux para mostrar los atributos de capacidades extendidas asociados con archivos y directorios en el sistema de archivos.

| Opción          | Descripción                                                                                                                              |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| -r, --recursive | Realiza la búsqueda de forma recursiva en los directorios especificados para encontrar archivos con atributos de capacidades extendidas. |
| -v, --version   | Muestra la versión de `getcap` instalada en el sistema.                                                                                  |
| -h, --help      | Muestra el mensaje de ayuda que describe el uso y las opciones disponibles de `getcap`.                                                  |

### **ffuf (Enumeración de directorios)**

Es un fuzzer web rápido escrito en Go que permite el descubrimiento típico de directorios, el descubrimiento de hosts virtuales (sin registros DNS) y el fuzzing de parámetros GET y POST.

| Flag       | Descripción                                                                                                           |
| ---------- | --------------------------------------------------------------------------------------------------------------------- |
| `-u`       | Especifica la URL objetivo.                                                                                           |
| `-w`       | Especifica el diccionario a utilizar para fuzzear.                                                                    |
| `-t`       | Establece el número de hilos para el proceso de fuzzing.                                                              |
| `-methods` | Especifica los métodos HTTP a utilizar en las solicitudes.                                                            |
| `-H`       | Agrega encabezados HTTP personalizados a las solicitudes.                                                             |
| `-mc`      | Haga coincidir los códigos de estado HTTP, o "todos" para todo. (predeterminado: 200-299.301.302.307.401.403.405.500) |
| `-c`       | Configura la cadena de búsqueda que indica una coincidencia.                                                          |
| `-o`       | Especifica el archivo de salida para los resultados.                                                                  |
| `-fc`      | Filtra el código de estado HTTP de las respuestas.                                                                    |
| `-s`       | No imprimir información adicional (modo silencioso) (predeterminado: falso)                                           |
| `-p`       | Segundos de "retraso" entre solicitudes, o un rango de retraso aleatorio. Por ejemplo "0,1" o "0,1-2,0"               |
| `-e`       | Lista de extensiones separadas por comas. Extiende la palabra clave FUZZ.                                             |

### **SSH**

SSH (Secure Shell) es un protocolo de red seguro que se utiliza para establecer una conexión segura entre un cliente y un servidor, permitiendo así la transferencia de datos de forma cifrada a través de una red no segura, como Internet. SSH proporciona una forma segura de acceder y administrar sistemas remotos, así como de transferir archivos de manera segura.

| Comando                                                    | Descripción                                                                                            |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| `ssh user@host`                                            | Conectar al servidor SSH especificado como `usuario` en `host`.                                        |
| `ssh -i ruta/clave_privada user@host`                      | Conectar al servidor SSH utilizando una clave privada específica.                                      |
| `scp archivo usuario@host:ruta/destino`                    | Copiar un archivo desde tu máquina local al servidor SSH remoto.                                       |
| `scp usuario@host:ruta/archivo destino_local`              | Copiar un archivo desde el servidor SSH remoto a tu máquina local.                                     |
| `ssh -L puerto_local:localhost:puerto_remoto usuario@host` | Crear un túnel SSH local que reenvíe el tráfico desde el puerto local al puerto remoto en el servidor. |
| `ssh -D puerto_local usuario@host`                         | Crear un proxy SOCKS dinámico utilizando SSH.                                                          |
| `ssh-keygen -t rsa -b 4096 -C "correo@ejemplo.com"`        | Generar un nuevo par de claves SSH.                                                                    |
| `ssh-copy-id usuario@host`                                 | Copiar tu clave pública al archivo `~/.ssh/authorized_keys` en el servidor SSH remoto.                 |

### **John The Ripper**

es una herramienta de código abierto utilizada para realizar ataques de fuerza bruta y ataques de diccionario contra contraseñas cifradas. Es una de las herramientas más populares y potentes disponibles para crackear contraseñas en sistemas Unix y Windows.

Dirección de un diccionario: `/usr/share/wordlists/fasttrack.txt`

> [!TIP] Consideracion antes de usar esta herramienta
> Recordar que descifrar un SSHKEY por ejemplo, es necesario convertirlo a hash, usando el comando: `sudo /usr/share/john/ssh2john.py sshkey.txt > hash`

| Comando                                    | Descripción                                                                                                                 |
| ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| `john --wordlist=diccionario.txt hashfile` | Realiza un ataque de diccionario utilizando el archivo de diccionario especificado contra el hash almacenado en `hashfile`. |
| `john --incremental hashfile`              | Realiza un ataque de fuerza bruta incremental contra el hash almacenado en `hashfile`.                                      |
| `john --single hashfile`                   | Realiza un ataque de fuerza bruta utilizando un solo hash almacenado en `hashfile`.                                         |
| `john --show hashfile`                     | Muestra las contraseñas crackeadas de `hashfile`.                                                                           |
| `john --test`                              | Realiza una prueba de rendimiento de John the Ripper.                                                                       |
| `john --format=formato hashfile`           | Especifica el formato del hash utilizado en `hashfile`.                                                                     |
| `john --list=formats`                      | Muestra una lista de los formatos de hash compatibles.                                                                      |
| `john --make-charset=charset.lst`          | Genera un archivo de lista de caracteres personalizado para usar en ataques de fuerza bruta.                                |
| `john --session=nombre_de_sesión hashfile` | Inicia una sesión de trabajo con un nombre específico.                                                                      |
| `john --restore=nombre_de_sesión`          | Restaura una sesión de trabajo previamente guardada.                                                                        |

### **wget (Descargar archivos en servidores web)**

es una utilidad de línea de comandos utilizada para descargar archivos desde servidores web utilizando el protocolo HTTP, HTTPS y FTP. Es una herramienta muy útil para automatizar descargas de archivos desde la línea de comandos en sistemas Unix y Linux

| Comando                                         | Descripción                                                               |
| ----------------------------------------------- | ------------------------------------------------------------------------- |
| `wget URL`                                      | Descarga el archivo especificado en la URL.                               |
| `wget -c URL`                                   | Continúa una descarga interrumpida.                                       |
| `wget -i archivo.txt`                           | Descarga una lista de URLs listadas en un archivo de texto.               |
| `wget --limit-rate=velocidad URL`               | Limita la velocidad de descarga a la velocidad especificada.              |
| `wget --mirror -p --convert-links -P ./URL`     | Descarga un sitio web completo para su visualización local.               |
| `wget --user=usuario --password=contraseña URL` | Descarga un archivo desde un sitio web que requiere autenticación.        |
| `wget --spider URL`                             | Simplemente verifica si el archivo en la URL existe.                      |
| `wget --no-check-certificate URL`               | Descarga el archivo desde una URL HTTPS sin verificar el certificado SSL. |
| `wget --help`                                   | Muestra la ayuda y la lista de opciones de `wget`.                        |

### Sqlmap (Pendiente)

