
Crear un CMS (Content Management System o gestor de contenidos) es diferente a la construcción de una aplicación web normal: su arquitectura es más como la construcción de un contenedor de aplicaciones. Al diseñar tal sistema, es necesario construir pensando en la extensibilidad como una de las principales necesidades. Esto puede ser un reto ya que la extensibilidad se consigue vía una arquitectura abierta y esto, puede comprometer la usabilidad de la aplicación: todo lo que el sistema requiere será construido a base de módulos desconocido en el futuro, incluso a nivel de interfaz de usuario. Orquestar estas pequeñas partes desconocidas entre sí, es lo que trata de hacer Orchard.



This document explains the architectural choices we made in Orchard and how they are solving that particular problem of getting both flexibility and a good user experience.


# Architecture

<table cellspacing="2" cellpadding="2" border="1" style="width:100%">
<tr>
<td colspan="4" align="center">Modulos
</tr><tr>
<td colspan="4" align="center">Núcleo
</tr><tr>
<td colspan="4" align="center">Orchard Framework
</tr><tr>
<td align="center" style="width:25%">ASP.NET MVC
<td align="center" style="width:25%">NHibernate
<td align="center" style="width:25%">Autofac
<td align="center" style="width:25%">Castle
</tr><tr>
<td colspan="2" align="center">.NET
<td colspan="2" align="center">ASP.NET
</tr><tr>
<td colspan="4" align="center">IIS or Windows Azure
</tr>
</table>

#Los cimientos de Orchard

The Orchard CMS is built on existing frameworks and libraries. Here are a few of the most fundamental ones:

Orchard CMS se basa en en Frameworks y librerías preexistentes. Estas son algunas de las más importantes:

- [ASP.NET MVC](http://www.asp.net/mvc): ASP.NET MVC es un framework de desarrollo web moderno, que fomenta la separación de compontentes.
- [NHibernate](http://nhforge.org/): NHibernate es una herramienta de mapeo objeto-relacional. Se ocupa de la persistencia de los ContentItem de Orchard en la base de datos y simplifica considerablemente el modelo de datos mediante la eliminación por completo de la persistencia en el desarrollo de módulos. Puedes ver ejemplos de código en cualquier Content Type básico, como por ejemplo, Pages.

- [Autofac](http://code.google.com/p/autofac/): Autofac es un [contenedor de inversión de control o loC](http://en.wikipedia.org/wiki/Inversion_of_control). Orchard hace un uso intensivo de inyección de dependencias. La creación de una dependencia inyectable en Orchard es tan simple como escribir una clase que implementa una interfaz IDependency o de una clase derivada de IDependency, y el consumo de la dependencia es tan simple como pasar un parámetro al constructor del tipo correcto. El ámbito y el ciclo de vida de la dependencia inyectada será gestionado por Orchard. Puedes ver ejemplos en el código fuente de IAuthorizationService, RolesBasedAuthorizationService y XmlRpcHandler.


- [Castle Dynamic Proxy](http://www.castleproject.org/dynamicproxy/index.html): Castle se usa para la generación de dynamic proxy (proxy dinámico). Orchard se construyen en la parte superior de estos frameworks establecidas como capas adicionales de abstracción. Se encuentran en partes muy profundias en la arquitectura y no se necesita conocimiento de NHibernate, Castillo, o autofac para trabajar con Orchard.


# Orchard Framework


El  Orchard framework es la capa más profunda de Orchard. Contiene el motor de la aplicación o al menos las partes que no se pudieron aislar en módulos. Estas partes son por lo general las cosas que hasta los módulos más fundamentales tendrán que confiar. Puede pensar en él como la biblioteca de clases base de Orchard.

## Arrancancdo Orchard

Cuando una aplicación web Orchard empieza a funcionar, un host de Orchard se crea. Un host es un producto único en el nivel de dominio de aplicación.

A continuación, el host tendrá la Shell para el tenant (inquilino) actual mediante el ShellContextFactory. Los tenant son instancias de la aplicación que están aislados por lo que los usuarios pueden contar, pero que se están ejecutando en el dominio de aplicación misma, mejorando la densidad de sitios. La shell es un producto único en el nivel de tenant y en realidad podría decirse que representa el tenant. Es el objeto que efectivamente proporcionan el  aislamiento tenant-nivel, manteniendo el modelo de programación de módulo agnóstico acerca de múltiples clientes.

La shell, una vez creadoa, obtendrá la lista de extensiones disponibles en el ExtensionManager. Las extensiones son módulos y temas. La implementación por defecto escanea los módulos y directorios de temas para obtener las extensiones.

Al mismo tiempo, la shell obtendrá la lista de configuraciones para los tenant de la ShellSettingsManager. La implementación predeterminada obtiene los valores de la subcarpeta apropiada de App_Data pero las implementaciones alternativas pueden obtener los distintos lugares. Por ejemplo, tenemos una aplicación que está utilizando en Azure con almacenamiento de blob en lugar App_Data porque no es fiable escribir en ese ambiente.


The shell then gets the CompositionStrategy object and uses it to prepare the IoC container from the list of available extensions for the current host and from the settings for the current tenant. The result of this is not an IoC container for the shell, it is a ShellBlueprint, which is a list of dependency, controller and record blueprints.

La shell obtiene el objeto CompositionStrategy y lo utiliza para preparar el contenedor IoC de la lista de extensiones disponibles para el host actual y las opciones para el inquilino actual. El resultado de esto no es un contenedor IoC para la carcasa, es un ShellBlueprint, que es una lista de los planos de dependencia, del controlador y de registro blueprint.

La lista de ShellSettings (que son por tenant) y ShellBluePrint son entonces arrojados a ShellContainerFactory.CreateContainer para obtener una ILifetimeScope, que básicamente, permite al contenedor IoC limitar el ámbito a nivel de tenant para que los módulos se pueden inyectar las dependencias cuyo ámbito de el tenant actual sin tener que hacer nada específico.

## Inyección de dependencias

La manera estándar de crear dependencias inyectables en Orchard es crear una interfaz que se deriva de IDependency o una de sus interfaces derivadas y luego implementar dicha interfaz. En el lado del consumidor, se puede coger un parámetro del tipo de interfaz en su constructor. El framework de aplicación descubrirá todas las dependencias y se hará cargo de una instancia y la inyección de instancias según sea necesario.

Hay tres diferentes ámbitos posibles para las dependencias, y la elección se lleva a cabo mediante la derivación desde la interfaz de la derecha:

- Solicitud: un ejemplo de dependencia se crea para cada nueva petición HTTP y se destruye una vez que la solicitud ha sido procesada. Utilice esta derivando su interfaz de IDependency. El objetivo debería ser razonablemente barato para crear.
- Objeto: una nueva instancia es creada cada vez que un objeto toma una dependencia en la interfaz. Las instancias no son compartidas. Utilice esta derivando de ITransientDependency. Los objetos deben ser extremadamente barato de crear.
- Shell: una sola instancia se crea por shell / tenant Utilice esta derivando de ISingletonDependency. Sólo la usamos para objetos que deben mantener un estado común para el tiempo de vida de la shell.

### Reemplazando dependencias existentes

Es posible reemplazar las dependencias existentes en la decoración de su clase con el atributo OrchardSuppressDependency, que lleva el nombre de tipo completo para reemplazar como argumento.

### Ordering Dependencies

Algunas dependencias no son únicas sino más bien partes de una lista. Por ejemplo, los controladores están activos al mismo tiempo. En algunos casos, tendrá que modificar el orden en que dichas dependencias se consumen. Esto puede hacerse modificando el manifiesto para el módulo, utilizando la propiedad de la característica de prioridad. Aquí hay un ejemplo de esto:

    
    Features:
        Orchard.Widgets.PageLayerHinting:
            Name: Page Layer Hinting
            Description: ...
            Dependencies: Orchard.Widgets
            Category: Widget
            Priority: -1



## ASP.NET MVC

Orchard se basa en ASP.NET MVC, pero con el fin de añadir cosas como el aislamiento theming y el tenant, tiene que introducir un nivel adicional de indirección que presentará en el lado de ASP.NET MVC los conceptos que se espera y la voluntad en el lado Orchard partiendo las cosas en el nivel de los conceptos Orchard.

Por ejemplo, cuando una vista específica se solicita, nuestro LayoutAwareViewEngine la lanza. En sentido estricto, no es un motor de vista nuevo, ya que no se refiere a la representación real, pero contiene la lógica para encontrar la vista adecuada en función del tema actual y luego delega la renderización de trabajo a los motores de vista activos

Del mismo modo, contamos con proveedores de rutas, carpetas modelo y las fábricas de controladores cuyo trabajo consiste en actuar como un punto de entrada único para ASP.NET MVC y enviar las llamadas a los objetos propiamente de ámbito inferior.

En el caso de las rutas, podemos tener n proveedores de rutas (por lo general procedentes de módulos) y un editor ruta que será lo que habla con ASP.NET MVC. Lo mismo ocurre con binders del modelo y las fábricas de controlador.

## Sistema de contenido

Los contenidos en Orchard se manejan bajo un sistema de tipo real que _ es en algunos aspectos más ricos y más dinámico que el subyacente sistema de tipo NET, con el fin de proporcionar la flexibilidad necesaria en un CMS Web:. los tipos deben estar compuestos sobre la marcha en tiempo de ejecución y reflejar las preocupaciones de gestión de contenidos.

### ContentTypes, ContentParts, y ContentFields

Orchard puede manejar tipos arbitrarios de contenido, incluyendo algunos que se crean dinámicamente por el administrador del sitio de una manera libre de código. Los content type son agregaciones de piezas de contenido que ocupan de un problema particular. La razón de esto es que muchas problemas abarcan más de un content type.

Por ejemplo, un blog, un producto y un clip de vídeo todos puedan tener una dirección enrutable, comentarios y etiquetas. Por esa razón, la dirección enrutable, comentarios y etiquetas son tratados en Orchard como una parte de contenido independiente. De esta manera, el módulo de gestión comentario puede ser desarrollado sólo una vez y se aplican a content type arbitrarios, incluidas los que el autor del módulo de comentarios no sabía nada.

Content Parts de ellos mismos pueden tener propiedades y content fileds . Content fileds también son reutilizables de la misma manera que los content parts son: un tipo de campo específico en que se usará por parte de varios content part y content type . La diferencia entre content parts y content fields reside en la escala a la que trabajan y en su semántica.

ContentFields are a finer grain than ContentParts. For example, a field type might describe a phone number or a coordinate, whereas a ContentPart would typically describe a whole concern such as commenting or tagging.

Pero la diferencia importante aquí es semántica: si quieres escribir un ContentPart si implementa un "es" una relación, y usted podría escribir un ContentField si se implementa un "tiene una" relación.

Por ejemplo, una camisa ** es un producto ** y ** tiene una SKU ** y un precio. No diría que una camisa tiene un producto o una camisa _ es un precio o un SKU.

A partir que usted sabe que el tipo de contenido camiseta se hace de un ProductPart, y que el ProductPart se hará a partir de un MoneyField llamado "precio" y un SKU Stringfield.

Otra diferencia es que tiene sólo una parte de un tipo dado por ContentType, que tiene sentido a la luz del "es" una relación, mientras que un ContentPart puede tener cualquier número de campos de un tipo determinado. Otra forma de decir esto es que ContentFields en una parte es un diccionario de cadenas en valores de tipo de campo, mientras que el ContentType es una lista de ContentPart (sin nombres).

Esto proporciona otra forma de elegir entre ContentPart ContentField: si crees que la gente quiere más de una instancia del objeto por ContentType, tiene que ser un campo.

### Anatomía de un Content Type

 ContentType, como hemos visto, está construido a partir de piezas . ContentParts, code-wise se asocian típicamente con::

- a Record (registro), que es una representación POCO de datos de la ContentPart
- una clase del modelo _ es la parte actual y que se deriva del `` ContentPart <T> donde T es el tipo de registro
- un repositorio. El repositorio no necesita ser implementado por el autor del módulo ya que Orchard será capaz de utilizar sólo uno genérico.
- handlers (manejadores). Handlers   implementan IContentHandler son un conjunto de controladores de eventos tales como OnCreated o OnSaved. Básicamente, estos se aferran ciclo de vida del ContentItem para llevar a cabo una serie de tareas. También pueden participar en la composición real de los elementos de contenido de sus constructores. Hay una colección de filtros de la ContentHandler base que habilita el controlador para agregar comportamiento común para el tipo de contenido. 
Por ejemplo, Orchard ofrece un StorageFilter _ hace que sea muy fácil de declarar cómo la persistencia de un ContentPart debe ser manipulada: simplemente escribir `Filters.Add (StorageFilter.For (myPartRepository));` Orchard se hará cargo de la persistencia de la base de datos del datos de myPartRepository.
Otro ejemplo de un filtro es el ActivatingFilter que se encarga de hacer del soldadura real de las piezas en un tipo: llamando a `Filters.Add (new ActivatingFilter <BodyAspect> (BlogPostDriver.ContentType.Name));` añade del ContentPart cuerpo al blog puestos.
-los drivers. Los drivers son más amigables, manipuladores más especializados (y en consecuencia menos flexible) y están asociados con un tipo específico ContentPart (que derivan de `ContentItemDriver <T> donde T es un tipo ContentPart). Controladores en del otro lado no tiene que ser específica para un tipo ContentPart. Los drivers pueden verse como controladores de una parte específica. Suelen construir shapes para ser redenderizados por el motor del tema.

## Content Manager
Todos los contenidos son accesibles en Orchard a través del objeto de Content Manager, que es como se hace posible el uso de contenidos de un tipo que no se sabe de antemano.

ContentManager tiene métodos para consultar del almacén de contenido, a los contenidos de la versión y gestionar su estado de publicación.

## Transacciones

Orchard crea automáticamente una transacción para cada solicitud HTTP. Eso significa que todas las operaciones que tienen lugar durante una petición son parte de un "ambiente" de la transacción. Si el código de solicitud durante la transacción se aborta , todas las operaciones de datos se deshace. Si la transacción no está explícitamente cancelado por el contrario, todas las operaciones se comprometan al final de la solicitud sin una confirmación explícita.


## Solicitud del ciclo de vida

En esta sección, vamos a tomar el ejemplo de una solicitud de un post específico.

Cuando llega una solicitud para un post específico, la primera aplicación analiza las rutas disponibles que han sido aportadas por los distintos módulos y encuentra la  ruta coincidente del módulo blog. La ruta se puede resolver solicitando la acción del elemento controlador de entrada del blog, que buscará el puesto de gestor de contenidos. La acción entonces consigue un modelo de objetos de página en el gestor de contenidos (por BuildDisplay ) basada en el objeto principal de esta solicitud, el mensaje que se ha recuperado desde el gestor de contenidos.

Un blog tiene su propio controlador, pero ese no es el caso para todos los ContentTypes. Por ejemplo, los tipos de contenido dinámico será servido por la  ItemController más genérica de la parte Routable Core. La acción de visualización de la  ItemController hace casi lo mismo que el controlador de entrada en el blog estaba haciendo: se pone el elemento de contenido desde el gestor de contenidos de slug y luego construye la  POM de los resultados.

El motor de vista  entonces se resolverá el vista correcta en función del tema actual y usando el tipo de binders del modelo, junto con las convenciones sobre los nombres de Orchard vista.

En la vista, la creación de una forma más dinámica que puede suceder, como las definiciones de zona.

La representación real es realizado por el motor del tema _ se va a encontrar la plantilla adecuada o el método de la forma de hacer cada una de las Shapes que encuentra en la  POM, por orden de aparición de forma recursiva.

## Widgets

Los widgets son ContentTypes que tienen el ContentPart Widget y el estereotipo widget. Como cualquier otro ContentTypes , que se componen de ContentParts y ContentFields. Eso significa que pueden ser editadas usando la misma edición y lógica de renderización como otro ContentTypes . También comparten la  mismos bloques de construcción, lo que significa que cualquier contentPart existente potencialmente pueden ser reutilizados como parte de un control casi sin esfuerzo.

Los widgets se añaden a las páginas a través de las capas de widget. Las capas son conjuntos de widgets. Tienen un nombre, una regla que determina qué páginas del sitio deben aparecer en ella, y una lista de widgets y la colocación zona asociada, ordenarlas, y los ajustes.

Las reglas unidos a cada una de las capas se expresan con expresiones IronRuby. Esas expresiones se pueden utilizar cualquiera de las implementaciones IRuleProvider en la aplicación. Orchard viene con dos de las implementaciones de serie: url y autenticado.

## Ajustes del sitio

Un sitio en Orchard es un ContentItem, que hace posible que los módulos para soldar piezas adicionales. Así es como módulos pueden contribuir configuración del sitio.

Configuración del sitio son por tenant

## Event Bus
Orchard y sus módulos de exponen puntos de extensibilidad mediante la creación de interfaces para las dependencias, las implementaciones de las cuales se puede obtener inyectado.

Enchufarlo a un punto de extensibilidad se realiza ya sea mediante la implementación de su interfaz, o mediante la implementación de una interfaz que tiene el mismo nombre y los mismos métodos. En otras palabras, Orchard no requiere correspondencia estrictamente inflexible de tipos de interfaz, que permite a los plug-ins para ampliar un punto de extensibilidad sin tener una dependencia en el ensamblado donde se define.

Esta es sólo una implementación del bus evento Orchard. Cuando un punto de extensibilidad llama a implementaciones de inyección, un mensaje se publica en el bus de eventos. Uno de los objetos escucha para el evento se distribuirá bus de los mensajes a los métodos en clases derivadas de una interfaz nombre apropiado.

## Comandos

Muchas de las acciones en un sitio Orchard se puede realizar desde la línea de comandos, así como de la interfaz de usuario admin. Estos comandos están expuestos por los métodos de aplicación de ICommandHandler clases que están decoradas con un atributo CommandName.

La Consola de Orchard descubre comandos disponibles en tiempo de ejecución mediante la simulación del entorno del sitio web y la inspección de los ensamblados mediante la reflexión. El entorno en el que la ejecución de comandos sucede debe ser lo más cercano posible al sitio real de operación.

## Buscando e Indexando

La búsqueda e indexación se implementan utilizando Lucene por defecto, aunque la implantación por defecto podría ser reemplazado con otro motor de indexación.

## Caching

El caché en Orchard se basa en la memoria caché de ASP.NET, pero se expone una API auxiliar que se puede utilizar a través de una dependencia del tipo iCache , mediante una llamada al método Get. Obtiene  una clave y una función que se puede utilizar para generar el valor de la entrada de la caché, si la memoria caché no contiene ya la entrada solicitada.

La principal ventaja de la utilización de la API de Orchard para el almacenamiento en caché es que trabaja por tenant transparente.

## Sistema de ficheros

El sistema de archivos en Orchard se abstrae de modo que el almacenamiento puede ser dirigido al sistema de archivo físico o a un almacenamiento alternativo, como almacenamiento de blob Azure, en función del entorno. El módulo de Media es un ejemplo de un módulo que  que es abstraído sistema de archivos.

## Usuarios y Roles

Usuarios en Orchard son ContentItems (aunque no se les puedan establecerse rutas) que hace que sea fácil para un módulo de perfil, por ejemplo, extenderlos con campos adicionales.
Los roles son un ContentPart que se adhiere a los usuarios.

## Permisos

Cada módulo puede exponer un conjunto de permisos, así como la forma en dichos permisos deben concederse de forma predeterminada para funciones predeterminadas de Orchard.

## Tareas

Los módulos pueden programar tareas llamando CreateTask en una dependencia del tipo IScheduledTaskManager tipo. La tarea puede ser ejecutado mediante la aplicación de IScheduledTaskHandler. El método de proceso puede examinar el nombre del tipo de tarea y decidir si se debe manejar la situación.

Las tareas se ejecutan en un subproceso independiente que proviene del grupo de subprocesos de ASP.NET.

## Notificaciones

Los módulos pueden emerger mensajes a la interfaz de usuario de administración obteniendo una dependencia de INotifier y llamando a uno de sus métodos. Varias notificaciones se pueden crear como parte de cualquier solicitud.

## Localización

La localización de la aplicación y sus módulos se hace envolviendo recursos de cadena en una llamada al método T: `<%: T (" Esta cadena se puede localizar ")%>`. Ver [Using the localization helpers](Using-the-localization-helpers) para más detalles y directrices. El gestor de recursos de Orchard puede cargar las cadenas de recursos adaptados a partir de los ficheros PO ubicados en lugares específicos de la
Content item localization is done through a different mechanism: localized versions of a content item are physically separate content items that are linked together by a special part.

La cultura actual a utilizar se determina por el gestor de cultura. La implementación predeterminada devuelve la referencia cultural que se ha configurado en la configuración del sitio, pero una implementación alternativa podría llegar desde el perfil de usuario o de la configuración del navegador.

## Logging

El registro se realiza a través de una dependencia del tipo ILogger . Diferentes implementaciones pueden enviar las entradas del registro para distintos tipos de almacenamiento. Orchard viene con una aplicación que utiliza [Castle.Core.Logging](http://api.castleproject.org/html/N_Castle_Core_Logging.htm) para el login.

# Orchard Core

El ensamblado Orchard.Core contiene un conjunto de módulos que son necesarios para que Orchard funcione. Otros módulos pueden tomar con seguridad las dependencias de los módulos que estarán siempre disponibles.

Ejemplos de módulos básicos son los feeds, navegación o enrutable.

# Módulos
La distribución por defector de Orchard viene con varios módulos preinstaldos como el de bloggin o el de páginas, pero también se pueden  incluir módulos de terceras partes.

Un módulo es un área ASP.NET MVC con un archivo  manifest.txt que extiende Orchard.

Un módulo contiene normalmente los controladores de eventos, tipos de contenido y sus plantillas de representación por omisión, así como algunas de interfaz de usuario admin.

Los módulos pueden ser dinámicamente compilado desde el código fuente cada vez que se realiza un cambio en su csproj o uno de los archivos que hace referencia csproj. Esto permite una "libreta" estilo de desarrollo que no requiere compilación explícita por parte del desarrollador o incluso el uso de un IDE como Visual Studio.

# Themes (temas)

Es un principio de diseño básico en que Orchard que todo el código HTML que se produce puede sustituible desde temas, incluido el marcado producida por los módulos. Los convenios definen qué archivos deben ir a donde en la jerarquía de archivos del tema.

El mecanismo de representación en todo Orchard se basa en las Shapes. El trabajo del motor del tema es encontrar el tema actual y teniendo en cuenta que el tema, determinar cuál es la mejor manera de hacer que cada Shape lo redenderice. Cada shape puede tener un procesamiento predeterminado que puede estar definido por un módulo como una plantilla en la carpeta de vistas o como un método en forma de código. Esa representación predeterminada puede ser anulado por el tema actual. El tema que al tener su propia versión de una plantilla para ese shape o su método propio  de esa shape.

Los temas pueden tener un padre, que permite a los hijos como especializaciones o adaptaciones de un tema principal. Orchard viene con un tema base llamada el Theme Machine que se ha diseñado para que sea fácil de usar como un tema principal.

Los temas pueden contener código en la mayor parte de los módulos de la misma manera hacer: pueden tener su propio csproj y beneficiarse de la compilación dinámica. Esto permite que los temas para definir los métodos de shape, pero también para exponer la interfaz de usuario de administración para la configuración que puedan tener.

La selección del tema actual se lleva a cabo por las clases que implementan IThemeSelector, que devuelven el nombre del tema y una prioridad para cualquier solicitud. Esto permite que muchos selectores contribuir a la elección del tema. Orchard viene con cuatro implementaciones de IThemeSelector:

- SiteThemeSelector selecciona el tema que está actualmente configurado para el tenant o en el sitio con una prioridad más baja.
- AdminThemeSelector toma el control y devuelve el tema de administración con una alta prioridad siempre que el URL es una URL de administración.
- PreviewThemeSelector anula el tema actual del sitio con el tema de ser previamente si el usuario actual es el que inició la previsualización de tema.
- SafeModeThemeSelector es el selector disponible sólo cuando la aplicación está en "modo seguro", lo que ocurre normalmente durante la instalación. Cuenta con una prioridad muy baja.

Un ejemplo de un selector tema podría ser uno que promueve un tema móvil cuando el agente de usuario es reconocido a pertenecer a un dispositivo móvil.
