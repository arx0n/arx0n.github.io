
# CIDrift

CIDrift es una pequeña herramienta desarrollada en Bash para calcular información de redes IPv4 a partir de una dirección IP utilizando notación CIDR.

La idea surgió para facilitar el trabajo en auditorías. Por ejemplo, si me dicen que tengo que auditar esta red → `10.10.5.1/13`, puedo saber rápidamente en qué rango de IPs debo moverme y hacerme una idea de cuántos hosts puede haber.

## ¿Qué calcula?
CIDrift permite obtener:

- **Máscara de Red**
- **Dirección de Red**
- **Dirección Broadcast**
- **Hosts disponibles**

Además, la herramienta realiza una validación básica de la dirección IPv4 y del prefijo CIDR introducido.

## Uso

Puedes utilizar CIDrift directamente desde la terminal:

```bash
cidrift 192.168.1.10/24
```

También puedes ejecutarlo sin argumentos y proporcionar la dirección IP cuando la herramienta la solicite:

```bash
cidrift
```

Ejemplo:

```text
[+] Dirección IP -> 192.168.1.10/24
[+] Total hosts -> 254
[+] Network Mask -> 255.255.255.0
[+] Network ID -> 192.168.1.0
[+] Broadcast Address -> 192.168.1.255
```

## Instalación

Puedes encontrar el código fuente y el instalador en GitHub:

[**Cidrift - GitHub**](https://github.com/arx0n/cidrift)

Para instalarlo:

```bash
git clone https://github.com/arx0n/cidrift.git
cd cidrift
sudo ./install.sh
```

Una vez instalado, podrás utilizar CIDrift desde cualquier ubicación:

```bash
cidrift 192.168.1.10/24
```

## Características

- IPv4
- Notación CIDR
- Validación de entradas
- Cálculo de la máscara de red
- Cálculo de la dirección de la Red
- Cálculo de la dirección Broadcast
- Cálculo de los hosts utilizables
- Interfaz de terminal
- Sin dependencias externas

## Código fuente

CIDrift es un proyecto abierto y su código fuente está disponible en GitHub.

[**GitHub → arx0n/cidrift**](https://github.com/arx0n/cidrift)

---

**CIDrift · Made by arx0n**

