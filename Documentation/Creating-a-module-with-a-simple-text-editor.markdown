En este tutorial, aprenderás cómo desarrollar un módulo de comercio simple utilizando sólo un editor de texto.

Si no tienes [Web Platform Installer](http://www.microsoft.com/web/downloads/platform.aspx) en tu PC, descargalo antes de comenzar con este tutorial.


# Configurando el sitio Orchard

En primer lugar, crea un sitio web nuevo Orchard. Si ya tiene un sitio configurado, puede omitir esta sección y pasar directamente a [la sección de generación de código](#codegen). Para iniciar la instalación, abra el **Administrador de IIS**, haga clic en **Sitios** y haga clic en **Add Web Site**. 


![Figure 1. Creating the Orchard site](../Attachments/Creating-a-module-with-a-simple-text-editor/01_NewWebSite.PNG)

En el formulario **Agregar nuevo sitio** , rellene los campos para que la nueva página web apunte a una carpeta, por ejemplo _ \ inetpub \ wwwroot \ Orchard_. Asigne un nombre al nuevo sitio **Orchard** y asígnale un puerto no utilizado, por ejemplo el 90. Utilice el grupo de aplicaciones predeterminado (. NET 4.0 canalización integrada). Haga clic en **Aceptar**.

![Figure 2. Configuring the site](../Attachments/Creating-a-module-with-a-simple-text-editor/s_02_NewSiteSettings.png)

Desde el menú de **Inicio**, inicie **Web Platform Installer**, seleccione **Orchard CMS**, haga clic en **Agregar**, y después haga clic en **Instalar**.

![](../Upload/screenshots/orchard_install.png)

Después de que acepte los términos de la licencia, Orchard será instalado.

Abra una ventana de comandos y cambie el directorio actual para apuntar a la raíz del sitio. A continuación, ejecute `bin \ orchard.exe`.


[![Figure 6. The Orchard command line](../Attachments/Creating-a-module-with-a-simple-text-editor/s_07_OrchardCLI.png)](../Attachments/Creating-a-module-with-a-simple-text-editor/07_OrchardCLI.png)

Escriba `help` comandos para obtener la lista de comandos disponibles. Por ahora, sólo los comandos 'help' y 'setup' están disponibles. Sin embargo, gracias a la forma en la Orchard está desarrollado y los módulos se activan, habrán nuevos comandos a medida que añada característivas. (La líne de comandos de Orchard descubre los comandos de los módulos dentro de la aplicación.).

Para levantar el sitio, teclee el siguiente comando

    
    setup /SiteName:Orchard /AdminUsername:admin /AdminPassword:123456
          /DatabaseProvider:SqlCe


Esto es equivalente a levantarlo desde la interfaz web.

Deje abierta la ventana de comandos. (De hecho, no la cierre hasta que haya terminado este tutorial.)


# Generando el código para el módulo

Ahora ya está listo para comenzar a desarrollar el módulo de comercio.
Orchard ofrece una función de generación de código que configura la estructura de un módulo vacío para ayudarle a empezar.
De forma predeterminada, la generación de código está desactivada. Así que primero debe instalar y activar la función.
La forma más sencilla de hacerlo es ir a los módulos de la interfaz de usuario de administrador y haga clic en la "Gallery".
Haga una búsqueda para "la generación de código" y luego instale el módulo.

Para habilitar la generación de código, si no lo hizo justo después de la instalación, puede introducir el siguiente comando en la ventana de comandos:

    
    feature enable Orchard.CodeGeneration


Vas a utilizar la utilidad de generación de código para crear un módulo de comercio. Introduce el siguiente comando:

    
    codegen module SimpleCommerce

Abre una ventana del explorador de windows y navega hacia la recién creada carpeta: \inetpub\wwwroot\Orchard\Modules\SimpleCommerce_ . Abre el fichero _module.txt_ usando un editor de texto.


Cambie la descripción a "A simple commerce module". También cambie la descripción del grupo Feature a " A simple product part". Guarde el archivo y ciérrelo. El siguiente ejemplo muestra el archivo _module.txt_ completo después de los cambios.


    
    Name: SimpleCommerce
    AntiForgery: enabled
    Author: The Orchard Team
    Website: http://orchardproject.net
    Version: 0.5.0
    OrchardVersion: 0.5.0
    Description: A simple commerce module
    Features:
        SimpleCommerce:
            Name: Simple Commerce
            Description: A simple product part.
            Category: Commerce


# Creando el modelo para el content part

A continuación, va a crear un modelo de datos que es una repesentation de lo que se almacenará en la base de datos.
En _Modules/SimpleCommerce/Models_, crea un fichero _Product.cs_ con el siguiente contenido:

    
    using System.ComponentModel.DataAnnotations;
    using Orchard.ContentManagement;
    using Orchard.ContentManagement.Records;
    
    namespace SimpleCommerce.Models {
      public class ProductPartRecord : ContentPartRecord {
        public virtual string Sku { get; set; }
        public virtual float Price { get; set; }
      }
    
      public class ProductPart : ContentPart<ProductPartRecord> {
        [Required]
        public string Sku {
          get { return Record.Sku; }
          set { Record.Sku = value; }
        }
    
        [Required]
        public float Price {
          get { return Record.Price; }
          set { Record.Price = value; }
        }
      }
    }


El código tiene dos propiedades, 'Sku' y 'Price', son propiedades virtuales necesarias para habilitar la creación de un proxy dinámico que manejará la capa de persistencia de forma transparente.


El código también define una parte de contenido que se deriva de `ContentPart&lt;ProductPartRecord&gt;` y  que expone el SKU y el precio del registro como propiedades públicas. Las propiedades tienen atributos que saldrán a la superficie en la interfaz de usuario como pruebas de validación.

Para que la aplicación pueda coger el fichero que acabamos de crear, debes añadir la referencia en el fichero del proyecto. Abre _SimpleCommerce.csproj_ y busca "assemblyinfo.cs". Después de esa línea añade la siguiente:

    
    <Compile Include="Models\Product.cs" />


Guarde el archivo, pero se dejalo abierto, se harán cambios adicionales a lo largo del tutorial.


Navega hasta el sitio en tu navegador para estar seguro de que la aplicación vía compilación dinámica coge el nuevo part y record. Sabrás que todo ha funcionado cuando vayas a **Features** en el panel de administración y veas el nuevo **SimpleCommerce** como una nueva funcionalidad.

En la línea de comandos habilita la nueva funcionalidad usando el siguiente comando: 

    
    feature enable SimpleCommerce


# Creando la migración de datos inicial

La migración de datos es un patrón que permite a una aplicación o componente añadir nuevas versiones sin pérdida de datos. La idea principal es que el sistema hace un seguimiento de la versión actual instalada y cada una migración de datos se especifican los cambios para pasar de una versión a la siguiente. Si el sistema detecta que hay una nueva versión instalada y los datos actuales son de una versión anterior, el administrador del sitio se le pedirá que actualice. Entonces, el sistema ejecuta todos los métodos de migración necesarias hasta que la versión de los datos y la versión del código queden sincronizados.



Comience por crear la migración inicial para el nuevo módulo, que acaba de crear las tablas de datos que se necesitan. En la ventana de comandos, escriba el siguiente comando:

    codegen datamigration SimpleCommerce


Esto creará el fichero _Migrations.cs_, con el siguiente contenido:

    using System;
    using System.Collections.Generic;
    using System.Data;
    using Orchard.ContentManagement.Drivers;
    using Orchard.ContentManagement.MetaData;
    using Orchard.ContentManagement.MetaData.Builders;
    using Orchard.Core.Contents.Extensions;
    using Orchard.Data.Migration;
    
    namespace SimpleCommerce.DataMigrations {
        public class Migrations : DataMigrationImpl {
    
            public int Create() {
                // Creating table ProductPartRecord
                SchemaBuilder.CreateTable("ProductPartRecord", table => table
                    .ContentPartRecord()
                    .Column("Sku", DbType.String)
                    .Column("Price", DbType.Single)
                );
    
    
    
                return 1;
            }
        }
    }


El nombre del método `create` es la convención para la migración de datos inicial. Se llama al método `SchemaBuilder.CreateTable`  que crea una tabla que tiene columnas _ProductPartRecord_ _Sku_ y _Price_ además de las columnas de la tabla _ContentPartRecord_ básica.

Note que el método devuelve 1, que es la versión inicial.

Agregue otro paso de migración a la anterior con el fin de ilustrar la forma que más adelante puede alterar el esquema existente de metadatos. En este caso, añadiremos una característica que permitirá a la parte que debe asignarse a cualquier tipo de contenido. Agregue el método siguiente a la clase de migración de datos:


    
    public int UpdateFrom1() {
      ContentDefinitionManager.AlterPartDefinition("ProductPart",
        builder => builder.Attachable());
      return 2;
    }


Esta nueva migración se denomina `UpdateFrom1`, que es la convención para la actualización desde la versión 1. Su migración siguiente debería llamarse `UpdateFrom2` y devolver 3 y así sucesivamente.

Asegúrese de que la siguiente línea se encuentra presente en el archivo _.csproj_. (Ya debería haber sido añadido por la herramienta de generación de código.)

    <Compile Include="Migrations.cs" />


Vaya a la pantalla ** Features ** en el dashboard. Verá una advertencia que indica que una de las características necesita ser actualizada, y el módulo **Simple Commerce** se muestra en rojo. Haga click en ** Update ** para asegurarse de que las migraciones se ejecutan y que el módulo está al día.


# Añadiendo un Handler

Un handler en Orchard es análogo a un filtro en ASP.NET MVC. Es un trozo de código que está destinado a ejecutarse cuando suceden eventos específicos en la aplicación, pero que no son específicos de un tipo de contenido determinado. Por ejemplo, podría construir un módulo de análisis que escucha el evento'Loaded' con el fin de registrar las estadísticas de uso del sistio. Para ver qué eventos de handlers pueden ser sobreescritos, exámine el código fuente de `ContentHandlerBase`.

El handler que necesita en el módulo no va a ser muy complicado, pero sí incorpora algunas tuberías que son necesarias para establecer la persistencia del part. Esperamos que este tipo de tuberías desaparezca en una versión futura de Orchard, posiblemente en favor de un enfoque más declarativo como el uso de atributos.

Crea una carpeta _Handlers_ y añade un fichero _ProductHandler.cs_ que contenga el siguiente código:

    
    using Orchard.ContentManagement.Handlers;
    using SimpleCommerce.Models;
    using Orchard.Data;
    
    namespace SimpleCommerce.Handlers {
      public class ProductHandler : ContentHandler {
        public ProductHandler(IRepository<ProductPartRecord> repository) {
          Filters.Add(StorageFilter.For(repository));
        }
      }
    }


Añade la referencia del fichero en el archivo del proyecto _.csproj_ file, para que la compilación dinámica pueda cogerlo, escribiendo la siguiente línea:

    <Compile Include="Handlers\ProductHandler.cs" />


# Añadiendo un Driver

Un Driver en Orchard es análogo a un controlador en ASP.NET MVC, bien adaptado al aspecto de composición que es necesario en los sistemas de gestión de contenido. Está especializada en una parte de contenido específico y puede especificar un comportamiento personalizado para acciones conocidas como mostrar un elemento en el front-end o editarlo en el panel de administración.

Un driver sobreescribe las acciones de mostrar y editar. Para nuestro ProductPart, crea una carpeta nueva _Drivers_ en esa carpeta, crea un archivo _ProductDriver.cs_ que contendrá el siguiente código:
    
    using SimpleCommerce.Models;
    using Orchard.ContentManagement.Drivers;
    using Orchard.ContentManagement;
    
    namespace SimpleCommerce.Drivers {
      public class ProductDriver : ContentPartDriver<ProductPart> {
        protected override DriverResult Display(
            ProductPart part, string displayType, dynamic shapeHelper)
        {
          return ContentShape("Parts_Product",
              () => shapeHelper.Parts_Product(
                  Sku: part.Sku,
                  Price: part.Price));
        }
    
        //GET
        protected override DriverResult Editor(ProductPart part, dynamic shapeHelper)
        {
          return ContentShape("Parts_Product_Edit",
              () => shapeHelper.EditorTemplate(
                  TemplateName: "Parts/Product",
                  Model: part,
                  Prefix: Prefix));
        }
     
        //POST
        protected override DriverResult Editor(
            ProductPart part, IUpdateModel updater, dynamic shapeHelper)
        {
          updater.TryUpdateModel(part, Prefix, null, null);
          return Editor(part, shapeHelper);
        }
      }
    }


El código en el método `display` crea un shape para utilizar al renderizar el elemento en el front-end. Esa shape tiene propiedades Sku `` y `price` copiadas del part.

Actualiza el archivo _.csproj_ con la siguiente línea:

    <Compile Include="Drivers\ProductDriver.cs" />


El método `Editor`  también crea un shape llamado `EditorTemplate`. La shape tiene una propiedad `TemplateName' que indica a Orchard dónde buscar la plantilla de redenderizado. El código también es`pecifica que el modelo de plantilla será del part, no de la shape (que sería el valor por defecto).

La colocación de las parts dentro del front-end o dashboard debe especificarse utilizando un archivo _placement.info_ que se encuentra en la raíz del módulo. Ese archivo, como una vista, se puede reemplazar en un tema. Cree el archivo _placement.info_ con el siguiente contenido:

    
    <Placement>
        <Place Parts_Product_Edit="Content:3"/>
        <Place Parts_Product="Content:3"/>
    </Placement>


Añade el fichero _placement.info_ al proyecto _.csproj_ file usando la siguiente línea:

    
    <Content Include="placement.info" />


# Creando el template

La última cosa por hacer para que la content part nueva pueda funcionar es escribir las dos plantillas (frontend y administración) que se configuran en el driver.

Cree la plantilla frontal en primer lugar. Cree una carpeta bajo _Parts_ _Views_ y añada un archivo _Product.cshtml_ que contiene el siguiente código:
    
    <br/>
    @T("Price"): <b>$@Model.Price</b><br />
    @Model.Sku<br/>


Esta es una manera muy simple de redenderizar una shape. Note que el método `T` contiene un literal "Price". Esto activa la [localización](Creating-global-ready-applications) en este texto.

La vista de la administración es un poco más pesada por las llamadas de helpers HTML. Crea una carpeta _EditorTemplates_ bajo _Views_ y una carpeta dentro, _Parts_ . Agregue un _Product.cshtml_ a la carpeta _Parts_ que contenga el código siguiente:
    
    @model SimpleCommerce.Models.ProductPart
    <fieldset>
        <label class="sub" for="Sku">@T("Sku")</label><br />
        @Html.TextBoxFor(m => m.Sku, new { @class = "text" })<br />
        <label class="sub" for="Price">@T("Price")</label><br />
        @Html.TextBoxFor(m => m.Price, new { @class = "text" })
    </fieldset>


Agregue las dos plantillas en el archivo _.csproj_ con las siguientes líneas:

    
    <Content Include="Views\Parts\Product.cshtml" />
    <Content Include="Views\EditorTemplates\Parts\Product.cshtml" />


# Uniendo todas las piezas en un Content Type

La content part que hemos preparado ya podría ser compuesto en la interfaz de usuario de administración en un content type (ver [Creando Content Types personalizados](Creating-custom-content-types)), pero la meta de este tutorial es que continues escribiendo el módulo desde un editor de texto.

Ahora se va a construir un nuevo `producto` content type que incluirá parte del 'Producto' y una serie de parts que se pueden obtener a partir de Orchard. Hasta ahora, este tutorial se ha centrado en el dominio específico de un módulo e-commerce. Ahora esto va a cambiar y se va a empezar a integrar en Orchard.

Para construir el tipo de contenido a partir de una nueva migración, abra el archivo _Migrations.cs_ y agregue el método siguiente a la clase:
    
    public int UpdateFrom2() {
      ContentDefinitionManager.AlterTypeDefinition("Product", cfg => cfg
        .WithPart("CommonPart")
        .WithPart("RoutePart")
        .WithPart("BodyPart")
        .WithPart("ProductPart")
        .WithPart("CommentsPart")
        .WithPart("TagsPart")
        .WithPart("LocalizationPart")
        .Creatable()
        .Indexed());
      return 3;
    }


También añade `using Orchard.Indexing;` al principio del fichero.

Lo que estás haciendo es crear (o actualizar) el `producto` content type y añadiéndole la posibilidad de tener su propia URL y  título ('RoutePart'), tener una descripción de texto enriquecido (`BodyPart`), para ser un producto que se puede comentar (`CommentsPart`), para ser poder usar tags (`TagsParts') y estar localizable (`LocalizationPart`). También se ha añadido que pueda ser creado, lo que añadirá una entrada desde el menú **Create Product**, y también entrará en el índice de búsqueda (`Indexed').

Para activar su nuevo módulo, abra el Orchard dashboard y haga click en **Modules**. Selecciona la pestaña **Features**, busca el módulo **SimpleCommerce**, y haga click en **Enable**.

![](../Upload/screenshots/simpleCommerce_enable.png)

Para añadir un nuevo content type **Producto**, haga click en **Content** en el dashboard, seleccione la pestaña Content Type, busque **Product** y  haga click en **Create New Product**.

![](../Upload/screenshots_675/simpleCommerce_product_675.png)

Ahora tienes un editor que cuenta con los campos de producto 'Sku' y 'price'.

El código de este módulo puede ser descargado de: [Orchard.Module.SimpleCommerce.0.5.0.zip](../Attachments/Creating-a-module-with-a-simple-text-editor/Orchard.Module.SimpleCommerce.0.5.0.zip)
