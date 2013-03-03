Hay un montón de características disponibles para Orchard que están listas para usarse.
Y en muchas ocasiones más características todavía desde la [gallery][1].
Este artículo proporciona una breve descripción de cada característica desarrollada por Orchard.

Esta es una lista organizada alfabéticamente por el nombre de los módulos, en la que dos secciones
principales separan los módulos que forman parte del núcleo y los que no.
Las características principales no pueden ser desabilitadas y todas están siempre activadas,
pero hay características que están implementadas en módulos del núcleo que pueden activarse y desactivarse.
Estas características simplemente no están en la categoría "Core".

Cada módulo detalla sus características, y siempre del mismo modo si está disponible desde el paquete de WebPI, o solo desde la galería o a través de las release del código fuente.

# Módulos principales (Core modules)

## Common

Las tres partes principales del contenido Body,Common e Identity, así como también el campo de texto Text field, está implementadas en el módulo Common.

La parte Body representa un bloque de texto enriquecido. Este está configurado por defecto para usar HTML
a través de TinyMCE, pero puede ajustarse para usar texto plano, Markdown o cualquier otro tipo de texto personalizado.

La parte Common captura las fechas de creación, modificación y publicación, así como también el propietario del content item.
El contenedor del item (container item) también puede encontrarse en este part, de manera que en este part,
también podemos implementar las relaciones de jerarquía del tipo padre / hijo como pueden ser blog / entrada o carpeta / archivo.

La parte identidad también es útil para proporcionar una identidad única a un content item que no tiene
una de forma natural. Por ejemplo, las páginas son identificadas de forma natural por su ruta relativa dentro del sitio web, o por ejemplo
los usuarios son identificados por el nombre de usuario, sin embargo, los widgets no tienen una identidad natural como éstas.
Con el objetivo de permitir la importación y exportación de estos items, la part Identity puede ser añadida.
Ésta part crea GUIDs (números de identificación) que son usados como una identidad única que puede relacionar los intercambios de contenidos entre distintas instancias de Orchard.

El campo de texto (Text field) es muy similar a la part Body, excepto que es un campo en lugar de una part.
El Text field activa el uso de múltiples instancias de un mismo item individual. Tiene otros pequeños beneficios
como la caja de una línea individual o de texto sin formato.

## Containers

Este módulo introduce cuatro partes que son útiles para crear jerarquías de contenido sencillas.
Esto es la base para el módulo Orchard.Lists que ha llegado a ser algo así como las Listas obsoletas que han sido
desestimadas en favor de una clasificación de modelos en el contenido como las taxonomías o unos mejores
mecanismos de búsqueda como por ejemplo las proyecciones.

La part Container puede ser agregada al tipo para marcar su habilidad de contener ciertos tipos de content items.
Éste también tiene propiedades que especifican el concepto por el que pueden ordenarse y la paginación.

La part Containable especifica que un tipo puede ser contenido. Esta trabaja mano a mano con la part Container
y permiten que el editor de contenido pueda escoger un contenedor para el item que se quiere editar.

El Container Widget part es similar al Container part, excepto que está hecho para ser usado dentro del Container Widget,
además posee habilidades especiales para filtrar para ser capaz de mostrar sólo un subconjunto de items presentes en el Container.
No es solo un contenedor en sí mismo, sino que además referencia y re-usa un container item existente.

La part de propiedades personalizadas ofrece tres propiedades personalizadas de texto que son útiles para implementar el filtrar y ordenar.
Esta part está desfasada ya que las proyecciones ahora permiten filtrar y ordenar en campos (fields).

## Contents

Este módulo principal crea la infraestructura para content types personalizados (tipos de contenido personalizados).

### Features

* Contents (Core): la infraestructura para tipos personalizados (custom types).
* Content Control Wrapper (desactivado por defecto): añade un botón de editar en el front-end.

## Dashboard

Implementa el panel de administración como un caparazón extensible.

## Feeds

El módulo principal Feeds proporciona la infraestructura que otros módulos pueden usar para distribuir a través de enlaces RSS, Atom o cualuier otro tipo de feeds.

## Navigation

Desde Orchard 1.5, la aplicación se distribuye con un menú de navegación jerárquico y extensible.

Pueden llamarse más de una instancia de menú, que pueden construirse con este módulo. Los items del menú pueden ser proporcionados por cualquier número de proveedores. El módulo ofrece items personalizados que pueden apuntar a cualquier URL, y items de contenido de menú que apuntan a un item de contenido específico.

Otros módulos, como son los taxonomies o los projections, pueden proporcionar sus propios proveedores de items de menú dinámicos.

Los menús pueden pueden ser renderizados como unos widgets menú, o como unos breadcrum o menús locales.


Un content item puede ser añadido a los menús suando el Navigation part. El Menu part, que cubría satisfactoriamente esta función antes de Orchard 1.5, está desfasado pero todavía proporciona compatibilidad con versiones anteriores (back-compatibility) de los datos de Orchard.

Este módulo principal también proporciona el menú de administración.

## Reports

El módulo principal Reports (reportes) establece la infraestructura para generar y mostrar reportes básicos.
Es usado durante la primera configuración para hacer un log de varias operaciones de inicio, incluidas las operaciones de base de datos. 

## Scheduling

Las APIs para este módulo principal pueden ser usadas por otros módulos como el Publish Later (publica después) para planificar operaciones que serán ejecutadas en el futuro.

## Settings

La mayoría de las configuraciones del sitio en Orchar se almacenan en un content item.
El content type Site (tipo de contenido sitio) es el que es usado para esto, y como cualquier otro content type en Orchard, puede extenderse. Esto permite que los módulos puedan contribuir a su propia configuración. El módulo Settings es el que permite este escenario. 

## Shapes

Shapes en Orchard son unidades básicas de interfaz de usuario (UI) a partir de las cuales es construido todo el HTML. Es posible añadir nuevos shapes a través de módulos dinámicamente, pero el módulo Shapes ya proporciona algunas shapes básicas y estándard.

Las shapes principales (Core Shapes) están definidas a través de código en CoreShapes.cs. Otras shapes están definidas en vistas como archivos ".cshtml".
Cualquier shape, tranto principal como no, pueden ser sobreescritas por plantillas (templates) en un tema (theme).

### Core shapes (shapes principales)

* **ActionLink:** usa valores por ruta (route values) para construir un link HTML.
* **DisplayTemplate, EditorTemplate:** son usando internamente para constrir las vistas públicas (display view) y de edición de los content items.
* **Layout**: es el shape más exterior, que junto con su Wrapper y su Document, define la estructura básica de HTML para ser renderizada. También define las zonas de nivel superior.
* **List**: es un shape estandard para renderizar listas de shapes.
* **Menu, MenuItem, LocalMenu, LocalMenuItem y MenuItemLink:** son las sapes que renderiza la navegación.
* **Pager (paginador) y shapes asociados y alternates:** los shapes usados para renderizar la paginación.
* **Partial:** un shape que permite renderizar una plantilla como una vista parcial, usando un modelo específico. Crear un shape dinámico es normalmente la técnica más adecuada.
* **Resource, HeadScripts, FootScripts, Metas, HeadLinks, StyleSheetLinks, Style:** shapes usados para renderizar recursos como etiquetas de scripts u hojas de estilo.
* **Zone:** un shape especial que está hecho para contener otros shapes, normalmente contienen widgets pero no están limitados a ellos.

### Templated shapes

* **Breadcrumb**: una representación jerárquica de una lista de enlaces de items de menú.
* **ErrorPage and NotFound:** las páginas que son renderizadas en el caso de que ocurra un error de servidor o un error 404 de recurso no encontrado.
* **Header:** la cabecera visual de la página (no la etiqueta head del documento).
* **Message:** este shape es usado para rendereizar información, alertas o mensajes de error.
* **User:** muestra los enlaces de login, logout, panel de usuario y cambio de contraseña.

## Title (título)

Este módulo sencillo inserta el part Title que es usado por la mayoría de tipos de contenido (content types).

## XmlRpc

Este módulo implementa las APIs necesarias para crear contenido como páginas y entradas de blog para aplicaciones como Windows Live Writer.
El módulo Orchard.Blogs se construye sobre el módulo XmlRpc para permitir específicamente la creación de entradas de blog.

# Módulos secundarios

## Markdown (WebPI, desactivado por defecto)

[Markdown][2] es un formato de texto con una sintaxis fácil de leer por las personas (human-readable) usado para describir texto enriquecido sin la complejidad del HTML. Algunas personas prefieren escribir Markdown en lugar de utilizar un editor de texto WYSYWYG, como el editor de texto por defecto que viene con Orchard, TinyMCE.

Una vez has activado la característica Markdown, puedes crear nuevos content types o editar los existentes para usar el formato Markdown en lugar de HTML abriendo la configuración del part Body y en la configuración flavor cambiando de "html" a "markdown". El editor de Markdown se mostrará en lugar del TinyMCE en el editor del content item.

## Orchard.Alias (WebPI)

El módulo Alias establece la infraestructura para mapear direcciones amigables a content items y rutas personalizadas. Este módulo es la base sobre la que está construido Autoroute.

### Características

* **Alias:** esta es la pieza principal de la infraestructura para que aliases (pseudónimos) funcione.
* **Alias UI (desactivado por defecto):** proporciona la interfaz de usuario para modificar, crear y eliminar aliases.

## Orchard.AntiSpam (WebPI, desactivado por defecto)

El módulo AntiSpam proporciona varias funcionalidades para la cucha contra la publicidad, correo y mensajes no deseados (spam) y las piezas de la infraestructura. Hace posible prevenir el spam en contenido arbitrario ( las versiones previas de Orchard sólo ofrecían servicios anti-spam en los comentarios).

Con el módulo AntiSpam, puedes añadir un captcha, filtros anti-spam externos o establecer límites de aceptación de contenido añadiendo algunos part a tus tipos, incluyendo formularios personalizados.

### Características

* **Anti-Spam:** las piezas principales de la infraestructura anti-spam. También proporciona la parte [ReCaptcha][3] que puede ser añadida a content types (tipos de contenido) para añadir CAPTCHA a su formulario de edición.
* **Akismet Anti-Spam Filter:** permite usar el servicio de terceros [Akismet][4] con los content types de Orchard.
* **TypePad Anti-Spam Filter:** permite usar el servicio de terceros [TypePad][5] con los content types de Orchard.

## Orchard.ArchiveLater (Archiva después)

Usando el part proporcionado por este módulo, puedes programar que un content item sea archivado.

Este módulo está disponible desde los paquetes de código fuente o [desde la galería][28].

## Orchard.Autoroute (WebPI)

Autoroute is built on top of the Alias feature.

Esta poderosa funcionalidad hace posible que los creadores de tipos de contenido (content type) puedan especificar una arquitectura de enlaces basada en tokens.

Por ejemplo, si quieres que una URL de tu entrada de blog sea de la forma posts/2012/7/el-mejor-post-que-has-leido-nunca puedes ir al editor del content type para entradas de blog, desarrollar la configuración para el part Autorute y establecer el patrón a "posts/{Content.Date.Format:yyyy}/{Content.Date.Format:MM}/{Content.Slug}".

Autoroute está construido sobre la característica Alias.

## Orchard.Blogs (WebPI)

El módulo Blogs proporciona las características y funciones de blog para Orchard. Este módulo está fuertemente relacionado con la composición de content type y otras características como los Comments.

### Ver también

* [Añade un blog a tu sitio web][6]
* [Crea entradas de blog con Windows Live Writer][7]

## Orchard.CodeGeneration

Este módulo proporciona a los desarrolladores comandos de scaffolding que ayudan a la creación de nuevos módulos y temas.

El módulo CodeGeneration está disponible desde los paquetes de código fuente o [desde la galería][29].

## Orchard.Comments (WebPI)

Puedes usar el part Comments proporcionado por este módulo o cualquier otro content type para permitir a los usuarios de tu sitio web opinar y comentar el contenido.

### See Also

* [Moderar comentarios][8]

## Orchard.ContentPermissions (WebPI, off by default)

Without this module, Orchard only provides configurable permissions for whole content types.
This module provides a part that can be added to any content type to restrict viewing
permissions per content item instead of per content type.

## Orchard.ContentPicker

This module provides an extensible content item picker that can be used to build
relationships between content items.

This module is available from source code packages or [from the gallery][30].

## Orchard.ContentTypes (WebPI, off by default)

Enable this module to enable the creation and modification of content types from the admin UI.

### See Also

* [Creating custom content types][9]

## Orchard.CustomForms (WebPI, off by default)

Custom forms are built as content types, typically using fields. Once you've built the content
type for your custom form, you can enable its instances to be created from the front-end by
anonymous users.

This is useful, for example, to create contact forms: enable the feature, create a "Contact Form"
content type, add name, e-mail and message text fields (select TextArea as the display option in the
field's settings), click on "Forms" in the admin menu, click "Add a new  Custom Form", 
select "Contact Us" as the content type for the form and publish. If you enables the Rules feature,
you can then create a rule that sends an e-mail when an item of the "Contact Us" type is
published. You should also grant the "Submit Contact Form" permission to the anonymous role
from the Users/Roles admin screen under "Custom Forms" in order to allow anonymous users to post contact forms.

## Orchard.DesignerTools

This module contains a few features that help with the development of themes.

### Features

* Shape Tracing: provides a Firebug-like tool that can be used to explore the server-side shape structure of the page, generate alternates, and inspect the model, placement and templates for any shape.
* URL Alternates: adds alternates for all shapes based on the current URL, of the form "someshape-url-thecurrenturl" or "someshape-url-homepage".
* Widget Alternates: adds alternates for specific widgets and layers.

### See Also:

* [Customizing Orchard using designer helper tools][31]

This module is available from source code packages or [from the gallery][32].

## Orchard.Email

This module implements an email messaging channel that can be used for example to send
email notifications from rules.

This module is available from source code packages or [from the gallery][38].

## Orchard.Fields (WebPI)

Orchard.Fields provides Input, Boolean, DateTime, Numeric, Link, Enumeration, and Media Picker fields
that can be used in custom content types.

## Orchard.Forms (WebPI)

This developer-targeted module provides shapes that are useful to dynamically build forms from code.

This module is used as a dependency by Projector and Rules.

## Orchard.ImportExport

The definition for content types, as well as the content itself, can be exported from one Orchard
instance, and imported into another using this module. The format that is used for the transfer
is the same XML format that is used in recipes.

This module is available from source code packages or [from the gallery][33].

## Orchard.Indexing, Orchard.Search and Lucene

Those three modules constitute the default full-text search infrastructure for Orchard.
The indexing module populates the index from content items. The Lucene module provides
the specific index implementation that indexing populates and that search queries.
The search index queries the index and formats results.

This module is available from source code packages or from the gallery: [search][34],
[indexing][35] and [Lucene][36].

## Orchard.jQuery (WebPI)

Used as a dependency by other modules, this provides jQuery and jQueryUI scripts.

## Orchard.Lists (deprecated)

This module provides a simple implementation for lists of content items, following
a folder/file metaphor where a content item can belong to only one list.

This module is deprecated and we recommend users switch to Taxonomies and Projections,
which enable much richer scenarios.

## Orchard.Localization

This module provides a part that can be added to a content type to make it localizable.
The items of the modified types can have several versions that differ by culture.

This module is available from source code packages or [from the gallery][37].

## Orchard.Media (WebPI)

Orchard.Media manages the contents of the /media folder.

### See Also

* [Adding and managing media content][10]

## Orchard.MediaPicker (WebPI)

MediaPicker adds the media picker to the content body editor.

## Orchard.Messaging

This module provides the messaging infrastructure that module such as Rules can
use. It is only useful as a dependency for other modules that implement
specific channels, such as e-mail or Twitter.

This module is available from source code packages or [from the gallery][39].

## Orchard.Modules (WebPI)

This is the module that provides the admin UI to enable and disable features.

### See Also

* [Installing and upgrading modules][11]

## Orchard.MultiTenancy

Hosting multiple Orchard sites on separate applications means duplicating everything
for each site. The multi-tenancy module enables the hosting of multiple Orchard sites
within a single IIS application, thus saving a lot of resources, and reducing maintenance
costs. Each site's data is strictly segregated from the others through a table prefix
or complete database separation.

### See Also

* [Setting up a multi-tenant Orchard site][40]

This module is available from source code packages or [from the gallery][41].

## Orchard.Packaging (WebPI)

This module handles the packaging of themes and modules.

### Features

* Packaging commands: core services and command-line commands to package and install modules.
* Gallery: integrates the [gallery][1] into Orchard.
* Package Updates: enables module updates from the admin dashboard.

### See Also

* [Installing modules and themes from the gallery][12]

## Orchard.Pages (WebPI)

The Pages modules adds the Page content type, and associated commands.

### See Also

* [Adding pages to your site][13]

## Orchard.Projections (WebPI)

This tremendously useful module enables the creation of arbitrary queries over
the contents of the site, and then to present the results in flexible layouts,
without leaving the admin dashboard.

### See Also

* [Presentation video on Projections][14]

## Orchard.PublishLater (WebPI, off by default)

The PublishLater part can be added to draftable content types and allows scheduled
publication of contents.

### See Also

* [Saving, scheduling and publishing drafts][15]

## Orchard.Recipes (WebPI)

Recipes are XML files that describe a set of operations on the contents and configuration
of the site. Recipes are used at setup to describe predefined initial configurations
(Orchard comes with default, blog and core recipes). They can also be included with
modules to specify additional operations that get executed after installation.
Finally, the import/export feature uses this same recipe format to transfer contents.

### See Also

* [Making a web site recipe][16]

## Orchard.Roles (WebPI)

Security APIs in Orchard do not make many presuppositions about authentication, membership
and permissions, but we do ship role-based security as a default security implementation.
Users can belong to one or many groups, and permissions are granted to groups rather than
users.

### See Also

* [Managing users and roles][17]
* [Understanding permissions][18]

## Orchard.Rules (WebPI, off by default)

Orchard events can be picked up by rules and trigger actions. For example, the publication
event on the comment content type can be picked-up by a user-defined rule and trigger
the action of sending an e-mail to the owner of the blog.
The Rules module provides simple admin UI to create and manage rules.

## Orchard.Scripting (WebPI)

In order to enable simple programmability of the application without requiring the
development of a whole module, certain key areas of Orchard expose extensibility through
scripting. For example, widget layer visibility is defined by rules that are written
as simple script expressions. The scripting infrastructure is language-agnostic, and
new languages could be added by a module. Orchard comes with one implementation that
is a simple expression language whose syntax is a subset of Ruby.

### Features

* Scripting: the scripting infrastructure.
* Lightweight Scripting: a simple expression language that is a subset of Ruby.
* Scripting Rules: makes it possible for rules to be triggered by arbitrary scripted expressions.

## Orchard.Scripting.DLR

This module, built on Orchard.Scripting, enables the possibility to use DLR languages
such as Ruby and Python as scripting languages.

## Orchard.Setup (WebPI, off after setup)

This module is always disabled except before the application has been setup. It is responsible
for implementing the setup mechanism. It contains the original recipes in its Recipes subfolder.

### See Also

* [Installing Orchard][20]
* [Making a web site recipe][16]

## Orchard.Tags (WebPI)

Tags are a very simple way to categorize contents. It is a flat and easily extensible structure.
For more elaborate classifications, we strongly recommend the use of the [Contrib.Taxonomies][21]
module.

### See Also

* [Organizing content with tags][22]

## Orchard.TaskLease

In web farm environments, it's often useful to send messages across all servers in the farm. This
module implements a way for code to communicate tasks to the whole server farm.

This module is available from source code packages or [from the gallery][42].

## Orchard.Themes (WebPI)

This module provides the infrastructure for easy customization of the look and feel of the site
through the definition of themes, which are a set of scripts, stylesheets and template overrides.

### See Also

* [Installing themes][23]
* [Previewing and applying a theme][24]
* [Customizing the default theme][25]

## Orchard.Tokens (WebPI, off by default)

Tokens are contextual environment variables that are used in dynamic expressions. For example,
the Autoroute feature makes it possible to define URL patterns for content items of a given
type. Those patterns rely on tokens that will be dynamically evaluated in a specific context.
The "{Content.Date.Format:yyyy}/{Content.Slug}" would be evaluated for the specific content item
it applies to and would be resolved to something like "2012/the-title".

## Orchard.Users (WebPI)

This is the module that implements the default user management in Orchard.

### See Also

* [Managing users and roles][17]

## Orchard.Warmup (WebPI, off by default)

Cold starts in ASP.NET applications can be slow, and shared hosting environments create the
conditions for frequent such cold starts. In order to mitigate this situation, the warmup
feature can prepare static versions of the most common pages of the site so those can be served
as soon as possible even if the application is not entirely done warming up.

## Orchard.Widgets (WebPI)

Widgets are reusable pieces of UI that can be positioned on any page of the site. Which widgets
get displayed on what pages is determined by layer rules.

### Features

* Widgets: the core widget feature and admin UI.
* Page Layer Hinting: adds a message when publishing a new page that prompts the user to create a new layer for that page.
* Widget Control Wrapper: Adds an edit button on the front-end for easier modification.

### See Also

* [Managing widgets][26]
* [Writing a widget][27]

## TinyMCE (WebPI)

Used as a dependency by other features, this provides the scripts necessary to implement the TinyMCE WYSYWYG HTML editor.

## UpgradeTo15 / UpgradeTo14 (WebPI, off by default)

Orchard 1.4 brought breaking changes in the way URLs and titles are managed. 1.3 and previous versions
were using the Route part to handle a static URL and title. 1.4 deprecated this in favor of the new
alias, autoroute and title part. The upgrade module contains special scripts that upgrade old content
to the new way of doing things.
Orchard 1.4 also introduced new field types (see Orchard.Fields), and because some users may have
used equivalent Contrib.* fields from gallery modules, the upgrade module provides an upgrade path
to the new fields.

  [1]: http://gallery.orchardproject.net
  [2]: http://daringfireball.net/projects/markdown/
  [3]: http://www.google.com/recaptcha
  [4]: http://akismet.com/
  [5]: http://antispam.typepad.com/
  [6]: http://docs.orchardproject.net/Documentation/Adding-a-Blog-to-Your-Site
  [7]: http://docs.orchardproject.net/Documentation/Blogging-with-LiveWriter
  [8]: http://docs.orchardproject.net/Documentation/Moderating-comments
  [9]: http://docs.orchardproject.net/Documentation/Creating-custom-content-types
  [10]: http://docs.orchardproject.net/Documentation/Adding-and-managing-media-content
  [11]: http://docs.orchardproject.net/Documentation/Installing-and-upgrading-modules
  [12]: http://docs.orchardproject.net/Documentation/Installing-modules-and-themes-from-the-gallery
  [13]: http://docs.orchardproject.net/Documentation/Adding-Pages-to-Your-Site
  [14]: http://www.youtube.com/watch?feature=player_embedded&v=SP912eWoOoo
  [15]: http://docs.orchardproject.net/Documentation/Saving-scheduling-and-publishing-drafts
  [16]: http://docs.orchardproject.net/Documentation/Making-a-Web-Site-Recipe
  [17]: http://docs.orchardproject.net/Documentation/Managing-users-and-roles
  [18]: http://docs.orchardproject.net/Documentation/Understanding-permissions
  [20]: http://docs.orchardproject.net/Documentation/Installing-Orchard
  [21]: http://gallery.orchardproject.net/List/Modules/Orchard.Module.Contrib.Taxonomies
  [22]: http://docs.orchardproject.net/Documentation/Organizing-content-with-tags
  [23]: http://docs.orchardproject.net/Documentation/Installing-themes
  [24]: http://docs.orchardproject.net/Documentation/Previewing-and-applying-a-theme
  [25]: http://docs.orchardproject.net/Documentation/Customizing-the-default-theme
  [26]: http://docs.orchardproject.net/Documentation/Managing-widgets
  [27]: http://docs.orchardproject.net/Documentation/Writing-a-widget
  [28]: http://gallery.orchardproject.net/List/Modules/Orchard.Module.Orchard.ArchiveLater
  [29]: http://gallery.orchardproject.net/List/Modules/Orchard.Module.Orchard.CodeGeneration
  [30]: http://gallery.orchardproject.net
  [31]: http://docs.orchardproject.net/Documentation/Customizing-Orchard-using-Designer-Helper-Tools
  [32]: http://gallery.orchardproject.net/List/Modules/Orchard.Module.Orchard.DesignerTools
  [33]: http://gallery.orchardproject.net/List/Modules/Orchard.Module.Orchard.ImportExport
  [34]: http://gallery.orchardproject.net/List/Modules/Orchard.Module.Orchard.Search
  [35]: http://gallery.orchardproject.net/List/Modules/Orchard.Module.Orchard.Indexing
  [36]: http://gallery.orchardproject.net/List/Modules/Orchard.Module.Lucene
  [37]: http://gallery.orchardproject.net/List/Modules/Orchard.Module.Orchard.Localization
  [38]: http://gallery.orchardproject.net/List/Modules/Orchard.Module.Orchard.Email
  [39]: http://gallery.orchardproject.net/List/Modules/Orchard.Module.Orchard.Messaging
  [40]: http://docs.orchardproject.net/Documentation/Setting-up-a-multi-tenant-Orchard-site
  [41]: http://gallery.orchardproject.net/List/Modules/Orchard.Module.Orchard.MultiTenancy
  [42]: http://gallery.orchardproject.net/List/Modules/Orchard.Module.Orchard.TaskLease
