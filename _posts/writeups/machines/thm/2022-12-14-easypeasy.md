---
title: '[THM] Easy Peasy'
date: 2022-12-14 19:30 PM
categories: [Machines, THM]
tags: [easy, linux, machines, ffuf, curso]
permalink: /:categories/:title
img_path: /assets/img/machines/thm/easypeasy
---

![banner](banner.png)

## Enumeration

### NMAP 

Para empezar veremos los puertos que tenemos accesibles, para esto usamos **nmap** a todos los puertos.

```console
$ nmap -sV -sC -p- -oN ports 10.10.93.173
Nmap scan report for 10.10.93.173
Host is up (0.089s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT      STATE SERVICE VERSION
80/tcp    open  http    nginx 1.16.1
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Welcome to nginx!
|_http-server-header: nginx/1.16.1
6498/tcp  open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 304a2b22acd95609f2da122057f46cd4 (RSA)
|   256 bf86c9c7b7ef8c8bb994ae0188c0854d (ECDSA)
|_  256 a172ef6c812913ef5a6c24034cfe3d0b (ED25519)
65524/tcp open  http    Apache httpd 2.4.43 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.43 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### Fuzzing

Ahora realizamos fuzzing a los directorios después de ver que en *robots.txt* no hay nada interesante. La herramienta que usaré es [ffuf](https://github.com/ffuf/ffuf).

```console
$ ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -u http://10.10.226.197/FUZZ -ic -c   

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.5.0 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.226.197/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
________________________________________________

                        [Status: 200, Size: 612, Words: 79, Lines: 26, Duration: 54ms]
hidden                  [Status: 301, Size: 169, Words: 5, Lines: 8, Duration: 49ms]
:: Progress: [22089/87651] :: Job [1/1] :: 429 req/sec :: Duration: [0:00:33] :: Errors: 0 :
```

> Vemos un directorio interesante...

En ese directorio no se encontraba nada excepto una imagen. Seguiremos con [ffuf](https://github.com/ffuf/ffuf), pero esta vez haremos fuzzing dentro del directorio */hidden*.

```console
$ ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -u http://10.10.226.197/hidden/FUZZ -ic -c

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.5.0 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.226.197/hidden/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
________________________________________________

                        [Status: 200, Size: 390, Words: 47, Lines: 19, Duration: 53ms]
whatever                [Status: 301, Size: 169, Words: 5, Lines: 8, Duration: 48ms]
:: Progress: [39589/87651] :: Job [1/1] :: 767 req/sec :: Duration: [0:00:53] :: Errors: 0 ::
```
> Aquí empezamos con cosas más llamativas en el *source code*.

## Foothold

### Primera flag

Esta la encontraremos como he mencionado antes en el código fuente de la web, en concreto en el directorio */hidden/whatever*.

![whatever](whatever-web.png)

> Es una simple imagen, pero si miramos el código...

Aquí vemos una etiqueta con el atributo **hidden** el cuál contiene un texto encriptado en formato **base64**.

![source-code](source.png) 

Para desencriptarlo lo haremos directamente con terminal de la siguiente forma.

```console
$ echo "ZmxhZ3tmMXJzN19mbDRnfQ==" | base64 -d                                       
flag{f1rs7_fl4g}
```
> Primera flag: **flag{f1rs7_fl4g}**
{: .prompt-tip }

### Segunda flag

Aparte de esa primera página tenemos que recordar que con **nmap** también vimos que el servicio apache está abierto en el puerto 65524. Entraremos desde el navegador a ver si se encuentra algo.

Al entrar nos encontramos con el index por defecto de apache, revisaremos *robots.txt* y *código fuente*. 

![apache](apache.png) 

Dentro del fichero *robots.txt* tenemos algo que llama la atención como *User-Agent*.

```shell
User-Agent:*
Disallow:/
Robots Not Allowed
User-Agent:a18672860d0510e5ab6699730763b250
Allow:/
This Flag Can Enter But Only This Flag No More Exceptions
```
{: file="/robots.txt" }

Ese hash lo copiaremos y lo pasaremos por `hash-identifier` para saber que tipo de hash estamos tratando.

```console
$ hash-identifier a18672860d0510e5ab6699730763b250                                
   #########################################################################
   #     __  __                     __           ______    _____           #
   #    /\ \/\ \                   /\ \         /\__  _\  /\  _ `\         #
   #    \ \ \_\ \     __      ____ \ \ \___     \/_/\ \/  \ \ \/\ \        #
   #     \ \  _  \  /'__`\   / ,__\ \ \  _ `\      \ \ \   \ \ \ \ \       #
   #      \ \ \ \ \/\ \_\ \_/\__, `\ \ \ \ \ \      \_\ \__ \ \ \_\ \      #
   #       \ \_\ \_\ \___ \_\/\____/  \ \_\ \_\     /\_____\ \ \____/      #
   #        \/_/\/_/\/__/\/_/\/___/    \/_/\/_/     \/_____/  \/___/  v1.2 #
   #                                                             By Zion3R #
   #                                                    www.Blackploit.com #
   #                                                   Root@Blackploit.com #
   #########################################################################
--------------------------------------------------

Possible Hashs:
[+] MD5
[+] Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))
```
> Nos da 2 posibilidades pero la que es más común es el MD5.

Si vamos probando por páginas de crack de hashes puede que nos lleve un rato por eso recomiendo que la búsqueda del hash sea directamente en el buscador. Así nos devolvera páginas de crackear hashes que ya la tengan registrada en la base de datos.

![duckduck](duckduck.png)

> Segunda flag: **flag{1m_s3c0nd_fl4g}**
{: .prompt-tip }

### Tercera flag

Para esta será algo tan sencillo como hacer *CTRL+F* en el código fuente del index del apache, y buscar por 'flag'. Directamente aparecerá en texto plano la flag para introducirla como tercera flag.

![flag3](flag3.png) 

> Tercera flag: **flag{9fdafbd64c47471a8f54cd3fc64cd312}**
{: .prompt-tip }

### Cuarta "flag"

En este caso seguiremos buscando en el código fuente y como para el anterior buscaremos, pero esta vez por 'hidden', con esto encontraremos otro párrafo oculto, contiene lo siguiente:

`its encoded with ba....:ObsJmP173N2X6dOrAgEAL0Vu`

Es otro texto en base, ahora es base62. 

Para su desencriptación usaremos [Cyberchef](https://cyberchef.io) la cuál es una muy buena página para temas de criptografía como este.

![cyberchef](cyberchef.png) 

> Cuarta "flag": **/n0th1ng3ls3m4tt3r**
{: .prompt-tip }

### Quinta flag

Entrando al directorio encontrado en la cuarta "flag", podemos ver lo siguiente:

![nothing](nothing.png) 

Si le echamos un vistazo al *source code*, encontramos otro hash y una imagen.

```html
<center>
<img src="binarycodepixabay.jpg" width="140px" height="140px"/>
<p>940d71e8655ac41efb5f8ab850668505b86dd64186a66e57d1483e7f5fe6fd81</p>
</center>
```

Descargaremos la imagen y copiaremos el hash a un archivo para poder tratar de una mejor forma. La imagen la dejaremos para luego, ahora nos pondremos con el hash.

Introducimos el valor del hash en un archivo.

`$ echo "940d71e8655ac41efb5f8ab850668505b86dd64186a66e57d1483e7f5fe6fd81" > hash.txt`

> En TryHackMe tendremos que descargar un diccionario que nos proporcionan
{: .prompt-info }

Usaremos `john` para crackear este hash, el formato será GOST ya que no sabemos realmente con qué tratamos.

El comando y la salida de este es:

```console
$ john --wordlist=easypeasy.txt --format=gost hash.txt
Using default input encoding: UTF-8
Loaded 1 password hash (gost, GOST R 34.11-94 [64/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
mypasswordforthatjob (?)     
1g 0:00:00:00 DONE (2022-12-15 08:52) 12.50g/s 51200p/s 51200c/s 51200C/s vgazoom4x..flash88
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

> Y aquí tendremos la quinta "flag": mypasswordforthatjob
{: .prompt-tip}

### Sexta flag

Antes descargamos la imagen que había en el directorio oculto del Apache, ahora usaremos `steghide` porque lo más probable es que la anterior contraseña que hemos extraído del hash sea la *passphrase* de datos ocultos dentro de la imagen utilizado técnicas de esteganografía.

```console
$ steghide extract -sf flag.jpeg -p mypasswordforthatjob
wrote extracted data to "secrettext.txt".
```

Vemos como se ha extraído datos ocultos, este contiene lo siguiente:

```shell
username:boring
password:
01101001 01100011 01101111 01101110 01110110 01100101 01110010 01110100 01100101 01100100 01101101 01111001 01110000 01100001 01110011 01110011 01110111 01101111 01110010 01100100 01110100 01101111 01100010 01101001 01101110 01100001 01110010 01111001
```
{: file="secrettext.txt" }

Pasaremos el binario a texto en mi caso he usado [binarytotext.net](https://binarytotext.net/).

![binary](binary.png) 

> Sexta "flag": **iconvertedmypasswordtobinary**
{: .prompt-tip}

### Séptima flag

Tendremos que iniciar ssh con las credenciales que tenemos, y revisar en la carpeta personal del usuario por la flag.

![ssh](ssh.png)

Vemos que viene una flag pero está rara, esta cifrada con Caesar con una rotación de 13 posiciones, también llamado ROT13. Si usamos [dcode.fr](https://www.dcode.fr/rot-13-cipher) para desencriptarlo veremos que nos devuelve la flag.

![rot](rot.png) 

> Séptima flag: **flag{n0wits33msn0rm4l}**
{: .prompt-tip}

Con esto habríamos completado todas las respuestas de la máquina excepto la última que se dejará para más adelante.

### Privilege Escalation

> Próximamente...

## Conclusion

Una máquina de iniciación que toca varias ramas al estilo CTF. 

### Techniques and Tools

1. nmap
2. ffuf
3. steghide
4. john
5. RCE (Remote Code Execution)
6. PE (Privilege Escalation)

