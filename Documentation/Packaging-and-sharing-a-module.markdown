
Después de desarrollar un módulo de extensión de Orchard es probable que desee compartirlo con los demás. Orchard proporciona una característica de empaquetado del módulo que se puede utilizar para crear un archivo de paquete que contiene el módulo. Para activar esta función, visite la sección "Features" del panel de administración de Orchard y active la opción "Orchard.Packaging".

![](../Upload/screenshots_675/enable_packaging.png)

Como alternativa, puede activar la función de Orchard.Packaging desde la línea de comandos de Orchard de línea de comandos. Para ello, ejecute bin\orchard.exe desde la raíz de la instalación de Orchard o la raíz del proyecto Orchard.Web si se ejecuta contra un [repositorio](Setting-up-a-source-enlistment). 
    
    orchard> feature enable Orchard.Packaging


Cuando la función Orchard.Packaging está activada, la línea de comandos de Orchard es compatible con comandos adicionales que se pueden utilizar para crear un paquete ([archivo .nupkg](http://nuget.org)) de cualquier módulo de su instalación Orchard y para instalar un nuevo módulo desde un paquete o fichero con extensión .nupkg.
    
    package create <extensionName> <path>
        Create a package for the extension <extensionName>
        (an extension being a module or a theme).
        The package will be output at the <path> specified.
        The default filename is Orchard.[Module|Theme].<extensionName>.<extensionVersion>.nupkg.
        For example, "package create SampleModule c:\temp" will create the package
        "c:\temp\Orchard.Module.SampleModule.1.0.0.nupkg".
    
    package install <packageId> <location> /Version:<version>
            Install a module or a theme from a package file.
    
    package uninstall <packageId>
        Uninstall a module or a theme.
        The <packageId> should take the format Orchard.[Module|Theme].<extensionName>.
        For example, "package uninstall Orchard.Module.SampleModule" will uninstall the Module under the "~/Modules/SampleModule" directory and
        "package uninstall Orchard.Theme.SampleTheme" will uninstall the Theme under the "~/Themes/SampleTheme" directory.
    
    user create /UserName:<username> /Password:<password> /Email:<email>
            Creates a new User
    

Ejecutando el comando "package create", puedes crear un archivo zip del un módulo.
    
    orchard> package create Lucene C:\Temp
    Package "C:\Temp\Orchard.Module.Lucene.1.0.0.nupkg" successfully created


Orchard usa el formato de paquetes de [NuGet](http://nuget.org) para crear los paquetes de módulos (básicamente archivos .zip con meta información sobre el paquete). NuGet esta basado en el formato de paquetes OPC, puedes aprender más al respecto en http://en.wikipedia.org/wiki/Open_Packaging_Conventions](http://en.wikipedia.org/wiki/Open_Packaging_Conventions).

Una vez que haya creado un paquete de módulos, usted lo puede compartir fácilmente con otros. Orchard ofrece la posibilidad de navegar e instalar un módulo de la sección "Modules" del panel de administración de Orchard. Consulte el tema [Instalación y actualización de módulos](Installing-and-upgrading-modules) para más detalles.

Adicionalmente, Orchard provee una Galería de funcionalidades que te permite registrar una o más feeds de extensiones de módulo (con formato OData). Los usuarios pueden instalar fácilmente módulos gracias a los feeds registrados. Por defecto los feeds de la galería se exponen desde esta web, en [http://orchardproject.net/gallery/server/FeedService.svc](http://orchardproject.net/gallery/server/FeedService.svc/Packages). Para más información, dirigite al tema [Module gallery feeds](Module gallery feeds). 

Puedes visitar la galería desde el panel de administración y instalar los módulos y temas disponibles online. El sitio web [http://orchardproject.net/gallery](http://orchardproject.net/gallery) también provee un front-end para buscar y descargar módulos y temas.

Puedes fácilmente subir tu módulo personalizado a nuestra Galería y compartirlo con otros usuarios de Orchard. [Crea una cuenta](http://orchardproject.net/gallery/Users/Account/Register) y [contribuye con tu módulo aquí](http://orchardproject.net/gallery/Contribute/Index).
