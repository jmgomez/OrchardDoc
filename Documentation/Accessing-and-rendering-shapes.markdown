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

Puedes poner tus temas en el proyecto de acuerdo a las siguientes reglas:

El contenido de cada item en las plantillas de shapes deben estar en el directorio _views/items_
* Los temas de los shape `Parts_` deben estar en _views/parts_ .
* Los temas de los shape `Fields_` deben estar en  _views/fields_ .
* Las plantillas del shape `EditorTemplate` deben estar en _views/EditorTemplates/_`template`+nombre .
Por ejemplo, un `EditorTemplate` con el nombre de tema _Parts/Routable.RoutePart_  tiene su tema en 
_views/EditorTemplates/Parts/Routable.RoutePart.cshtml_.
* Por último, todos los demás temas de shapes deben estar en la carpeta _views_ .

> **Nota**`  `La extensión de temas puede ser cualquier extensión soportada por cualquier view engine activo, como son _.cshtml_,_.vbhtml_ o _.ascx_.

## Desde nombre de archivo de un Tema hacia el nombre de un Shape

Las reglas para mapear desde el nombre de archivo de un tema a su correspondiente nombre de shape son las siguientes:

* El Punto (.) y la barra invertida han de cambiarse por el guión bajo (_).
Ten en cuenta que esto no significa que un archivo _example.cshtml_ en un subdirectorio _myviews_ del directorio _Views
sea equivalente a un archivo _myviews_example.cshtml en la carpeta _Views_.
El tema de shape debería estar en el directorio esperado (consulta las reglas anteriores)
* El guión (-) cambia al doble guión bajo (\_\_)

Por ejemplo, _Views/Hello.World.cshtml_ será usado para renderizar un shape llamado `Hello_World`,
y _Views/Hello.World-85.cshtml_ será usado para renderizar un shape llamado `Hello_World__85`.

## Otra alternativa para renderizar Shapes

Como te habrás dado cuenta, un widget HTML en la zona `AsideSecond` (por ejemplo) podría renderizarse 
por un tema _widget.cshtml_, por un tema _widget-htmlwidget.cshtml_ o por otro _widget-asidesecond.cshtml_
si este existiera en el tema actual.
Cuando existen varias posibilidades para renderizar el mismo contenido,
esto hace referencia a las _alternates_ de un shape,
lo cual permite una gran variedad de nuevos escenarios para sobreescribir temas.

Las alternativas forman un grupo que corresponde al mismo shape si difieren sólo por un doble guión bajo al final.
Por ejemplo, `Hello_World`, `Hello_World__85`, y `Hello_World__DarkBlue` son un grupo de alternativas para 
un shape del tipo `Hello_World`. `Hello_World_Summary`, sin embargo, no pertenece a este grupo, y podría corresponder a un shape `Hello_World_Shape`, no a un shape `Hello_World`.

## ¿Qué alternativa debería ser renderizada?

Aunque no tenga alternativas, un shape siempre es creado con su nombre base, por ejemplo `Hello_World`.
Las anternativas ofrecen al desarrollador de temas opciones de nombres adicionales para sus temas más allá
del nombre por defecto como en el ejemplo: _hello.world.cshtml_.
El sistema escogerá entre las alternativas las plantillas más especializadas, de esta manera 
si _hello.world-orange.cshtml_ existe tendrá preferencia sobre _hello.world.cshtml_.

## Items de contenido de alternativas incorporados

La tabla superior muestra los nombres posibles de temas para los items de contenido.
Ahora debería estar claro que el nombre de un shape es construido a partir de `Content`
y el tipo para mostrar el shape (por ejemplo `Content_Summary`).

El sistema también añade automáticamente el tipo de contenido y el ID del contenido así como las alternativas
(por ejemplo `Content_Summary__Page` y `Content_Summary__42`).

Para más información sobre el uso de las alternativas, mira [Alternates](Alternates).

# Renderizar Shapes usando temas

A shape template is a fragment of markup that is used to render the shape.
The default view engine in Orchard is the Razor view engine.
Therefore, shape templates use Razor syntax by default.
For an introduction to Razor syntax, see [Template File Syntax Guide](Template-file-syntax-guide).
Un tema de sepa es un fragmento de código etiquetado que es usado para renderizar el shape.
El view engine por defecto en Orchard es Razor.
De la misma manera, los temas de shapes usan la sintaxis de Razor por defecto.
Para una introducción a la sintaxis de Razor echa un vistazo a [Template File Syntax Guide](Template-file-syntax-guide).

El siguiente ejemplo nos muestra a un tema para renderizar una part `Map` como una imagen.

    <img alt="Location" border="1" src="http://maps.google.com/maps/api/staticmap? 
         &zoom=14
         &size=256x256
         &maptype=satellite&markers=color:blue|@Model.Latitude,@Model.Longitude
         &sensor=false" />

Este ejemplo muestra un elemento `img` en el cual el atributo `src` contiene una URL
y un conjunto de parámetros pasados como valores querystring.
En este querystring, `@Model` representa al shape que se pasó junto al tema.
De la misma manera, `@Model.Latitude` es la propiedad  `Latitude` del shape, y
 `Latitude` es la propiedad `Longitude` del shape.

El próximo ejemplo muestra la plantilla para el editor.
Esta plantilla permite al usuario introducir los valores de la latidud y la longitud.
    
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


La expresión `@Html.LabelFor` crea elementos label usando el nombre de las propiedades del shape.
La expresión `@Html.TextBoxFor` crea elementos de caja de texto, los input type="text", donde los usuario introducen los valores para las propiedades del shape.
La expresión `@Html.ValidationMessageFor` crea mensajes formateados que se muestran cuando el usuario ha introducido valores incorrectos.

## Wrappers

Los Wrappers te permiten personalizar la renderización de un shape añadiendo etiquetado alrededor del shape.
Por ejemplo, _Document.cshtml_ es un wrapper para el shape `Layout`, porque este especifica
el código de marcado que envuelve al shape `Layout`.
Para más información sobre las relaciones entre `Document`y `Layout`,
echa un vistazo a [Guía de sintaxis del archivo de tema](Template-file-syntax-guide).

Normalmente, añades un archivo wrapper a la carpeta _Views_ de tu theme.
Por ejemplo, para añadir un wrapeer para `Widget`, añadirás un archivo _Widget.Wrapper.cshtml_
a la carpeta _Views_ de tu tema.
Si activas la funcionalidad **Shape Tracing**, verás los nombres disponibles para los wrapper de un shape.
Puedes también especificar un wrapper en el archivo _placement.info_.
Para más información de cómo especificar un wrapper,
mira [Comprendiendo el archivo placement.info](Understanding-placement-info).


# Creando un método de Shape

Otro modo de crear y renderizar un shape es crear un método que define y renderiza al mismo tiempo un shape.
El método debe marcarse con el atributo `Shape` (la clase `Orchard.DisplayManagement.ShapeAttribute`).
El método devuelve un objeto `IHtmlString` en lugar de usar un tema.
El objeto devuelvo contiene el etiquetado que renderiza el shape.

El siguiente ejemplo muestra el shape `DateTimeRelative`.
Este shape toma un valor `DateTime` en el pasado y devuelve un string que relaciona el valor con el tiempo actual. 
    
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
