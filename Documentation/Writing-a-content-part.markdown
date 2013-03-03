Un ContentPart (parte o Trozo de Contenido) es una pieza de funcionalidad reutilizable o interfaz de usuario (o ambos) que se puede añadir a cualquier ContentType (tipo de Contenido) en Orchard. Algunos ejemplos de ContentParts son, el RoutePart (para permitir que un ContentItem a sea direccionable con una URL desde el front-end), el TagsPart (para aplicación de palabras clave / etiquetas a un elemento de contenido), y la MenuPart (para permitir que un ContentItem que se pueda añadir en el menú principal).

En este tutorial se explica el proceso de creación de un ContentPart nuevo desde cero, utilizando la función SScaffolding en Orchard como una herramienta de productividad. Aunque este tutorial supone un desarrollo en Visual Studio, no es estrictamente necesario disponer de Visual Studio para desarrollar una ContentPart - siéntase libre de utilizar su editor preferido.

En este tutorial, vamos a construir un MapPart personalizado, que puede ser configurado con los valores de latitud y longitud con el fin de mostrar un mapa de un ContentItem en particular.


> **Importante:** Antes de que generes la estructura de ficheros para tu modulo, necesitarás descargar, instalar y activar la función de **Code Generation**. Para más información, vea [Command-line Code Generation](Command-line-scaffolding).

Vamos a añadir un nuevo módulo "Maps" que contiene nuestra aplicación MapPart, como un nuevo proyecto en la solución de Orchard. Suponiendo que usted ha alistado el árbol de fuentes Orchard, inicie Visual Studio 2010 y abra el archivo Orchard.sln que se encuentra bajo el "src" de la carpeta raíz.

![](../Upload/screenshots/vs_soln_explorer.png)

Escriba en la línea de comandos de Orchard "codegen module Maps /IncludeInSolution:true".  El parámetro "IncludeInSolution" dice a Orchard que debe referenciarlo en el archivo Orchard.sln.

    
    orchard> codegen module Maps /IncludeInSolution:true
    Creating module Maps
    Module Maps created successfully



Después de ejecutar este comando, Visual Studio preguntará para volver a cargar el archivo de la solución, dígale que sí.

![](../Upload/screenshots_85/vs_soln_reload.png)

El módulo de Maps aparecerá añadido a la solución, junto con algunos archivos y carpetas predeterminadas para poder iniciar la escritura del módulo.

![](../Upload/screenshots/vs_soln_maps_module.png)

Abra el archivo Module.txt en la raíz del proyecto Maps. Este archivo define la información acerca de su módulo, como un nombre, descripción, versión, autor, y una categoría de características expuestas por el módulo. El archivo Module.txt también puede contener información adicional, como las dependencias, que no vamos a cubrir aquí. Nuestro módulo es muy simple, y sólo contiene una única funcionalidad "Maps" que no tiene dependencias adicionales. Edite el archivo Module.txt como se indica a continuación.

    
    Name: Maps
    AntiForgery: enabled
    Author: The Orchard Team
    Website: http://orchardproject.net
    Version: 1.0.0
    OrchardVersion: 1.0.0
    Description:  Adds a map image to content items, based on longitude and latitude.
    Features:
        Maps:
            Description: Adds a map image to content items, based on longitude and latitude.
            Category: Geolocation


Ahora vamos a empezar a escribir la parte del mapa. Para empezar, necesitamos una clase para contener los datos para el ContentPart. Las clases de datos convencionalmente se añaden a los "modelos" de la carpeta del proyecto. Haga clic en la carpeta Models en Visual Studio y seleccione "Agregar clase>" en el menú contextual y de nombre a su nuevo archivo,  Map.cs:

![](../Upload/screenshots_675/vs_add_model_class.png)


En Orchard, los datos de un ContentPart son representados por una Record class, que representa que campos son almacenados en una tabla de una base da datos, y una clase ContentPart que usa el Record para alamacenarlo. Añade el MapRecord (ContentPartRecord) a la clase MapPart (ContentPart) como sigue:

    
    using System.ComponentModel.DataAnnotations;
    using Orchard.ContentManagement;
    using Orchard.ContentManagement.Records;
    
    namespace Maps.Models {
        public class MapRecord : ContentPartRecord {
            public virtual double Latitude { get; set; }
            public virtual double Longitude { get; set; }
        }
    
        public class MapPart : ContentPart<MapRecord> {
            [Required]
            public double Latitude
            {
                get { return Record.Latitude; }
                set { Record.Latitude = value; }
            }
    
            [Required]
            public double Longitude
            {
                get { return Record.Longitude; }
                set { Record.Longitude = value; }
            }
        }
    }

Ahora construye el proyecto para asegurarte de que tu clase Record compila sin errores.

![](../Upload/screenshots/vs_build_maps.png)

A continuación, vamos a crear una migración de datos para nuestro módulo Maps. ¿Por qué necesitamos una clase de migración? La razón es que la definición de un registro y clase de elementos para almacenar los datos en realidad no afectar a la base de datos de ninguna manera. La migración de datos es lo que dice Orchard cómo actualizar el esquema de base de datos cuando la función de mapas está activada (la migración se ejecuta cuando la función está activada). La migración también puede actualizar el esquema de base de datos de versiones anteriores de un módulo para el esquema que requiere una versión más reciente de un módulo - este es un tema avanzado que no se tratará en este tutorial.

Para crear una clase de migración de datos nueva, puede utilizar la función de generación de código de Orchard.  Ejecute  "codegen datamigration Maps" desde la línea de comandos de Orchard.

    
    orchard> codegen datamigration Maps
    Creating Data Migration for Maps
    Data migration created successfully in Module Maps


Visual Studio le solicitará cargar la solución de nuevo. Después de aceptar la solicitud, la nueva clase de migración de datos aparecerá en el proyecto.

![](../Attachments/Writing-a-content-part/vs_sol_migration.PNG)

La clase de migración añadida vía la consola de generación de código contiene un solo método Create(), que define una tabla de la estructura de la base de datos, basada en las clases Record del proyecto. Debido a que sólo tenemos una clase MapRecord con dos propiedades, latitud y longitud, la clase de migración es bastante simple. Note que el método Create es llamado cuando la característica es activida, y la base de datos se actulizará en consecuencia.


    
    using System;
    using System.Collections.Generic;
    using System.Data;
    using Maps.Models;
    using Orchard.ContentManagement.Drivers;
    using Orchard.ContentManagement.MetaData;
    using Orchard.ContentManagement.MetaData.Builders;
    using Orchard.Core.Contents.Extensions;
    using Orchard.Data.Migration;
    
    namespace Maps.DataMigrations {
        public class Migrations : DataMigrationImpl {
    
            public int Create() {
        		// Creating table MapRecord
    			SchemaBuilder.CreateTable("MapRecord", table => table
    				.ContentPartRecord()
    				.Column("Latitude", DbType.Double)
    				.Column("Longitude", DbType.Double)
    			);
    
                ContentDefinitionManager.AlterPartDefinition(
                    typeof(MapPart).Name, cfg => cfg.Attachable());
    
                return 1;
            }
        }
    }


Añade la línea de AlterPartDefinition para que la migración sea acoplable a cualquier ContentType. También debemos escribir 'using Maps.Models;' al principio del fichero.

Ahora añadiremos el Handler al MapPart. Un handler en Orchard es una clase que define un compartamiento para una parte, manejando los eventos o manipulando el modelo de datos antes de redenderizar el ContentPart. El MapPart es muy simple, en este caso, nuestra clase handler sólo especifica un IRepository para el MapRecord que debe usarse para el almacenamiento. Añade el siguiente código a Handlers/MapHandelr.cs

    
    using Maps.Models;
    using Orchard.ContentManagement.Handlers;
    using Orchard.Data;
    
    namespace Maps.Handlers {
        public class MapHandler : ContentHandler {
            public MapHandler(IRepository<MapRecord> repository) {
                Filters.Add(StorageFilter.For(repository));
            }
        }
    }


También añadadiremos un driver para nuesto MapPart. Un driver en Orchard es una clase que define cómo se mostrar un shape para los distintos contextos en los que actua nuestro part. Por ejemplo, cuando estamos mostrando el Map en el front-end, el método "Display" define el nombre de la plantilla a usar para los diferentes displayTypes (por ejemplo, "detalles" o "resumen"). Similarmente, el método "Editor" del driver define la plantilla a usar para mostrar el editor del MapPart (para introducir valores en la latidud o longitud). Vamos a mantener esta parte simple y sólo usar "Map" para el nombre del shape que vamos a usar en los contextos de mostrar y editar (y en todos los displayTypes). Añade la clase MapDriver como sigue:



    
    using Maps.Models;
    using Orchard.ContentManagement;
    using Orchard.ContentManagement.Drivers;
    
    namespace Maps.Drivers {
        public class MapDriver : ContentPartDriver<MapPart> {
            protected override DriverResult Display(
                MapPart part, string displayType, dynamic shapeHelper) {
    
                return ContentShape("Parts_Map", () => shapeHelper.Parts_Map(
                    Longitude: part.Longitude,
                    Latitude: part.Latitude));
            }
    
            //GET
            protected override DriverResult Editor(
                MapPart part, dynamic shapeHelper) {
    
                return ContentShape("Parts_Map_Edit",
                    () => shapeHelper.EditorTemplate(
                        TemplateName: "Parts/Map",
                        Model: part,
                        Prefix: Prefix));
            }
            //POST
            protected override DriverResult Editor(
                MapPart part, IUpdateModel updater, dynamic shapeHelper) {
    
                updater.TryUpdateModel(part, Prefix, null, null);
                return Editor(part, shapeHelper);
            }
        }
    }


Ahora podemos añadir las vistas para mostrar y editar. Primero añadimos una carpeta "Parts" y otra "EditorTemplates/Parts" a la carpeta "Views" en el proyecto Maps, luego añadimos un fichero Masp.cshtml a ambas carpetas,  Views/EditorTemplates/Parts y Views/Parts y escrimos el siguiente código:

    
    @model Maps.Models.MapPart
    
    <fieldset>
      <legend>Map Fields</legend>
                
      <div class="editor-label">
        @Html.LabelFor(model => model.Latitude)
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

And:
    
    <img alt="Location" border="1" src="http://maps.google.com/maps/api/staticmap? 
         &zoom=14
         &size=256x256
         &maptype=roadmap
         &markers=color:blue|@Model.Latitude,@Model.Longitude
         &sensor=false" />


Ambas plantillas se representan como parte de una más grande, la página compuesta. Debido a que el sistema necesita saber el orden y el lugar donde se hará dentro de la página compuesta, tenemos que añadir un archivo placement.info en la raíz del directorio del módulo:

    
    <Placement>
        <Place Parts_Map="Content:10"/>
        <Place Parts_Map_Edit="Content:7.5"/>
    </Placement>



Es decir que la shape Parts_Map (que es redenderizada en Views/Parts/Maps.cshtml a menos que se sobreescriba en el tema actual) debe ser redenderizada en la "Content" zone si está disponible, en la décima posición. También situa la plantilla del editor shape/template en la zona principal en la segunda posición.

Para activar el Map part, ve a la sección "Features" del panel de administración y activala.

![](../Upload/screenshots/enable_maps.png)


Usted puede probar la MapPart conectándolo a cualquier tipo de contenido en el sistema, a través del "ContentTypes" del panel de administración de Orchard. Vamos a añadir un ContentType existente, es decir, el personalizado "Evento" ContentType que hemos construido en [Creación de tipos de contenido personalizados](Creating-custom-content-types). Si usted no ha leído ese documento todavía o no tienen el tipo "Event" , siga adelante y añada el Mapa para el ContentType de Page en su lugar (siguiendo los mismos pasos a continuación).


En la pantlla "Mange Content Types" del panel de administración, haga click en "Edit" para editar la definición de este tipo (debes tener activada la funcionalidad Orchard.ContentTypes).

![](../Upload/screenshots_675/edit_content_type_event.png)

En la lista de parts para el "Event", haga clic en "Add" para añadir una part.

![](../Upload/screenshots_675/add_parts_event.png)

El map  es mostrado en una la lista de parts disponibles para agregar. Selecciónelo y haga clic en "Save"

![](../Upload/screenshots_85/add_parts_event_map.png)

Ahora vaya a "Manage Content" y edite un EventContentItem. Note que puede añadir la latitud y la longitud para el content part.

![](../Upload/screenshots_675/edit_event_map_fields.png)

En el front-end de tu sitio, puedes ver el efecto que tiene el Map part redenderizando el EventContentItem.

![](../Upload/screenshots_675/view_event_map.png)

# Obteniendo el código
El MapPart descrito en este documento está disponible desde aquí: [Orchard.Module.Maps.1.0.0.zip](/Attachments/Writing-a-content-part/Orchard.Module.Maps.1.0.0.zip), listo para instalar y usar con el código fuente.
