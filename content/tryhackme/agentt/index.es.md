# Agent T

## Información

- **Plataforma:** Try Hack Me
- **Dificultad:** Fácil
- **Sistema:** Linux
- **IP:** `10.129.138.6`
- **Objetivo:** Descubrir la flag del usuario **root**.

## Reconocimiento

Primero creamos una carpeta para la máquina. Posteriormente, con `mkt`, que es una función que tengo definida en mi `zshrc`, creamos una serie de directorios que me ayudarán a organizarme mejor.

![AGENTT](/images/THM/AGENTT/mkt.png) 

Una vez tengamos esto, ya procedemos a trabajar en la máquina. Vamos a ver si tenemos conexión con ella y así proceder a la fase de reconocimiento. Vamos a lanzar una traza ICMP a la máquina víctima.

![AGENTT](/images/THM/AGENTT/ping.png) 

De aquí podemos sacar dos cosas importantes: la primera es que sabemos que nos estamos enfrentando a una máquina Linux por el TTL. Recordad que **Linux - TTL = 64 | Windows - TTL = 128**. 

Al trazar la traza ICMP, el paquete pasa por un nodo intermediario que, entre otras cosas, hace que el TTL disminuya en una unidad. Por proximidad, podemos determinar que estamos ante una máquina Linux. Aparte, nos dice que 1 paquete se ha enviado y 1 se ha recibido, por lo tanto, tenemos conexión con la máquina.

## Enumeración

``` bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.129.138.6 -oG allPorts
``` 

![AGENTT](/images/THM/AGENTT/nmap.png) 

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

Como vemos, nos reporta que el puerto **80** está abierto. Podemos ver qué servicio corre en él, en este caso **80 → HTTP**.

Me gustaría saber también qué versión se está ejecutando, así que vamos a lanzar otro **nmap** que va a estar más enfocado en decirme qué versión y qué servicio corren en cada puerto.

```bash
nmap -sCV -p,22,80 10.129.138.6
```

![AGENTT](/images/THM/AGENTT/nmap2.png) 

<details>
<summary>Explicación del comando</summary>

- `-sCV` → Ejecuta scripts básicos de reconocimiento a la par que detecta las versiones para los puertos que especifiquemos.
</details>

Esta versión en concreto de **php** me suena que tiene algo a lo que sacarle partido pero vamos a pasarnos un poco por la web primero. 

## Acceso Inicial

Vamos a empezar investigando la página web a ver qué nos encontramos. Hoy antes de meterme voy a usar un comando llamado `whatweb` que nos reporta un poco de información sobre esta misma.

![AGENTT](/images/THM/AGENTT/whatweb.png) 

Como veis nos complementa un poco más las versiones del servidor web, frameworks, cms, lenguajes, librerías... Es bastante útil para tener una idea de como esta montada la web.
Vamos a meternos a ver que tiene esta máquina.

![AGENTT](/images/THM/AGENTT/web.png) 

Como no veo nada raro, vamos a probar a hacer un reconocimiento de directorios con `gobuster`. 

``` bash
gobuster dir -u http://10.129.138.6 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```

![AGENTT](/images/THM/AGENTT/web2.png) 

<details>
<summary>Explicación del comando</summary>

- `dir` → Indica que vamos a realizar un descubrimiento de directorios.
- `-u` → Especifica la URL sobre la que queremos realizar el escaneo.
- `-w` → Indica el diccionario que vamos a utilizar para realizar el descubrimiento.
- `-x` → Permite especificar las extensiónes que queremos buscar.
</details>

Curiosamente nos reporta que todas las respuestas han devuelto un 200 como código de estado por lo que no nos sirve de nada intentar sacar directorios.
Vamos a tirar de una herramienta llamada `searchsploit` para ver si el **PHP** es vulnerable.

## Explotación

``` bash
searchsploit php 8.1.0-dev
```

![AGENTT](/images/THM/AGENTT/searchsploit.png) 

Como podemos ver, sí existe una vulnerabilidad tipo **RCE**, es decir que podemos ejecutar comandos remotamente. Vamos a descargárnoslo con el parámetro `-m` acompañado del número del exploit sin la extensión de este mismo, en este caso `49933`.

![AGENTT](/images/THM/AGENTT/49933.png) 

Vamos a ver el código del archivo:

![AGENTT](/images/THM/AGENTT/catt.png) 

Este script aprovecha una vulnerabilidad de PHP 8.1.0-dev para ejecutar comandos mediante la cabecera User-Agent y conseguir una pseudo-shell en el servidor. Vamos a ejecutarlo con `python3` y a ver que nos reporta.

## Escalada de privilegios

![AGENTT](/images/THM/AGENTT/sploit.png) 

Como veis, se confirma que tenemos acceso mediante **ejecución remota de comandos** o, dicho de manera más técnica, un **RCE**. He tratado de ver si me podía mover entre directorios y como veis no se puede. Pero por otro lado ya somos el usuario `root`. 

Voy a tratar de buscar un archivo que contenga la flag con el comando `find`.

``` bash
find / -type f -name "*.txt" 2>/dev/null
```

![AGENTT](/images/THM/AGENTT/find.png) 

<details>
<summary>Explicación del comando</summary>

- `find /` → Busca archivos y directorios desde la raíz del sistema.
- `-type f` → Sirve para indicar que queremos buscar, en este caso `f` para buscar archivos.
- `-name "*.txt"` → Sirve para indicarle el nombre del archivo, en este caso ponemos un `*` para decirle que busque **todos** los ficheros `.txt`.
- `2>/dev/null` → Redirige los errores de permisos u otros errores al agujero negro de **/dev/null** para que no aparezcan por pantalla.

</details>

Como veis nos encuentra un archivo llamado `flag.txt` y se ubica en la raíz, vamos a usar `cat` para listar su contenido y ya lo tendríamos.

## Conclusión

Máquina bastante sencilla, hemos tirado de reconocimiento web básico, hemos usado la herramienta `searchsploit` para buscar vulnerabilidades y mediante un programa con `python` hemos entrado por todo lo alto como usuario **root**. 

## Comentarios

{{< comments-es >}}