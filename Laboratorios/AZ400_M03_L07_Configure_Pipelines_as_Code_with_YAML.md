---
lab:
  title: "Configurar canalizaciones como código con YAML"
  module: "Módulo 03: Diseñar e implementar una estrategia de publicación"
---

# Configurar canalizaciones como código con YAML

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [navegador compatible con Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configure una organización de Azure DevOps:** si aún no tiene una organización de Azure DevOps que pueda usar para este laboratorio, créela siguiendo las instrucciones disponibles en [Create an organization or project collection](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifique una suscripción de Azure existente o cree una nueva.

- Verifique que tiene una cuenta de Microsoft o una cuenta de Microsoft Entra con el rol de Propietario en la suscripción de Azure y el rol de Administrador global en el inquilino de Microsoft Entra asociado a la suscripción de Azure. Para obtener más información, consulte [List Azure role assignments using the Azure portal](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) y [View and assign administrator roles in Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Descripción general del laboratorio

Muchas organizaciones prefieren definir sus canalizaciones de compilación y publicación mediante YAML. Esto les permite acceder a las mismas funciones de canalización que quienes usan el diseñador visual, pero con un archivo de marcado que se puede gestionar como cualquier otro archivo de origen. Las definiciones de compilación en YAML se pueden agregar a un proyecto simplemente agregando los archivos correspondientes a la raíz del repositorio. Azure DevOps también proporciona plantillas predeterminadas para tipos de proyectos populares y un diseñador de YAML para simplificar el proceso de definición de tareas de compilación y publicación.

## Objetivos

Después de completar este laboratorio, podrá:

- Configurar canalizaciones de CI/CD como código con YAML en Azure DevOps.

## Tiempo estimado: 45 minutos

## Instrucciones

### Ejercicio 0: Configure los requisitos previos del laboratorio

En este ejercicio, configurará los requisitos previos del laboratorio.

#### Tarea 1: (omitir si ya se realizó) Crear y configurar el proyecto del equipo

En esta tarea, creará un proyecto de Azure DevOps **eShopOnWeb_MultiStageYAML** que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del navegador abra su organización de Azure DevOps. Haga clic en **New Project**. Asigne al proyecto el nombre **eShopOnWeb_MultiStageYAML** y deje los demás campos con sus valores predeterminados. Haga clic en **Create**.

   ![Captura de pantalla del panel para crear un proyecto nuevo.](images/create-project.png)

#### Tarea 2: (omitir si ya se realizó) Importar el repositorio Git de eShopOnWeb

En esta tarea, importará el repositorio Git de eShopOnWeb que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del navegador abra su organización de Azure DevOps y el proyecto **eShopOnWeb_MultiStageYAML** que creó anteriormente. Haga clic en **Repos > Files**, luego en **Import a Repository**. Seleccione **Import**. En la ventana **Import a Git Repository**, pegue la siguiente URL https://github.com/MicrosoftLearning/eShopOnWeb.git y haga clic en **Import**:

   ![Captura de pantalla del panel para importar el repositorio.](images/import-repo.png)

1. El repositorio está organizado de la siguiente manera:
   - La carpeta **.ado** contiene canalizaciones YAML de Azure DevOps.
   - La carpeta **.devcontainer** contiene la configuración para desarrollar con contenedores (ya sea localmente en VS Code o en GitHub Codespaces).
   - La carpeta **infra** contiene plantillas de infraestructura como código de Bicep y ARM que se usan en algunos escenarios del laboratorio.
   - La carpeta **.github** contiene definiciones de flujos de trabajo YAML de GitHub.
   - La carpeta **src** contiene el sitio web de .NET 8 que se usa en los escenarios del laboratorio.

1. Vaya a **Repos > Branches**.
1. Pase el cursor sobre la rama **main** y luego haga clic en los puntos suspensivos a la derecha de la columna.
1. Haga clic en **Set as default branch**.

    > **Nota**: si la rama main ya es la rama predeterminada, la opción **Set as default branch** aparece deshabilitada. En ese caso, continúe con las instrucciones.

#### Tarea 3: Crear recursos de Azure

En esta tarea, creará una aplicación web de Azure mediante Azure Portal.

1. Desde el equipo del laboratorio, inicie un navegador web, vaya al [**Azure Portal**](https://portal.azure.com) e inicie sesión con la cuenta de usuario que tiene el rol de Propietario en la suscripción de Azure que usará en este laboratorio y tiene el rol de Administrador global en el inquilino de Microsoft Entra asociado a esa suscripción.
1. En Azure Portal, en la barra de herramientas, haga clic en el icono de **Cloud Shell** situado directamente a la derecha del cuadro de búsqueda.
1. Si se le solicita que seleccione **Bash** o **PowerShell**, elija **Bash**.

   > **Nota**: si es la primera vez que inicia **Cloud Shell** y aparece la ventana emergente **Getting started**, seleccione **No storage account required** y la suscripción que está usando en este laboratorio, y haga clic en **Apply**.

1. En el símbolo del sistema **Bash** del panel de Cloud Shell, ejecute el siguiente comando para crear un grupo de recursos (reemplace el marcador de posición `<region>` por el nombre de la región de Azure más cercana a usted, como 'centralus', 'westeurope' o cualquier otra región de su elección).

   ```bash
   LOCATION='<region>'
   ```

   ```bash
   RESOURCEGROUPNAME='az400m03l07-RG'
   az group create --name $RESOURCEGROUPNAME --location $LOCATION
   ```

1. Para crear un plan de servicio de aplicaciones de Windows, ejecute el siguiente comando:

   ```bash
   SERVICEPLANNAME='az400m03l07-sp1'
   az appservice plan create --resource-group $RESOURCEGROUPNAME --name $SERVICEPLANNAME --sku B3
   ```

    > **Nota**: si obtiene un error como "The subscription is not registered to use namespace 'Microsoft.Web'" al ejecutar el comando anterior, ejecute `az provider register --namespace Microsoft.Web` y luego vuelva a ejecutar el comando que generó el error.

1. Cree una aplicación web con un nombre único.

   ```bash
   WEBAPPNAME=eshoponWebYAML$RANDOM$RANDOM
   az webapp create --resource-group $RESOURCEGROUPNAME --plan $SERVICEPLANNAME --name $WEBAPPNAME
   ```

   > **Nota**: anote el nombre de la aplicación web. Lo necesitará más adelante en este laboratorio.

1. Cierre Azure Cloud Shell, pero deje Azure Portal abierto en el navegador.

### Ejercicio 1: Configurar canalizaciones de CI/CD como código con YAML en Azure DevOps

En este ejercicio, configurará canalizaciones de CI/CD como código con YAML en Azure DevOps.

#### Tarea 1: Agregar una definición de compilación YAML

En esta tarea, agregará una definición de compilación YAML al proyecto existente.

1. Regrese al panel **Pipelines** dentro del centro **Pipelines**.
1. En la ventana **Create your first Pipeline**, haga clic en **Create pipeline**.

   > **Nota**: usaremos el asistente para crear una nueva definición de canalización YAML basada en nuestro proyecto.

1. En el panel **Where is your code?**, haga clic en la opción **Azure Repos Git (YAML)**.
1. En el panel **Select a repository**, haga clic en **eShopOnWeb_MultiStageYAML**.
1. En el panel **Configure your pipeline**, desplácese hacia abajo y seleccione **Existing Azure Pipelines YAML File**.
1. En el panel **Selecting an existing YAML File**, especifique los siguientes parámetros:
   - Branch: **main**
   - Path: **.ado/eshoponweb-ci.yml**
1. Haga clic en **Continue** para guardar estos ajustes.
1. En la pantalla **Review your Pipeline YAML**, haga clic en **Run** para iniciar el proceso de compilación de la canalización.
1. Espere a que la canalización de compilación se complete correctamente. Ignore las advertencias relacionadas con el código fuente, ya que no son relevantes para este ejercicio del laboratorio.

   > **Nota**: cada tarea del archivo YAML está disponible para revisión, incluidas las advertencias y los errores.

#### Tarea 2: Agregar entrega continua a la definición YAML

En esta tarea, agregará entrega continua a la definición basada en YAML de la canalización que creó en la tarea anterior.

> **Nota**: ahora que los procesos de compilación y prueba son correctos, podemos agregar la entrega a la definición YAML.

1. En el panel de ejecución de la canalización, haga clic en el símbolo de puntos suspensivos en la parte superior derecha y, en el menú desplegable, haga clic en **Edit pipeline**.
1. En el panel que muestra el contenido del archivo **eShopOnWeb_MultiStageYAML/.ado/eshoponweb-ci.yml**, vaya hasta el final del archivo (línea 56) y presione **Enter/Return** para agregar una línea vacía nueva.
1. Trabajando en la línea **57**, agregue el siguiente contenido para definir la etapa **Release** en la canalización YAML.

   > **Nota**: puede definir las etapas que necesite para organizar y seguir mejor el progreso de la canalización.

   ```yaml
   - stage: Deploy
     displayName: Deploy to an Azure Web App
     jobs:
       - job: Deploy
         pool:
           vmImage: "windows-latest"
         steps:
   ```

1. Coloque el cursor en una línea nueva al final de la definición YAML.

   > **Nota**: este será el lugar donde se agregarán nuevas tareas.

1. En la lista de tareas del lado derecho del panel del código, busque y seleccione la tarea **Azure App Service Deploy**.
1. En el panel **Azure App Service deploy**, especifique la siguiente configuración y haga clic en **Add**:

   - en la lista desplegable **Azure subscription**, seleccione la suscripción de Azure en la que implementó los recursos de Azure al principio del laboratorio, haga clic en **Authorize** y, cuando se le solicite, autentíquese con la misma cuenta de usuario que usó durante la implementación de los recursos de Azure.
   - en la lista desplegable **App Service name**, seleccione el nombre de la aplicación web que implementó al principio del laboratorio.
   - en el cuadro de texto **Package or folder**, **actualice** el valor predeterminado a `$(Build.ArtifactStagingDirectory)/**/Web.zip`.
   - Abra la sección **Application and Configuration Settings** y, en el cuadro de texto **App settings**, agregue `-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development`.

1. Confirme la configuración en el panel Assistant haciendo clic en el botón **Add**.

   > **Nota**: esto agregará automáticamente la tarea de implementación a la definición de la canalización YAML.

1. El fragmento de código agregado al editor debería verse similar al siguiente, reflejando su nombre para los parámetros azureSubscription y WebAppName:

   ```yaml
   - task: AzureRmWebAppDeployment@4
     inputs:
       ConnectionType: "AzureRM"
       azureSubscription: "AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)"
       appType: "webApp"
       WebAppName: "eshoponWebYAML369825031"
       packageForLinux: "$(Build.ArtifactStagingDirectory)/**/Web.zip"
       AppSettings: "-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development"
   ```

1. Valide que la tarea aparezca como hija de la tarea **steps**. Si no es así, seleccione todas las líneas de la tarea agregada, presione la tecla **Tab** dos veces para indentar cuatro espacios y hacer que quede como hija de **steps**.

   > **Nota**: el parámetro **packageForLinux** es engañoso en el contexto de este laboratorio, pero es válido para Windows o Linux.

   > **Nota**: de forma predeterminada, estas dos etapas se ejecutan de forma independiente. Como resultado, el resultado de la compilación de la primera etapa podría no estar disponible para la segunda sin cambios adicionales. Para implementar estos cambios, agregaremos una tarea nueva para descargar el artefacto de implementación al inicio de la etapa de implementación.

1. Coloque el cursor en la primera línea bajo el nodo **steps** de la etapa **deploy** y presione Enter/Return para agregar una línea vacía nueva (línea 64).
1. En el panel **Tasks**, busque y seleccione la tarea **Download build artifacts**.
1. Especifique los siguientes parámetros para esta tarea:
   - Download Artifacts produced by: **Current Build**
   - Download Type: **Specific Artifact**
   - Artifact Name: **seleccione "Website" de la lista** (o escriba "`Website`" directamente si no aparece automáticamente en la lista)
   - Destination Directory: **$(Build.ArtifactStagingDirectory)**
1. Haga clic en **Add**.
1. El fragmento de código agregado debería verse similar a esto:

   ```yaml
   - task: DownloadBuildArtifacts@1
     inputs:
       buildType: "current"
       downloadType: "single"
       artifactName: "Website"
       downloadPath: "$(Build.ArtifactStagingDirectory)"
   ```

1. Si la sangría de YAML está desalineada, con la tarea agregada aún seleccionada en el editor, presione la tecla **Tab** dos veces para indentarla cuatro espacios.

   > **Nota**: aquí también puede agregar una línea vacía antes y después para facilitar la lectura.

1. Haga clic en **Validate and save** y, en el panel **Validate and save**, haga clic en **Save** nuevamente para confirmar el cambio directamente en la rama main.

   > **Nota**: como el YAML original de CI no estaba configurado para activar automáticamente una nueva compilación, debemos iniciarla manualmente.

1. En el menú izquierdo de Azure DevOps, vaya a **Pipelines** y seleccione **Pipelines** nuevamente.
1. Abra la canalización **eShopOnWeb_MultiStageYAML** y haga clic en **Run pipeline**.
1. Confirme la ejecución en el panel que aparece haciendo clic en **Run**.
1. Observe las 2 etapas diferentes: **Build .Net Core Solution** y **Deploy to Azure Web App**.
1. Espere a que la canalización se inicie y espere hasta que la etapa de compilación se complete correctamente.
1. Cuando la etapa de implementación quiera iniciar, aparecerá un mensaje **Permissions Needed** junto con una barra naranja que dice:

   ```text
   This pipeline needs permission to access a resource before this run can continue to Deploy to an Azure Web App
   ```

1. Haga clic en **View**.
1. En el panel **Waiting for Review**, haga clic en **Permit**.
1. Valide el mensaje en la ventana **Permit access?** y confirme haciendo clic en **Permit**.
1. Esto activa la etapa de implementación. Espere a que se complete correctamente.

   > **Nota**: si la implementación falla por un problema de sintaxis del pipeline YAML, puede usar esta referencia:

   ```yaml
   #NAME THE PIPELINE SAME AS FILE (WITHOUT ".yml")
   # trigger:
   # - main

   resources:
    repositories:
      - repository: self
        trigger: none

   stages:
   - stage: Build
     displayName: Build .Net Core Solution
     jobs:
     - job: Build
       pool:
         vmImage: ubuntu-latest
       steps:
       - task: DotNetCoreCLI@2
         displayName: Restore
         inputs:
           command: 'restore'
           projects: '**/*.sln'
           feedsToUse: 'select'

       - task: DotNetCoreCLI@2
         displayName: Build
         inputs:
           command: 'build'
           projects: '**/*.sln'

       - task: DotNetCoreCLI@2
         displayName: Test
         inputs:
           command: 'test'
           projects: 'tests/UnitTests/*.csproj'

       - task: DotNetCoreCLI@2
         displayName: Publish
         inputs:
           command: 'publish'
           publishWebProjects: true
           arguments: '-o $(Build.ArtifactStagingDirectory)'

       - task: PublishBuildArtifacts@1
         displayName: Publish Artifacts ADO - Website
         inputs:
           pathToPublish: '$(Build.ArtifactStagingDirectory)'
           artifactName: Website

       - task: PublishBuildArtifacts@1
         displayName: Publish Artifacts ADO - Bicep
         inputs:
           PathtoPublish: '$(Build.SourcesDirectory)/infra/webapp.bicep'
           ArtifactName: 'Bicep'
           publishLocation: 'Container'

    - stage: Deploy
      displayName: Deploy to an Azure Web App
      jobs:
      - job: Deploy
        pool:
          vmImage: 'windows-latest'
        steps:
        - task: DownloadBuildArtifacts@1
          inputs:
            buildType: 'current'
            downloadType: 'single'
            artifactName: 'Website'
            downloadPath: '$(Build.ArtifactStagingDirectory)'
        - task: AzureRmWebAppDeployment@4
          inputs:
            ConnectionType: 'AzureRM'
            azureSubscription: 'AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)'
            appType: 'webApp'
            WebAppName: 'eshoponWebYAML369825031'
            packageForLinux: '$(Build.ArtifactStagingDirectory)/**/Web.zip'
            AppSettings: '-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development'
   ```

#### Tarea 3: Revisar el sitio implementado

1. Regrese a la ventana del navegador que muestra Azure Portal y navegue hasta el panel que muestra las propiedades de la aplicación web de Azure.
1. En el panel de la aplicación web de Azure, haga clic en **Overview** y, en el panel de resumen, haga clic en **Browse** para abrir el sitio en una nueva pestaña del navegador.
1. Verifique que el sitio implementado se cargue como se espera en la nueva pestaña del navegador y muestre el sitio web de comercio electrónico eShopOnWeb.

### Ejercicio 2: Configurar la configuración del entorno para canalizaciones de CI/CD como código con YAML en Azure DevOps

En este ejercicio, agregará aprobaciones a una canalización basada en YAML en Azure DevOps.

#### Tarea 1: Configurar entornos de la canalización

Las canalizaciones YAML como código no tienen Release/Quality Gates como las que se usan en las canalizaciones clásicas de Azure DevOps. Sin embargo, se pueden configurar similitudes para canalizaciones YAML mediante **Environments**. En esta tarea, usará este mecanismo para configurar aprobaciones para la etapa de compilación.

1. Desde el proyecto de Azure DevOps **eShopOnWeb_MultiStageYAML**, navegue a **Pipelines**.
1. En el menú de la izquierda de Pipelines, seleccione **Environments**.
1. Haga clic en **Create Environment**.
1. En el panel **New Environment**, agregue un nombre para el entorno llamado **`approvals`**.
1. En **Resources**, seleccione **None**.
1. Confirme la configuración pulsando el botón **Create**.
1. Una vez creado el entorno, seleccione la pestaña **Approvals and Checks** del nuevo entorno **approvals**.
1. En **Add your first check**, seleccione **Approvals**.
1. Agregue el nombre de su cuenta de usuario de Azure DevOps al campo **approvers**.

   > **Nota**: en un escenario real, esto reflejaría el nombre del equipo de DevOps que trabaja en este proyecto.

1. Confirme la configuración de aprobación pulsando el botón **Create**.
1. Por último, debemos agregar la configuración necesaria de "environment: approvals" al código YAML de la canalización para la etapa de implementación. Para ello, vaya a **Repos**, navegue a la carpeta **.ado** y seleccione el archivo de canalización como código **eshoponweb-ci.yml**.
1. En la vista de contenido, haga clic en el botón **Edit** para cambiar al modo de edición.
1. Vaya al inicio del trabajo de implementación (**-job: Deploy** en la línea 60).
1. Agregue una línea vacía justo debajo y luego agregue el siguiente fragmento:

   ```yaml
   environment: approvals
   ```

   El fragmento resultante debería verse así:

   ```yaml
   jobs:
     - job: Deploy
       environment: approvals
       pool:
         vmImage: "windows-latest"
   ```

1. Como el entorno es una configuración específica de una etapa de implementación, no se puede usar en "jobs". Por lo tanto, debemos realizar cambios adicionales en la definición actual del trabajo.
1. En la línea **60**, cambie "- job: Deploy" por **- deployment: Deploy**.
1. A continuación, bajo la línea **63** (vmImage: windows-latest), agregue una línea vacía.
1. Pegue el siguiente fragmento YAML:

   ```yaml
   strategy:
     runOnce:
       deploy:
   ```

1. Seleccione el fragmento restante (desde la línea **67** hasta el final) y use la tecla **Tab** para corregir la sangría del YAML.

   El fragmento YAML resultante debería verse así ahora, reflejando la **Deploy Stage**:

   ```yaml
   - stage: Deploy
     displayName: Deploy to an Azure Web App
     jobs:
       - deployment: Deploy
         environment: approvals
         pool:
           vmImage: "windows-latest"
         strategy:
           runOnce:
             deploy:
               steps:
                 - task: DownloadBuildArtifacts@1
                   inputs:
                     buildType: "current"
                     downloadType: "single"
                     artifactName: "Website"
                     downloadPath: "$(Build.ArtifactStagingDirectory)"
                 - task: AzureRmWebAppDeployment@4
                   inputs:
                     ConnectionType: "AzureRM"
                     azureSubscription: "AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)"
                     appType: "webApp"
                     WebAppName: "eshoponWebYAML369825031"
                     packageForLinux: "$(Build.ArtifactStagingDirectory)/**/Web.zip"
                     AppSettings: "-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development"
   ```

1. Confirme los cambios en el archivo YAML haciendo clic en **Commit** y luego en **Commit** nuevamente en el panel de confirmación que aparece.
1. Vaya al menú del proyecto de Azure DevOps a la izquierda, seleccione **Pipelines**, luego **Pipelines** y observe la canalización **EshopOnWeb_MultiStageYAML** usada anteriormente.
1. Abra la canalización.
1. Haga clic en **Run Pipeline** para iniciar una nueva ejecución; confirme haciendo clic en **Run**.
1. Al igual que antes, la etapa de compilación se inicia según lo esperado. Espere a que se complete correctamente.
1. A continuación, como ya hemos configurado el entorno _environment:approvals_ para la etapa de implementación, se pedirá una confirmación de aprobación antes de que inicie.
1. Esto es visible en la vista de la canalización, donde aparece **Waiting (1 check in progress)**. También se muestra un mensaje de notificación que dice **1 approval needs review before this run can continue to Deploy to an Azure Web App**.
1. Haga clic en el botón **Review** junto a este mensaje.
1. En el panel que aparece, **Waiting for review**, haga clic en el botón **Approve**.
1. Esto permite que la etapa de implementación se inicie y despliegue correctamente el código fuente de la aplicación web de Azure.

   > **Nota**: aunque este ejemplo solo usó aprobaciones, recuerde que otras comprobaciones, como Azure Monitor, REST API, etc., pueden utilizarse de manera similar.

   > [!IMPORTANT]
   > Recuerde eliminar los recursos creados en Azure Portal para evitar cargos innecesarios.

## Revisión

En este laboratorio, configuró canalizaciones de CI/CD como código con YAML en Azure DevOps.
