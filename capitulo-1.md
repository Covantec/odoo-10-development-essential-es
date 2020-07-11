# Odoo 10 Development Essentials


## Capítulo 1. Iniciando con desarrollo Odoo

Antes de sumergirse en el desarrollo de Odoo, es necesario configurar el
entorno de desarrollo, y para esto se debe aprender las tareas básicas
de administración.

Odoo está desarrollado usando el lenguaje de programación Python y usa
PostgreSQL como base de datos para almacenamiento de datos; estos son los
requisitos principales para trabajar con Odoo. Para ejecutar Odoo desde
el código fuente, necesitará primero instalar las librerías Python de las
cuales depende. Luego el código fuente de Odoo debe descargarse desde
GitHub y ejecutado desde el código fuente. Aunque es posible descargar
un ZIP o tarball, es mejor obtener el código fuente usando el control de
versiones de la aplicación desde GitHub, así además tendría Odoo instalado
en su equipo.


## Creando un huésped para el servidor Odoo

Se recomienda usar sistemas *Debian*/*Ubuntu* para el servidor Odoo, aunque
puede trabajar desde el sistema operativo de su preferencia, sea
*Windows*, *Mac*, o *Linux*.

Odoo puede ser ejecutado en una gran variedad de sistemas operativos,
entonces, ¿por qué elegir *Debian* por encima de otros sistemas operativos?
Porque *Debian* es considerado la plataforma de despliegue de referencia por
el equipo de desarrollo de Odoo; el soporte para Odoo es mejor. Por lo tanto
será más fácil encontrar ayuda y recursos adicionales si se trabaja con
*Debian*/*Ubuntu*.

También es la plataforma con la que la mayoría de desarrolladores trabajan y donde
se implementan más despliegues. Así que inevitablemente, se espera que los
desarrolladores de Odoo se sientan cómodos con la plataforma *Debian*/*Ubuntu*. Incluso
si tienes un pasado *Windows*, será importante que tenga algún conocimiento acerca de
esto.

En este capítulo, aprenderás cómo armar y trabajar con Odoo hospedado en un sistema
*Debian*, usando sólo la linea de comando. Para aquellos en casa con un sistema *Windows*,
se cubrirá cómo configurar una máquina virtual para hospedar el servidor Odoo. Como
ventaja, las técnicas que aprenderás aquí también le permitirán administrar Odoo en
servidores en la nube, donde su único acceso será a través de Secure Shel (SSH).

### Nota

Mantén en mente que estas instrucciones están destinadas a organizar un nuevo sistemas
para desarrollo. Si desea probar algunos de ellos en un sistema existente, siempre toma
una copia de seguridad antes de tiempo para poder restaurarlo en caso de que algo salga
mal.


## Provisión para un huésped Debian

Como se explicó antes, necesitará un huésped basado en *Debian* para su servidor
Odoo. Si estos son sus primeros pasos con Linux, puede gustarte notar que *Ubuntu* es una
distribución Linux basada en *Debian*, así que son muy similares.

Está garantizado que Odoo trabaje con la versión actual estable de *Debian* o *Ubuntu*.
Al tiempo de escribir, estos son *Debian* 8 "Jessie" y *Ubuntu* 16.04.1 LTS (Xenial Xerus).
Ambos, vienen con Python 2.7, el cual es necesario para correr Odoo. Es importante señalar
que Odoo no soporta Python 3 aún, así que Python 2 es requerido.

Si ya está ejecutando *Ubuntu* o alguna otra distribución con basada en *Debian*, ¡estas
listo!; Esto también puede ser utilizado como un huésped para Odoo.

Para los sistemas operativos Windows y Mac, instala Python, PostgreSQL, y todas las
dependencias; luego, ejecuta Odoo desde la fuente nativa. Sin embargo, esto puede ser un reto,
así que su consejo es que uses una máquina virtual que ejecute un servidor *Debian* o
*Ubuntu*. Eres bienvenido a utilizar su software de virtualización preferido para obtener
un sistema de trabajo *Debian* en una máquina virtual.

En caso de que necesites alguna guía, aquí hay unos consejos con respecto al software de
virtualización. Existen varias opciones, tales como *Microsoft Hyper-V* (disponible en algunas
versiones de *Windows* recientes), *Oracle VirtualBox* y *VMWare Workstation Player* (*VMWare
Fusion* para Mac). La *VMWare Workstation Player* es probablemente más fácil de utilizar
y descargas fáciles de usar pueden ser halladas en https://my.vmware.com/wev/vmware/downloads.

Con respecto a la imagen de Linux a utilizar, será más amigable para el usuario instalar
*Ubuntu Server* que *Debian*. Si está empezando con Linux, le recomiendo que pruebes una
imagen lista para usar. *TurnKey Linux* proporciona imágenes previamente instaladas fáciles
de usar en varios formatos, incluyendo ISO. El formato ISO funcionará con cualquier programa
de virtualización que elijas, incluso en una máquina de metal desnudo que pueda tener. Una
buena opción podría ser la imagen LAPP, que incluye Python y PostgreSQL, y se puede encontrar
en http://www.turnkeylinux.org/lapp.

Una vez instalada y arrancada, deberá ser capaz de iniciar sesión en una línea de comando
shell.


## Creando una cuenta de usuario para Oddo

Si ha iniciado sesión usando la cuenta de súper usuario `root`, su primera tarea debe ser
crear una cuenta de usuario normal para su trabajo, ya que se considera una mala práctica
trabajar como `root`. En particular, el operador de Odoo se rehusará a correr si lo inicias
como `root`.

Si ha iniciado sesión usando *Ubuntu*, probablemente no necesitará esto, ya que el proceso
de instalación debe haberte guiado para la creación de un usuario.

Primero, asegúrese de que `sudo` esté instalado. Su usuario de trabajo lo necesitará.
Si se inició sesión como `root`, ejecute los siguientes comandos:

Instalar las actualizaciones del sistema, ejecutando el siguiente comando:

```
# apt-get update && apt-get upgrade
```

Asegúrese que tiene instalado el comando `sudo`, ejecutando el siguiente comando:

```
# apt-get install sudo
```

Crear un usuario `odoo` con los poderes *sudo*, ejecutando el siguiente comando:

```
# useradd -m -g sudo -s /bin/bash odoo
```

Defina una contraseña para el nuevo usuario `odoo`, ejecutando el siguiente comando:

```
# passwd odoo
```

Puedes cambiar el nombre de usuario  `odoo` al que usted quiere. La opción `-m` asegura
que su directorio de inicio sea creado. La opción `-g sudo`.

Ahora puede iniciar sesión como el nuevo usuario y configurar Odoo.


## Instalando Odoo desde la fuente

Los paquetes de instalación rápida de Odoo, pueden encontrarse en la dirección
[nigthly.odoo.com](https://nigthly.odoo.com/), disponible como *Windows* `(.exe)`,
*Debian* (`.deb`), *CentOS* (`.rpm`), y código fuente en formato tarballs (`. tar .gz`).

Como desarrollador, preferirá instalarlos directamente del repositorio GitHub. Esto
terminará dándole más control sobre versiones y actualizaciones.

Para mantener las cosas ordenadas, va a trabajar en un directorio `/odoo-dev`  dentro
de su directorio `home`.

### Nota

A lo largo del libro, asumirá que `/odoo-dev` es el directorio donde su servidor de
Odoo está instalado.

Primero, asegúrate de haber iniciado sesión como el usuario creado ahora o durante el
proceso de instalación, no como el usuario `root`. Asumiendo que su usuario es `odoo`,
confírmelo con el siguiente comando:

Mostrar el valor de la variable de entorno de usuario `$HOME`, ejecutando el siguiente
comando:

```
$ echo $HOME
/home/odoo
```

Mostrar el valor de la variable de entorno de usuario `$USER`, ejecutando el siguiente
comando:

```
$ echo $USER
odoo
```

Ejecutar el comando `whoami` para mostrar el nombre del usuario creado, ejecutando el
siguiente comando:

```
$ whoami
odoo
```

Ahora puede utilizar este script. Le muestra cómo instalar Odoo desde la fuente a un
sistema *Debian*/*Ubuntu*.

Primero, instale las dependencias básicas para comenzar, para eso ejecute los siguientes
comandos.

Instalar las actualizaciones del sistema, ejecutando el siguiente comando:

```
$ sudo apt-get update && sudo apt-get upgrade
```

Instalar el paquete `git`, ejecutando el siguiente comando:

```
$ sudo apt-get install git
```

Instalar el paquete `nodejs` y su administrador de paquete `npm`, ejecutando el siguiente
comando:

```
$ sudo apt-get install npm
```

Crear enlace simbólico `node` al ejecutar `nodejs`, ejecutando el siguiente comando:

```
$ sudo ln -s /usr/bin/nodejs /usr/bin/node
```

Instalar el compilador `less`, ejecutando el siguiente comando:

```
$ sudo npm install -g less less-plugin-clean-css
```

Partiendo de la versión 9.0, el cliente web de Odoo requiere que el preprocesador `less` CSS
esté instalado en el sistema para que las páginas web puedan ser renderizadas correctamente.
Para instalar esto, necesita *Node.js* y *npm*.

Luego, necesita obtener el código fuente Odoo e instalar sus dependencias. El código fuente
Odoo incluye un script de utilidades, dentro del directorio `odoo/setup/`, para ayudarle a
instalar las dependencias requeridas en un sistema *Debian*/*Ubuntu*:

Debe crear el directorio de trabajo, ejecutando el siguiente comando:

```
$ mkdir ~/odoo-dev
```

Valla dentro del directorio de trabajo, ejecutando el siguiente comando:

```
$ cd ~/odoo-dev
```

Obtenga una copia del código fuente de Odoo, ejecutando el siguiente comando:

```
$ git clone https://github.com/odoo/odoo.git -b 10.0 --depth=1
```

Instalar dependencias del sistema Odoo, ejecutando el siguiente comando:

```
$ ./odoo/setup/setup_dev.py setup_deps
```

Instalar PostgreSQL y súper usuario DB para el usuario unix, ejecutando el siguiente comando:

```
$ ./odoo/setup/setup_dev.py setup_pg
```

Al final, Odoo debería estar listo para utilizarse. El símbolo ~ es n atajo para su directorio `home`
(por ejemplo,  `/home/odoo`).
La opción `git -b 10.0` indica a Git que descargue específicamente la rama 10.0 de Odoo. Al tiempo de la
escritura, esto es redundante ya que 10.0 es la rama por defecto; sin embargo, esto puede cambiar, entonces,
puede hacer el script a prueba del futuro. La opción `--depth=1` indica a Git que descargue sólo la última
revisión, en vez del último historial de cambio completo, haciendo la descarga más pequeña y más veloz.

Para iniciar un servidor Odoo, solo ejecuta:

```
$ ~/odoo-dev/odoo/odoo-bin
```

### Consejo

En Odoo 10, el script `odoo.py`, utilizado en versiones previas para iniciar el servidor, fue reemplazado con
`odoo-bin`.

De forma predeterminada, las instancias Odoo escuchan en el puerto 8069, por lo que si apunta un navegador
a `http://<dirección-servidor>:8069`, llegará a estas instancias. Cuando lo acceda por primera vez, nos
muestra un asistente para crear una nueva base de datos, como se muestra en la siguiente captura de pantalla:

![Database](img/1-01.jpg)

Como desarrolladores, necesitará trabajar con varias bases de datos, así que es más convenientes más
conveniente crearlos desde la línea de comandos, así que aprenderá cómo hacerlo. Ahora presione **Ctrl + C**
en el terminal para detener el servidor Odoo y volver al prompt de comando.


## Inicializando una nueva base de datos Odoo

Para ser capaces de crear una nueva base de datos, su usuario debe ser un súper usuario PostgreSQL. El siguiente
comando crea un súper usuario PostgreSQL para el usuario actual Unix.

```
$ sudo createuser --superuser $(whoami)
```
Para crear una nueva base de datos, use el comando `createdb`. Cree una base de datos `demo`:

```
$ createdb demo
```

Para inicializar ésta base de datos con el esquema de datos Odoo, debe ejecutar Odoo en la base de datos vacía, usando
la opción `-d`:

```
$ ~/odoo-dev/odoo/odoo-bin -d demo
```

Esto tomará un par de minutos para inicializar una base de datos `demo`, y terminará con un mensaje de registro INFO,
**Módulos cargados**.

### Nota

Ten en cuenta que puede no ser el último mensaje de registro, y puede estar en las últimas tres o cuatro líneas.
Con esto, el servidor estará listo para escuchar las peticiones del cliente.

De forma predeterminada, esto inicializará la base de datos con datos de demostración, que a menudo es útil para
las base de datos de desarrollo. Para inicializar una base de datos sin datos de demostración, agregue la opción
`--without-demo-data=all` al comando.

Ahora abre `http: // <server-name>: 8069` con su navegador para que se presente la pantalla de inicio de sesión.
Si no conoces el nombre del servidor, escribe el comando `hostname` en el terminal para encontrarlo o el comando
`ifconfig` para encontrar la dirección IP.

Si está hospedando Odoo en una máquina virtual, es posible que debe establecer algunas configuraciones de red
para poder acceder desde su sistema huésped. La solución más simple es cambiar el tipo de red de la máquina
virtual de NAT a Bridged. Con esto, en lugar de compartir la dirección IP del huésped, la máquina virtual invitada
tendrá su propia dirección IP. También es posible utilizar NAT, pero eso requiere que configures el reenvío de
puertos para que su sistema sepa que algunos puertos, como `8069`, deben ser manejados por la máquina virtual. En
caso de que tenga problemas, espera que estos detalles le ayuden a encontrar información relevante en la
documentación del software de virtualización elegido.

La cuenta de administrador predeterminada es `admin` con su contraseña `admin`. Al iniciar sesión, se le presenta
el menú  **Apps**, que muestra las aplicaciones disponibles:

![Database](img/1-02.jpg)

Siempre que desee detener la instancia del servidor Odoo y volver a la línea de comandos, presione **Ctrl + C** en
el indicador de bash. Al presionar la tecla de flecha hacia arriba le llevará el comando de shell anterior, por lo
que es una forma rápida de iniciar Odoo de nuevo con las mismas opciones. Las teclas ***Ctrl + C*** seguido por la
tecla de flecha hacia arriba y ***Enter*** son una combinación utilizada con frecuencia para reiniciar el servidor
Odoo durante el desarrollo.


## Administrar sus bases de datos

Usted ha visto cómo crear e inicializar nuevas bases de datos Odoo desde la línea de comandos. Hay más comandos que
vale la pena saber para administrar las bases de datos.

Ya sabe cómo usar el comando `createdb` para crear bases de datos vacías, pero también puede crear una nueva
base de datos copiando una existente, usando la opción  `--template`.

Asegúrate de que su instancia de Odoo está detenida y no tiene ninguna otra conexión abierta en la base de datos
`demo` que acaba de crear y, a continuación, ejecute esto:

```
$ createdb --template=demo demo-test
```

De hecho, cada vez que creo una base de datos, se utiliza una plantilla. Si no se especifica ninguna, se utiliza
una predeterminada llamada `template1`.

Para listar las bases de datos existentes en su sistema, utiliza la utilidad `psql` de PostgreSQL con la opción
`-l`:

```
$ psql -l
```

Al ejecutarlo se listarán las dos bases de datos que ha creado hasta ahora: `demo` y `demo-test`. La lista
también mostrará la codificación utilizada en cada base de datos. El valor predeterminado es UTF-8, que es la
codificación necesaria para las bases de datos Odoo.

Para eliminar una base de datos que ya no necesita (o quieres crear nuevamente) para utilizar el comando `dropdb`:

```
$ dropdb demo-test
```

Ahora ya sabes lo básico para trabajar con bases de datos. Para obtener más información sobre PostgreSQL,
consulta la documentación oficial en http://www.postgresql.org/docs/.

### Nota

**ADVERTENCIA:**

El comando `drop` de la base de datos  destruirá irrevocablemente sus datos. Ten cuidado al usarlo y mantén siempre
copias de seguridad de bases de datos importantes antes de usar este comando.


## Una palabra sobre las versiones de productos Odoo

Al momento de la redacción de este texto, la última versión estable de Odoo es la versión 10, marcada en GitHub como
rama 10.0. Esta es la versión con la que trabajará a lo largo del libro.

### Nota

Es importante notar que las bases de datos de Odoo son incompatibles entre las versiones principales de Odoo. Esto
significa que si ejecuta un servidor Odoo 10 contra una base de datos creada para una versión principal anterior
de Odoo, no funcionará.

El trabajo de migración no trivial es necesario antes de que una base de datos pueda ser usada con una versión más
reciente del producto.

Lo mismo ocurre con los módulos addon: como regla general, un módulo addon desarrollado para una versión mayor Odoo
no funcionará con otras versiones. Cuando descargue un módulo de la comunidad desde la Web, asegúrese de que esté
orientado a la versión Odoo que está utilizando.

Por otra parte, se espera que las versiones principales (9.0, 10.0) reciban actualizaciones frecuentes, pero éstas
deben ser en su mayoría correcciones de errores.
Se asegura que son "API estable", lo que significa que las estructuras de datos del modelo y los identificadores de
elementos de vista se mantendrán estables. Esto es importante porque significa que no habrá ningún riesgo de ruptura
de módulos personalizados debido a cambios incompatibles en los módulos de núcleo ascendentes.

Tenga en cuenta que la versión en la rama `master` resultará en la siguiente versión estable principal, pero hasta
entonces, no es "API estable" y no debes utilizarla para crear módulos personalizados. Hacerlo es como moverse en
arena movediza: no puedes estar seguro de cuándo se introducirán algunos cambios que romperán su módulo personalizado.

## Más opciones de configuración del servidor

El servidor Odoo soporta bastantes otras opciones. Podrá comprobar todas las opciones disponibles con más opciones de configuración del servidor con el parámetro `--help`:

```
$ ./odoo-bin --help
```

Revisará algunas de las opciones más importantes en las siguientes secciones. Usted comenzará por ver cómo se pueden guardar las opciones actualmente activas en un archivo de configuración.

### Archivos de configuración del servidor Odoo

La mayoría de las opciones se pueden guardar en un archivo de configuración. De forma predeterminada, Odoo utilizará el archivo `.odoorc` en su directorio personal. En sistemas Linux su ubicación predeterminada está en el directorio de inicio (`$ HOME`) y en la distribución de *Windows* está en el mismo directorio que el ejecutable utilizado para iniciar Odoo.

## Nota

En versiones anteriores de Odoo/OpenERP, el nombre del archivo de configuración predeterminado era `.openerp-serverrc`. Para compatibilidad con versiones anteriores, Odoo 10 seguirá utilizando esto si está presente y no se encuentra ningún archivo `.odoorc`.

En una instalación limpia, el archivo de configuración `.odoorc` no se crea automáticamente. Debe usar la opción `--save` para crear el archivo de configuración predeterminado, si aún no existe, y almacenar la configuración actual de la instancia en el:

Servir configuración al archivo

```
$ ~/odoo-dev/odoo/odoo-bin --save --stop-after-init
```

Aquí, también uso la opción `--stop-after-init` para detener el servidor después de que termine sus acciones. Esta opción se utiliza con frecuencia cuando se ejecutan pruebas o se solicita ejecutar una actualización de módulo para comprobar si está instalada correctamente.

Ahora puede inspeccionar lo que se guardó en este archivo de configuración predeterminado:

show the configuration file

```
$ more ~/.odoorc
```

Esto mostrará todas las opciones de configuración disponibles con sus valores predeterminados. Su edición será efectiva la próxima vez que inicie una instancia de Odoo. Escriba `q` para salir y volver al prompt.

También puede optar por usar un archivo de configuración específico, usando la opción `--conf=<filepath>`. Los archivos de configuración no necesitan tener todas las opciones que acabas de ver. Sólo los que realmente cambian un valor por defecto deben estar allí.

## Cambiando el puerto de escucha

La opción de comando `--xmlrpc-port=<port>` les permite cambiar el puerto de escucha de una instancia de servidor desde el predeterminado 8069. Esto se puede usar para ejecutar más de una instancia al mismo tiempo, en la misma máquina.

Va a probar esto. Abre dos ventanas de terminal. En el primero, ejecute esto:

```
$ ~/odoo-dev/odoo/odoo-bin --xmlrpc-port=8070
```

Ejecuta el siguiente comando en el segundo terminal:

```
$ ~/odoo-dev/odoo/odoo-bin --xmlrpc-port=8071
```

Ahí lo tienes: dos instancias Odoo en el mismo servidor de escucha en diferentes puertos! Las dos instancias pueden utilizar bases de datos iguales o diferentes, dependiendo de los parámetros de configuración utilizados. Y los dos podrían estar ejecutando las mismas o diferentes versiones de Odoo.

### La opción filtro de la base de datos

Cuando se desarrolla con Odoo, es frecuente trabajar con varias bases de datos, ya veces incluso con diferentes versiones de Odoo. Detener e iniciar diferentes instancias de servidor en el mismo puerto y cambiar entre distintas bases de datos puede provocar que las sesiones de cliente web se comporten de forma incorrecta.

El acceso a su instancia utilizando una ventana del navegador que se ejecuta en modo privado puede ayudar a evitar algunos de estos problemas.

Otra buena práctica es habilitar un filtro de base de datos en la instancia del servidor para asegurarse de que sólo permite las solicitudes de la base de datos con la que querrá trabajar, ignorando todos las demás. Esto se hace con la opción `--db-filter`. Acepta una expresión regular que se utiliza como filtro para los nombres de base de datos válidos. Para que coincida con un nombre exacto, la expresión debe comenzar con un `^` y terminar con `$`.

Por ejemplo, para permitir sólo la base de datos `demo` use este comando:

```
$ ~/odoo-dev/odoo/odoo-bin --db-filter=^demo$
```


### Administrar los mensajes de registro del servidor

La opción `--log-level` les permite establecer la verbosidad del registro. Esto puede ser muy útil para entender lo que está sucediendo en el servidor. Por ejemplo, para habilitar el nivel de registro de depuración, use la opción `--log-level=debug`.

Los siguientes niveles de registro pueden ser particularmente interesantes:

+  `debug_sql` para inspeccionar consultas SQL generadas por el servidor.

+  `debug_rpc` para detallar las peticiones recibidas por el servidor.

+  `debug_rpc_answer` para detallar las respuestas enviadas por el servidor.

De forma predeterminada, la salida del registro se dirige a la salida estándar (la pantalla de la consola), pero se puede dirigir a un archivo de registro con la opción `--logfile=<filepath>`.

Finalmente, la opción `--dev=all` mostrará el depurador de Python (`pdb`) cuando se genera una excepción. Es útil hacer un análisis _post-mortem_ de un error de servidor. Ten en cuenta que no tiene ningún efecto en la verbosidad del registrador. Puedes encontrar más detalles sobre los comandos del depurador de Python en https://docs.python.org/2/library/pdb.html#debugger-commands.


### Desarrollando desde su estación de trabajo

Puedes estar ejecutando Odoo con un sistema *Debian*/*Ubuntu* en una máquina virtual local o en un servidor a través de la red. Pero puede que prefiera hacer el trabajo de desarrollo en su estación de trabajo personal, utilizando su editor de texto favorito o IDE. Este suele ser el caso de los desarrolladores que trabajan desde estaciones de trabajo *Windows*. Pero también puede ser el caso de los usuarios de Linux que necesitan trabajar en un servidor Odoo a través de la red local.

Una solución para esto es para permitir el uso compartido de archivos en el huésped Odoo para que los archivos sean fáciles de editar desde su estación de trabajo. Para las operaciones del servidor Odoo, como un reinicio del servidor, puede usar un shell SSH (como PuTTY en *Windows*) junto con su editor favorito.


#### Usando un editor de texto Linux

Tarde o temprano, necesitará editar archivos desde la línea de comandos del shell. En muchos sistemas *Debian*, el editor de texto predeterminado es vi. Si no se sientes cómodo con él, probablemente podrías usar una alternativa más amigable. En los sistemas *Ubuntu*, el editor de texto predeterminado es `nano`. Es posible que prefiera este, ya que es más fácil de usar. En caso de que no esté disponible en su servidor, se puede instalar con:

```
$ sudo apt-get install nano
```

En las siguientes secciones, asumirá `nano` como el editor preferido. Si prefieres cualquier otro editor, siéntete libre de adaptar los comandos en consecuencia.


## Instalando y configurando Samba

El servicio Samba ayuda a que los servicios de compartición de archivos de Linux sean compatibles con los sistemas Microsoft *Windows*. Podrá instalarlo en su servidor *Debian*/*Ubuntu* con este comando:


```
$ sudo apt-get install samba samba-common-bin
```

El paquete `samba` instala los servicios de intercambio de archivos y el paquete `samba-common-bin` es necesario para la herramienta `smbpasswd`. De forma predeterminada, los usuarios autorizados a acceder a archivos compartidos deben registrarse con él. Necesita registrar a su usuario, `odoo` por ejemplo, y establecer una contraseña para su acceso a compartir archivos:

```
$ sudo smbpasswd -a odoo
```

Después de esto, se le pedirá una contraseña para usar para acceder al directorio compartido, y el usuario `odoo` podrá acceder a los archivos compartidos para su directorio personal, aunque será de sólo lectura. Querrá tener acceso de escritura, por lo que necesita editar el archivo de configuración de Samba para cambiarlo de la siguiente manera:

```
$ sudo nano /etc/samba/smb.conf
```

En el archivo de configuración, busque la sección `[homes]`. Edita sus líneas de configuración para que coincidan con la configuración de la siguiente manera:

```
[homes]
    comment = Home Directories
    browseable = yes
    read only = no
    create mask = 0640
    directory mask = 0750
```

Para que los cambios de configuración tengan efecto, reinicia el servicio:

```
$ sudo /etc/init.d/smbd restart
```

#### Consejo


##### Descargando el código ejemplo

Puedes descargar los archivos de códigos de ejemplo para todos los libros Packt que hayas comprado desde su cuenta en http://www.packtpub.com. Si compraste este libro en algún otro sitio, puedes entrar a http://www.packtpub.com/support y registrate para que le envíen los archivos directamente por correo electrónico.

Para acceder a los archivos desde *Windows*, puede asignar una unidad de red para la ruta `\\<my-server-name>\odoo`
utilizando el nombre de usuario y la contraseña específicos definidos con `smbpasswd` Al intentar iniciar sesión con el usuario `odoo`, podría encontrar problemas con *Windows* agregando el dominio del equipo al nombre de usuario (por ejemplo, `MYPC\odoo`). Para evitar esto, utilice un dominio vacío añadiendo un carácter `\` al inicio de sesión (por ejemplo, `\odoo`):

![Directory](img/1-03.jpg)


Si ahora abre la unidad asignada con el Explorador de *Windows*, podrá acceder y editar el contenido del directorio home del usuario `odoo`:

![Home](img/1-04.jpg)

Odoo incluye un par de herramientas que son muy útiles para los desarrolladores, y va a hacer uso de ellas a lo largo del libro. Son características técnicas y el modo de desarrollo. Estos están desactivados por defecto, por lo que este es un buen momento para aprender a habilitarlos.


### Activación de las herramientas de desarrollo
Las herramientas de desarrollo proporcionan una configuración y funciones avanzadas de servidor. Estos incluyen un menú de depuración en la barra de menú superior junto con opciones de menú adicionales en el menú **Settings**, en particular el menú **Technical**.

Estas herramientas vienen deshabilitadas de forma predeterminada y, para habilitarlas, debe iniciar sesión como administrador. En la barra de menú superior, seleccione el menú **Settings**. En la parte inferior derecha, debajo de la versión Odoo, encontrará dos opciones para habilitar el modo de desarrollo; cualquiera de ellas habilitará los menús **Debug** y **Technical**. La segunda opción, **Activate the developer mode (whit assest)**, también deshabilita la minificación de JavaScript y CSS utilizada por el cliente web, facilitando la depuración del comportamiento del cliente:

![Depuracion](img/1-05.jpg)

Después de eso, la página se vuelve a cargar y debería verse un icono de error en la barra de menú superior, justo antes del avatar y nombre de usuario de la sesión que proporciona las opciones de modo de depuración. Y en la opción **Settings** en el menú superior, deberá ver una nueva sección del menú **Technical** que da acceso a muchos internos de la instancia de Odoo:

![Cliente](img/1-06.jpg)


#### Consejo

La opción **Technical** del menú les permite inspeccionar y editar todas las configuraciones Odoo almacenadas en la base de datos, desde la interfaz de usuario hasta la seguridad y otros parámetros del sistema. Aprenderás más sobre muchos de estos a lo largo del libro.


## Instalación de módulos de terceros

Hacer nuevos módulos disponibles en una instancia Odoo para que puedan instalarse es algo que los recién llegados a Odoo suelen encontrar confuso. Pero no tiene que ser así, así que va a desmitificarlo.


## Encontrar módulos comunitarios

Hay muchos módulos Odoo disponibles en Internet. La tienda de aplicaciones de Odoo en https://apps.odoo.com es un catálogo de módulos que se pueden descargar e instalar en su sistema. La **Odoo Community Association (OCA)** coordina las contribuciones de la comunidad y mantiene bastantes repositorios de módulos en GitHub en https://github.com/OCA/.

Para agregar un módulo a una instalación de Odoo, podrá copiarlo en el directorio `addons` junto con los módulos oficiales. En su caso, el directorio `addons` está en `~/odoo-dev/odoo/addons/`. Esto podría no ser la mejor opción para nosotros, ya que su instalación de Odoo se basa en un repositorio de código controlado por versiones, y querrá mantenerlo sincronizado con el repositorio de GitHub.

Afortunadamente, puede usar ubicaciones adicionales para los módulos para que pueda mantener sus módulos personalizados en un directorio diferente, sin tenerlos mezclados con los oficiales.

Como ejemplo, va a descargar el código de este libro, disponible en GitHub, y hacer disponibles esos módulos addon en su instalación de Odoo.

Para obtener el código fuente de GitHub, ejecute los siguientes comandos:

```
$ cd ~/odoo-dev
```

```
$ git clone https://github.com/dreispt/todo_app.git -b 10.0
```
Usted uso la opción `-b` para asegurarse de que está descargando los módulos para la versión 10.0.

Después de esto, tendrá un directorio nuevo `/todo_app` junto al directorio `/odoo`, que contiene los módulos. Ahora debe informar a Odoo sobre este nuevo directorio de módulos.


### Configurandola ruta addons

El servidor Odoo tiene una opción de configuración llamada `addons_path` para establecer dónde el servidor debe buscar módulos. De forma predeterminada, esto apunta al directorio `/addons`, donde se ejecuta el servidor Odoo.

Podrá proporcionar no sólo una, sino una lista de directorios donde se pueden encontrar módulos. Esto les permite mantener sus módulos personalizados en un directorio diferente, sin tenerlos mezclados con los addons oficiales.

Va a iniciar el servidor con una ruta addons que incluye su nuevo directorio de módulos:

```
$ cd ~/odoo-dev/odoo
```

```
$ ./odoo-bin -d demo --addons-path="../todo_app,./addons"
```

Si miras más de cerca el registro del servidor, notarás una línea que informa de la ruta de complementos en uso: `INFO? Odoo: addons paths: [...]`. Confirma que contiene su directorio `todo_app`.


### Actualizando la lista de aplicaciones

Todavía necesita pedirle a Odoo que actualice su lista de módulos antes de que estos nuevos módulos estén disponibles para la instalación.

Para ello, necesita activar el modo desarrollador, ya que proporciona la opción de menú **Actualizar Lista de Aplicaciones**. Se puede encontrar en el menú superior de **Aplicaciones**.

Después de actualizar la lista de módulos, puede confirmar que los nuevos módulos están disponibles para la instalación. Utilice la opción de menú **Aplicaciones** para ver la lista de módulos locales. Busca `todo` y deberá ver los nuevos módulos disponibles.

Ten en cuenta que la segunda opción de menú **App Store** muestra la lista de módulos del almacén de aplicaciones Odoo en lugar de los módulos locales:

![Apps](img/1-07.jpg)


## Resumen

En este capítulo, aprendió a configurar un sistema *Debian* para alojar Odoo e instalarlo desde el código fuente de GitHub. También aprendió a crear bases de datos Odoo y ejecutar instancias de Odoo. Para permitir a los desarrolladores utilizar sus herramientas favoritas en su estación de trabajo personal, se explicará cómo configurar el uso compartido de archivos en el huésped Odoo.

Ahora deberá tener un ambiente Odoo en funcionamiento para trabajar y estar cómodos con la administración de bases de datos e instancias.

Con esto en su lugar, está listos para ir directamente a la acción. En el próximo capítulo, creará su primer módulo Odoo desde cero y entenderá los principales elementos que involucra.

¡Entonces inicie!
