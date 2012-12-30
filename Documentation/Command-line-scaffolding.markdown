La generación de código es un módulo Orchard que automatiza la tarea de creación de archivos y extensiones adicionales. Esta característica es útil para los desarrolladores que quieran crear controladores,  migraciónes , módulos y temas. Sin embargo, la función de generación de código no se instala de forma predeterminada al instalar Orchard.

You can install the code generation feature using the Orchard Gallery. Open the admin panel, click **Modules** under the **Gallery** heading.

Puede instalar la función de generación de código mediante la Orchard Gallery. Abra el panel de administración, haga clic en **Modules** bajo el **Gallery** epígrafe.

![](../Upload/screenshots_675/gallery_modules_675.PNG)

Busca el m´´odulo **Code Generation**, y haga click en **Install**.

![](../Upload/screenshots_675/gallery_code_generation_675.png)

Para habilitar la generación de código haga click en **Features** debajo de **Configuration**, busca la característica (feature) **Code Generation**, y haga click en **Enable**.

![](../Upload/screenshots/enable_codegen.png)

Para activar la función desde Orchard de línea de comandos, abra la línea de comandos de Orchard y escriba el siguiente comando. Para obtener más información acerca de la Huerta de línea de comandos, consulte [Using the command-line interface](Using-the-command-line-interface).

    
    orchard> feature enable Orchard.CodeGeneration
    Enabling features Orchard.CodeGeneration
    Orchard.CodeGeneration was enabled


Una vez que la función de generación de código está activado, los nuevos comandos están disponibles para la creación de un módulo, temas, la migración de datos, o el controlador. En la actualidad, los comandos de generación de código añaden archivos a la localización adecuada

    
    codegen controller <module-name> <controller-name>
            Create a new Orchard controller in a module
    
    codegen datamigration <feature-name>
            Create a new Data Migration class
    
    codegen module <module-name> [/IncludeInSolution:true|false]
            Create a new Orchard module
    
    codegen theme <theme-name> [/CreateProject:true|false][/IncludeInSolution:true|false][/BasedOn:<theme-name>]
            Create a new Orchard theme


Para ver un tutorial de uso de la función de generación de código para crear un nuevo módulo y migraciónes de datos, consulte [Writing a content part](Writing-a-content-part).
