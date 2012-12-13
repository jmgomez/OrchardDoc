
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

Content items are instances of content types. Said differently, content types are classes of content items. We said in the previous section that examples of content items are pages, blog posts, and products. Those three examples also describe three content types: page, blog post and product. In other words what we call a blog post is just an item of type blog post.
Los Content items son instancias de content types. Dicho de otra manera, content types son las clases de los content items. Hemos dicho en la sección anterior que ejemplos de content items son páginas, entradas de blog y productos. Estos tres ejemplso también describen tres content types: page, blog post y product. Es decir, a lo que llamamos entrada de blog es solo un item del tipo blog post.

## Content Part

En Orchard, los content types están construidos a partir de pequeñas partes, llamadas content parts. Las content parts son átomos de contenido que son suficientes para proporcionar un comportamiento específico y coherente y que pueden reusarse con otros content types.

![A blog post is made from parts.](../Attachments/Basic-Orchard-Concepts/ContentParts.PNG)

Por ejemplo, los comentarios, etiquetas y valoraciones son content parts porque definen comportamientos específicos y pueden reusarse por cualquier content type. No hay nada en el content part comentarios que lo hagan específico a un content type concreto como por ejemplo las entradas de blog. Los comentarios pueden ser igualmente útiles en entradas de blog como pueden serlo en las páginas del sitio o los productos.

There can be only one of each part on any given content type.
Sólo puede haber una tipología de content part en cada conten type dado. Por ejemplo, no pueden haber dos content part de la clase commments(comentarios) en el content type blog post.

## Content Field

Content fields are pieces of information that can be added to a content type. Content fields have a name and a type and are specific to a content type. There can be several of each field type on any given content type.

For example, a Product content type can have a text field representing its Sku, a numeric field representing its price, and another numeric field representing its weight. Each of these fields probably only makes sense on a product.

> Note: it would be possible to create a product part with three properties that would be roughly equivalent to this set of fields. This would have the advantage of making it possible to transform any content type into a product. Each approach is a valid choice and Orchard allows for both.

## Module

The various possible custom possible extensions that can be built for Orchard are typically built as modules. A module is a set of extensions for Orchard that are grouped under a single sub-folder of the Modules directory that can be found under the Orchard web site.

Optional modules for Orchard can be found in the Orchard Gallery (see the menu entry on top of this page).

![The Module management screen](../Attachments/Basic-Orchard-Concepts/Modules.PNG)

## Feature

A module can contain one or more features, which is a logical grouping of functionality that can be enabled or disabled individually. For example, a custom authentication module could have separate features for OpenID, FaceBook, LiveID, Twitter or Google authentication that can each be turned on or off.

Features can depend on each other, whether they are in the same module or not.

![The feature management screen](../Attachments/Basic-Orchard-Concepts/Features.PNG)

## Manifest

A manifest is a small text file that describes a module or a theme to the system.

Here is an example of a manifest:

    
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


# UI composition

Orchard manages content that is composed from parts. It needs a mechanism that orchestrates the display while taking into account the composite nature of the content. This is why we talk about UI composition, as elementary bits and pieces of content need to be composed into a harmonious and consistent whole. Several concepts contribute to this UI composition.

## Theme

When designing a web site, it is important to be able to modify the visual look of every single aspect of the site. Orchard provides a clean separation between the content management and the visual rendering of the content.

A theme is a packaged look and feel for an Orchard site. It can consist of any combination of style sheets, images, layouts, templates and even custom code. It is even possible to create a theme that inherits from another, which is very useful if you are trying to make only small modifications on an existing theme.

![The same site can be displayed differently by switching themes.](../Attachments/Basic-Orchard-Concepts/ThemeComparison.png)

## Layout

A layout is a file in a theme that defines the general organization of the pages of the site that use it. A layout typically defines a set of zones where contents or widgets can be inserted.

![The layout for the theme machine, with its various collapsible zones](../Attachments/Anatomy-of-a-theme/TheThemeMachineZoneScreenshot.PNG)

## Template

Each content part, each field and each widget will need to be graphically represented in the front-end, transforming the data that it represents into a form that can be read by the users of the site. A template is the recipe that formats that data and transforms it into HTML for the browser to display. You can think of a template as plain HTML with well-defined "holes" where data gets inserted.

Here is an example of a template that displays the title from the Route part:

    
    <h1>@Model.Title</h1>


## Shape

Before displaying something using a template, that something gets transformed into a shape, which is a very malleable object that contains all the information required in order to display it. Before getting rendered by templates, everything gets mapped into a tree of shapes that is a sort of abstract representation of the contents of the final page. The advantage of such trees of shapes is that any module can modify existing shapes or create new ones.

The layout, zones, widgets and content parts all get represented as shapes as part of the rendering process.

One could imagine for example a Gravatar module that would add avatar icon shapes to the comment shapes that were created by the comment module. In the same way, the layers from the widget module are adding widget shapes to the zone shapes of the layout shape.

## Placement

When rendering the collections of parts and fields -or any other shapes- that compose a page or content item, Orchard needs to know in what order to do so. Placement.info files are XML files that describe rules that can be used to determine what shapes go into what zones and in what order. This enables not only the rendering of each shape to be customized, but also the order in which they get rendered.

Here is an example of a placement file:

    
    <Placement>
        <Place Parts_Map="Content:10"/>
        <Place Parts_Map_Edit="Content:7.5"/>
    </Placement>


## Zone

Zones are specific parts of a layout that can be customized by inserting widgets. In some themes, zones are collapsible, which means that they disappear if they contain no active widget.

## Widget

A widget is a small fragment of UI that can be added to some or all pages of the site. Examples of widgets are tag clouds, maps, archives, a search form, or recent blog posts.

![A few widgets](../Attachments/Basic-Orchard-Concepts/Widget.PNG)

## Layer

A layer is a group of widgets (with their specific configuration, which includes their positioning -zone name and ordering-) that is activated by a specific rule.

For example, the TheHomePage layer is activated by a rule that specifically selects the home page. The Default layer is always active no matter what page is displayed. The Authenticated layer is only active when users have identified themselves.

When more than one layer is active on any given page, all the widgets from all those layers get displayed at the same time. Orchard orders them based on their position string.

# Security

## Users and roles

In Orchard, users can be attributed roles, which can be seen as stereotypes of users. Permissions can then be attributed to roles in order to define who can do what on the site (more on this in the next section). Any user can have one or several roles.

Site owners can create their own roles but Orchard comes with built-in roles that should cover most sites' requirements:

* Administrator: have full control over the site's settings and contents.
* Editor: does not create content but edit and publish content created by authors.
* Moderator: validates user-created contents such as comments.
* Author: writes and publishes his own content.
* Contributor: writes content but does not necessarily have the rights to publish it.
* Anonymous: an unknown user, someone who hasn't logged in.
* Authenticated: any user who has logged in.

Neither Anonymous nor Authenticated can be assigned to a user manually. Rather, they are determined dynamically at runtime.

## Privileges and Permissions

All users don't have the same rights and privileges in Orchard: the site owner can choose who can create content, who can write or validate comments, etc. Rights and privileges are represented as permissions. In Orchard, permissions are granted to roles but are not explicitly denied. In other words if a user belongs to any role that has a given permission, he has that permission. To revoke a permission, you need to either remove a user with the role the permission has been granted to or you need to remove that permission for the whole role.

Some permissions are "effectively granted". This means that they have not been explicitly granted, but that they have been implied by another permission. For example, if you grant the site owner permission, you are implicitly granting all the other permissions.

![](../Attachments/Basic-Orchard-Concepts/Permissions.PNG)

Permissions, as well as their default settings for the built-in roles, are defined by modules. This means that if you build your own module, you can define specific permissions to accompany it.

## Site owner

The site owner, sometimes also called "super user" is a special user that is defined at setup time and that has all the rights on the site. It can be changed from the settings admin screen if you have the permission to do so.

There is a permission called "Site Owners Permission" that grants the same right and that is granted by default to only members of the Administrator role. We advise never to grant that permission to any other role.

# Development

In this section we will describe concepts that are only required for module developers.

## ASP.NET MVC

ASP.NET MVC is the Web framework that Orchard is built on.

## Handler

A handler is similar to an MVC filter in that it contains code that will execute for specific events of the request life-cycle. They are typically used to set-up data repositories or to do additional operations when something gets loaded.

## Driver

Drivers are similar to MVC controllers, but they act at the level of a content part instead of at the level of the full request. They typically prepare shapes for rendering and handle post-backs from admin editors.

## Record

A record is a class that models the database representation of a content part. They are POCOs where each property must be virtual.

## Model

What plays the part of the model for a content part is the part class itself. Some parts also define view models, in the form of strongly-typed classes or of more flexible dynamic shapes.

## Migration

A migration is a description of the operations to execute when first installing a feature or when upgrading it from a version to the next. This enables smooth upgrades of individual features without data loss. Orchard includes a data migration framework.

## Injection
Inversion of Control, or injection, is widely used in Orchard. When any piece of code requires a dependency, it will typically demand the injection of one or several instances of a specific interface. The framework will take care of selecting, instantiating and injecting the right implementations at runtime.

