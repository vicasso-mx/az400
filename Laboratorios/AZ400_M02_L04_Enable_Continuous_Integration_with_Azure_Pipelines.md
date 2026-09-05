---
lab:
    title: 'Habilitar la integración continua con Azure Pipelines'
    module: 'Módulo 02: Implementar CI con Azure Pipelines y GitHub Actions'
---

# Habilitar la integración continua con Azure Pipelines

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [navegador compatible con Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configure una organización de Azure DevOps:** si aún no tiene una organización de Azure DevOps que pueda usar para este laboratorio, créela siguiendo las instrucciones disponibles en [Create an organization or project collection](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

## Descripción general del laboratorio

En este laboratorio, aprenderá a definir canalizaciones de compilación en Azure DevOps mediante YAML.
Las canalizaciones se utilizarán en dos escenarios:

- Como parte del proceso de validación de Pull Request.
- Como parte de la implementación de la integración continua.

## Objetivos

Después de completar este laboratorio, podrá:

- Incluir la validación de compilación como parte de un Pull Request.
- Configurar la canalización de CI como código con YAML.

## Tiempo estimado: 30 minutos

## Instrucciones

### Ejercicio 0: (omitir si ya se realizó) Configure los requisitos previos del laboratorio

En este ejercicio, configurará los requisitos previos del laboratorio, que consisten en un nuevo proyecto de Azure DevOps con un repositorio basado en [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarea 1: (omitir si ya se realizó) Crear y configurar el proyecto del equipo

En esta tarea, creará un proyecto de Azure DevOps **eShopOnWeb** que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del navegador abra su organización de Azure DevOps. Haga clic en **New Project**. Asigne al proyecto el nombre **eShopOnWeb** y deje los demás campos con sus valores predeterminados. Haga clic en **Create**.

#### Tarea 2: (omitir si ya se realizó) Importar el repositorio Git de eShopOnWeb

En esta tarea, importará el repositorio Git de eShopOnWeb que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del navegador abra su organización de Azure DevOps y el proyecto **eShopOnWeb** que creó anteriormente. Haga clic en **Repos > Files**, luego en **Import a Repository**. Seleccione **Import**. En la ventana **Import a Git Repository**, pegue la siguiente URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> y haga clic en **Import**:

1. El repositorio está organizado de la siguiente manera:
    - La carpeta **.ado** contiene canalizaciones YAML de Azure DevOps.
    - La carpeta **.devcontainer** contiene la configuración para desarrollar usando contenedores (ya sea localmente en VS Code o en GitHub Codespaces).
    - La carpeta **infra** contiene plantillas de infraestructura como código de Bicep y ARM que se usan en algunos escenarios del laboratorio.
    - La carpeta **.github** contiene definiciones de flujos de trabajo de YAML de GitHub.
    - La carpeta **src** contiene el sitio web de .NET que se usa en los escenarios del laboratorio.

#### Tarea 3: (omitir si ya se realizó) Establecer la rama principal como rama predeterminada

1. Vaya a **Repos > Branches**.
1. Pase el cursor sobre la rama **main** y luego haga clic en los puntos suspensivos a la derecha de la columna.
1. Haga clic en **Set as default branch**.

### Ejercicio 1: Incluir la validación de compilación como parte de un Pull Request

En este ejercicio, incluirá la validación de compilación para validar un Pull Request.

#### Tarea 1: Importar la definición de compilación YAML

En esta tarea, importará la definición de compilación YAML que se usará como una Branch Policy para validar los pull requests.

Comencemos importando la canalización de compilación denominada [eshoponweb-ci-pr.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci-pr.yml).

1. Vaya a **Pipelines > Pipelines**.
1. Haga clic en el botón **Create Pipeline** o **New Pipeline**.
1. Seleccione **Azure Repos Git (YAML)**.
1. Seleccione el repositorio **eShopOnWeb**.
1. Seleccione **Existing Azure Pipelines YAML File**.
1. Seleccione la rama **main** y el archivo **/.ado/eshoponweb-ci-pr.yml**, y luego haga clic en **Continue**.

    La definición de compilación consta de las siguientes tareas:
    - **DotNet Restore**: con NuGet Package Restore, puede instalar todas las dependencias del proyecto sin tener que almacenarlas en el control de código fuente.
    - **DotNet Build**: compila un proyecto y todas sus dependencias.
    - **DotNet Test**: controlador de pruebas de .NET que se usa para ejecutar pruebas unitarias.
    - **DotNet Publish**: publica la aplicación y sus dependencias en una carpeta para su implementación en un sistema de hospedaje. En este caso, es **Build.ArtifactStagingDirectory**.

1. En el panel **Review your pipeline YAML**, haga clic en el símbolo de flecha hacia abajo junto al botón **Run** y luego haga clic en **Save**.
1. La canalización tomará un nombre basado en el nombre del proyecto. Vamos a **renombrarla** para identificarla mejor. Vaya a **Pipelines > Pipelines** y haga clic en la canalización creada recientemente. Haga clic en los puntos suspensivos y seleccione la opción **Rename/Move**. Asigne el nombre **eshoponweb-ci-pr** y haga clic en **Save**.

#### Tarea 2: Branch Policies

En esta tarea, agregará políticas a la rama main y solo permitirá cambios mediante Pull Requests que cumplan con las políticas definidas. Desea asegurarse de que los cambios en una rama se revisen antes de fusionarse.

1. Vaya a la sección **Repos > Branches**.
1. En la pestaña **Mine** del panel **Branches**, pase el cursor sobre la entrada de la rama **main** para mostrar el símbolo de puntos suspensivos en el lado derecho.
1. Haga clic en los puntos suspensivos y, en el menú emergente, seleccione **Branch Policies**.
1. En la pestaña **main** de la configuración del repositorio, habilite la opción **Require minimum number of reviewers**. Agregue **1** revisor y marque la casilla **Allow requestors to approve their own changes** (dado que usted es el único usuario de su proyecto para este laboratorio).
1. En la pestaña **main** de la configuración del repositorio, en la sección **Build Validation**, haga clic en **+** (Agregar una nueva política de compilación). En la lista **Build pipeline**, seleccione **eshoponweb-ci-pr** y luego haga clic en **Save**.

#### Tarea 3: Trabajar con Pull Requests

En esta tarea, usará el portal de Azure DevOps para crear un Pull Request, usando una nueva rama para fusionar un cambio en la rama protegida **main**.

1. Vaya a la sección **Repos** del menú de navegación de eShopOnWeb y haga clic en **Branches**.
1. Cree una nueva rama llamada **Feature01** basada en la rama **main**.
1. Haga clic en **Feature01** y navegue hasta el archivo **/eShopOnWeb/src/Web/Program.cs** como parte de la rama **Feature01**.
1. Haga clic en el botón **Edit** en la parte superior derecha.
1. Realice el siguiente cambio en la primera línea:

    ```csharp
    // Testing my PR
    ```

1. Haga clic en **Commit > Commit** (deje el mensaje de confirmación predeterminado).
1. Aparecerá un mensaje que propone crear un Pull Request (ya que su rama **Feature01** ahora tiene cambios por delante en comparación con **main**). Haga clic en **Create a Pull Request**.
1. En la pestaña **New pull request**, deje los valores predeterminados y haga clic en **Create**.
1. El Pull Request mostrará algunos requisitos pendientes según las políticas aplicadas a la rama de destino **main**.
    - Debe haber al menos 1 usuario que revise y apruebe los cambios.
    - La validación de compilación; verá que la compilación **eshoponweb-ci-pr** se activó automáticamente.

1. Cuando todas las validaciones se hayan realizado correctamente, en la parte superior derecha haga clic en **Approve**. Ahora, en el menú desplegable **Set auto-complete**, puede hacer clic en **Complete**.
1. En la pestaña **Complete Pull Request**, haga clic en **Complete Merge**.

### Ejercicio 2: Configurar la canalización de CI como código con YAML

En este ejercicio, configurará la canalización de CI como código con YAML.

#### Tarea 1: Importar la definición de compilación YAML para CI

En esta tarea, agregará la definición de compilación YAML que se usará para implementar la integración continua.

Comencemos importando la canalización de CI llamada [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Vaya a **Pipelines > Pipelines**.
1. Haga clic en el botón **New Pipeline**.
1. Seleccione **Azure Repos Git (YAML)**.
1. Seleccione el repositorio **eShopOnWeb**.
1. Seleccione **Existing Azure Pipelines YAML File**.
1. Seleccione la rama **main** y el archivo **/.ado/eshoponweb-ci.yml**, y luego haga clic en **Continue**.

    La definición de CI consta de las siguientes tareas:
    - **DotNet Restore**: con NuGet Package Restore, puede instalar todas las dependencias del proyecto sin tener que almacenarlas en el control de código fuente.
    - **DotNet Build**: compila un proyecto y todas sus dependencias.
    - **DotNet Test**: controlador de pruebas de .NET que se usa para ejecutar pruebas unitarias.
    - **DotNet Publish**: publica la aplicación y sus dependencias en una carpeta para su implementación en un sistema de hospedaje. En este caso, es **Build.ArtifactStagingDirectory**.
    - **Publish Artifact - Website**: publica el artefacto de la aplicación (creado en el paso anterior) y lo hace disponible como artefacto de canalización.
    - **Publish Artifact - Bicep**: publica el artefacto de infraestructura (archivo Bicep) y lo hace disponible como artefacto de canalización.

1. Haga clic en **Run** y espere a que la canalización se ejecute correctamente.

#### Tarea 2: Habilitar la integración continua

La definición predeterminada de la canalización de compilación no habilita la integración continua.

1. Haga clic en la opción **Edit pipeline** que aparece en el menú de puntos suspensivos junto al botón **Run new** en la parte superior derecha.
1. Ahora, debe reemplazar las líneas **# trigger:** y **# - main** con el siguiente código:

    ```YAML
    trigger:
      branches:
        include:
        - main
      paths:
        include:
        - src/web/*
    ```

    Esto desencadenará automáticamente la canalización de compilación si se realiza algún cambio en la rama main y en el código de la aplicación web (la carpeta src/web).

    Como habilitó Branch Policies, debe pasar por un Pull Request para actualizar el código.

1. Haga clic en el botón **Validate and save** para validar y guardar la definición de la canalización.
1. Seleccione **Create a new branch for this commit**.
1. Mantenga el nombre de rama predeterminado y marque **Start a pull request**.
1. Haga clic en **Save**.
1. La canalización tomará un nombre basado en el nombre del proyecto. Vamos a **renombrarla** para identificarla mejor. Vaya a **Pipelines > Pipelines** y haga clic en la canalización creada recientemente. Haga clic en los puntos suspensivos y seleccione la opción **Rename/Move**. Asigne el nombre **eshoponweb-ci** y haga clic en **Save**.
1. Vaya a **Repos > Pull Requests**.
1. Haga clic en la solicitud de extracción **"Update eshoponweb-ci.yml for Azure Pipelines"**.
1. Cuando todas las validaciones se hayan realizado correctamente, en la parte superior derecha haga clic en **Approve**. Ahora puede hacer clic en **Complete**.
1. En la pestaña **Complete Pull Request**, haga clic en **Complete Merge**.

#### Tarea 3: Probar la canalización de CI

En esta tarea, creará un Pull Request usando una nueva rama para fusionar un cambio en la rama protegida **main** y activar automáticamente la canalización de CI.

1. Vaya a la sección **Repos** y haga clic en **Branches**.
1. Cree una nueva rama llamada **Feature02** basada en la rama **main**.
1. Haga clic en la nueva rama **Feature02**.
1. Vaya al archivo **/eShopOnWeb/src/Web/Program.cs** y haga clic en **Edit** en la parte superior derecha.
1. Elimine la primera línea:

    ```csharp
    // Testing my PR
    ```

1. Haga clic en **Commit > Commit** (deje el mensaje de confirmación predeterminado).
1. Aparecerá un mensaje que propone crear un Pull Request (ya que su rama **Feature02** ahora tiene cambios por delante en comparación con **main**).
1. Haga clic en **Create a Pull Request**.
1. En la pestaña **New pull request**, deje los valores predeterminados y haga clic en **Create**.
1. El Pull Request mostrará algunos requisitos pendientes según las políticas aplicadas a la rama de destino **main**.
1. Cuando todas las validaciones se hayan realizado correctamente, en la parte superior derecha haga clic en **Approve**. Ahora, en el menú desplegable **Set auto-complete**, puede hacer clic en **Complete**.
1. En la pestaña **Complete Pull Request**, haga clic en **Complete Merge**.
1. Regrese a **Pipelines > Pipelines**; verá que la compilación **eshoponweb-ci** se activó automáticamente después de que se fusionó el código.
1. Haga clic en la compilación **eshoponweb-ci** y luego seleccione la última ejecución.
1. Después de su ejecución correcta, haga clic en **Related > Published** para comprobar los artefactos publicados:
    - Bicep: el artefacto de infraestructura.
    - Website: el artefacto de la aplicación.

## Revisión

En este laboratorio, habilitó la validación de pull request mediante una definición de compilación y configuró la canalización de CI como código con YAML en Azure DevOps.
