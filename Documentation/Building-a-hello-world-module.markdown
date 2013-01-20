En este artículo se describe cómo crear un prqueño módulo para Orchard que sólo mostrará un "hola mundo" de la página.
Otro ejemplo de módulo puede encontrarse aquí: [Quick Start - Get module blueprint](http://orchardjumpstart.codeplex.com/)


# Introducción

Orchard está construido en la cima de ASP.NET MVC, lo que significa que si ya sabes ASP.NET MVC se debe sentir como en casa. Si no, no te preocupes que vamos a explicar todo lo que estamos haciendo.

MVC es un patrón en el que las preocupaciones están claramente separados: hay un modelo (M) para los datos, un controlador (C) que orquesta la interfaz de usuario y determina cómo funciona el modelo, y una vista (V), cuya única responsabilidad es la de mostrar lo que el controlador se lo da.

En el caso de nuestro módulo Hello World, no vamos a tener todos los datos por lo que el modelo no será de interés para nosotros. Sólo se dispone de un controlador y una vista. Todos los demás será la infraestructura necesaria para declarar lo que estamos haciendo para Orchard. Volveremos a estos conceptos y recapitular una vez que hemos construido nuestro módulo.

Los módulos en Orchard son conjuntos de extensiones que se pueden empaquetar con el fin de ser reutilizados en otros sitios Orchard. Los módulos se implementan como Áreas MVC. Las áreas en MVC son sub-sitios que contienen un conjunto de características que actúan de forma relativamente aislada de las otras partes del sitio. Un módulo de Orchard es simplemente un área con un archivo de manifiesto. Se pueden utilizar las API Orchard (pero no tiene por qué).

# Generando la estructura del módulo

Antes de generar la estructura de archivos para el módulo, es necesario descargar, instalar y activar la función de **generación de código** para Orchard. Para más información, ver [Command-line Code Generation](Command-line-scaffolding).

Una vez que haya activado la generación de código, abra la línea de comandos de Orchard, y cree el módulo Hello World con el siguiente comando:

    
    codegen module HelloWorld


# Modificando el manifiesto

Ahora debería tener una carpeta HelloWorld nueva en la carpeta Modules de tu sitio web Orchard. En esta carpeta, encontrará un archivo module.txt. Ábrelo y personalícelo de la siguiente manera:

    
    name: HelloWorld
    antiforgery: enabled
    author: The Orchard Team
    website: http://orchardproject.net
    version: 0.5.0
    orchardversion: 0.5.0
    description: The Hello World module is greeting the world and not doing much more. 
    features:
        HelloWorld:
            Description: A very simple module.
            Category: Sample


Este archivo de texto describe el módulo al sistema. La información contenida en este archivo se utiliza por ejemplo, en la pantalla de administración

> Nota: ten cuidado con las tabulaciones y los espacios en blanco en este fichero.

# Añadiendo la ruta

El módulo tendrá que manejar la URL / HelloWorld relativa en su sitio web Orchard. Para declarar qué hacer cuando esa URL es llamada, cree el archivo Routes.cs siguiente en la carpeta HelloWorld:

    
    using System.Collections.Generic;
    using System.Web.Mvc;
    using System.Web.Routing;
    using Orchard.Mvc.Routes;
    
    namespace HelloWorld {
        public class Routes : IRouteProvider {
            public void GetRoutes(ICollection<RouteDescriptor> routes) {
                foreach (var routeDescriptor in GetRoutes())
                    routes.Add(routeDescriptor);
            }
    
            public IEnumerable<RouteDescriptor> GetRoutes() {
                return new[] {
                    new RouteDescriptor {
                        Priority = 5,
                        Route = new Route(
                            "HelloWorld",
                            new RouteValueDictionary {
                                {"area", "HelloWorld"},
                                {"controller", "Home"},
                                {"action", "Index"}
                            },
                            new RouteValueDictionary(),
                            new RouteValueDictionary {
                                {"area", "HelloWorld"}
                            },
                            new MvcRouteHandler())
                    }
                };
            }
        }
    }


Una ruta es una descripción de la asignación entre las direcciones URL y las acciones del controlador. Este código asigna el URL de HelloWorld al área HelloWorld con el controlador principal y la acción Index.

# Creando el controlador

El nuevo módulo también tiene una carpeta Controladores. Cree el archivo HomeController.cs siguiente en la carpeta Controladores:

    
    using System.Web.Mvc;
    using Orchard.Themes;
    
    namespace HelloWorld.Controllers {
        [Themed]
        public class HomeController : Controller {
            public ActionResult Index() {
                return View("HelloWorld");
            }
        }
    }


Este es el controlador que se encargará de las solicitudes de URL HelloWorld. La acción por omisión, index, solicita que la vista se redenderice.


Observe el atributo Themed sobre la clase del controlador que solicitará que la vista se acople con el tema actualmente activo.

# Creando la vista

En la carpeta Vistas, cree una carpeta con el nombre Home. En la carpeta Home/Views, cree el archivo HelloWorld.cshtml siguiente:

    
    <h2>@T("Hello World!")</h2>


El archivo especifica el contenido básicos de nuestra vista. El resto del maquetado se añadirá en función del tema por defecto.

Observe que hemos utilizado la función auxiliar T que prepara esta vista para ser localizada. Esto no es obligatorio, pero es un bonito detalle.

# Añadiendo los nuevos ficheros al proyecto

Ya casi hemos terminado. La única tarea que queda es declarar al sistema el conjunto de archivos en el módulo para la compilación dinámica.

Abre el ficheo the HelloWorld.csproj y añade las siguientes línes dentro del &lt;/ItemGroup&gt; tags:

    
    <ItemGroup>
      <Compile Include="Routes.cs"/>
      <Compile Include="Controllers\HomeController.cs"/>
    </ItemGroup>


También agregue lo siguiente a la sección ItemGroup que ya cuenta con otras etiquetas de contenido:
    
    <Content Include="Views\Home\HelloWorld.cshtml" />


# Activando el módulo

Por último, es necesario activar el nuevo módulo. En la línea de comandos, escriba:
    
    feature enable HelloWorld


También podría haber hecho esto desde el panel "Features" en la pantalla de interfaz de usuario de administración del sitio.

# Usar el modulo

Ahora puede navegar hasta la URL /HelloWorld de su sitio Orchard en tu navegador web favorito y obtener un bonito mensaje Hello World:

![The UI for our completed module](../Attachments/Building-a-hello-world-module/HelloWorld.png)

# Conclución

En este tutorial, hemos construido un módulo  simple que maneja una ruta (/ HelloWorld) por acción index del controlador home y sirve una vista sencilla que se ajuste por el tema actual. Lo hemos hecho con sólo herramientas libres y de una manera que difiere muy poco de lo que se puede hacer en un área normal ASP.NET MVC. Algunas cosas nos la proporcionó el Framework de Orchard, como por ejemplo la activación/desactivación del módulo o maquetar la vista sin ningún esfuerzo por nuestra parte.

Hopefully this will get you started with Orchard and prepare you to build more elaborate modules.

El código de este ejemplo se lo puede descargar desde: [HelloWorld.zip](../Attachments/Building-a-hello-world-module/HelloWorld.zip)
