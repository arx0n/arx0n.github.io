# PICKLERICK

## Información

- **Plataforma:** Try Hack Me
- **Dificultad:** Fácil
- **Sistema:** Linux
- **IP:** `10.130.186.22`
- **Objetivo:** Descubrir **3 ingredientes secretos** ocultos en la máquina

## Reconocimiento

Primero creamos una carpeta para la máquina. Posteriormente, con `mkt`, que es una función que tengo definida en mi `zshrc`, creamos una serie de directorios que me ayudarán a organizarme mejor.

![PICKLERICK](/images/THM/PICKLERICK/mkt.png) 

Una vez tengamos esto, ya procedemos a trabajar en la máquina. Vamos a ver si tenemos conexión con ella y así proceder a la fase de reconocimiento. Vamos a lanzar una traza ICMP a la máquina víctima.

![PICKLERICK](/images/THM/PICKLERICK/ping.png) 

De aquí podemos sacar dos cosas importantes: la primera es que sabemos que nos estamos enfrentando a una máquina Linux por el TTL. Recordad que **Linux - TTL = 64 | Windows - TTL = 128**. 

Al trazar la traza ICMP, el paquete pasa por un nodo intermediario que, entre otras cosas, hace que el TTL disminuya en una unidad. Por proximidad, podemos determinar que estamos ante una máquina Linux. Aparte, nos dice que 1 paquete se ha enviado y 1 se ha recibido, por lo tanto, tenemos conexión con la máquina.

## Enumeración

``` bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.130.186.22 -oG allPorts
``` 

![PICKLERICK](/images/THM/PICKLERICK/nmap.png) 

<details>
<summary>Explicación del comando</summary>

- `-p-` → Escanea todo el rango de puertos, los 65535.
- `--open` → Los puertos pueden estar en diversos estados: abiertos, cerrados, filtrados. Así que este parámetro hará que solo tenga en cuenta los que vea abiertos.
- `-sS` → Realiza un SYN scan que hace que el escaneo vaya mucho más rápido, a la par que agresivo y preciso.
- `--min-rate 5000` → Para tramitar paquetes no más lentos que 5000 paquetes por segundo. Combinado con el parámetro anterior, hace que el escaneo vaya volando.
- `-vvv` → Me da un poco más de información.
- `-n` → Para que no aplique resolución DNS.
- `-Pn` → Para que no aplique descubrimiento de hosts.
- `-oG` → Me exporta el escaneo en formato grepeable.
</details>

Como vemos, nos reporta que los puertos **22** y **88** están abiertos. Podemos ver qué servicio corre en cada puerto, en este caso, el **22 → SSH** y el **88 → HTTP**.

Me gustaría saber también qué versión se está ejecutando, así que vamos a lanzar otro **nmap** que va a estar más enfocado en decirme qué versión y qué servicio corren en cada puerto.

```bash
nmap -sCV -p,22,80 10.130.186.22
```

![PICKLERICK](/images/THM/PICKLERICK/nmap2.png) 

<details>
<summary>Explicación del comando</summary>

- `-sCV` → Ejecuta scripts básicos de reconocimiento a la par que detecta las versiones para los puertos que especifiquemos.
</details>

## Acceso Inicial

Vamos a empezar investigando la página web a ver qué nos encontramos.

![PICKLERICK](/images/THM/PICKLERICK/web.png) 

Como no veo nada raro, vamos a probar a ver el código fuente de la página. Entre otras cosas, nos ayuda un poco a entender cómo está hecha la web y, de vez en cuando, puedes encontrar cosas interesantes...

![PICKLERICK](/images/THM/PICKLERICK/web2.png) 

¡EUREKA! Como veis, alguien ha dejado comentado un usuario, cosa bastante torpe por parte del programador, pero bueno.

Vamos a seguir investigando para ver qué podemos hacer con el usuario y ver si encontramos los ingredientes secretos. Antes de seguir a ciegas por la web, voy a tratar de hacer un **descubrimiento de directorios** y para eso voy a emplear la herramienta `gobuster`.

``` bash
gobuster dir -u http://10.130.186.22 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```

![PICKLERICK](/images/THM/PICKLERICK/gobuster.png) 

<details>
<summary>Explicación del comando</summary>

- `dir` → Indica que vamos a realizar un descubrimiento de directorios.
- `-u` → Especifica la URL sobre la que queremos realizar el escaneo.
- `-w` → Indica el diccionario que vamos a utilizar para realizar el descubrimiento.
- `-x` → Permite especificar las extensiones que queremos buscar.
</details>

Como podemos observar, hay cosas interesantes: un panel de **"login"**, un directorio **assets** y el que me llama la atención es el **robots.txt**. Vamos a investigar un poco.

![PICKLERICK](/images/THM/PICKLERICK/robots.png) 

Hay un texto raro que no sé bien bien lo que es, pero como tampoco estoy muy orientado hacia dónde quiero ir, vamos a ir directamente al panel de **login** a ver si encontramos algo.

![PICKLERICK](/images/THM/PICKLERICK/login.png) 

Como no hay ninguna vulnerabilidad a la vista, vamos a probar de tirar con lo poco que tenemos. Usaré el usuario que nos encontramos al principio, **R1ckRul3**, y de contraseña la del `robots.txt` que habíamos visto antes, **Wuabbalubbadubdub**.

![PICKLERICK](/images/THM/PICKLERICK/login2.png) 

Ni tan mal, tenemos acceso a lo que parece ser una ejecución remota de comandos. Vamos a tratar de ver lo máximo que podemos sacar desde aquí. 

## Explotación

![PICKLERICK](/images/THM/PICKLERICK/whoami.png) 

Como veis, se confirma que tenemos acceso mediante **ejecución remota de comandos** o, dicho de manera más técnica, un **RCE**. Así que vamos a tratar de encontrar los ingredientes.

![PICKLERICK](/images/THM/PICKLERICK/pwd.png) 

Al ejecutar un `pwd`, puedo ver el directorio actual en el que me encuentro, así que vamos a ejecutar `ls` y así listar el contenido de **/var/www/html**.

![PICKLERICK](/images/THM/PICKLERICK/ls.png) 

¡Tenemos el primer ingrediente! Vamos a tratar de ver cuál es, podemos usar el comando `cat` para leer el archivo.

![PICKLERICK](/images/THM/PICKLERICK/cat.png) 

¡Vaya! Parece ser que nos lo van a poner complicado. Aquí podríamos hacer varias cosas, como es un archivo que está en el **/var/www/html**, podemos tratar de ejecutarlo desde la web o usar un comando alternativo a **cat**. Voy a usar `less` porque me parece más sencillo y cómodo.

![PICKLERICK](/images/THM/PICKLERICK/less.png) 

¡Tenemos el primer ingrediente! Faltan otros dos, vamos a tratar de encontrarlos. Si os habéis fijado, al hacer el `ls` había más archivos, vamos a ver qué otras cosas sacamos.

![PICKLERICK](/images/THM/PICKLERICK/clue.png) 

Como veis, tenemos otro archivo de texto llamado **clue.txt**. Esta vez sí que voy a ejecutarlo desde la web, así vemos varias maneras de hacer una misma cosa.

![PICKLERICK](/images/THM/PICKLERICK/clue2.png) 

Nos da lo que parece ser una especie de pista que vamos a aprovechar. Vamos a tratar de encontrar archivos a nivel de sistema que contengan la palabra **ingredient** y para ello usaremos `find`.

``` bash
find / -iname "ingredient" 2>/dev/null
```

![PICKLERICK](/images/THM/PICKLERICK/find.png) 

<details>
<summary>Explicación del comando</summary>

- `find /` → Busca archivos y directorios desde la raíz del sistema.
- `-iname "ingredient"` → Busca elementos cuyo nombre sea `ingredient`, sin distinguir entre mayúsculas y minúsculas.
- `2>/dev/null` → Redirige los errores de permisos u otros errores al agujero negro de **/dev/null** para que no aparezcan por pantalla.

</details>

¡Uueeepa! Tenemos el segundo ingrediente. Como no sé en qué formato está, voy a tirar de una vez con el comando `strings` y así nos ahorramos tiempo.

![PICKLERICK](/images/THM/PICKLERICK/2nd.png) 

¡Ya tendríamos el segundo ingrediente! Como el comando `find` no me ha reportado más ingredientes, sospecho que el tercer ingrediente estará accesible con el usuario **root** y tendrá un nombre que no nos esperamos, así que vamos de una con la escalada de privilegios.

## Escalada de privilegios

Muy bien, vamos a tirar de comandos básicos para ver qué podemos hacer. Voy a empezar con el clásico `sudo -l` para ver qué comandos podemos usar mediante sudo y si necesitamos contraseña o no.

![PICKLERICK](/images/THM/PICKLERICK/sudo.png) 

Pues sí que era fácil, como podemos ver, tiene acceso a todos los comandos y no necesita contraseña. Se me ocurre listar de primeras lo que tenemos en el directorio personal de **root** usando el comando `sudo ls /root`.

![PICKLERICK](/images/THM/PICKLERICK/lsroot.png) 

Y como veis, tenía razón, el nombre del tercer ingrediente no lo podríamos haber sacado con `find` (o sí), así que hemos hecho bien en proceder de esta forma. Vamos a listarlo y ya estaría.

![PICKLERICK](/images/THM/PICKLERICK/root.png) 

Ya tendríamos el tercer y último ingrediente y habríamos resuelto la máquina.

## Conclusión

Máquina bastante sencilla, hemos tirado de reconocimiento web básico, hemos conseguido acceso a un panel en el que hemos podido ejecutar comandos y os propongo un reto. 

## RETO

Como veis, hemos encontrado los tres ingredientes desde la web, cosa que no está mal, pero no hemos podido ser **root** de manera más clara. Os propongo que transforméis el **RCE** en una **Reverse Shell**. Así podremos ganar acceso total al sistema y no solo tener lo necesario para resolver la máquina. ¡Suerte!

![PICKLERICK](/images/THM/PICKLERICK/reto.png) 

La IP ha cambiado porque mientras hacía el writeup se me cerró la conexión a la máquina y la tuve que volver a spawnear.