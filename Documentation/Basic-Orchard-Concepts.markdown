
Orchard es un sistema de gestión de contenidos basado en (Web CMS). Orchard tiene como principal objetivo ayudarte a construir sitios web a partir de piezas existentes. Estas piezas vienen en diferentes tamaños y formas que es necesario entender bien si pretendes llegar a ser productivo con ellas. En este artículo profundizaremos en estas piezas y explicaremos sus nombres y comportamientos.

# Definiciones generales de CMS

## Contenido

La 'C' en CMS significa "content" en inglés, obviamente contenido en español. De modo que parece razonable decir que el contenido es todo lo que CMS gestiona. Con más exactitud, el contenido es cualquier elemento de un sitio web que tiene información en sí mismo. Por ejemplo, una entrada de un blog, un comentario, un producto, también el menú de navegación o el logotipo de tu compañía. Todos ellos son piezas de contenido individuales y perfectamente identificables. Si estás pensando en este momento, que contenido es todo lo que hay en un sitio web, estás en lo cierto. Si estabas pensando que esto es algo difuso y poco claro, podrías estar en lo cierto, pero afortunadamente vamos a detallar las peculiaridades y diferencias entre los diferentes tipos de contenido en las próximas secciones.

## Panel de administración, dashboard o back-end

El panel de administración es el apartado donde administras el sitio web y su contenido. El acceso y el uso a este está restringido, de modo que sólo los usuarios que tienen el permiso "Access admin panel" pueden tuilizarlo. El panel de administración es la 'M' en CMS.

![](../Attachments/Basic-Orchard-Concepts/Adminpanel.PNG)

## CMS

Y la 'S' en CMS significa "System" en inglés, sistema en español. Usar la palabra sistema no es tan difuso y sin sentido como parece. Es muy importante que un CMS gestione el contenido de una manera sistemática: esto significa que todo el contenido es administrado de forma homogénea, lo que permite compartir recursos.

For example, you can manage blog posts, pages and products using common tools, and all of those can get comments, ratings or tagging from common modules. This gives you a more consistent experience and facilitates the creation of new types of content.

Por ejemplo, puedes gestionar entradas en el blog, páginas y productos usando herramientas comunes para ambos, y todos ellos pueden disponer de comentarios, valoraciones o etiquetado gracias a módulos comunes a todos.

## Front-end

El front-end, en español también conocido como página pública en contraste con el panel de administración que es de acceso restringido, es el apartado del sitio web sin restricciones de acceso y que acepta tanto usuarios anónimos como usuarios registrados. En otras palabras, es la cara pública de tu sitio web, todo excepto el panel de administración.

![](../Attachments/Basic-Orchard-Concepts/FrontEnd.PNG)

## Setup

Setup es el proceso que debes seguir para que tu sitio web empiece a funcionar (y nada más, todo el resto del trabajo se debe hacer después, como por ejemplo crear el contenido).

![](../Attachments/Basic-Orchard-Concepts/Setup.PNG)

# Conceptos de Orchar

## Content Item

Un content item, es una pequeña pieza de contenido, con frecuencia a sociada a una única dirección URL del sitio web. Algunos ejemplos de content items son páginas, entradas del blog o productos. 

## Content type (tipo de contenido)

Los Content items son instancias de content types. Dicho de otra manera, content types son las clases de los content items. Hemos dicho en la sección anterior que ejemplos de content items son páginas, entradas de blog y productos. Estos tres ejemplso también describen tres content types: page, blog post y product. Es decir, a lo que llamamos entrada de blog es solo un item del tipo blog post.

## Content Part

En Orchard, los content types están construidos a partir de pequeñas partes, llamadas content parts. Las content parts son átomos de contenido que son suficientes para proporcionar un comportamiento específico y coherente y que pueden reusarse con otros content types.

![A blog post is made from parts.](../Attachments/Basic-Orchard-Concepts/ContentParts.PNG)

Por ejemplo, los comentarios, etiquetas y valoraciones son content parts porque definen comportamientos específicos y pueden reusarse por cualquier content type. No hay nada en el content part comentarios que lo hagan específico a un content type concreto como por ejemplo las entradas de blog. Los comentarios pueden ser igualmente útiles en entradas de blog como pueden serlo en las páginas del sitio o los productos.

Sólo puede haber una tipología de content part en cada conten type dado. Por ejemplo, no pueden haber dos content part de la clase commments(comentarios) en el content type blog post.

## Content Field (contenido campo)

Content fields are pieces of information that can be added to a content type. Content fields have a name and a type and are specific to a content type. There can be several of each field type on any given content type.

Los Content fields son porciones de información que pueden ser añadidas a un content type. Los Content fields tienen un nombre y tipo que le son específicos para un content type.

Por ejemplo, un content type Producto puede tener un campo de texto representando su código de referencia (en inglés SKU), un campo numérico conteniendo su precio y otro campo numérico representando su peso. Cada uno de estos campos es posible que solo tengan sentido en el ámbito de un producto.

> Nota: Es posible crear un producto part con tres propiedades que podrían ser a groso modo equivalentes a este conjunto de campos mencionado. Esto tiene la ventaja de hacer posible que se pueda transformar cualquier tipo de contenido (content type) en un producto. Cualquier punto de vista elegido es una opción válida y Orchard las permite.

## Module (Módulo)

Las posibles extensiones personalizadas que pueden construirse para Orchard, son construidas normalmente como módulos (modules). Un módulo es un conjunto de extensiones para Orchard que son agrupadas bajo una única sub-carpeta en el directorio Modules, que puede encontrarse en cualquier website desarrollado sobre Orchard.

Los módulos opcionales para Orchard pueden encontrarse en Orchard Gallery, una galería de módulos disponibles para Orchard (echa un vistazo al enlace en el menú superior de esta página).

![The Module management screen](../Attachments/Basic-Orchard-Concepts/Modules.PNG)

## Feature (Característica)

Un módulo puede contener una o más características, las cuales son una agrupación lógica de funcionalidades que pueden ser activadas o desactivadas individualmente. Por ejemplo, un módulo de autenticación personalizado puede tener características separadas para OpenID, FaceBook, LiveID, Twitter o Google las cuales pueden encenderse o apagarse.

Las características pueden depender unas de otras, aunque estén o no estén en un mismo módulo.

![The feature management screen](../Attachments/Basic-Orchard-Concepts/Features.PNG)

## Manifest (Manifiesto)

Un manifiesto es un archivo de texto de pequeño tamaño que escribe un módulo o tema para el sistema.

Este es un ejemplo de manifiesto:

    
    Name: Comments
    AntiForgery: enabled
    Author: The Orchard Team
    Website: http://orchardproject.net
    Version: 0.9.0
    OrchardVersion: 0.9.0
    Description: The comments system implemented by this module can be applied to arbitrary Orchard content types, such as blogs and pages. It includes comment validation and spam protection through the Akismet service.
    Features:
        Orchard.Comments:
            Name: Comments
            Description: Standard content item comments.
            Dependencies: Settings
            Category: Social


# UI composition (composición de interfaz de usuario)

Orchard administra contenido que está compuesto por partes. Esto necesita de un mecanismo para orquestrar la forma de mostrarle cuando se tiene en cuenta la naturaleza de la composición del contenido. Esta es la razón por la que hablamos de UI composition, como unos bits y piezas elementales de contenido necesarias para ser compuestas dentro de un todo armonioso y consistente. Varios conceptos contribuyen a la UI composition.

## Theme (Tema)

Al diseñar un sitio web, es importante poder modificar el aspecto visual de cada detalle del sitio. Orchard ofrece una separación bien clara entre la administración del contenido y el renderizado visual del contenido.

Un tema (theme) es un diseño empaqueado para un sitio de Orchard. Consiste en una combinación de hojas de estilo, imágenes, maqueta o estructura, plantillas y también código personalizado. También es posible crear un tema que hereda de otro tema, lo cual es muy útil si pretendes hacer pequeñas modificaciones a un tema existente.

![El mismo sitio puede mostrarse diferente cambiando entre temas](../Attachments/Basic-Orchard-Concepts/ThemeComparison.png)

## Layout (maqueta, distribución, estructura)

A layout is a file in a theme that defines the general organization of the pages of the site that use it. A layout typically defines a set of zones where contents or widgets can be inserted.
Una maqueta es un archivo que forma parte de un theme y que define la distribución general del contenido de las páginas que la usan. Una maqueta normalmente define una serie de zonas donde se coloca el contenido o los widgets.

![The layout for the theme machine, with its various collapsible zones](../Attachments/Anatomy-of-a-theme/TheThemeMachineZoneScreenshot.PNG)

## Template (plantilla)

Each content part, each field and each widget will need to be graphically represented in the front-end, transforming the data that it represents into a form that can be read by the users of the site. A template is the recipe that formats that data and transforms it into HTML for the browser to display. You can think of a template as plain HTML with well-defined "holes" where data gets inserted.

Cada content part, cada field y cada widget debe estar representado graficamente en el front-end (web pública), transformando la información a la que representa de forma que pueda ser leída e interpretada por los usuarios del sitio web. Una plantilla es una receta (recipe en inglés) que formatea los datos y los transforma en HTML para que los navegadores puedan mostrarlo.
Puedes pensar en las plantillas como texto plano en HTML con "huecos" bien definidos en los que los datos son encajados.

Aquí tienes un pequeño ejemplo de una plantilla que muestra el título de un Route part:

    
    <h1>@Model.Title</h1>


## Shape (Figura)

Antes de mostrar algo usando una plantilla, ese algo debe ser transformado en un shape, el cual es un objeto maleable que contiene toda la información requerida para mostrarlo. Antes de ser renderizado por las plantillas, toda la información es mapeada en un árbol de shapes que es una especie de representación abstracta del contenido que aparecerá finalmente en la página. La ventaja de estos árboles de shapes es que cada módulo puede modificar shapes existentes o crear otros nuevos.

La maqueta, las zonas, los widgets y los content parts todos ellos son representados como shapes y al mismo tiempo como parte del proceso de renderizado.

Uno puede imaginar por ejemplo un módulo de Gravatar que podría añadir un shape avatar al shape comentario que ha sido creado por el módulo comentarios. De la misma manera, las capas de un módulo widget son shapes widget añadidas a un shape zona que forma parte de un shape maqueta (layout shape).

## Placement (disposición o colocación)

Cuando se renderiza una colección de parts y fields -o cualquier otro shape- que compone una página o un content item, Orchard necesita conocer en qué  orden hacerlo. Los archivos Placement.info son archivos XML que describen las reglas que deben usarse para definir qué shapes van dentro de qué zonas y en qué orden. Esto no permite solo renderizar cada shape que debe ser personalizado, si no que también el orden en el que van a ser renderizados.

Este es un ejemplo del archivo Placement.info:

    
    <Placement>
        <Place Parts_Map="Content:10"/>
        <Place Parts_Map_Edit="Content:7.5"/>
    </Placement>


## Zone (Zona)

Las zonas son partes específicas de la maqueta que pueden personalizarse insertando widgets en ellas. En algunos temas, las zonas pueden desplegarse y plegarse, lo cual significa que pueden desaparecer si no contienen ningún widget activo.

## Widget

Un widget es un pequeño fragmente de interfaz de usuario (en inglés User Interfaze, UI), que puede agregarse a alguna o todas las páginas del sitio web. Ejemplos de widgets son la nube de palabras, los mapas, listado de entradas de blog, un formulario de búsqueda o el listado de las últimas entradas del blog.

![Algunos widgets](../Attachments/Basic-Orchard-Concepts/Widget.PNG)

## Layer (Capa)

Una capa es un grupo de widgets (con su configuración específica que incluye su posición: el nombre de la zona y el orden) que es activado por una regla concreta.

Por ejemplo, la capa TheHomePage es activada por una regla que que selecciona específicamente la página de inicio. La capa por defecto está siempre activa sin importar qué página se muestra en el navegador. La capa Autenticado (Authenticated) sólo está activa cuando los usuarios se han loggeado correctamente.

Cuando más de una capa está activa en una página dada, todos los widgets de esas capas son mostrados al mismo tiempo. Orchard los ordena según su cadena de posición.

# Seguridad

## Usuarios y roles

En Orchard, los usuarios pueden ser roles atribuídos, los cuales pueden ser vistos como estereotipos de usuarios. Los permisos pueden ser atribuidos a los roles para definir quién puede hacer qué en el sitio web (se profundizará más sobre esto en la próxima sección). Cualquier usuario pueden tener uno o más roles.

Los propietarios de un sitio web pueden crear sus propios roles, pero Orchard proporciona de inicio un conjunto de roles que deberían cubrir las necesidades de la mayoría de sitios web:

* Administrator: tiene control absoluto sobre la configuración del sitio y el contenido.
* Editor: no crea contenido pero edita y publica el contenido creado por los autores.
* Moderator: valida el contenido creado por los usuarios, por ejemplo los comentarios.
* Author: escribe y publica su propio contenido.
* Contributor: escribe contenido y no necesariamente tiene derechos de publicación sobre éstos.
* Anonymous: es un usuario anónimo que no tiene por qué estar registrado en la página.
* Authenticated: cualquier usuario que se ha identificado con usuario y contraseña en el sitio web.

Ni Anonymous ni Authenticated pueden ser asignados a un usuario manualmente. Por el contrario, son asignados dinámicamente en tiempo de ejecución.

## Privilegios y permisos

No todos los usuarios tienen los mismos derechos y privilegios en Orchard: el propietario del sitio web puede elegir quien crea contenido, quien lo escribe o quien valida comentarios, etc. Los derechos y privilegios son representados como permisos. En Orchard, los permisos son asignados a roles pero no se especifica la denegación de permisos. Es decir, is un usuario tiene asignado un role y este role tiene un permiso dado, el usuario tiene ese permiso. Para revocar un permiso, necesitas eliminar al usuario del listado de usuarios con ese permiso o eliminar el permiso del role que tiene el usuario.

Algunos permisos son "efectivamente garantizados". Esto quiere decir que esos permisos no han sido garantizados explícitamente, pero sin embargo esos permisos están implícitos en otro permiso. Por ejemplo, si garantizas el permiso de propietario web, estás garantizando de forma implícita todos los demás permisos existentes.

![](../Attachments/Basic-Orchard-Concepts/Permissions.PNG)

Permissions, as well as their default settings for the built-in roles, are defined by modules. This means that if you build your own module, you can define specific permissions to accompany it.

## Site owner (propietario del sitio web)

El propietario del sitio web, en algunas ocasiones llamado súper usuario (super user en inglés) es un usuario especial que se define al configurar el sitio por primera vez y que tiene todos los derechos del sitio web. Puede cambiarse desde el panel de administración en el apartado de configuración si se tiene permisos para hacerlo.

Hay un permiso llamado "Site Owners Permission" (Permiso de propietario del sitio) que garantiza el mismo derecho que si hubiera sido asignado con la configuración inicial. Aconsejamos no garantizar nunca este permiso a ningún otro role distinto a administrador.


# Desarrollo

En esta sección vamos a describir los conceptos que solo son requeridos para desarrolladores de módulos.

## ASP.NET MVC

ASP.NET MVC es la plataforma web sobre la que está construido Orchard.

## Handler

Un handler es similar a un filtro MVC que contiene el código que ejecutará eventos específicos durante el ciclo de vida de una petición. Los handler son usados normalmente para establecer repositorios de datos o realizar operaciones adicionales al cargar algo.

## Driver

Los drivers son similares a los controles MVC, pero actúan al nivel del content part en lugar de al nivel de toda la petición. Los drivers normalmente preparan shapes para renderizarlas y administran post-backs desde los editores del panel de administración.

## Record

Un record es una clase que modela la representación de una base de datos de un content part. Son objetos POCO en los que cada propiedad debe ser virtual.

## Model

Un modelo es una clase de content part. Algunas parts también definen modelos de vista (view models), en la forma de clases fuertemente tipadas o shapes dinámicas más flexibles.

## Migration

La migración es una descripción de operaciones para ejecutar en la primera instalación de una característica o al actualizar la característica de una versión a otra. Esto activa actualizaciones suaves (smooth upgrades) de características individuales sin perder la información. Orchard incluye una plataforma para migración de datos.

## Injection (Inyección de dependencias)
Inversion of Control (IoC), o inyeccion es ampliamente utilizado en Orchard. Cuando una porción de códico requiere una depenfencia, normalmente pedirá que le sea inyectada una o varias instancias de una interfaz eespecífica. La plataforma se preocupará de seleccionar, instanciar e inyectar las implementaciones correctas en tiempo de ejecución.
