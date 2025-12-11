# HTB: Dancing
- Dirección IP Objetivo: 10.129.18.194
- Sistema Operativo Objetivo: Microsoft Windows
- Dificultad: Muy Fácil (Debugging)
- Autor: Daniel Barbeyto Torres
- Fecha: 2025-12-11

## I. Resumen Ejecutivo (Executive Summary)
La máquina objetivo, Dancing, fue comprometida a nivel de usuario mediante la explotación de una configuración errónea en el servicio SMB (Server Message Block). El servidor permitía el acceso anónimo o null session a la compartición personalizada llamada WorkShares. Este acceso sin credenciales permitió la enumeración de archivos dentro del directorio de un usuario, donde se encontró y descargó la flag de compromiso (flag.txt).

## II. Enumeración y Descubrimiento (Reconnaissance)
### 2.1. Verificación de Conectividad (Ping)
Se verificó la accesibilidad de la máquina objetivo mediante un ping ICMP. 

### 2.2. Escaneo de Puertos y Servicios (Nmap)
Se procedió a realizar un escaneo detallado de puertos utilizando Nmap con la detección de servicios (-sV). 

Comando Ejecutado:

```bash
sudo nmap -sV 10.129.18.194
```

![a](img/dancing_1.md)

Resultado: El escaneo reveló que la máquina es un sistema Windows y expone los siguientes servicios:

135/tcp (msrpc): Microsoft Windows RPC
139/tcp (netbios-ssn): Microsoft Windows netbios-ssn
445/tcp (microsoft-ds):  Puerto SMB/CIFS. Esto indica que hay un recurso compartido activo que se puede intentar explorar potencialmente. 

## III. Proceso de Explotación: Abuso de SMB
### 3.1. Enumeración de Recursos Compartidos (Shares)
Para enumerar el contenido compartido en el sistema remoto, se utilizó la utilidad smbclient. 

Se intentó una conexión sin credenciales (sesión nula o como invitado/anónimo ), dejando el campo de contraseña en blanco, para listar las comparticiones disponibles. 


Comando Ejecutado:

```bash
smbclient -L 10.129.18.194
```

![a](img/dancing_2.md)

Resultado: El comando reveló cuatro comparticiones distintas:
- ADMIN$: Compartición administrativa oculta, típicamente inaccesible sin credenciales de administrador. 
- C$: Compartición administrativa para el volumen de disco C:, donde se aloja el sistema operativo. 
- IPC$: Compartición de comunicación entre procesos (IPC), no forma parte del sistema de archivos. 
- WorkShares: Una compartición personalizada. 

### 3.2. Conexión a las Comparticiones
Se procedió a intentar una conexión a las comparticiones ADMIN$, C$ y WorkShares sin proporcionar credenciales, buscando un fallo de configuración. 

Comandos Ejecutados (intentando sesión nula):

```bash
smbclient \\\\10.129.18.194\\ADMIN$ 
smbclient \\\\10.129.18.194\\C$
smbclient \\\\10.129.18.194\\WorkShares
```

![a](img/dancing_3.md)

Se recibió un error NT_STATUS_ACCESS_DENIED al intentar acceder a ADMIN$ y C$, lo que indica que se requieren credenciales válidas para dichas comparticiones. 

Sin embargo, el acceso a WorkShares fue exitoso, indicando que un admnistrador configuró incorrectamente los permisos, permitiendo el inicio de sesión sin contraseña.

## IV. Post-Explotación y Recuperación de la Flag
### 4.1. Localización y Extracción de la Flag
Una vez dentro de la compartición WorkShares, se exploró el contenido mediante el comando ls. Se encontraron dos directorios, Amy.J y James.P.

Se navegó a ambos directorios. En el directorio James.P se localizó el archivo flag.txt.

Comandos Ejecutados (dentro de la shell SMB):

```bash
smb: \> cd James.P
smb: \James.P\> ls
smb: \James.P\> get flag.txt
```

![a](img/dancing_4.md)

El archivo se descargó correctamente al sistema local. 

## 4.2. Confirmación del Compromiso
Se salió del cliente SMB y se visualizó el contenido del archivo flag.txt en la terminal local. 

```bash
ls
cat flag.txt
```

![a](img/dancing_5.md)

## V. Conclusión y Remedios
### 5.1. Impacto
La máquina Dancing presentaba un fallo de seguridad crítico al permitir el acceso anónimo a la compartición WorkShares. Este acceso no autenticado llevó directamente a la exposición de archivos sensibles y al compromiso de la máquina.

### 5.2. Recomendaciones de Seguridad
Se recomienda la aplicación de las siguientes medidas correctivas:
- Deshabilitar el Acceso Anónimo/Sesión Nula: Restringir el acceso a todas las comparticiones SMB para que requieran autenticación de usuario.
- Revisión de Permisos: Revisar los permisos de las comparticiones personalizadas (WorkShares) para asegurar que solo los usuarios autorizados (y con credenciales) puedan acceder a ellas.
- Seguridad de Archivos: Asegurar que los archivos sensibles como las flags no se almacenen en directorios compartidos accesibles públicamente.
