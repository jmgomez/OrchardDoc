


El objetivo de este artículo es familiarizarte con Orchard. Hay tantos conceptos para entender que es necesario dar un vistazo general antes de entrar en profundidad. Por lo tanto, este artículo comienza con una base común que cualquier usuario debe entender y profundiza progresivamente al tiempo que introduce los términos técnicos pertinentes .

Cuando termines de leer esto, tendrás un conocimiento que te permitirá jugar con Orchard (como diseñador / desarrollador) sin confundirte por su arquitectura de alto nivel y su terminología. Cuando se introduce,  cada término nuevo se pone en negrita para darle enfásis y se asegure de entenderlo antes de continuar con la lectura.

Este artículo contiene una gran cantidad de enlaces a otras páginas con explicación específica, de modo que puedes utilizarlo como punto de partida. Para responder a algunas preguntas generales sobre qué es Orchard, lee [Frequently Asked Questions](frequently-asked-questions). Para una presentación más técnica, [How Orchard works](How-Orchard-works).


# Orchard como...
La mejor manera de introducir los conceptos básicos de Orchard, es echar un vistazo a lo que un usuario puede hacer con los roles: usuario normal (también conocido como lector / anónimo / invitado), administrador, diseñador y desarrollador.

## Usuario
Como **usuario**, un sitio  Orchard es como cualquier otro **website**: Tiene una página de inicio, desde donde se puede acceder a otras páginas a través de enlaces. Dependiendo del tipo de sitio, el contenido puede variar (pueden ser páginas estáticas, blog, wiki, comercio electrónico, etc)

## Administrador
El **administrador** tiene acceso a varias partes del sitio:

1. Cuando está instalando Orchard  [installing Orchard](Installing-Orchard), verá la página de instalación. Este paso concluye en la creación de una base de datos donde se almacenan todos los contenidos y la configuración de la página web.

2. Por supuesto, como usuario, también puede ver el fornt-end.
3. Puedes abrir el **[dashboard](Getting-Around-the-Admin-Panel)** (aka control panel/back-end) por dos razones:
     1.. [Configure the website](Getting-Started):  editar la configuración y el aspecto de la página web (o instalar / activar / actualizar)

     2. [Editar el contenido](Getting-Started) de la página web

4.  **[Command line](Using-the-command-line-interface)**: Es posible hacer scripts de la mayoría de acciones de la administración desde la línea de comandos, por lo que es fácil de automatizar ciertas operaciones

  
## Diseñador

El diseñador puede modificar [la apariencia de una pagina web](Previewing-and-applying-a-theme). Puede editar las configuraciones de un theme existente o crear uno. Un **[theme](Anatomy-of-a-theme)** es todo lo que interviene en la representación visual de la web. A veces se llama skin o la plantilla. Transforma el  **content** (datos generados por el usuario) en HTML que se puede visualizar en un navegador. Por ejemplo: cuando se escribe una entrada en el blog, el theme define dónde y cómo se muestran los menús, títulos, cuerpo, comentarios, etc


Depending on how much customization is required, the designer may [edit some or all elements of the theme](Customizing-the-default-theme). These elements are of the following types:

Dependiendo de la cantidad de personalización es necesario, el diseñador puede [editar algunos o todos los elementos del tema](Customizing-the-default-theme). Estos elementos son de los tipos siguientes:

*Los documentos que definen el **layout** y sus zonas: Esta es la representación general de una página en el sitio web (sin ningún contenido real). Por ejemplo: Se dice que si el sitio web debe tener una, dos o tres columnas. Así, una **zona** es un contenedor que forma parte del layout y está listo para recibir cualquier contenido. Tenga en cuenta que un theme flexible (como el proporcionado por Orchard) puede adaptarse para ocultar las zonas vacías.

* **Views**: Es la representación visual de un contenido específico. Una vista es típicamente un archivo con la extensión CSHTML o ASPX. Proporciona el código HTML para utilizar cuando se muestra un tipo de contenido específico. Por lo tanto, una página con muchos contenidos (menú, post, comentarios, etc) se creó mediante la composición de todos los puntos de vista individuales pertinentes.

* **Stylesheets**, **Javascript** y archivos Media: Se utilizan para modificar el aspecto definido en las vistas. Son archivos como "Site.css", jQuery o las imágenes de fondo, bordes y los iconos.

* **[Widget](Managing-widgets)**: Una página web muestra típicamente un contenido principal (como un post), pero a menudo también tiene pequeños trozos de información a los lados. Por ejemplo: una nube de etiquetas, una lista de los últimos posts, un feed de Twitter, etc


* Layers and the binding between content to specific zones: A **layer** is like the description of a group of pages. Once defined, you can tell where to put each content (or widget). Eg: A layer grouping all the blog posts can be defined, then we can make a Tag cloud widget appear on the side

Layers (capas) y los bindings entre el contenido y zonas específicas: Un **Layer** es como una capa de un grupo de páginas. Una vez definido, puede decirle dónde poner cada contenido (o widget). Por ejemplo: Un layer puede agrupar todas las entradas del blog, entonces podemos hacer un widget que actue como nube de tags y que se situe a su lado.

El tratamiento avanzado de themes puede incluir algún tipo de programación para personalizar aún más la vista.

## Desarrollador

El **desarrollador** tiene una visión completa de la arquitectura de Orchard y puede extenderlo.
Orchard está organizado en módulos. Cada **módulo** proporciona un bloque de construcción (también conocido como add on/ plugin) a la página web con un objetivo distinto. Por ejemplo, puede haber:


* **Módulo de extensión** : Añade algunas característica (de bajo nivel) que benefica a la página web. pe: La capacidad de [buscar contenido](Search-and-indexing) o de [usar un editor externo, como liveWriter para editar los posts](Blogging-with-LiveWriter)
* Content module (Módulo de contenido): Agrega todo lo necesario (a nivel de código y visual) para ver o editar los tipos de contenido que hay (como posts).
* Módulo Widget : Añade un pequeño contenido visual para un mostrar a un lado de los módulos existentes de contenido existente (como un nube de tag a un blog).
* Theme module: Cambia parte de la vista de un Theme (Esto es lo que, comúnmente, crearía un diseñador.)
* Todo lo demás: Un módulo puede tener muchas extensiones, tipos de contenido, widgets y temas en un solo paquete.

Orchard is designed to be highly extensible; this means that almost anything that you interact with can be extended, replaced or disabled.
Out of the box, Orchard comes with a number of modules to provide a good user/administrator experience; but a designer/developer can change them or [create more](Building-a-hello-world-module). It is also possible to [share your modules](Packaging-and-sharing-a-module) with the Orchard community and to [install modules](Installing-and-upgrading-modules) developed by others.

Orchard está diseñado para ser altamente extensible, lo que significa que casi cualquier cosa con la que se interectúe dentro de Orchard, se puede extender, reemplazar o desactivar.
Por defecto, Orchard viene con una serie de módulos para proporcionar lo necesario a un usuario / administrador, pero un diseñador / desarrollador puede modificar o [crear más](Building-a-hello-world-module). También es posible [compartir  módulos](Packaging-and-sharing-a-module) con la comunidad de Orchard e [instalar módulos](Installing-and-upgrading-modules) desarrollados por otros.

Orchard v0.8 viene con un sólo theme (llamado "[The Theme Machine](Anatomy-of-a-theme)"). However, it has enough zones to allow various arrangements. This is important because, by default, a site can only have one theme (unless you develop a module allowing more), so the theme must be flexible enough to allow pages to have different layouts. If you are not satisfied, you can copy it and add more zones.

Sin embargo, tiene zonas suficientes para permitir diferentes posiciones/combinaciones. Esto es importante porque, de forma predeterminada, un sitio sólo puede tener un theme (a menos que desarrolle un módulo que permita más), por lo que el theme debe ser lo suficientemente flexible como para permitir  páginascon diferentes diseños. Si necesita más, puede copiar y añadir nuevas zonas.


[The Orchard Gallery](Gallery-overview) contiene más themes y módulos listos para instalar. Asegúrese de  saber qué características adicionales están disponibles.

# Contenido

Con el fin de completar su sitio web, Orchard le pepor defecto. Pero también le permite definir su propio contenido. Esto es importante porque es posible que desee tener eventos o perfiles o cualquier otra cosa que no sea compatible por defecto. En esta sección se explica esta capacidad.


* Content: Los datos que normalmente se muestran en el front-end. Se usa este término como una forma genérica de llamar a cualquier cosa que esté generada por el usuario..

* [Content Type &amp; Item](Creating-custom-content-types): Un **content type** (tipo de contenido) es como una clase dinámica; define una estructura de datos para un tipo específico de contenido. Esa estructura incluso puede ser cambiada por el administrador. Un **content item** (elemento de contenido) es una instancia del content type. Por lo tanto, BlogPost puede ser un tipo de contenido, pero la entrada en sí, es un content item.

* **[Content Part](Writing-a-content-part)**: Because many content types share many aspects; these aspects can be created independently and reused in each content type. That's what a content part is. Eg: A blog post can have comments; a photo can also have comments; so, instead of implementing the "comments" feature twice, we can create it as a content part and reuse it for both content types.

Debido a que muchos Content Type  tienen muchos aspectos en común, estos aspectos pueden ser creados de forma independiente y se reutiliza en cada Content Type. Precisamente esto, es un Content Part. Por ejemplo: Un blog puede tener comentarios, la foto también puede tener comentarios, así que, en lugar de implementar "Comentarios" dos veces, podemos crearlo como un Content Part y volver a utilizarlo para ambos tipos de contenido.

* **[Content Field](Creating-a-custom-field-type)**: In the same spirit of reusability, we can have smaller types that must behave in a certain way. Eg: Most content types will need Date, phone number, email address, etc. They aren't simple properties since we can attach some behavior (like validation rules) but they aren't content parts either (too "small"). 

Con el mismo espíritu de reutilización, podemos tener tipos más pequeños que deben comportarse de una determinada manera. Por ejemplo: La mayoría de los content type necesitarán fecha, número de teléfono, dirección de correo electrónico, etc. y no son propiedades simples ya que podemos unir un poco de comportamiento (como las reglas de validación), pero tampoco son content part (demasiado "pequeños").

* **Record**: In order to be able to save a content type/part (in a SQL database), it needs to be "linked" to a record. It is a class with all the properties that should be saved. Eg: A Map part must save its coordinates (Latitude &amp; Longitude), so it will be linked to a record with these two properties; and Orchard will do the rest to load/save it. You will not have to deal with records unless you [develop your own module](Building-a-hello-world-module). But it is useful to understand this concept in case you encounter it.

Con el fin de ser capaz de guardar un Content Type/Part (en una base de datos SQL), éste debe ser "unido" a un registro. Que es una clase con todas las propiedades que se deben guardar. Por ejemplo: Un Part Map debe guardar sus coordenadas (latitud y longitud), por lo que estará vinculado a un registro con estas dos propiedades, y Orchard hará el resto para cargar / guardar. No tendrás que lidiar con los registros a menos que [develop your own module](Building-a-hello-world-module). Pero la información presentada es útil para entender este concepto en caso de que se lo encuentre.


Note that a content type can only have one of each kind of content parts. But it can have many fields of the same kind. The reason is in the semantic meaning of these concepts. For example, a blog post can only have one commenting aspect and it can have many dates (creation date, last update date, etc.).

Tenga en cuenta que un Content Type sólo puede tener uno de cada Content Part. Pero puede tener muchos Content Fields de la misma clase. La razón está en el significado semántico de estos conceptos. Por ejemplo, una entrada de blog sólo puede tener un tipo "comentario" pero sin embargo puede tener muchas fechas (fecha de creación, fecha de última actualización, etc.)


Orchard es un [proyecto open source](frequently-asked-questions), siéntete libre de [contribuir](Contributing-patches) con cualquier característica/módulo que quieras.

# Conclusión

Vamos a detenernos aquí. En este punto, usted debe tener una buena comprensión de lo que es Orchard. Cualquier artículo que habla sobre [cómo usarlo](MainPage) debería ser más fácil de entender. El siguiente paso es entrar en detalles un poco más en profundidad sobrelos módulos, temas, y [la arquitectura a bajo nivel de Orchard](How-Orchard-works). Esto es útil cuando empiezas a aprender[cómo extender Orchard](Building-a-hello-world-module).