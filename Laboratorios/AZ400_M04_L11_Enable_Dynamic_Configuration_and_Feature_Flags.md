---
lab:
    title: 'Habilitar configuración dinámica y Feature Flags'
    module: 'Módulo 04: Implementar una implementación continua segura mediante Azure Pipelines'
---

# Habilitar configuración dinámica y Feature Flags

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [navegador compatible con Azure DevOps](https://learn.microsoft.com/azure/devops/server/compatibility).

- **Configure una organización de Azure DevOps:** si aún no tiene una organización de Azure DevOps que pueda usar para este laboratorio, créela siguiendo las instrucciones disponibles en [Create an organization or project collection](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization?view=azure-devops).

- Identifique una suscripción de Azure existente o cree una nueva.

- Verifique que tiene una cuenta de Microsoft o una cuenta de Microsoft Entra con el rol de Colaborador o Propietario en la suscripción de Azure. Para obtener más información, consulte [List Azure role assignments using the Azure portal](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) y [View and assign administrator roles in Azure Active Directory](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Descripción general del laboratorio

[Azure App Configuration](https://learn.microsoft.com/azure/azure-app-configuration/overview) ofrece un servicio para administrar configuraciones de la aplicación y feature flags de forma centralizada. Los programas modernos, especialmente los que se ejecutan en la nube, generalmente tienen muchos componentes distribuidos. Propagar la configuración entre estos componentes puede provocar errores difíciles de solucionar durante la implementación de la aplicación. Use App Configuration para almacenar todas las configuraciones de su aplicación y proteger sus accesos en un solo lugar.

## Objetivos

Después de completar este laboratorio, podrá:

- Habilitar la configuración dinámica.
- Administrar feature flags.

## Tiempo estimado: 45 minutos

## Instrucciones

### Ejercicio 0: (omitir si ya se realizó) Configure los requisitos previos del laboratorio

En este ejercicio, configurará los requisitos previos del laboratorio, que consisten en un nuevo proyecto de Azure DevOps con un repositorio basado en [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarea 1: (omitir si ya se realizó) Crear y configurar el proyecto del equipo

En esta tarea, creará un proyecto de Azure DevOps **eShopOnWeb** que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del navegador abra su organización de Azure DevOps. Haga clic en **New Project**. Asigne al proyecto el nombre **eShopOnWeb** y seleccione **Scrum** en el menú desplegable **Work Item process**. Haga clic en **Create**.

#### Tarea 2: (omitir si ya se realizó) Importar el repositorio Git de eShopOnWeb

En esta tarea, importará el repositorio Git de eShopOnWeb que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del navegador abra su organización de Azure DevOps y el proyecto **eShopOnWeb** que creó previamente. Haga clic en **Repos > Files**, luego en **Import**. En la ventana **Import a Git Repository**, pegue la siguiente URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> y haga clic en **Import**:

1. El repositorio está organizado de la siguiente manera:
    - La carpeta **.ado** contiene canalizaciones YAML de Azure DevOps.
    - La carpeta **.devcontainer** contiene la configuración para desarrollar con contenedores (ya sea localmente en VS Code o en GitHub Codespaces).
    - La carpeta **infra** contiene plantillas de infraestructura como código de Bicep y ARM que se usan en algunos escenarios del laboratorio.
    - La carpeta **.github** contiene definiciones de flujos de trabajo YAML de GitHub.
    - La carpeta **src** contiene el sitio web de .NET 8 usado en los escenarios del laboratorio.

#### Tarea 3: (omitir si ya se realizó) Establecer la rama main como rama predeterminada

1. Vaya a **Repos > Branches**.
1. Pase el cursor sobre la rama **main** y luego haga clic en los puntos suspensivos a la derecha de la columna.
1. Haga clic en **Set as default branch**.

### Ejercicio 1: (omitir si ya se realizó) Importar y ejecutar las canalizaciones de CI/CD

En este ejercicio, importará canalizaciones de CI/CD para compilar e implementar la aplicación eShopOnWeb. La canalización de CI ya está preparada para compilar la aplicación y ejecutar pruebas. La canalización de CD implementará la aplicación en una Azure Web App.

#### Tarea 1: Importar y ejecutar la canalización de CI

Comencemos importando la canalización de CI llamada [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Vaya a **Pipelines > Pipelines**.
1. Haga clic en el botón **Create Pipeline** (si no hay canalizaciones) o en **New pipeline** (si ya se han creado canalizaciones).
1. Seleccione **Azure Repos Git (Yaml)**.
1. Seleccione el repositorio **eShopOnWeb**.
1. Seleccione **Existing Azure Pipelines YAML File**.
1. Seleccione la rama **main** y el archivo **/.ado/eshoponweb-ci.yml**, y luego haga clic en **Continue**.
1. Haga clic en el botón **Run** para ejecutar la canalización.
1. La canalización tomará un nombre basado en el nombre del proyecto. Vamos a **renombrarla** para identificarla mejor. Vaya a **Pipelines > Pipelines** y haga clic en la canalización creada recientemente. Haga clic en los puntos suspensivos y seleccione la opción **Rename/Remove**. Asigne el nombre **eshoponweb-ci** y haga clic en **Save**.

#### Tarea 2: Importar y ejecutar la canalización de CD

Importemos la canalización de CD llamada [eshoponweb-cd-webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml).

1. Vaya a **Pipelines > Pipelines**.
1. Haga clic en el botón **New pipeline**.
1. Seleccione **Azure Repos Git (Yaml)**.
1. Seleccione el repositorio **eShopOnWeb**.
1. Seleccione **Existing Azure Pipelines YAML File**.
1. Seleccione la rama **main** y el archivo **/.ado/eshoponweb-cd-webapp-code.yml**, y luego haga clic en **Continue**.
1. En la definición YAML de la canalización, personalice:
   - **YOUR-SUBSCRIPTION-ID** con el identificador de su suscripción de Azure.
   - **az400eshop-NAME** reemplace NAME para que sea globalmente único.
   - **AZ400-EWebShop-NAME** con el nombre del grupo de recursos definido antes en el laboratorio.

1. Haga clic en **Save and Run** y espere a que la canalización se ejecute correctamente.

    > **Nota**: debe hacer clic en **Save and Run** dos veces. Si la ventana **Jobs** muestra un mensaje de permiso necesario, seleccione **Deploy** en la ventana **Jobs**, luego **View** y **Permit** dos veces para completar la ejecución de la canalización.

    > **Nota**: la implementación puede tardar varios minutos en completarse.

    La definición de CD consta de las siguientes tareas:
    - **Resources**: está preparada para activarse automáticamente al completarse la canalización de CI. También descarga el repositorio del archivo Bicep.
    - **AzureResourceManagerTemplateDeployment**: implementa la Azure Web App mediante una plantilla Bicep.

1. La canalización tomará un nombre basado en el nombre del proyecto. Vamos a **renombrarla** para identificarla mejor. Vaya a **Pipelines > Pipelines** y haga clic en la canalización creada recientemente. Haga clic en los puntos suspensivos y seleccione la opción **Rename/Remove**. Asigne el nombre **eshoponweb-cd-webapp-code** y haga clic en **Save**.

### Ejercicio 2: Administrar Azure App Configuration

En este ejercicio, creará el recurso de App Configuration en Azure, habilitará la identidad administrada y luego probará la solución completa.

> **Nota**: este ejercicio no requiere habilidades de programación. El código del sitio web ya implementa las funciones de Azure App Configuration.

Si desea saber cómo implementar esto en su aplicación, consulte estos tutoriales: [Use dynamic configuration in an ASP.NET Core app](https://learn.microsoft.com/azure/azure-app-configuration/enable-dynamic-configuration-aspnet-core) y [Manage feature flags in Azure App Configuration](https://learn.microsoft.com/azure/azure-app-configuration/manage-feature-flags).

#### Tarea 1: Crear el recurso de App Configuration

1. En Azure Portal, busque el servicio **App Configuration**.
1. Haga clic en **Create app configuration** y luego seleccione:
    - Su suscripción de Azure.
    - El grupo de recursos creado previamente (debería llamarse **AZ400-EWebShop-NAME**).
    - La ubicación.
    - Un nombre único como **appcs-NAME-REGION**, por ejemplo.
    - Seleccione la capa de precios **Free**.
1. Haga clic en **Review + create** y luego en **Create**.
1. Después de crear el servicio de App Configuration, vaya a **Overview** y copie/guarde el valor del **Endpoint**.

#### Tarea 2: Habilitar Managed Identity

1. Vaya a la Web App implementada mediante la canalización (debería llamarse **az400-webapp-NAME**).
1. En la sección **Settings**, haga clic en **Identity** y luego cambie el estado a **On** en la sección **System Assigned**; haga clic en **save > yes** y espere unos segundos a que termine la operación.
1. Regrese al servicio de App Configuration y haga clic en **Access control** y luego en **Add role assignment**.
1. En la sección **Role**, seleccione **App Configuration Data Reader**.
1. En la sección **Members**, marque **Manage Identity** y luego haga clic en **+ Select members**. En el campo **Managed Identity**, seleccione **App Service (1)**, elija su aplicación web y haga clic en **Select**.
1. Haga clic en **Review and assign** dos veces para completar la asignación de roles.

#### Tarea 3: Configurar la Web App

Para asegurarse de que su sitio web esté accediendo a App Configuration, debe actualizar su configuración.

1. Regrese a su Web App.
1. En la sección **Settings**, haga clic en **Environment Variables**.
1. Agregue dos configuraciones de aplicación nuevas:
    - Primera configuración de la aplicación
        - **Name:** UseAppConfig
        - **Value:** true
    - Segunda configuración de la aplicación
        - **Name:** AppConfigEndpoint
        - **Value:** *el valor que guardó/copió anteriormente desde App Configuration Endpoint. Debería verse como <https://appcs-NAME-REGION.azconfig.io>*

1. Haga clic en **Apply** y luego en **Confirm** y espere a que se actualicen los ajustes.
1. Vaya a **Overview** y haga clic en **Browse**.
1. En este punto, no verá cambios en el sitio web porque App Configuration aún no contiene datos. Eso es lo que hará en las siguientes tareas.

#### Tarea 4: Probar la administración de configuración

1. En su sitio web, seleccione **Visual Studio** en la lista desplegable **Brand** y haga clic en el botón con la flecha (**>**).
1. Verá un mensaje que dice *"THERE ARE NO RESULTS THAT MATCH YOUR SEARCH"*. El objetivo de este laboratorio es poder actualizar ese valor sin modificar el código del sitio web ni volver a implementarlo.
1. Para probarlo, regrese a App Configuration.
1. En la sección **Operations**, seleccione **Configuration Explorer**.
1. Haga clic en **Create > Key-value** y agregue:
    - **Key:** eShopWeb:Settings:NoResultsMessage
    - **Value:** *escriba su mensaje personalizado*
1. Haga clic en **Apply**; luego vuelva a su sitio web y actualice la página.
1. Debería ver su nuevo mensaje en lugar del valor predeterminado anterior.

#### Tarea 5: Probar el Feature Flag

Continuemos probando el Feature manager.

1. Para probarlo, regrese a App Configuration.
1. En la sección **Operations**, seleccione **Feature manager**.
1. Haga clic en **Create** y agregue:
    - **Enable feature flag:** activado
    - **Feature flag name:** SalesWeekend
1. Haga clic en **Apply**; luego vuelva a su sitio web y actualice la página.
1. Debería ver una imagen con el texto "ALL T-SHIRTS ON SALE THIS WEEKEND".
1. Puede deshabilitar esta función en App Configuration y luego verá que la imagen desaparece.

   > [!IMPORTANT]
   > Recuerde eliminar los recursos creados en Azure Portal para evitar cargos innecesarios. Asegúrese de deshabilitar la canalización **eshoponweb-cd-webapp-code** o volverá a crear un grupo de recursos eliminado y sus recursos asociados después de la próxima ejecución de **eshoponweb-ci**.

## Revisión

En este laboratorio, aprendió a habilitar la configuración de forma dinámica y a administrar feature flags.
