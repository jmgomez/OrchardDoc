Un _shape_ (silueta o figura) es un modelo dinámico de datos
Pues pensar en los shapes como en contenedores de datos que son manipulados por plantillas para el renderizado.
La función de un shape es reemplazar los modelos de vista estáticos de ASP.NET MVC usando
un modelo que puede actualizarse en tiempo de ejecución&#151;esto es, usando un shape dinámico.

Este artículo introduce el concepto de shapes y explica cómo trabajar con él.
Está pensado para desarrolladores de módulos y temas (theme) quienes poseen, al menos, un conocimiento básico de módulos de Orchard.
Para información sobre crear módulos, echa un vistazo a [Construyendo un módulo Hola Mundo](Building-a-hello-world-module).
Para información sobre objetos dinámicos, mira [Creando y usando objetos dinámicos, fuente MSDN Library](http://msdn.microsoft.com/en-us/library/ee461504.aspx).

> **Nota**`
`A lo largo del artículo se usa indistintamente las palabras plantilla y tema para referirse al mismo concepto en inglés theme y nunca al concepto pattern.

# Introducción a Shapes

Los shapes son modelos de datos dinámicos que usan plantillas de shape para hacer los datos visibles para su uso de la forma que quieras.
Las plantillas de shapes son fragmentos de etiquetado (markup) para renderizar shapes.
Algunos ejemplos de shapes incluyen menús, items de menú, items de contenido, documentos y mensajes.

Un shape es un objeto modelo de datos que deriva de la clase `Orchard.DisplayManagement.Shapes.Shape`.
La clase `Shape` nunca es instanciada. En su lugar, los shapes son creados en tiempo de ejecución por un shape factory.
La shape factory por defecto es `Orchard.DisplayManagement.Implementation.DefaultShapeFactory`.
Los shapes creados por la shape factory son objetos dinámicos

> **Nota**`
`Los objetos dinámicos son nuevos de .NET Framework 4.
Como objeto dinámico, un shape expone sus miembros en tiempo de ejecución en lugar de tiempo de compilación.
Esto contrasta con el objeto model de ASP.NET MVC que es un objeto estático definido en tiempo de compilación.

La información sobre el shape se encuentra en la propiedad `ShapeMetadata` del propio shape.
Esta información inlucye el tipo de shape, la forma en que será mostrado, la posición, el prefix, los wrappers, alternates
el contenido de objetos hijo (child content) y un valor boleano `WasExecuted`.

Puedes acceder a los metadatos de un shape como se muestra en el siguiente ejemplo:
    
    var shapeType = shapeName.Metadata.Type;

Después de que el objeto shape haya sido creado, el shape es renderizado con ayuda de una plantilla de shape (shape template).
Una plantilla de shape es una porción de etiquetado HTML (vista parcial) que es responsable de mostrar el shape.
Otra posibilidad es usar un atributo (`Orchard.DisplayManagement.ShapeAttribute`)
que te permite escribir código que crea y muestra el shape sin usar una plantilla.

# Creando Shapes

Para los desarrolladores de módulos, la necesidad más común a la hora de desarrollar shapes es cómo transportar los datos desde un driver a una plantilla para renderizarlo.
Un driver deriva de la clase `Orchard.ContentManagement.Drivers.ContentPartDriver` y normalmente sobreescribe (overrides) los métodos `Display`y `Editor` de esa clase.
Los métodos `Display` y `Editor` devuelven un objeto `ContentShapeResult`, el cual es análogo al objeto `ActionResult` devuelvo por los action method de ASP.NET MVC.
El método `ContentShape` te ayuda a crear el shape y devolverlo dentro de un objeto `ContentShapeResult`.

Aunque el método `ContentShape` está sobrecargado, el uso más común es pasarle dos
parámetros&#151;el tipo de shape y una expresión con una función dinámica que define el shape.
El tipo de shape nombra el sape y une el shape a la plantilla que debe usarse para renderizarlo.
Las premisas para decidir los nombres (naming conventions or naming guidelines) son discutidas más adelante en "Decidiendo los nombres de Shapes y Plantillas"(Accessing-and-rendering-shapes#NamingShapesandTemplates).

La expresión de función puede describirse mejor si usamos un ejemplo.
El siguiente ejemplo muestra el modo `Display` de un driver que devuelve un resultado shape (shape result),
el cual será usado para mostrar una parte `Map`.

    protected override DriverResult Display(
        MapPart part, string displayType, dynamic shapeHelper)
    {
    return ContentShape("Parts_Map",
                         () => shapeHelper.Parts_Map(
                               Longitude: part.Longitude, 
                               Latitude: part.Latitude));
    }

La expresión usa un objeto dinámico(`shapeHelper`) para definir un shape `Parts_Map`y sus atributos.
La expresión añade la propiedad `Longitude` al shape y establece su contenido igual a la propiedad `Latitude` del part.
La expresión también añade una propietad `Latitude` con el mismo valor que la propiedad `Latitude` del part.
El método `ContentShape`crea un objeto resultado que es devuelto por el método `Display`.

El próximo ejemplo muestra una clase driver completa que envía un shape result a una plantilla tanto para 
ser mostrada como editada en una parte `Map`. El método `Display`es usado para mostrar el mapa.
El métido `Editor`marcado con "GET" es usado para mostrar el shape result en una vista de edición para recoger las entradas de datos del usuario.
El métido `Editor`marcado con "POST" es usado para actualizar la vista de edición usando los vatos proporcionados por el usuario.
Estos métodos usan distintas sobrecargas del método `Editor`.


    
    using Maps.Models;
    using Orchard.ContentManagement;
    using Orchard.ContentManagement.Drivers;
    
    namespace Maps.Drivers
    {
        public class MapPartDriver : ContentPartDriver<MapPart>
        {
            protected override DriverResult Display(
                MapPart part, string displayType, dynamic shapeHelper)
            {
                return ContentShape("Parts_Map",
                                    () => shapeHelper.Parts_Map(
                                          Longitude: part.Longitude, 
                                          Latitude: part.Latitude));
            }
    
            //GET
            protected override DriverResult Editor(
                MapPart part, dynamic shapeHelper)
            {
                return ContentShape("Parts_Map_Edit",
                                    () => shapeHelper.EditorTemplate(
                                          TemplateName: "Parts/Map", 
                                          Model: part));
            }
    
            //POST
            protected override DriverResult Editor(
                MapPart part, IUpdateModel updater, dynamic shapeHelper)
            {
                updater.TryUpdateModel(part, Prefix, null, null);
                return Editor(part, shapeHelper);
            }
        }
    }

El método `Editor`marcado "GET" usa el método `ContentShape` para crear un shape para una plantilla de edición.
En este caso, el nombre del tipo es `Parts_Map_Edit`  y el objeto `shapeHelper` crea un shape del tipo `EditorTemplate`.
Este es un shape especial que tiene una propiedad `TemplateName` y una propiedad `Model`.
La propiedad `TemplateName` almacena una ruta parcial a la plantilla.
Esta vez `"Parts/Map"` hace que Orchard busque una plantilla dentro de tu módulo que la siguiente ruta:

_Views/EditorTemplates/Parts/Map.cshtml_

La propiedad `Model` toma el nombre del archivo del modelo part, pero quitando la extensión de archivo.

# Decidiendo los nombres de shapes y de los temas (themes)

Como habrás notado, el nombre dado a un tipo shape une el shape a la plantilla que será usada para renderizarlo.
Por ejemplo, supongamos que creas un part llamado `Map`y que este muestra un mapa para unas longitud y latitud dadas.
El nombre del tipo sape debería ser `Parts_Map. Por convención, todos los shape part empiezan con `Parts_`seguidos del nombre del part (en este caso `Map`). Proporcionando este nombre (`Parts_Map`), Orchard busca el tema en tu módulo a través de la siguiente ruta:

_views/parts/Map.cshtml_

La siguiente tabla resume las convenciones usadas para llamar a los tipo shape y los temas.

Aplicado a             | Convención de nombrado de Shape                                          | Tipo Shape Example                                   | Tema Example
---------------------- | ----------------------------------------------------------------- | ---------------------------------------------------- | ------------------------------------------
Content shapes         | Content\_\_\[ContentType\]                                        | Content\_\_BlogPost                                  | Content-BlogPost
Content shapes         | Content\_\_\[Id\]                                                 | Content\_\_42                                        | Content-42
Content shapes         | Content\_\_\[DisplayType\]                                        | Content\_\_Summary                                   | Content.Summary
Content shapes         | Content\_\[DisplayType\]\_\_\[ContentType\]                       | Content\_Summary\_\_BlogPost                         | Content-BlogPost.Summary
Content shapes         | Content\_\[DisplayType\]\_\_\[Id\]                                | Content\_Summary\_\_42                               | Content-42.Summary
Content.Edit shapes    | Content\_Edit\_\_\[DisplayType\]                                  | Content\_Edit\_\_Page                                | Content-Page.Edit
Content Part templates | \[ShapeType\]\_\_\[Id\]                                           | Parts\_Common\_Metadata\_\_42                        | Parts/Common.Metadata-42
Content Part templates | \[ShapeType\]\_\_\[ContentType\]                                  | Parts\_Common\_Metadata\_\_BlogPost                  | Parts/Common.Metadata-BlogPost
Field templates        | \[ShapeType\]\_\_\[FieldName\]                                    | Fields\_Common\_Text\_\_Teaser                       | Fields/Common.Text-Teaser
Field templates        | \[ShapeType\]\_\_\[PartName\]                                     | Fields\_Common\_Text\_\_TeaserPart                   | Fileds/Common.Text-TeaserPart
Field templates        | \[ShapeType\]\_\_\[ContentType\]\_\_\[PartName\]                  | Fields\_Common\_Text\_\_Blog\_\_TeaserPart           | Fields/Common.Text-Blog-TeaserPart
Field templates        | \[ShapeType\]\_\_\[PartName\]\_\_\[FieldName\]                    | Fields\_Common\_Text\_\_TeaserPart\_\_Teaser         | Fields/Common.Text-TeaserPart-Teaser
Field templates        | \[ShapeType\]\_\_\[ContentType\]\_\_\[FieldName\]                 | Fields\_Common\_Text\_\_Blog\_\_Teaser               | Fields/Common.Text-Blog-Teaser
Field templates        | \[ShapeType\]\_\_\[ContentType\]\_\_\[PartName\]\_\_\[FieldName\] | Fields\_Common\_Text\_\_Blog\_\_TeaserPart\_\_Teaser | Fields/Common.Text-Blog-TeaserPart-Teaser
LocalMenu              | LocalMenu\_\_\[MenuName\]                                         | LocalMenu\_\_main                                    | LocalMenu-main
LocalMenuItem          | LocalMenuItem\_\_\[MenuName\]                                     | LocalMenuItem\_\_main                                | LocalMenuItem-main
Menu                   | Menu\_\_\[MenuName\]                                              | Menu\_\_main                                         | Menu-main
MenuItem               | MenuItem\_\_\[MenuName\]                                          | MenuItem\_\_main                                     | MenuItem-main
Resource               | Resource\_\_\[FileName\]                                          | Resource\_\_flower\.gif                              | Resource-flower.gif
Style                  | Style\_\_\[FileName\]                                             | Style\_\_site\.css                                   | Style-site.css
Widget                 | Widget\_\_\[ContentType\]                                         | Widget\_\_HtmlWidget                                 | Widget-HtmlWidget
Widget                 | Widget\_\_\[ZoneName\]                                            | Widget\_\_AsideSecond                                | Widget-AsideSecond
Zone                   | Zone\_\_\[ZoneName\]                                              | Zone\_\_AsideSecond                                  | Zone-AsideSecond

You should put your templates in the project according to the following rules:

* Content item shape templates are in the _views/items_ folder.
* `Parts_` shape templates are in the _views/parts_ folder.
* `Fields_` shape templates are in the _views/fields_ folder.
* The `EditorTemplate` shape templates are in the _views/EditorTemplates/_`template` name folder.  
For example, an `EditorTemplate` with a template name of _Parts/Routable.RoutePart_ has its template
at _views/EditorTemplates/Parts/Routable.RoutePart.cshtml_.
* All other shape templates are in the _views_ folder.

> **Note**`  `The template extension can be any extension supported by an active view engine, such as _.cshtml_, _.vbhtml_, or _.ascx_.

## From Template File Name to Shape Name

More generally, the rules to map from a template file name to the corresponding shape name are the following:

* Dot (.) and backslash (\) change to underscore (_).
Note that this does not mean that an _example.cshtml_ file in a _myviews_ subdirectory of _Views_
is equivalent to a _myviews_example.chtml_ file in _Views_.
The shape templates must still be in the expected directory (see above).
* Hyphen (-) changes to a double underscore (\_\_).

For example, _Views/Hello.World.cshtml_ will be used to render a shape named `Hello_World`,
and _Views/Hello.World-85.cshtml_ will be used to render a shape named `Hello_World__85`.

## Alternate Shape Rendering

As noted, an HTML widget in the `AsideSecond` zone (for example) could be rendered
by a _widget.cshtml_ template, by a _widget-htmlwidget.cshtml_ template,
or by a _widget-asidesecond.cshtml_ if they exist in the current theme.
When various possibilities exist to render the same content,
these are referred to as _alternates_ of the shape,
and they enable rich template overriding scenarios.

Alternates form a group that corresponds to the same shape if they differ only by a double-underscore suffix.
For example, `Hello_World`, `Hello_World__85`, and `Hello_World__DarkBlue` are an alternate group
for a `Hello_World` shape. `Hello_World_Summary`, conversely, does not belong to that group
and would correspond to a `Hello_World_Shape` shape, not to a `Hello_World` shape.
(Notice the difference between "\_\_" and "\_".)

## Which Alternate Will Be Rendered?

Even if it has alternates, a shape is always created with the base name, such as `Hello_World`.
Alternates give additional template name options to the theme developer beyond the default
(such as _hello.world.cshtml_).
The system will choose the most specialized template available among the alternates,
so _hello.world-orange.cshtml_ will be preferred to _hello.world.cshtml_ if it exists.

## Built-In Content Item Alternates

The table above shows possible template names for content items.
It should now be clear that the shape name is built from `Content`
and the display type (for example `Content_Summary`).

The system also automatically adds the content type and the content ID as alternates
(for example `Content_Summary__Page` and `Content_Summary__42`).

For more information about how to use alternates, see [Alternates](Alternates).

# Rendering Shapes Using Templates

A shape template is a fragment of markup that is used to render the shape.
The default view engine in Orchard is the Razor view engine.
Therefore, shape templates use Razor syntax by default.
For an introduction to Razor syntax, see [Template File Syntax Guide](Template-file-syntax-guide).

The following example shows a template for displaying a `Map` part as an image. 

    <img alt="Location" border="1" src="http://maps.google.com/maps/api/staticmap? 
         &zoom=14
         &size=256x256
         &maptype=satellite&markers=color:blue|@Model.Latitude,@Model.Longitude
         &sensor=false" />

This example shows an `img` element in which the `src` attribute contains a URL
and a set of parameters passed as query-string values.
In this query string, `@Model` represents the shape that was passed into the template.
Therefore, `@Model.Latitude` is the `Latitude` property of the shape,
and `@Model.Longitude` is the `Longitude` property of the shape.

The following example shows the template for the editor.
This template enables the user to enter values for the latitude and longitude.
    
    @model Maps.Models.MapPart

    <fieldset>
        <legend>Map Fields</legend>
                
        <div class="editor-label">
            @Html.LabelFor(model => model.Longitude)
        </div>
        <div class="editor-field">
            @Html.TextBoxFor(model => model.Latitude)
            @Html.ValidationMessageFor(model => model.Latitude)
        </div>
                
        <div class="editor-label">
            @Html.LabelFor(model => model.Longitude)
        </div>
        <div class="editor-field">
            @Html.TextBoxFor(model => model.Longitude)
            @Html.ValidationMessageFor(model => model.Longitude)
        </div>
    </fieldset>


The `@Html.LabelFor` expressions create labels using the name of the shape properties.
The `@Html.TextBoxFor` expressions create text boxes where users enter values for the shape properties.
The `@Html.ValidationMessageFor` expressions create messages that are displayed if users enter an invalid value.

## Wrappers

Wrappers let you customize the rendering of a shape by adding markup around the shape.
For example, _Document.cshtml_ is a wrapper for the `Layout` shape, because it specifies
the markup code that  surrounds the `Layout` shape.
For more information about the relationship between `Document` and `Layout`,
see [Template File Syntax Guide](Template-file-syntax-guide).

Typically, you add a wrapper file to the _Views_ folder of  your theme.
For example, to add a wrapper for `Widget`, you add a _Widget.Wrapper.cshtml_ file to
the _Views_ folder of your theme.
If you enable the **Shape Tracing** feature, you'll see the available wrapper names for a shape.
You can also specify a wrapper in the _placement.info_ file.
For more information about how to specify  a wrapper,
see [Understanding the placement.info File](Understanding-placement-info).

# Creating a Shape Method

Another way to create and render a shape is to create a method that both defines and renders the shape.
The method must be marked with the `Shape` attribute (the `Orchard.DisplayManagement.ShapeAttribute` class).
The method returns an `IHtmlString` object instead of using a template;
the returned object contains the markup that renders the shape. 

The following example shows the `DateTimeRelative` shape.
This shape takes a `DateTime` value in the past and returns a string that relates the value to the current time.
    
    public class DateTimeShapes : IDependency {
        private readonly IClock _clock;
    
        public DateTimeShapes(IClock clock) {
            _clock = clock;
            T = NullLocalizer.Instance;
        }
    
        public Localizer T { get; set; }
    
        [Shape]
        public IHtmlString DateTimeRelative(HtmlHelper Html, DateTime dateTimeUtc) {
            var time = _clock.UtcNow - dateTimeUtc;
    
            if (time.TotalDays > 7)
                return Html.DateTime(dateTimeUtc, T("'on' MMM d yyyy 'at' h:mm tt"));
            if (time.TotalHours > 24)
                return T.Plural("1 day ago", "{0} days ago", time.Days);
            if (time.TotalMinutes > 60)
                return T.Plural("1 hour ago", "{0} hours ago", time.Hours);
            if (time.TotalSeconds > 60)
                return T.Plural("1 minute ago", "{0} minutes ago", time.Minutes);
            if (time.TotalSeconds > 10)
                return T.Plural("1 second ago", "{0} seconds ago", time.Seconds);
    
            return T("a moment ago");
        }
    }
