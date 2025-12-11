# HTB: Meow
- Dirección IP Objetivo: 10.129.18.148
- Dificultad: Muy Fácil (Debugging)
- Autor: Daniel Barbeyto Torres
- Fecha: 2025-12-11

## I. Resumen Ejecutivo (Executive Summary)
La máquina objetivo, Meow, fue comprometida a nivel de root sin necesidad de explotación de vulnerabilidades complejas. El vector de ataque principal fue la exposición del servicio Telnet (puerto 23/tcp) configurado con credenciales por defecto (o sin contraseña) para el usuario root. Esto permitió el acceso inmediato de administrador al sistema y la subsiguiente recuperación de la flag de compromiso.

## II. Enumeración y Descubrimiento (Reconnaissance)
Cuando se comienza un test de penetración o cualquier tipo de evaluación de seguridad, el primer paso es conocido como Enumeración. Este paso consiste en documentar el estado actual de la máquina objetivo para aprender lo máximo posible de él.

### 2.1. Verificación de Conectividad (Ping)
Se verificó la accesibilidad de la máquina mediante un ping ICMP. Después de ver que el ping funciona, podemos ver que hay acceso posible a la máquina Miau.

### 2.2. Escaneo de Puertos y Servicios (Nmap)
Para comenzar el escaneo, se utilizó el comando Nmap (Network Mapper). Nmap manda requests a los puertos de la máquina objetivo para tratar de recibir una respuesta, de forma que puede determinar qué puertos están abiertos o cuáles no.

Se usó la flag -sV para determinar el nombre y la descripción de los servicios identificados, ya que algunos puertos son usados por servicios por defectos y otros pueden no ser estándar.

Comando Ejecutado:

```bash
sudo nmap -sV 10.129.18.148
```

![a](img/meow_1.png)

Resultado: El escaneo reveló que solamente el puerto 23/tcp se encontraba abierto, ejecutando el servicio Telnet (telnet Linux telnetd).

## III. Proceso de Explotación

### 3.1. Conexión al Servicio Telnet
Telnet es un protocolo de red del pleistoceno que sirve para manejar una máquina remotamente por línea de comandos. A nivel de lo que se ve, es muy parecido al SSH, pero SSH es cifrado y en Telnet se manda todo por texto plano (por eso SSH es más seguro).


Comando Ejecutado:

```bash
telnet 10.129.18.148
```
La conexión fue exitosa, presentando una prompt de inicio de sesión (Meow login:).

### 3.2. Abuso de Credenciales por Defecto (Acceso Root)
Al no disponer de credenciales, se buscó la manera de entrar. A veces, por fallos de configuración, algunas cuentas importantes suelen dejarse sin contraseña, lo cual es un problema muy grave. Un atacante puede usar un diccionario de nombres típicos (como admin, administrator, root) e ir probándolos por fuerza bruta.

Se introdujo el nombre de usuario: root (uno de los más comunes ). Se dejó el campo de contraseña vacío o se usó la credencial por defecto.

Se consiguió entrar en el sistema. El intento de login con credenciales triviales (usuario root sin contraseña) fue exitoso, otorgando una shell de administrador (root@Meow:~#) directamente.

## IV. Post-Explotación y Recuperación de la Flag
### 4.1. Localización y Extracción de la Flag
Una vez dentro del sistema con privilegios de root, se exploró el directorio home (~) para ver si se encontraba lo que se buscaba.


Comandos Ejecutados:

```bash
ls
cat flag.txt
```
En el directorio /root se encontró el archivo flag.txt, y se obtuvo la flag de compromiso.

## V. Conclusión y Remedios
### 5.1. Impacto
La máquina Meow presentaba un fallo de seguridad crítico al permitir el acceso con privilegios de root a través del protocolo Telnet utilizando credenciales por defecto. Esto resulta en un compromiso completo e inmediato del sistema.

### 5.2. Recomendaciones de Seguridad
Se recomienda encarecidamente la aplicación de las siguientes medidas correctivas:

1. Deshabilitar Telnet (Puerto 23/tcp): Telnet es un protocolo inseguro (caca ) que transmite datos en texto plano. Debe ser reemplazado por SSH (Secure Shell), que es cifrado (bien ).
2. Imponer Contraseñas Fuertes: Bajo ninguna circunstancia se deben utilizar o mantener cuentas con contraseñas vacías o credenciales por defecto.
