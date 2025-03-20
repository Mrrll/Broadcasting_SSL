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
-   [Configurar Apache en Xampp](#configurar-apache-en-xampp)
-   [Instalación y configuración laravel reverb](#instalación-y-configuración-laravel-reverb)
-   [Información del certificado](#información-del-certificado)

## Crear certificados

> [!NOTE]
> Hay que tener instalado `Openssl` que suele estar en `C:\xampp\apache\bin` podemos usar una carpeta si la creamos debemos trabajar esta parte dentro de la carpeta.

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
