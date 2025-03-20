# Broadcasting SSL

Proyecto WebSocket con laravel Reverb en un virtual host en xampp por `https`.

> [!NOTE]
> La finalidad de este proyecto es poder enviar un mensaje desde un evento utilizado una conexión websocket desde un virtual host. Nos hemos ido guiando por la documentación de laravel, hemos utilizado: [Broadcasting](https://laravel.com/docs/12.x/broadcasting), [Laravel Reverb](https://laravel.com/docs/12.x/reverb), [Events](https://laravel.com/docs/12.x/events) y [Agregar certificado SSL 🔒 a XAMPP – WINDOWS](https://www.youtube.com/watch?v=Tdv0dedPNRQ).

<a name="top"></a>

## Indice de Contenidos.

-   [Crear certificados](#crear-certificados)
    -   [Crear una root key](#crear-una-root-key)
    -   [Crear un root certificate](#crear-un-root-certificate)
    -   [Pedido de firma de certificado](#pedido-de-firma-de-certificado)
    -   [Certificado SSL](#certificado-ssl)
-   [Información del certificado](#información-del-certificado)
-   [Configurar Apache en Xampp](#configurar-apache-en-xampp)
    -   [Habilitar módulos](#habilitar-módulos)
    -   [Añadir certificados SSL](#añadir-certificados-ssl)
    -   [Virtual Host](#virtual-host)
    -   [Redirigir al host](#redirigir-al-host)
-   [Agregar Certificados SSL](#agregar-certificados-ssl)
-   [Instalación y configuración laravel reverb](#instalación-y-configuración-laravel-reverb)

## Crear certificados

> [!NOTE]
> Hay que tener instalado `Openssl` que suele estar en `C:\xampp\apache\bin` podemos usar una carpeta si la creamos debemos trabajar esta parte dentro de la carpeta en este caso he creado una carpeta :file_folder: llamada `certs` en la raíz del proyecto .

> [!CAUTION]
> El nombre de dominio tiene que ser el que vayamos a usar en este caso he usado `broadcasting.local`.

### Crear una root key

> Typee: en la Consola:
```console
openssl genrsa -des3 -out localRootCA.key 2048
```

> [!IMPORTANT]
> Para este paso deberemos elegir y recordar una passphrase o contraseña.

[Ir al Indice de Contenidos...](#indice-de-contenidos) :top:

### Crear un root certificate

> [!NOTE]
> Durante este paso utilizaremos la passphrase ingresada al generar la root key.

> Typee: en la Consola:
```console
openssl req -x509 -new -nodes -key localRootCA.key -sha256 -days 1024 -out localRootCA.pem
```

> [!IMPORTANT]
> Se pueden contestar las preguntas que aparecen en este paso con un punto o un espacio en blanco, sin embargo cuando nos soliciten el **Common Name** debemos ingresar el nombre del dominio o virtual host en este caso es `broadcasting.local`, si quiere puede contestar las preguntas para el certificado mira en el siguiente enlace [Información del certificado](#información-del-certificado). 

[Ir al Indice de Contenidos...](#indice-de-contenidos) :top:

### Pedido de firma de certificado

> [!NOTE]
> Es necesario emitir un archivo de solicitud de firma de certificado **(broadcasting.local.csr)** y una clave **(broadcasting.local.key)** para poder hacer la comprobación de nuestro certificado. Importante: se pueden contestar las preguntas que aparecen en este paso con un punto o un espacio en blanco, sin embargo cuando nos soliciten el **Common Name** debemos ingresar `broadcasting.local`. Para este paso también deberemos pensar una passphrase.

> [!CAUTION]
> El nombre de los archivos y el nombre de dominio tiene que ser el que vayamos a usar en este caso he usado `broadcasting.local`.

> Typee: en la Consola:
```console
openssl req -new -nodes -out broadcasting.local.csr -newkey rsa:2048 -keyout broadcasting.local.key
```

[Ir al Indice de Contenidos...](#indice-de-contenidos) :top:

### Certificado SSL

> [!NOTE]
> Para generar un certificado SSL, primero debemos generar un archivo de configuración. creamos un archivo y lo guardaremos como `x509v3.ext` y añadiremos el siguiente contenido.

```text
authorityKeyIdentifier = keyid, issuer
basicConstraints = CA: FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = broadcasting.local
```

> [!WARNING]
> Muy importante la parte de `DNS.1 = broadcasting.local`, si tienes otro nombre de dominio no te olvides de cambiarlo.

> [!NOTE]
> Finalmente, ahora podemos crear nuestro certificado SSL para XAMPP (la passphrase que nos pedirá es la que hemos utilizado para crear el `root key`).

> Typee: en la Consola:
```console
openssl x509 -req -in broadcasting.local.csr -CA localRootCA.pem -CAkey localRootCA.key -CAcreateserial -out broadcasting.local.crt -days 500 -sha256 -extfile x509v3.ext
```

> Pues ya tendríamos un certificado para nuestro virtual host en local. 👍

[Ir al Indice de Contenidos...](#indice-de-contenidos) :top:

## Información del certificado

__Country__ Name (2 letter code) [XX]: _Código de país en formato ISO 3166-1 (por ejemplo, "US" para Estados Unidos o "ES" para España)._

__State or Province Name__ (full name) []: _Nombre completo del estado o provincia._

__Locality Name__ (eg, city) []: _Nombre de la ciudad o localidad._

__Organization Name__ (eg, company) [Default Company Ltd]: _Nombre de la organización o empresa._

__Organizational Unit Name__ (eg, section) []: _Nombre de la unidad organizativa dentro de la empresa._

<ins>__Common Name__</ins> (e.g. server FQDN or YOUR name) []: _Nombre común del certificado, normalmente el dominio del sitio web (por ejemplo, example.com)._

__Email Address__ []: _Dirección de correo electrónico del administrador del certificado._

__A challenge password__ []: (Opcional) _Contraseña para proteger el certificado._

__An optional company name__ []: (Opcional) _Nombre alternativo de la empresa._

[Ir al Indice de Contenidos...](#indice-de-contenidos) :top:

## Configurar Apache en Xampp

> [!TIP]
> Lo primero de todo vamos a copiar los archivos de los certificados que acabamos de crear en las carpetas correspondientes de Xampp, Solo para tener ordenados nuestros archivos y no tener que ir cambiado rutas pero podemos de omitir este paso.

> [!NOTE]
> En este caso los archivos de los certificados los tengo en la carpeta `certs` en la raíz del proyecto, cambia las rutas por la tuyas y los nombres de los archivos.

> Typee: en la Consola:

PowerShell

```console
Copy-Item "C:\xampp\htdocs\broadcasting\certs\broadcasting.local.crt" -Destination "C:\xampp\apache\conf\ssl.crt\"
```
```console
Copy-Item "C:\xampp\htdocs\broadcasting\certs\broadcasting.local.key" -Destination "C:\xampp\apache\conf\ssl.key\"
```
(Opcional)
```console
Copy-Item "C:\xampp\htdocs\broadcasting\certs\broadcasting.local.csr" -Destination "C:\xampp\apache\conf\ssl.csr\"
```

CMD (Símbolo del sistema)

```console
copy "C:\xampp\htdocs\broadcasting\certs\broadcasting.local.crt" "C:\xampp\apache\conf\ssl.crt\"
```
```console
copy "C:\xampp\htdocs\broadcasting\certs\broadcasting.local.key" "C:\xampp\apache\conf\ssl.key\"
```
(Opcional)
```console
copy "C:\xampp\htdocs\broadcasting\certs\broadcasting.local.csr" "C:\xampp\apache\conf\ssl.csr\"
```

[Ir al Indice de Contenidos...](#indice-de-contenidos) :top:

> [!TIP]
> Es recomendable abrir los archivos en `Ejecutar como administrador`.

### Habilitar módulos

> [!NOTE]
> Abre el archivo __httpd.conf__ de Apache (ubicado en `C:\xampp\apache\conf\httpd.conf`) y asegúrate de tener descomentadas (sin el “#”) las siguientes líneas:

```text
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
LoadModule proxy_wstunnel_module modules/mod_proxy_wstunnel.so
```

[Ir al Indice de Contenidos...](#indice-de-contenidos) :top:

### Añadir certificados SSL

> [!NOTE]
> Abre el archivo __httpd-ssl.conf__ de Apache (ubicado en `C:\xampp\apache\conf\extra\httpd-ssl.conf`) y buscamos `SSLCertificateFile` veremos que hay varias líneas que ponen `SSLCertificateFile "conf/ssl.crt/server.crt"` hay unas que están comentadas con (“#”) o podemos descomentar y modificarla o podemos añadir debajo:

> [!TIP]
> Podemos tener tantos certificados como necesitemos.

```text
SSLCertificateFile "conf/ssl.crt/broadcasting.local.crt"
```

También buscaremos `SSLCertificateKeyFile` y haremos lo mismo pero con la key.

```text
SSLCertificateKeyFile "conf/ssl.key/broadcasting.local.key"
```

> [!IMPORTANT]
> Recuerda que si has omitido el paso de copiar los archivos a las carpetas correspondientes deberás de cambiar las rutas por las tuyas y el nombre del archivo.

[Ir al Indice de Contenidos...](#indice-de-contenidos) :top:

### Virtual Host

> [!NOTE]
> Abre el archivo __httpd-vhost.conf__ de Apache (ubicado en `C:\xampp\apache\conf\extra\httpd-vhost.conf`) y añadimos lo siguiente:

```text
<VirtualHost *:80>
    DocumentRoot "C:/xampp/htdocs/broadcasting/public"
    ServerName broadcasting.local
    <Directory "C:/xampp/htdocs/broadcasting/public">
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>

<VirtualHost *:443>
    DocumentRoot "C:/xampp/htdocs/broadcasting/public"
    ServerName broadcasting.local
    SSLEngine on
    SSLCertificateFile "C:/xampp/apache/conf/ssl.crt/broadcasting.local.crt"
    SSLCertificateKeyFile "C:/xampp/apache/conf/ssl.key/broadcasting.local.key"
    # Habilita el proxy y la traducción de encabezados
    ProxyPreserveHost On
    SSLProxyEngine On
    # Proxy para WebSockets
    <Location /app>
         ProxyPass "ws://127.0.0.1:6001/app"
         ProxyPassReverse "ws://127.0.0.1:6001/app"
    </Location>
    <Directory "C:/xampp/htdocs/broadcasting/public">
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

> [!IMPORTANT]
> Hay que tener en cuenta el nombre del dominio, la ruta del proyecto, las rutas de los certificados y la ruta del `websocket` y su puerto en este proyecto vamos a utilizar el puerto `6001`.

[Ir al Indice de Contenidos...](#indice-de-contenidos) :top:

### Redirigir al host

> [!NOTE]
> Abre el archivo __hosts__ de Windows (ubicado en `C:\Windows\System32\drivers\etc\hosts`) y añadimos lo siguiente:

```text
127.0.0.1   broadcasting.local
```

[Ir al Indice de Contenidos...](#indice-de-contenidos) :top:

## Agregar Certificados SSL

> [!NOTE]
> Vamos a agregar los certificados al navegador y a windows.

Firefox: Abrimos el navegador y pegamos:

```text
about:preferences#privacy
```

> [!IMPORTANT]
> No dirigimos a la sección `Certificados` y presionamos el botón `Ver certificados` y en la pestaña `Autoridades` le damos al botón `Importar` seleccionamos el archivo `localRootCA.pem` que hemos creado.

Chrome: Abrimos el navegador y pegamos:

```text
chrome://certificate-manager/
```

> [!IMPORTANT]
> Presionamos el botón `Gestionar certificados importados desde Windows` se abrirá una ventana y le damos al botón `Importar` luego a siguiente  seleccionamos el archivo `localRootCA.pem` que hemos creado y a siguiente marcamos la casilla `Colocar todos los certificados en el siguiente almacén` presionamos el botón `Examinar ...` y seleccionamos `Entidades de certificación raíz de confianza` aceptamos y lo típico siguiente siguiente y finalizar.

Windows:

> [!IMPORTANT]
> Le damos doble click al archivo `broadcasting.local.crt` presionamos el botón `Instalar certificado` seleccionamos la casilla `Equipo local` siguiente nos pedirá que confirmemos que haga cambios le damos a si y marcamos la casilla `Colocar todos los certificados en el siguiente almacén` presionamos el botón `Examinar ...` y seleccionamos `Entidades de certificación raíz de confianza` aceptamos y lo típico siguiente siguiente y finalizar.

[Ir al Indice de Contenidos...](#indice-de-contenidos) :top: