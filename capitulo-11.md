# Capítulo 11. Creación de características Frontend de un website

Odoo comenzó como un sistema backend, pero la necesidad de una interfaz de frontend pronto se sintió. Las primeras características del portal, basadas en la misma interfaz que el backend, no eran muy flexibles ni amigables con dispositivos móviles.

Para solucionar este vacío, la versión 8 introdujo nuevas características de sitio web, agregando un Sistema de Gestión de Contenidos (CMS) al producto. Esto les permitiría construir frontends hermosos y eficaces sin la necesidad de integrar un CMS de terceros.

Aquí aprenderá cómo desarrollar sus propios módulos complementarios orientados al usuario, aprovechando la función de sitio web proporcionada por Odoo.

## Mapa de ruta

Usted creará una página web en la que se enumerarán nuestras Tareas pendientes, lo que les permitirá navegar a una página detallada para cada tarea existente. También querrá poder proponer nuevas Tareas pendientes a través de un formulario web.

Con esto podrá cubrir las técnicas esenciales para el desarrollo de sitios web: crear páginas dinámicas, pasar parámetros a otra página, crear formularios y manejar su validación y lógica de cálculo.

Pero primero, se le ofrece una introducción a los conceptos básicos del sitio web con una página web **Hola Mundo** muy sencilla.

## Su primera página web

Va a crear un módulo de complemento para las características de su sitio web. Podrá llamarlo `todo_website`. Para introducir los fundamentos del desarrollo web de Odoo, implementará una sencilla página web **Hola Mundo**. Imaginativo, ¿verdad?

Como de costumbre, comenzará a crear su archivo de manifiesto. Crea el archivo `todo_website/__manifest__.py` con:

```
{
  'name': 'To-Do Website',
  'description': 'To-Do Tasks Website',
  'author': 'Daniel Reis',
  'depends': ['todo_kanban']
}
```

Esta construyendo sobre el módulo `todo_kanban`, de modo que tenga todas las características disponibles añadidas al modelo de Tareas pendientes a lo largo del libro.

Tenga en cuenta que ahora mismo no depende del módulo `website`. Aunque `website` ofrece un marco útil para crear sitios web con todas las funciones, las capacidades básicas de la Web se incorporan  en el núcleo del framework. Va a explorarlos.


## ¡Hola Mundo!

Para proporcionar su primera página web, agregará un objeto controlador. Podrá empezar por tener su archivo importado con el módulo:

Primero agrega un archivo `todo_website/__init__.py` con la siguiente línea:

```
from . import controllers
```

A continuación, agregue un archivo `todo_website/controllers/__init__.py` con la siguiente línea:

```
from . import main
```

Ahora agregue el archivo como tal para el controlador, `todo_website/controllers/main.py`, con el siguiente código:

```
# -*- coding: utf-8 -*-

from odoo import http

class Todo(http.Controller):

    @http.route('/helloworld', auth='public')
    def hello_world(self):
        return('<h1>Hello World!</h1>')
```


El módulo `odoo.http` proporciona las funciones web relacionadas con Odoo. Sus controladores, responsables de la representación de páginas, deben ser objetos heredados de la clase `odoo.http.Controller`. El nombre real utilizado para la clase no es importante; Aquí eligió utilizar `Main`.

Dentro de la clase del controlador tiene métodos, que coinciden rutas, hace algún procesamiento, y luego devuelve un resultado; La página que se mostrará al usuario.

El decorador `odoo.http.route` es lo que une un método a una ruta de URL. Su ejemplo utiliza la ruta `/hello`. Navegue hasta http://localhost:8069/hello y recibirá un mensaje de `Hello World`. En este ejemplo, el procesamiento realizado por el método es bastante trivial: simplemente devuelve una cadena de texto con el marcado HTML para el mensaje _Hello World_.

Probablemente haya notado que ha añadido el argumento `auth='public'` a la ruta. Esto es necesario para que la página esté disponible para usuarios no autenticados. Si lo elimina, sólo los usuarios autenticados pueden ver la página. Si no hay sesión activa, en su lugar se mostrará la pantalla de inicio de sesión.


## Hola Mundo! Con una plantilla Qweb

El uso de cadenas de Python para construir HTML se tornará aburrido muy rápido. Las plantillas QWeb hacen un trabajo mucho mejor en eso. Así que va a mejorar su página web de **Hola Mundo** para usar una plantilla en su lugar.

Las plantillas de QWeb se agregan a través de archivos de datos XML, y técnicamente son un tipo de vista, junto con vistas de formulario o árbol. En realidad se almacenan en el mismo modelo `ir.ui.view`.

Como de costumbre, los archivos de datos que se cargan deben declararse en el archivo de manifiesto, por lo que debes editar el archivo `todo_website/__manifest__.py` para agregar la clave:

```
  'data': ['views/todo_templates.xml'],
```

A continuación, agrega el archivo de data, `views/todo_web.xml`, con el siguiente contenido:

```
<odoo>
  <template id="hello" name="Hello Template">
    <h1>Hello World !</h1>
  </template>
</odoo>
```

### Nota

> El elemento `<template>` es en realidad un acceso directo para declarar un `<record>` para el modelo `ir.ui.view`, usando `type = "qweb"`, y una plantilla `<t>` dentro de él.

Ahora necesita que su método de controlador use esta plantilla:

```
from odoo.http import request

# ...
@http.route('/hello', auth='public')
def hello(self, **kwargs):
    return request.render('todo_website.hello')
```

La representación de la plantilla se proporciona mediante `request`, a través de su función `render()`.

### Consejo

> Observa que agrego `**kwargs` a los argumentos del método. Con esto, si cualquier parámetro adicional proporcionado por la petición HTTP, como una consulta de cadenas de texto o parámetros POST, puede ser capturado por el diccionario `kwargs`. Esto hace que su método sea más robusto, ya que proporcionar parámetros inesperados no causará errores.

## Ampliación de funciones web

Extensibilidad es algo que espera en todas las características de Odoo, y las características de la web no son una excepción. Y de hecho puede extender los controladores y plantillas existentes. Por ejemplo, se extenderá su página web de **Hola Mundo** para que tome un parámetro con el nombre para saludar: usando la `URL/hello?name=John` devolvería un saludo **Hello John!** .

La extensión se hace generalmente desde un módulo distinto, pero funciona de igual manera dentro del mismo módulo. Para mantener las cosas concisas y sencillas, lo hará sin crear un nuevo módulo.

Va añadir un nuevo archivo `todo_website/controllers/extend.py` con el siguiente código:

```
# -*- coding: utf-8 -*-

from odoo import http
from odoo.addons.todo_website.controllers.main import Todo

class TodoExtended(Todo):
    @http.route()
    def hello(self, name=None, **kwargs):
        response = super(TodoExtended, self).hello()
        response.qcontext['name'] = name
        return response
```

Aquí puede ver lo que necesita hacer para extender un controlador.

Primero uso una importación de Python para obtener una referencia a la clase de controlador que querrá extender. Comparados con los modelos, tienen un registro central, proporcionado por el objeto `env`, donde se puede obtener una referencia a cualquier clase de modelo, sin necesidad de conocer el módulo y el archivo que los implementa. Con los controladores no tiene eso, y necesita conocer el módulo y el archivo que implementa el controlador que querrá ampliar.

A continuación tiene que (re)definir el método desde el controlador que se está extendiendo. Tiene que estar decorado con al menos el simple `@http.route()` para que su ruta se mantenga activa. Opcionalmente, puede proporcionar parámetros a `route()`, y luego estará reemplazando y redefiniendo sus rutas.

El método extendido `hello()` ahora tiene un parámetro de nombre. Los parámetros pueden obtener sus valores de segmentos de la URL de la ruta, de los parámetros de consulta de una cadena o de los parámetros POST. En este caso, la ruta no tiene variable extraíble (se demostrará en un momento), y como está manejando peticiones `GET`, no `POST`, el valor del parámetro `name` será extraído de la consulta de cadena en la URL. Una URL de prueba podría ser `http://localhost:8069/hello?name=John`.

Dentro del método `hello()` ejecuta el método heredado para obtener su respuesta, y luego puede modificarlo de acuerdo a nuestras necesidades. El patrón común para los métodos de controlador es que terminen con una instrucción para renderizar una plantilla. En su caso:

```
return request.render('todo_website.hello')
```

Esto genera un objeto `http.Response`, pero la representación real se retrasa hasta el final del despacho.

Esto significa que el método de herencia todavía puede cambiar la plantilla QWeb y el contexto a utilizar para la representación. Usted podría cambiar la plantilla modificando `response.template`, pero no lo necesitará. Preferirá modificar `response.qcontext` para agregar la clave `name` al contexto de renderizado.

No olvides agregar el nuevo archivo Python en `todo_website/controllers/__init__.py:`

```
from . import main
from . import extend
```

Ahora necesita modificar la plantilla `QWeb`, Para que haga uso de esta información adicional. Agrega lo que sigue a `todo/website/views/todo_extend.xml`:

```
<odoo>
  <template id="hello_extended" name="Extended Hello World" inherit_id="todo_website.hello">
    <xpath expr="//h1" position="replace">
      <h1>
        Hello <t t-esc="name or 'Someone'" />!
      </h1>
    </xpath>
  </template>
</odoo>
```

Las plantillas de páginas web son documentos XML, al igual que los otros tipos de vista Odoo, y puede usar `xpath` para localizar elementos y luego manipularlos, tal como podrá con los otros tipos de vista. La plantilla heredada se identifica en el elemento `<template>` por el atributo `inherited_id`.

No debe olvidar declarar este archivo de datos adicional en su manifiesto, `todo_website/__manifest__.py`:

```
  'data': [
    'Views/todo_web.xml',
    'views/todo_extend.xml'
  ],
```

Después de esto, accediendo a `http://localhost:8069/hello?name=John` debe mostrarle un mensaje **Hello John!**.

También puede proporcionar parámetros a través de segmentos de URL. Por ejemplo, podrá obtener exactamente el mismo resultado de la URL `http://localhost:8069/hello/John` usando esta implementación alternativa:

```
class TodoExtended(Todo):

@http.route(['/hello','/hello/<name>])
def hello(self, name=None, **kwargs):
        response = super(TodoExtended, self).hello()
        response.qcontext['name'] = name
        return response
```

Como puedes ver, las rutas pueden contener `placeholders` correspondientes a los parámetros que se van a extraer y luego pasar al método. Los `placeholders` también pueden especificar un convertidor para implementar una asignación de tipo específica. Por ejemplo, `<int:user_id>` extraería el parámetro `user_id` como un valor entero.

Los convertidores son una característica proporcionada por la biblioteca `werkzeug`, utilizada por Odoo, y la mayoría de los disponibles se pueden encontrar en la documentación de la biblioteca `werkzeug`, en [http://werkzeug.pocoo.org/docs/routing/](http://werkzeug.pocoo.org/docs/routing/).

Odoo añade un conversor específico y particularmente útil: extraer un registro de modelo. Por ejemplo `@http.route('/hello/<model("res.users"):user>)` extrae el parámetro `user` como un objeto de registro para el modelo `res.users`.

## ¡HolaCMS!

Va a hacer esto aún más interesante, y crear su propio CMS simple. Para esto puede hacer que la ruta espere un nombre de plantilla (una página) en la URL y luego simplemente renderizarla. Usted podría entonces crear dinámicamente páginas web y hacerlas servir por su CMS.

Resulta que esto es bastante fácil de hacer:

```
@http.route('/hellocms/<page>', auth='public')
def hello (auto, página, ** kwargs):
     return http.request.render(page)
```

Ahora, abre `http://localhost:8069/hellocms/todo_website.hello` en su navegador web y verás su página de **Hello World!**

De hecho, el sitio web incorporado ofrece funciones de CMS, incluyendo una implementación más robusta de lo anterior, en la ruta del endpoint `/page`.

### Nota

En la jerga `werkzeug` el `endpoint` es un alias de la ruta, y representado por su parte estática (sin los `placeholders`). Para su ejemplo simple de CMS, el `endpoint` fue `/hellocms`.

La mayor parte del tiempo querrá que nuestras páginas se integren en el sitio web de Odoo. Así que para el resto de este capítulo en sus ejemplos va a trabajar con el módulo `website`.


## Creando sitios web

Las páginas dadas por los ejemplos anteriores no están integradas con el módulo `website`: no tiene pie de página, menú, etc. El módulo `website` de Odoo proporciona convenientemente todas estas características para que no tenga que preocuparse por ellas.

Para usarlo, debe comenzar instalando el módulo `website` en su instancia de trabajo y luego agregarlo como una dependencia a su módulo. La clave `depends` en __manifest__.py debe tener el siguiente aspecto:

```
  'depends': ['todo_kanban', 'website'],
```

Para utilizar el sitio web, también debe modificar el controlador y la plantilla.

El controlador necesita un argumento adicional `website=True` en la ruta:

```
@http.route('/hello', auth='public', website=True)
def hello(self, **kwargs):
    return request.render('todo_website.hello')
```

Y la plantilla debe insertarse dentro del diseño general del sitio web:

```
<template id="hello" name="Hello World">

  <t t-call="website.layout">
    <h1>Hello World!</h1>
  </t>

</template>
```

Con esto, el ejemplo **Hola Mundo** que utilizo antes debe ahora ser mostrado dentro de una página del sitio web de Odoo.

## Adición de elementos CSS y JavaScript

Sus páginas web pueden necesitar algunos recursos adicionales de CSS o JavaScript. Este aspecto de las páginas web es gestionado por el sitio web, por lo que necesita una forma de decirle que también utilice sus archivos.

Va a añadir un poco de CSS para agregar un efecto de tachado simple para las tareas realizadas. Para ello, crea el archivo `todo_website/static/src/css/index.css` con este contenido:

```
.todo-app-done {
    text-decoration: line-through;
}
```

Luego tiene que incluirlo en las páginas del sitio web. Esto se hace agregándolos en la plantilla `website.assets_frontend` responsable de cargar los activos específicos del sitio web. Edite el archivo de datos `todo_website/views/todo_templates.xml`, para ampliar esa plantilla:

```
<odoo>
  <template id="assets_frontend"
            name="todo_website_assets"
            inherit_id="website.assets_frontend">

    <xpath expr="." position="inside">
        <link rel="stylesheet" type="text/css" href="/todo_website/static/src/css/index.css" />
    </xpath>

  </template>
</odoo>
```

Pronto estará usando esta nueva clase de estilo `todo-app-done`. Por supuesto, los activos de JavaScript también se pueden agregar usando un enfoque similar.


## El controlador de lista de tareas pendientes

Ahora que ha pasado por lo básico, va a trabajar en su lista de tareas por hacer. Tendrá una URL `/todo` mostrándole una página web con una lista de Tareas por hacer.

Para ello, necesita un método controlador, la preparación de los datos a presentar, y una plantilla QWeb para presentar esa lista al usuario.

Edita el archivo `todo_website/controllers/main.py` para agregar este método:

```
#class Main(http.Controller):

    @http.route('/todo', auth='user' , website=True)
    def index(self, **kwargs):
        TodoTask = request.env['todo.task']
        tasks =  TodoTask.search([])
        return request.render(
            'todo_website.index', {'tasks': tasks})
```

El controlador recupera los datos que se van a utilizar y los pone a disposición de la plantilla renderizada. En este caso, el controlador requiere una sesión autenticada, ya que la ruta tiene el atributo `auth='user'`. Incluso si ese es el valor predeterminado, es una buena práctica declarar explícitamente que se requiere una sesión de usuario.

Con esto, la declaración Todo Task `search()` se ejecutará con el usuario de la sesión actual.

Los datos accesibles a los usuarios públicos son muy limitados, cuando uso ese tipo de ruta, a menudo necesita usar `sudo()` para elevar el acceso y hacer disponibles los datos de la página que de otro modo no serían accesibles.

Esto también puede ser un riesgo de seguridad, así que tenga cuidado en la validación de los parámetros de entrada y en las acciones realizadas. También mantenga el uso del conjunto de registros `sudo()`  limitado a las operaciones mínimas posibles.

El método `request.render()` espera que el identificador de la plantilla de QWeb se procese y un diccionario con el contexto disponible para la evaluación de la plantilla.

## La plantilla de lista de tareas pendientes

La plantilla de QWeb debe ser agregada por un archivo de datos, y puede agregarla al archivo de datos `todo_website/views/todo_templates.xml` existente:

```
<template id="index" name="Todo List">
  <t t-call="website.layout">
    <div id="wrap" class="container">
      <h1>Todo Tasks</h1>

      <!-- List of Tasks -->
      <t t-foreach="tasks" t-as="task">
        <div class="row">
          <input type="checkbox" disabled="True"
            t-att-checked=" 'checked' if task.is_done else {}" />
          <a t-attf-href="/todo/{{slug(task)}}">
            <span t-field="task.name"
                  t-att-class="'todo-app-done' if task.is_done else ''" />
          </a>
        </div>
      </t>

      <!-- Add a new Task -->
      <div class="row">
        <a href="/todo/add" class="btn btn-primary btn-lg">Add</a>
      </div>

    </div>
  </t>
</template>
```

El código anterior utiliza la directiva `t-foreach` para mostrar una lista de tareas. La directiva `t-att` usada en la casilla de verificación de entrada les permite agregar o no el atributo *checked* dependiendo del valor `is_done`.

Usted tiene una entrada de casilla de verificación, y querrá que se compruebe si la tarea se realiza. En HTML, se comprueba una casilla de verificación en función de que tenga o no el atributo `checked`. Para ello uso la directiva `t-att-NAME` para renderizar dinámicamente el atributo `checked` dependiendo de una expresión. En este caso, la expresión se evalúa como `None`, QWeb omitirá el atributo, lo cual es conveniente para este caso.

Al procesar el nombre de la tarea, se utiliza la directiva `t-attf` para crear dinámicamente la URL para abrir el formulario de detalle para cada tarea específica. Se utiliza la función especial `slug()` para generar una URL legible por humanos para cada registro. El enlace no funcionará por ahora, ya que todavía está por crear el controlador correspondiente.

En cada tarea también uso la directiva `t-att` para establecer el estilo `todo-app-done` solo para las tareas que se realizan.

Finalmente, tiene un botón Añadir para abrir una página con un formulario para crear una nueva Tarea. Lo usará para introducir el manejo de formularios web a continuación.

## La página de detalles de las tareas por hacer

Cada elemento de la lista de tareas es un enlace a una página de detalles. Debe implementar un controlador para esos enlaces, y una plantilla QWeb para su presentación. En este punto, este debería ser un ejercicio sencillo.

En el archivo `todo_website/controllers/main.py` agregue el método:

```
#class Main(http.Controller):

    @http.route('/todo/<model("todo.task"):task>', website=True)
    def index(self, task, **kwargs):
        return http.request.render(
            'todo_website.detail',
            {'task': task})
```

Observe que la ruta utiliza un `placeholder` con el convertidor `model("todo.task")`, asignando a la variable de tarea. Captura un identificador de tarea desde la URL, ya sea un número de ID simple o una representación de `slug`, y lo convierte en el objeto de registro de exploración correspondiente.

Y para la plantilla de QWeb añada el siguiente código al archivo de datos `todo_website/views/todo_web.xml`:

```
<template id="detail" name="Todo Task Detail">
<t t-call="website.layout">
  <div id="wrap" class="container">
    <h1 t-field="task.name" />
    <p>Responsible: <span t-field="task.user_id" /></p>
    <p>Deadline: <span t-field="task.date_deadline" /></p>
  </div>
</t>
</template>
```

Cabe destacar aquí el uso del elemento `<t t-field>`. Se encarga de la representación adecuada del valor del campo, al igual que en el backend. Presenta correctamente valores de fecha y valores Many-to-One, por ejemplo.


## Formularios Web

Los formularios son una característica común que se encuentran en los sitios web. Ya tiene todas las herramientas necesarias para implementar una: una plantilla de QWeb puede proporcionar el HTML para el formulario, la acción de envío correspondiente puede ser una URL, procesada por un controlador que puede ejecutar toda la lógica de validación y finalmente almacenar los datos en el Modelo adecuado.

Pero para formas no triviales esto puede ser una tarea exigente. No es tan simple realizar todas las validaciones necesarias y proporcionar retroalimentación al usuario sobre lo que está mal.

Puesto que esto es una necesidad común, el módulo `website_form` está disponible para ayudarle con esto. Va a ver cómo usarlo.

Mirando hacia atrás en el botón Agregar en la lista Tarea por hacer, puede ver que abre la URL `/todo/add`. Esto presentará un formulario para enviar una nueva tarea por hacer y los campos disponibles serán el nombre de la tarea, una persona (usuario) responsable de la tarea y un archivo adjunto.

Debe comenzar agregando la dependencia `website_form` a su módulo. Podrá reemplazar el módulo `website`, ya que mantenerlo explícitamente sería redundante. En el `todo_website/__manifest__.py` edite la clave `depends` a:

```
  'depends': ['todo_kanban', 'website_form'],
```

Ahora va a añadir la página con el formulario.


## La página formulario

Podrá comenzar implementando el método del controlador para soportar la renderización del formulario, en el archivo `todo_website/controllers/main.py`:

```
@http.route('/todo/add', website=True)
def add(self, **kwargs):
    users = request.env['res.users'].search([])
    return request.render(
        'todo_website.add', {'users': users})
```

Se trata de un controlador sencillo, que muestra la plantilla `todo_website.add` y que le proporciona una lista de usuarios, de modo que pueda utilizarse para crear un cuadro de selección.

Ahora para la plantilla QWeb correspondiente. Podrá añadirlo al archivo de datos `todo_website/views/todo_web.xml`:

```
<template id="add" name="Add Todo Task">
  <t t-call="website.layout">
    <t t-set="additional_title">Add Todo</t>
    <div id="wrap" class="container">
      <div class="row">
        <section id="forms">
          <form method="post" class="s_website_form container-fluid form-horizontal"
                action="/website_form/"
                data-model_name="todo.task"
                data-success_page="/todo"
                enctype="multipart/form-data">

                <!-- Form fields will go here! -->

                <!-- Submit button -->
                <div class="form-group">
                  <div class="col-md-offset-3 col-md-7 col-sm-offset-4 col-sm-8">
                    <a class="o_website_form_send btn btn-primary btn-lg">
                      Save
                    </a>
                    <span id="o_website_form_result"></span>
                  </div>
                </div>

          </form>
        </section>
      </div> <!-- rows -->
    </div> <!-- container -->
  </t> <!-- website.layout -->
</template>
```

Como es de esperar, puede encontrar el elemento `<t t-call="website.layout"` específico de Odoo, responsable de insertar la plantilla dentro del diseño del sitio web, y `<t t-set="additional_title">` que establece un titulo adicional, esperado por el layout del sitio web.

Para el contenido, la mayoría de lo que puede ver en esta plantilla se puede encontrar en un típico formulario Booststrap CSS. Pero también tiene algunos atributos y clases de CSS que son específicos para los formularios del sitio web. Lo marca en negrita en el código, por lo que es más fácil identificarlos.

Las clases CSS son necesarias para que el código JavaScript pueda realizar correctamente su lógica de manejo de formularios. Y luego tiene algunos atributos específicos en el elemento `<form>`:

+ `action` es un atributo de formulario estándar, pero debe tener el valor "/website_form/". Se requiere la barra diagonal.

+ `data-model_name` identifica el modelo al que escribir y se pasará al controlador `/website_form`.

+ `data-success_page` es la URL a redireccionar después de una presentación de formulario correcta. En este caso, se les enviará de nuevo a la lista de tareas.

No necesitará proporcionar su propio método de controlador para manejar el envío de formularios. La ruta `/website_form` lo hará por nosotros. Toma toda la información que necesita del formulario, incluyendo los atributos específicos que acaba de describir, y luego realiza validaciones esenciales en los datos de entrada, y crea un nuevo registro en el modelo de destino.

Para casos de uso avanzado, puede forzar que se use un método de controlador personalizado. Para ello debe añadir un atributo `data-force_action` al elemento `<form>`, con la palabra clave para que el controlador objetivo a utilizar. Por ejemplo, `data-force_action="todo-custom"` tendría la solicitud para llamar a la URL `/website_form/todo-custom`. Entonces deberá proporcionar un método de controlador adjunto a esa ruta. Sin embargo, hacer esto quedará fuera de su alcance aquí.

Todavía tiene que terminar su formulario, agregando los campos para obtener las entradas del usuario. Dentro del elemento `<form>`, añade:

```
<!-- Description text field, required -->
<div class="form-group form-field">
  <div class="col-md-3 col-sm-4 text-right">
    <label class="control-label" for="name">To do*</label>
  </div>
  <div class="col-md-7 col-sm-8">
    <input name="name" type="text" required="True"
           class="o_website_from_input form-control" />
  </div>
</div>
 
<!-- Add an attachment field -->
<div class="form-group form-field">
  <div class="col-md-3 col-sm-4 text-right">
    <label class="control-label" for="file_upload">
      Attach file
    </label>
  </div>
<div class="col-md-7 col-sm-8">
    <input name="file_upload" type="file"
           class="o_website_from_input form-control" />
  </div>
</div>
```

Aquí está agregando dos campos, un campo de texto regular para la descripción y un campo de archivo, para cargar un archivo adjunto. Todo el marcado se puede encontrar en los formularios regulares Bootstrap, a excepción de la clase `o_website_from_input`, necesarios para la lógica de formulario de `website` para preparar los datos para enviar.

La lista de selección de usuario no es muy diferente excepto que necesita usar una directiva `t-foreach` QWeb para representar la lista de usuarios seleccionables. Podrá hacer esto porque el controlador recupera ese conjunto de registros y lo pone a disposición de la plantilla bajo el nombre `users`:

```
<!-- Select User -->
<div class="form-group form-field">
  <div class="col-md-3 col-sm-4 text-right">
    <label class="control-label" for="user_id">
      For Person
    </label>
  </div>
  <div class="col-md-7 col-sm-8">
    <select name="user_id" class="o_website_from_input form-control">
      <t t-foreach="users" t-as="user">
        <option t-att-value="user.id">
          <t t-esc="user.name" />
        </option>
      </t>
    </select>
  </div>
</div>
```

Sin embargo, su formulario todavía no funcionará hasta que haga alguna configuración de seguridad de acceso.

## Seguridad de acceso y elemento de menú

Dado que este manejo genérico de formularios está bastante abierto y se basa en datos no confiables enviados por el cliente, por razones de seguridad necesita algún tipo de configuración del servidor en lo que el cliente puede hacer. En particular, los campos de modelo que se pueden escribir basados en datos de formulario deben estar en la lista blanca.

Para añadir campos a esta lista blanca, se proporciona una función de ayuda y puede usarla desde un archivo de datos XML. Debe crear el archivo `todo_website/data/config_data.xml` con:


```
<?xml version="1.0" encoding="utf-8"?>
<odoo>
  <data>

    <record id="todo_app.model_todo_task" model="ir.model">
      <field name="website_form_access">True</field>
    </record>

    <function model="ir.model.fields" name="formbuilder_whitelist">
      <value>todo.task</value>
      <value eval="['name', 'user_id', 'date_deadline']"/>
    </function>

  </data>
</odoo>
```

Para que un modelo pueda ser utilizado por los formularios, debe hacer dos cosas: activar un indicador en el modelo, y poner en lista blanca el campo que se pueda utilizar. Estas son las dos acciones que se están realizando en el archivo de datos anterior.

No olvide que, para que su módulo conozca este archivo de datos, debe agregarse a la clave `data` en el manifiesto del módulo.

También sería bueno que su página Todo esté disponible en el menú del módulo `website`. Va a añadirla usando el mismo archivo de datos. Agrega otro elemento `<data>` como este:

```
<data noupdate = "1">
  <record id="menu_todo" model="website.menu">
    <field name="name">Todo</field>
    <field name="url">/todo</field>
    <field name="parent_id" ref="website.main_menu" />
    <field name="sequence" type="int">50</ field>
  </record>
</data>
```

Como se puede ver, para agregar un elemento de menú del sitio web solo necesita crear un registro en el modelo `web.menu`, con un nombre, una URL y el identificador del elemento de menú principal. El nivel superior de este menú tiene como padre el elemento `website.main_menu`.

## Adición de lógica personalizada

Los formularios web les permiten conectar nuestras propias validaciones y cálculos al procesamiento de formularios. Esto se realiza mediante la implementación de un método `website_form_input_filter()` con la lógica del modelo de destino. Acepta un diccionario de valores, lo valida y realiza cambios en él y, a continuación, devuelve el diccionario de valores posiblemente modificado.

Lo utilizará para implementar dos funciones: eliminar cualquier espacio inicial y final del título de la tarea e imponer que el título de la tarea tenga al menos tres caracteres.

Agregue el archivo `todo_website/models/todo_task.py` que contiene el siguiente código:

```
# -*- coding: utf-8 -*-

from odoo import api, models
from odoo.exceptions import ValidationError

class TodoTask(models.Model):
    _inherit = 'todo.task'

    @api.model
    def website_form_input_filter(self, request, values):
        if 'name' in values:
            values['name'] = values['name'].strip()
            if len(values.['name']) < 3:
                raise ValidationError(
                    'Text must be at least 3 characters long')
        return values
```

El método `website_form_input_filter` realmente espera dos parámetros: el objeto `request` y el diccionario `values`. Los errores que impiden el envío de formularios deben generar una excepción `ValidationError`.

La mayor parte del tiempo este punto de extensión para los formularios debería permitirle evitar manipuladores de presentación de formularios personalizados.

Como de costumbre, debe importar este nuevo archivo de Python, añadiendo `from . import models` en el archivo `todo_website/__init__.py`, y añadiendo el archivo `todo_website/models/__init__.py` con un `from . import todo_task line`.


## Resumen

Ahora debes tener un buen entendimiento sobre lo esencial de las características del sitio web. Usted ha visto cómo usar controladores web y plantillas QWeb para renderizar páginas web dinámicas. A continuación, aprendió a utilizar el complemento de sitio web y crear nuestras propias páginas para ello. Por último, ha introducido el complemento de formularios de sitios web que les ayudó a crear un formulario web. Estos deben proporcionarle las habilidades básicas necesarias para crear las características del sitio web.

A continuación, aprenderá cómo tener aplicaciones externas interactuar con nuestras aplicaciones Odoo.
