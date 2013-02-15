Los Fields (campos) pueden ser usados en Orchard para [construir nuevos Contents Types](Creating-custom-content-types). Orchard actualmente viene con un solo tipo de fields, el campo de texto, pero es posible construir tus propios content fields que luego pueden ser utilizados para construir nuevos content types.

Este tema le enseñará cómo agregar un nuevo tipo de campo. Puede encontrar el código fuente de este tutorial aquí: [http://orcharddatetimefield.codeplex.com/](http://orcharddatetimefield.codeplex.com/).

Vamos a asumir que usa Visual Studio y tiene el código fuente aislado para este tema. Es posible construir este módulo sin Visual Studio, simplemente manipulando el arhivo csproj y añadiendo las referencias a los archivos pertinentes. Por favor consulta [Creando un módulo con un simple editor de texto](Creating-a-module-with-a-simple-text-editor) para obtener un ejemplo de cómo crear un editor sin visual studio.


# Objetivos

Conozca los pasos para agregar un nuevo FieldType a Orchard. El propósito es tener un content field Date y otro de Time para que cualquier ContentType existente o nuevo puede permitir al usuario seleccionar una fecha o una hora muy fácilmente.

![The date time field in the event editor](../Attachments/Creating-a-custom-field-type/EventEditor.PNG)

# Creando un módulo

Vamos a crear el FieldType nuevo dentro de un módulo Orchard nuevo para que pueda ser fácilmente distribuido. Vamos a utilizar la función de CodeGeneration para ello.

> **Importante:** Antes de que puedas generar la estructura de ficheros para tu módulo, necesitas descargar, instalar y activar la función de **Code Generation** de Orchard. Para más información ver  [Command-line Code Generation](Command-line-scaffolding).

Una vez que la función **Code Generation** ha sido activada, puedes escribir el siguiente comando 'codegen' en la línea de comandos de Orchard.

    
    codegen module CustomFields /IncludeInSolution:true

Esto debería cerar una nueva carpeta **CustomFields** dentro de Modules, escribiendo algunas carpetas y ficheros. Por ejemplo, deberías abrir el fichero de manifiesto **module.txt** y modificarlo:

    
    Name: CustomFields
    AntiForgery: enabled
    Author: Me
    Website: http://orcharddatetimefield.codeplex.com
    Version: 0.6.1
    OrchardVersion: 0.8.0
    Description: A bunch of custom fields for use in your custom content types.
    Features:
        CustomFields:
            Description: Custom fields for Orchard.
            Category: Fields
        DateTimeField:
            Description: A date and time field with a friendly UI.
            Category: Fields
            Dependencies: CustomFields, Orchard.jQuery, Common, Settings


Estamos definiendo dos características aquí porque este módulo eventualmente contienen más campos y queremos distinguir entre la función predeterminada del módulo (que tiene el mismo nombre que el mismo módulo y tiene que existir en cualquier módulo) y la función de campo de fecha. Que demuestra también categorías y dependencias.

# Modelando el campo

Creamos ahora una carpeta **Fields** dentro de nuestra carpeta **CustomsFields** y crear también fichero **DateTimeField.cs** dentro:

    
    using System;
    using System.Globalization;
    using Orchard.ContentManagement;
    using Orchard.ContentManagement.FieldStorage;
    using Orchard.Environment.Extensions;
    
    namespace CustomFields.DateTimeField.Fields {
        [OrchardFeature("DateTimeField")]
        public class DateTimeField : ContentField {
    
            public DateTime? DateTime {
                get {
                    var value = Storage.Get<string>();
                    DateTime parsedDateTime;
    
                    if (System.DateTime.TryParse(value, CultureInfo.InvariantCulture,
                        DateTimeStyles.AdjustToUniversal, out parsedDateTime)) {
    
                        return parsedDateTime;
                    }
    
                    return null;
                }
    
                set {
                    Storage.Set(value == null ?
                        String.Empty :
                        value.Value.ToString(CultureInfo.InvariantCulture));
                }
            }
        }
    }


El campo es definido como una clase que deriva de 'ContentField', lo que nos proporciona algunas características como el poder almacenar el valor del campo. El campo se almacenará como un string. Las conversiones se manajerán automáticamente, pero lo estamos haciendo explicitamente aquí para darte una buena idea de cómo deberían ser las cosas con ContentFields más complejos.

# Creando el View Model

It is good practice (although not mandatory) to create one or several view models that will be used as the model in the admin template that we will use to render instances of our field. Let's create the following **DateTimeFieldViewModel.cs** file in a new **ViewModels** folder:

    
    namespace CustomFields.DateTimeField.ViewModels {
    
        public class DateTimeFieldViewModel {
    
            public string Name { get; set; }
    
            public string Date { get; set; }
            public string Time { get; set; }
    
            public bool ShowDate { get; set; }
            public bool ShowTime { get; set; }
        }
    }

Esto no sólo expone la fecha y el tiempo como propiedades separadas, también tiene algunos parámetros que pueden ser pasados a la vista para personalizar el redenderizado.


# Creando las opciones de configuración para nuestro ContentField

Esta flexibilidad en la representación que nos acaba de introducir en el ViewModel puede exponer como son los ajustes para el campo. De esta manera, los administradores pueden configurar los campos en los content type que crean con el fin de adaptarlos a sus necesidades específicas.

Crea una carpeta **Settings** y añade el siguiente fichero **DateTimeFieldSetting.cs** con esto:

    
    namespace CustomFields.DateTimeField.Settings {
    
        public enum DateTimeFieldDisplays {
            DateAndTime,
            DateOnly,
            TimeOnly
        }
    
        public class DateTimeFieldSettings {
            public DateTimeFieldDisplays Display { get; set; }
        }
    }


Hemos definido aquí una enumeración que describe los posibles valores de nuestra configuración de la pantalla, que es el único parámetro para el campo. La clase de configuración en sí es simplemente una clase ordinaria con una propiedad escrita con esa enumeración.

# Escribiendo el Driver

Exactamente igual que con un part, un campo tiene un controlador que será responsable de manejar la visualización y edición  en este campo, al que será añadido a un ContentType.

Crear una carpeta **Drivers** y añade el siguiente fichero **DateTimeFieldDriver.cs**

    
    using System;
    using JetBrains.Annotations;
    using Orchard;
    using Orchard.ContentManagement;
    using Orchard.ContentManagement.Drivers;
    using Contrib.DateTimeField.Settings;
    using Contrib.DateTimeField.ViewModels;
    using Orchard.ContentManagement.Handlers;
    using Orchard.Localization;
    
    namespace CustomFields.DateTimeField.Drivers {
        [UsedImplicitly]
        public class DateTimeFieldDriver : ContentFieldDriver<Fields.DateTimeField> {
            public IOrchardServices Services { get; set; }
    
            // EditorTemplates/Fields/Custom.DateTime.cshtml
            private const string TemplateName = "Fields/Custom.DateTime";
    
            public DateTimeFieldDriver(IOrchardServices services) {
                Services = services;
                T = NullLocalizer.Instance;
            }
    
            public Localizer T { get; set; }
    
            private static string GetPrefix(ContentField field, ContentPart part) {
                // handles spaces in field names
                return (part.PartDefinition.Name + "." + field.Name)
                       .Replace(" ", "_");
            }
    
            protected override DriverResult Display(
                ContentPart part, Fields.DateTimeField field,
                string displayType, dynamic shapeHelper) {
    
                var settings = field.PartFieldDefinition.Settings
                                    .GetModel<DateTimeFieldSettings>();
                var value = field.DateTime;
                
                return ContentShape("Fields_Custom_DateTime", // key in Shape Table
                        field.Name, // used to differentiate shapes in placement.info overrides, e.g. Fields_Common_Text-DIFFERENTIATOR
                        // this is the actual Shape which will be resolved
                        // (Fields/Custom.DateTime.cshtml)
                        s =>
                        s.Name(field.Name)
                         .Date(value.HasValue ?
                             value.Value.ToLocalTime().ToShortDateString() :
                             String.Empty)
                         .Time(value.HasValue ?
                             value.Value.ToLocalTime().ToShortTimeString() :
                             String.Empty)
                         .ShowDate(
                             settings.Display == DateTimeFieldDisplays.DateAndTime ||
                             settings.Display == DateTimeFieldDisplays.DateOnly)
                         .ShowTime(
                             settings.Display == DateTimeFieldDisplays.DateAndTime ||
                             settings.Display == DateTimeFieldDisplays.TimeOnly)
                    );
            }
    
            protected override DriverResult Editor(ContentPart part,
                                                   Fields.DateTimeField field,
                                                   dynamic shapeHelper) {
    
                var settings = field.PartFieldDefinition.Settings
                                    .GetModel<DateTimeFieldSettings>();
                var value = field.DateTime;
    
                if (value.HasValue) {
                    value = value.Value.ToLocalTime();
                }
    
                var viewModel = new DateTimeFieldViewModel {
                    Name = field.Name,
                    Date = value.HasValue ?
                           value.Value.ToLocalTime().ToShortDateString() : "",
                    Time = value.HasValue ?
                           value.Value.ToLocalTime().ToShortTimeString() : "",
                    ShowDate =
                        settings.Display == DateTimeFieldDisplays.DateAndTime ||
                        settings.Display == DateTimeFieldDisplays.DateOnly,
                    ShowTime =
                        settings.Display == DateTimeFieldDisplays.DateAndTime ||
                        settings.Display == DateTimeFieldDisplays.TimeOnly
    
                };
    
                return ContentShape("Fields_Custom_DateTime_Edit",
                    () => shapeHelper.EditorTemplate(
                              TemplateName: TemplateName,
                              Model: viewModel,
                              Prefix: GetPrefix(field, part)));
            }
    
            protected override DriverResult Editor(ContentPart part,
                                                   Fields.DateTimeField field,
                                                   IUpdateModel updater,
                                                   dynamic shapeHelper) {
    
                var viewModel = new DateTimeFieldViewModel();
    
                if (updater.TryUpdateModel(viewModel,
                                           GetPrefix(field, part), null, null)) {
                    DateTime value;
    
                    var settings = field.PartFieldDefinition.Settings
                                        .GetModel<DateTimeFieldSettings>();
                    if (settings.Display == DateTimeFieldDisplays.DateOnly) {
                        viewModel.Time = DateTime.Now.ToShortTimeString();
                    }
    
                    if (settings.Display == DateTimeFieldDisplays.TimeOnly) {
                        viewModel.Date = DateTime.Now.ToShortDateString();
                    }
    
                    if (DateTime.TryParse(
                            viewModel.Date + " " + viewModel.Time, out value)) {
                        field.DateTime = value.ToUniversalTime();
                    }
                    else {
                        updater.AddModelError(GetPrefix(field, part),
                                              T("{0} is an invalid date and time",
                                              field.Name));
                        field.DateTime = null;
                    }
                }
    
                return Editor(part, field, shapeHelper);
            }
    
            protected override void Importing(ContentPart part, Fields.DateTimeField field, 
                ImportContentContext context) {
    
                var importedText = context.Attribute(GetPrefix(field, part), "DateTime");
                if (importedText != null) {
                    field.Storage.Set(null, importedText);
                }
            }
    
            protected override void Exporting(ContentPart part, Fields.DateTimeField field, 
                ExportContentContext context) {
                context.Element(GetPrefix(field, part))
                    .SetAttributeValue("DateTime", field.Storage.Get<string>(null));
            }
        }
    }


Enumeremos unas pocas de las cosas que estamos haciendo en el código para explicar cómo funciona.

El driver deriva de 'ContentFieldDriver<DateTimeField>' para que sea reconocido por Orchard y tenga un tipado fuerte cuando sea accedido desde el código del driver.

Empezamos por inyectar una dependencia del localizador (la propiedad 'T') de manera que podamos usar cadenas localizables en todo el código.

El método estático 'GetPrefix' es una convención definida que se utiliza para crear nombres de columnas en la base de datos para las instancias del ContentType.

Después tenemos dos acciones, 'Display' y 'Editor', cada una de ellas empieza por coger las 'settings' y 'value' para cada campo y construye un shape para mostrarlo.

> Nota: El atributo `UsedImplicitly' está aquí sólo para suprimir una advertencia de ReSharper. Podría ser retirado.

El objeto 'shapeHelper' prove algunos métodos helper para crear shapes, dos de ellos pueden ser vistos en acción.

El segundo método de 'Editor' es uno de los que es llamado cuando el form es enviado. 

The second `Editor` method is the one that is called when the admin form is submitted. Su trabajo consiste en asignar los datos presentados de nuevo en el campo y después de llamar al primer método Editor` para hacer que el editor en la pantalla de nuevo.

# Escribiendo plantillas

Tenemos que escribir las vistas que determinarán como nuestros campos van a ser representados en el panel de administración y en el front-end.

Crear el directorio **Fields** y otro **EditorTemplates** dentro de **Views**. Después crea otro directorio **Fields** dentro de EditorTemplates. En **Views/Fields**, crea el siguiente archivo, **Custom.DateTime.cshtml**:
    
    <p class="text-field"><span class="name">@Model.Name:</span> 
        @if(Model.ShowDate) { <text>@Model.Date</text> } 
        @if(Model.ShowTime) { <text>@Model.Time</text> }
    </p>

Este código redenderiza el nombre del campo, seguido de dos puntos y el valor de la fecha y el tiempo acorde con la configuración del campo.

Ahora crea un fichero con el mismo nombre dentro de **Views/EditorTemplates/Fields** con el siguiente contenido:

    
    @model CustomFields.DateTimeField.ViewModels.DateTimeFieldViewModel
    
    @{
    Style.Include("datetime.css");
    Style.Require("jQueryUI_DatePicker");
    Style.Require("jQueryUtils_TimePicker");
    Style.Require("jQueryUI_Orchard");
    
    Script.Require("jQuery");
    Script.Require("jQueryUtils");
    Script.Require("jQueryUI_Core");
    Script.Require("jQueryUI_Widget");
    Script.Require("jQueryUI_DatePicker");
    Script.Require("jQueryUtils_TimePicker");
    }
    
    <fieldset>
      <label for="@Html.FieldIdFor(m => Model.Date)">@Model.Name</label>
    
      @if ( Model.ShowDate ) {
      <label class="forpicker"
        for="@Html.FieldIdFor(m => Model.Date)">@T("Date")</label>
      <span class="date">@Html.EditorFor(m => m.Date)</span>
      }
    
      @if ( Model.ShowTime ) {
      <label class="forpicker"
        for="@Html.FieldIdFor(m => Model.Time)">@T("Time")</label>
      <span class="time">@Html.EditorFor(m => m.Time)</span>
      }
      @if(Model.ShowDate) { <text>@Html.ValidationMessageFor(m=>m.Date)</text> }
      @if(Model.ShowTime) { <text>@Html.ValidationMessageFor(m=>m.Time)</text> }
    </fieldset>
    
    @using(Script.Foot()) {
    <script type="text/javascript">
    $(function () {
      $("#@Html.FieldIdFor(m => Model.Date)").datepicker();
      $("#@Html.FieldIdFor(m => Model.Time)").timepickr();
    });
    </script>
    }



Esta plantilla está registrando algunos estilos y scripts (tenga en cuenta que si otras parts registran los mismos archivos, sólo se redenderizan una vez). A continuación, define el editor como un selector de fechas y un selector de tiempo de acuerdo a la configuración del campo. Los campos son cuadros de texto normales que están discretamente enriquecidos por selectores de fecha y hora utilizando jQuery UI.

Para especificar el orden y la ubicación en la que estas plantillas se representará en el página compuesta, tenemos que añadir un archivo placement.info en la raíz del directorio del módulo:
    
    <Placement>
        <Place Fields_Custom_DateTime_Edit="Content:2.5"/>
        <Place Fields_Custom_DateTime="Content:2.5"/>
    </Placement>



# Gestionando las opciones de configuración del campo.

No hemos terminado completamente aún. Todavía tenemos que gestionar y  persistir los valores para el campo.

Añade el siguiente fichero **DateTimeFieldEditorEvents.cs** en el directorio **Settings**:

    
    using System.Collections.Generic;
    using Orchard.ContentManagement;
    using Orchard.ContentManagement.MetaData;
    using Orchard.ContentManagement.MetaData.Builders;
    using Orchard.ContentManagement.MetaData.Models;
    using Orchard.ContentManagement.ViewModels;
    
    namespace CustomFields.DateTimeField.Settings {
        public class DateTimeFieldEditorEvents : ContentDefinitionEditorEventsBase {
    
            public override IEnumerable<TemplateViewModel>
              PartFieldEditor(ContentPartFieldDefinition definition) {
                if (definition.FieldDefinition.Name == "DateTimeField") {
                    var model = definition.Settings.GetModel<DateTimeFieldSettings>();
                    yield return DefinitionTemplate(model);
                }
            }
    
            public override IEnumerable<TemplateViewModel> PartFieldEditorUpdate(
              ContentPartFieldDefinitionBuilder builder, IUpdateModel updateModel) {
                var model = new DateTimeFieldSettings();
                if (builder.FieldType != "DateTimeField") {
                  yield break;
                }
                
                if (updateModel.TryUpdateModel(
                  model, "DateTimeFieldSettings", null, null)) {
                    builder.WithSetting("DateTimeFieldSettings.Display",
                                        model.Display.ToString());
                }
    
                yield return DefinitionTemplate(model);
            }
        }
    }


Es equivalente a un  driver, pero para la configuración del campo. El primer método obtiene los valores y determina qué plantilla utilizará, y el segundo actualiza el modelo con los valores del formulario presentado y a continuación, llama al primero.

La plantilla editor para el campo es definiada por el siguiente fichero **DateTimeFieldSettings.cshtml** que debes crear en un nuevo directorio **DefinitionTemplates** dentro de **Views**:

    
    @model CustomFields.DateTimeField.Settings.DateTimeFieldSettings
    @using CustomFields.DateTimeField.Settings;
    
    <fieldset>
        <label for="@Html.FieldIdFor(m => m.Display)"
          class="forcheckbox">@T("Display options")</label>  
        <select id="@Html.FieldIdFor(m => m.Display)"
          name="@Html.FieldNameFor(m => m.Display)">
            @Html.SelectOption(DateTimeFieldDisplays.DateAndTime, 
              Model.Display == DateTimeFieldDisplays.DateAndTime,
              T("Date and time").ToString())
            @Html.SelectOption(DateTimeFieldDisplays.DateOnly,
              Model.Display == DateTimeFieldDisplays.DateOnly,
              T("Date only").ToString())
            @Html.SelectOption(DateTimeFieldDisplays.TimeOnly,
              Model.Display == DateTimeFieldDisplays.TimeOnly,
              T("Time only").ToString())
        </select> 
    
        @Html.ValidationMessageFor(m => m.Display)
            
    </fieldset>

Esta plantilla crea una etiqueta para la configuración y luego un selector que permite activar al administrador del sitio una opción para la configuración del campo.

# Actualizando el fichero del proyecto

Si utiliza Visual Studio, usted puede saltar esta sección ya que el archivo de proyecto ya ha sido actualizado, simplemente guarde todo con (CTRL + SHIFT + S). De lo contrario, para que el motor de compilación dinámica de Orchard pueda recoger los archivos cs de nuestro nuevo módulo, necesitamos añadirlos al fichero **CustomFields.csproj**. Busca la línea `<compile Include="Properties\AssemblyInfo.cs"/>` en el fichero **CustomFields.csproj** y añade lo siguiente justo después:


    
    <Compile Include="Drivers\DateTimeFieldDriver.cs" />
    <Compile Include="Fields\DateTimeField.cs" />
    <Compile Include="Settings\DateTimeFieldEditorEvents.cs" />
    <Compile Include="Settings\DateTimeFieldSettings.cs" />
    <Compile Include="ViewModels\DateTimeFieldViewModel.cs" />


# Añadiendo la hoja de estilos

Crea un directorio **Styles** y crea el siguiente fichero **datetime.css**
    
    html.dyn label.forpicker {
        display:none;
    }
    
    html.dyn input.hinted {
        color:#ccc;
        font-style:italic;
    }
    .date input{
        width:10em;
    }
    .time input {
        width:6em;
    }


# Usando el Field

In order to be able to use the new field, you must first make sure that the **Orchard.ContentTypes** feature is enabled. Also enable our new **DateTimeField** feature under **Fields**. Once it is, you can click on **Manage content types** in the admin menu. Click **Create new type** and give it the name "Event". Click **Add** next to fields and type in "When" as the name of the field. Select our new **DateTime** field type as the type of the field.

Con el fin de poder utilizar el nuevo campo, primero debe asegurarse de que la función **Orchard.ContentTypes**está habilitada. También activa  nuestra nueva función **DateTimeField** debajo de **Fields**. Una vez que esté activada, puede hacer click en **Manage content types** en el menú de administración. Haga clic en **Create new type** nuevo tipo y dele el nombre de "Event". Haga clic en **Add** al lado de los campos y escriba "When" como el nombre del campo. Seleccione el nuevo ContentType **DateTime** como el tipo de campo.

Ahora en el editor de texto, debes de ver nuestro nuevo campo **When**, y deberás poder desplegar la sección de configuración  haciendo click en la izquierda de "**&gt;**":

![Configuring the field](../Attachments/Creating-a-custom-field-type/ConfiguringTheField.PNG)

Hemos elegido para mantener la fecha y la hora que se muestra. Los valores para el campo tienen también la oportunidad de determinar dónde está el campo o si aparecerá en el front-end si desea anular los valores predeterminados. Vamos a pasar eso por ahora. Agregue la part **Route** para que nuestros eventos puedan tener un título y guarde.

Ahora podremos añadir un nuevo evento haciendo click en **Create Event** en el menú de administración. El editor 


We can now add a new event by clicking **Create Event** in the admin menu. El editor que se crea para nosotros tiene un campo "When" con bonitos selectores:

![The date time field in the event editor](../Attachments/Creating-a-custom-field-type/EventEditor.PNG)
Create an event and save it. You can now view it on the site:

![The event as displayed on the front end](../Attachments/Creating-a-custom-field-type/Dinner.PNG)

# Obteniendo el código
Descarga el código de aquí: [CustomFields.zip](../Attachments/Creating-a-custom-field-type/CustomFields.zip). El código también está en [http://orcharddatetimefield.codeplex.com/](http://orcharddatetimefield.codeplex.com/).
