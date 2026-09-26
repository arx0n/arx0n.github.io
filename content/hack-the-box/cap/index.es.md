# CAP

## Información

- **Plataforma:** Hack The Box
- **Dificultad:** Fácil
- **Sistema:** Linux
- **IP:** `10.129.22.41`
- **Objetivo:** Descubir la flag del **usuario** y de **root**

## Reconocimiento

Primero creamos una carpeta para la máquina. Posteriormente, con 'mkt', que es una función que tengo definida en mi 'zshrc', creamos una serie de directorios que me ayudarán a organizarme mejor.

![CAP](/images/HTB/CAP/mkt.png) 

Una vez tengamos esto, ya procedemos a trabajar en la máquina. Vamos a ver si tenemos conexión con ella y así proceder a la fase de reconocimiento. Vamos a lanzar una traza ICMP a la máquina víctima.

![CAP](/images/HTB/CAP/ping.png) 

De aquí podemos sacar dos cosas importantes: la primera es que sabemos que nos estamos enfrentando a una máquina Linux por el TTL. Recordad que **Linux - TTL = 64 | Windows - TTL = 128**. 

Al trazar la traza ICMP, el paquete pasa por un nodo intermediario que, entre otras cosas, hace que el TTL disminuya en una unidad. Por proximidad podemos determinar que estamos ante una máquina Linux. Aparte, nos dice que 1 paquete se ha enviado y 1 se ha recibido, por lo tanto, tenemos conexión con la máquina.

## Enumeración


``` bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.129.22.41 -oG allPorts
``` 

![CAP](/images/HTB/CAP/nmap.png) 

<details>
<summary>Explicación del comando</summary>

- `-p-` → Escanea todo el rango de puertos, los 65535.
- `--open` → Los puertos pueden estar en diversos estados: abiertos, cerrados, filtrados. Así que este parámetro hará que solo tenga en cuenta los que vea abiertos.
- `-sS` → Realiza un SYN scan que hace que el escaneo vaya mucho más rápido, a la par que agresivo y preciso.
- `--min-rate 5000` → Para tramitar paquetes no más lentos que 5000 paquetes por segundo, combinado con el parámetro anterior hace que el escaneo vaya volando.
- `-vvv` → Me da un poco más de información.
- `-n` → Para que no aplique resolución DNS.
- `-Pn` → Para que no aplique descubrimiento de Hosts.
- `-oG` → Me exporta el escaneo en formato grepeable.
</details>


Como vemos, nos reporta que los puertos **21**, **22** y **88** están abiertos. Podemos ver qué servicio corre en cada puerto, en este caso el **21 → FTP**, el **22 → SSH** y el **88 → HTTP**. 

Me gustaría saber también qué versión se está ejecutando, así que vamos a lanzar otro **nmap** que va a estar más enfocado en decirme qué versión y qué servicio corren en cada puerto.

```bash
nmap -sCV -p21,22,80 10.129.22.41
```

![CAP](/images/HTB/CAP/nmap2.png) 

<details>
<summary>Explicación del comando</summary>

- `-sCV` → Ejecuta scripts básicos de reconocimiento a la par que detecta las versiones para los puertos que especifiquemos.
</details>

## Acceso Inicial

Como no veo nada que me llame en especial la atención, voy a probar a conectarme por **ftp** usando credenciales básicas tipo `admin-admin`, `admin-1234`, etc.

![CAP](/images/HTB/CAP/ftp.png) 

Pero nada parece funcionar, así que vamos a ir directamente a ver qué es la web y qué podemos explotar de ella.

![CAP](/images/HTB/CAP/web.png) 

Como podéis ver, la web es una especie de panel de ciberseguridad en la que parecen haber monitorizaciones y análisis de seguridad. Podemos observar que arriba está el usuario **Nathan**. Pero fuera de eso no veo nada raro, vamos a husmear por el menú del panel.

![CAP](/images/HTB/CAP/webb2.png) 

Se ve como un panel donde puedes descargarte datos de paquetes que tienen almacenados. Voy a probar a descargármelo a ver qué sale.

![CAP](/images/HTB/CAP/web3.png) 

Se ve que te descarga un archivo **.pcap** con un número como nombre, si os fijáis estoy en el directorio data y al final hay otro directorio, en este caso **1**. He ido probando números y el que más me ha llamado la atención es este:

![CAP](/images/HTB/CAP/web4.png) 

Como podéis ver, poniendo **/0** hay un número de paquetes importante, ya no es 1, así que voy a descargarme el archivo y vamos a ver si tiene chicha.

## Explotación

![CAP](/images/HTB/CAP/pcap.png) 

Me he movido del directorio **Descargas** el archivo al directorio actual de trabajo y, como podéis ver, es el mismo archivo que descargas desde la web. Vamos a intentar analizar lo que trae con `Tshark`.

![CAP](/images/HTB/CAP/tshark.png) 

Como veis, contiene bastante información sobre paquetes, si os fijáis, en un punto se ve una credencial, así que en cuanto lo he visto, para no perder tiempo, he decidido filtrar la salida y esto es el resultado:

![CAP](/images/HTB/CAP/tshark2.png) 

<details>
<summary>Explicación del comando</summary>

- `tshark` → Analiza tráfico de red vía terminal.
-`-r` → Este parámetro sirve para indicarle el archivo que nosotros queremos.
-`grep` → Sirve para buscar texto dentro de una entrada o archivo.
-`Ei (x)` → Permite usar expresiones regulares extendidas, es decir, nos deja que busquemos el texto que queramos ignorando mayúsculas y minúsculas. 
</details>

¡Tenemos credenciales! Hemos sacado una credencial que parece ser la del usuario **nathan**, el mismo que había en la web. Vamos a intentar entrar vía **SSH** y ver si logramos realizar un acceso a la máquina.

![CAP](/images/HTB/CAP/ssh.png) 

Con `sshpass` podemos poner la contraseña directamente. Como vemos, las credenciales son válidas y conseguimos acceso a la máquina. Voy a tratar de ver en qué ruta estoy y sacar la primera **flag**.

![CAP](/images/HTB/CAP/txt1.png) 

Como vemos, ya tendríamos la primera flag. Vamos a ver si podemos escalar privilegios y convertirnos en root.

## Escalada de privilegios

He estado mirando permisos **SUID** y no he encontrado nada, así que, como la máquina se llama **Cap**, he tratado de buscar alguna **capability**.

![CAP](/images/HTB/CAP/capabi.png) 

Como veis, `python3.8` tiene una capability, así que podemos escalar privilegios importando la librería **os**. Cambiaríamos el **UID** por el de root y ya estaría.

![CAP](/images/HTB/CAP/root1.png) 

Como os decía, al importarnos la librería **os** podemos ejecutar comandos a nivel de sistema y, una vez cambiado el **UID** por el de **root**, nos abrimos una **bash** y hemos escalado privilegios.

## Conclusión

Máquina bastante sencilla en la que hemos conseguido acceso inicial mediante la extracción de credenciales de un archivo **.pcap** y posteriormente hemos escalado privilegios aprovechando una **capability** asignada a `python3.8`.