---
lab:
    title: 'Implementar contenedores de Docker en aplicaciones web de Azure App Service'
    module: 'Módulo 02: Implementar CI con Azure Pipelines y GitHub Actions'
---

# Implementar contenedores de Docker en aplicaciones web de Azure App Service

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [navegador compatible con Azure DevOps](https://learn.microsoft.com/azure/devops/server/compatibility).

- **Configure una organización de Azure DevOps:** si aún no tiene una organización de Azure DevOps que pueda usar para este laboratorio, créela siguiendo las instrucciones disponibles en [Create an organization or project collection](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifique una suscripción de Azure existente o cree una nueva.

- Verifique que tiene una cuenta de Microsoft o una cuenta de Microsoft Entra con el rol de Colaborador o Propietario en la suscripción de Azure. Para obtener más información, consulte [List Azure role assignments using the Azure portal](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) y [View and assign administrator roles in Azure Active Directory](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Descripción general del laboratorio

En este laboratorio, aprenderá a usar una canalización de CI/CD de Azure DevOps para compilar una imagen personalizada de Docker, enviarla a Azure Container Registry y desplegarla como contenedor en Azure App Service.

## Objetivos

Después de completar este laboratorio, podrá:

- Compilar una imagen personalizada de Docker usando un agente Linux hospedado por Microsoft.
- Enviar una imagen a Azure Container Registry.
- Implementar una imagen de Docker como contenedor en Azure App Service mediante Azure DevOps.

## Tiempo estimado: 20 minutos

## Instrucciones

### Ejercicio 0: (omitir si ya se realizó) Configure los requisitos previos del laboratorio

En este ejercicio, configurará los requisitos previos del laboratorio, que consisten en un nuevo proyecto de Azure DevOps con un repositorio basado en [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarea 1: (omitir si ya se realizó) Crear y configurar el proyecto del equipo

En esta tarea, creará un proyecto de Azure DevOps **eShopOnWeb** que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del navegador abra su organización de Azure DevOps. Haga clic en **New Project**. Asigne al proyecto el nombre **eShopOnWeb** y elija **Scrum** en el menú desplegable **Work Item process**. Haga clic en **Create**.

#### Tarea 2: (omitir si ya se realizó) Importar el repositorio Git de eShopOnWeb

En esta tarea, importará el repositorio Git de eShopOnWeb que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del navegador abra su organización de Azure DevOps y el proyecto **eShopOnWeb** que creó anteriormente. Haga clic en **Repos > Files** y luego en **Import**. En la ventana **Import a Git Repository**, pegue la siguiente URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> y haga clic en **Import**:

1. El repositorio está organizado de la siguiente manera:
    - La carpeta **.ado** contiene canalizaciones YAML de Azure DevOps.
    - La carpeta **.devcontainer** contiene la configuración para desarrollar con contenedores (ya sea localmente en VS Code o en GitHub Codespaces).
    - La carpeta **infra** contiene plantillas de infraestructura como código de Bicep y ARM que se usan en algunos escenarios del laboratorio.
    - La carpeta **.github** contiene definiciones de flujos de trabajo YAML de GitHub.
    - La carpeta **src** contiene el sitio web de .NET 8 que se usa en los escenarios del laboratorio.

#### Tarea 3: (omitir si ya se realizó) Establecer la rama principal como rama predeterminada

1. Vaya a **Repos > Branches**.
1. Pase el cursor sobre la rama **main** y luego haga clic en los puntos suspensivos a la derecha de la columna.
1. Haga clic en **Set as default branch**.

### Ejercicio 1: Importar y ejecutar la canalización de CI

En este ejercicio, configurará la conexión de servicio con su suscripción de Azure y luego importará y ejecutará la canalización de CI.

#### Tarea 1: Importar y ejecutar la canalización de CI

1. Vaya a **Pipelines > Pipelines**.
1. Haga clic en el botón **New pipeline** (o **Create Pipeline** si no tiene otras canalizaciones creadas previamente).
1. Seleccione **Azure Repos Git (YAML)**.
1. Seleccione el repositorio **eShopOnWeb**.
1. Seleccione **Existing Azure Pipelines YAML file**.
1. Seleccione la rama **main** y el archivo **/.ado/eshoponweb-ci-docker.yml**, y luego haga clic en **Continue**.
1. En la definición de YAML de la canalización, personalice:
   - **YOUR-SUBSCRIPTION-ID** con su identificador de suscripción de Azure.
   - Reemplace **resourceGroup** por el nombre del grupo de recursos que se usó durante la creación de la conexión de servicio, por ejemplo, **AZ400-RG1**.

1. Revise la definición de la canalización. La definición de CI consta de las siguientes tareas:
    - **Resources**: descarga los archivos del repositorio que se usarán en las tareas siguientes.
    - **AzureResourceManagerTemplateDeployment**: implementa Azure Container Registry mediante una plantilla Bicep.
    - **PowerShell**: recupera el valor de **ACR Login Server** de la salida de la tarea anterior y crea un nuevo parámetro llamado **acrLoginServer**.
    - [**Docker**](https://learn.microsoft.com/azure/devops/pipelines/tasks/reference/docker-v0?view=azure-pipelines) **- Build**: compila la imagen de Docker y crea dos etiquetas (Latest y el ID actual de compilación).
    - **Docker - Push**: envía las imágenes a Azure Container Registry.

1. Haga clic en **Save and Run**.

1. Abra la ejecución de la canalización. Si ve un mensaje de advertencia que dice "This pipeline needs permission to access a resource before this run can continue to Build", haga clic en **View** y luego en **Permit** y **Permit** nuevamente. Esto permitirá que la canalización acceda a la suscripción de Azure.

    > **Nota**: la implementación puede tardar unos minutos en completarse.

1. La canalización tomará un nombre basado en el nombre del proyecto. Vamos a **renombrarla** para identificarla mejor. Vaya a **Pipelines > Pipelines** y haga clic en la canalización creada recientemente. Haga clic en los puntos suspensivos y seleccione la opción **Rename/move**. Asigne el nombre **eshoponweb-ci-docker** y haga clic en **Save**.

1. Vaya al [**Azure Portal**](https://portal.azure.com), busque Azure Container Registry en el grupo de recursos recién creado (debería llamarse **AZ400-RG1**). En el lado izquierdo haga clic en **Repositories** dentro de **Services** y asegúrese de que se haya creado el repositorio **eshoponweb/web**. Cuando haga clic en el enlace del repositorio, debería ver dos etiquetas (una de ellas es **latest**); esas son las imágenes enviadas. Si no las ve, revise el estado de su canalización.

### Ejercicio 2: Importar y ejecutar la canalización de CD

En este ejercicio, configurará la conexión de servicio con su suscripción de Azure y luego importará y ejecutará la canalización de CD.

#### Tarea 1: Importar y ejecutar la canalización de CD

En esta tarea, importará y ejecutará la canalización de CD.

1. Vaya a **Pipelines > Pipelines**.
1. Haga clic en el botón **New pipeline**.
1. Seleccione **Azure Repos Git (YAML)**.
1. Seleccione el repositorio **eShopOnWeb**.
1. Seleccione **Existing Azure Pipelines YAML File**.
1. Seleccione la rama **main** y el archivo **/.ado/eshoponweb-cd-webapp-docker.yml**, y luego haga clic en **Continue**.
1. En la definición de YAML de la canalización, personalice:
   - **YOUR-SUBSCRIPTION-ID** con su identificador de suscripción de Azure.
   - Reemplace **resourceGroup** por el nombre del grupo de recursos utilizado durante la creación de la conexión de servicio, por ejemplo, **AZ400-RG1**.
   - Reemplace **location** por la región de Azure donde se implementarán los recursos.

1. Revise la definición de la canalización. La definición de CD consta de las siguientes tareas:
    - **Resources**: descarga los archivos del repositorio que se usarán en las tareas siguientes.
    - **AzureResourceManagerTemplateDeployment**: implementa Azure App Service mediante una plantilla Bicep.
    - **AzureResourceManagerTemplateDeployment**: agrega una asignación de roles mediante Bicep.

1. Haga clic en **Save and Run**.

1. Abra la ejecución de la canalización. Si ve un mensaje de advertencia que dice "This pipeline needs permission to access a resource before this run can continue to Deploy", haga clic en **View** y luego en **Permit** y **Permit** nuevamente. Esto permitirá que la canalización acceda a la suscripción de Azure.

    > **Importante**: si no autoriza la canalización durante la configuración, encontrará errores de permisos durante la ejecución. Los mensajes de error comunes incluyen "This pipeline needs permission to access a resource" o "Pipeline run failed due to insufficient permissions". Para resolverlo, vaya a la ejecución de la canalización, haga clic en **View** junto a la solicitud de permisos y luego haga clic en **Permit** para conceder el acceso necesario a su suscripción y recursos de Azure.

    > **Nota**: la implementación puede tardar unos minutos en completarse.

    > [!IMPORTANT]
    > Si recibe el mensaje de error "TF402455: Pushes to this branch are not permitted; you must use a pull request to update this branch.", debe desmarcar la regla de protección de ramas "Require a minimum number of reviewers" que se habilitó en los laboratorios anteriores.

1. La canalización tomará un nombre basado en el nombre del proyecto. Vamos a **renombrarla** para identificarla mejor. Vaya a **Pipelines > Pipelines** y pase el cursor sobre la canalización creada recientemente. Haga clic en los puntos suspensivos y seleccione la opción **Rename/move**. Asigne el nombre **eshoponweb-cd-webapp-docker** y haga clic en **Save**.

    > **Nota 1**: el uso de la plantilla **/infra/webapp-docker.bicep** crea un plan de servicio de la aplicación, una aplicación web con identidad administrada asignada por el sistema habilitada y hace referencia a la imagen de Docker enviada previamente: **${acr.properties.loginServer}/eshoponweb/web:latest**.

    > **Nota 2**: el uso de la plantilla **/infra/webapp-to-acr-roleassignment.bicep** crea una nueva asignación de roles para la aplicación web con el rol AcrPull para poder recuperar la imagen de Docker. Esto podría hacerse en la primera plantilla, pero como la propagación de la asignación de roles puede tardar un poco, es una buena idea realizar las dos tareas por separado.

#### Tarea 2: Probar la solución

1. En Azure Portal, vaya al grupo de recursos recién creado; ahora debería ver tres recursos (App Service, App Service Plan y Container Registry).

1. Vaya a App Service y luego haga clic en **Browse**; esto lo llevará al sitio web.

1. Verifique que la aplicación eShopOnWeb se esté ejecutando correctamente. Si confirma esto, habrá completado el laboratorio con éxito.

> [!IMPORTANT]
> Recuerde eliminar los recursos creados en Azure Portal para evitar cargos innecesarios.

## Revisión

En este laboratorio, aprendió a usar una canalización de CI/CD de Azure DevOps para compilar una imagen personalizada de Docker, enviarla a Azure Container Registry y desplegarla como contenedor en Azure App Service.
