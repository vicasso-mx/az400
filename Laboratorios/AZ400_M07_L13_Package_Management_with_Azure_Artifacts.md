---
lab:
  title: "Gestión de paquetes con Azure Artifacts"
  module: "Módulo 07: Diseñar e implementar una estrategia de gestión de dependencias"
---

# Gestión de paquetes con Azure Artifacts

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [navegador compatible con Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configure una organización de Azure DevOps:** si aún no tiene una organización de Azure DevOps que pueda usar para este laboratorio, créela siguiendo las instrucciones disponibles en [AZ-400 Lab Prerequisites](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

- **Configure el proyecto de ejemplo eShopOnWeb:** si aún no tiene el proyecto de ejemplo eShopOnWeb que pueda usar para este laboratorio, créelo siguiendo las instrucciones disponibles en [AZ-400 Lab Prerequisites](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

- Visual Studio 2022 Community Edition disponible en la [página de descargas de Visual Studio](https://visualstudio.microsoft.com/downloads/). La instalación de Visual Studio 2022 debe incluir las cargas de trabajo **ASP.NET y desarrollo web**, **Azure development** y **.NET Core cross-platform development**.

- **SDK de .NET Core:** [Descargue e instale el SDK de .NET Core (2.1.400+)](https://go.microsoft.com/fwlink/?linkid=2103972)

- **Proveedor de credenciales de Azure Artifacts:** [Descargue e instale el proveedor de credenciales](https://go.microsoft.com/fwlink/?linkid=2099625).

## Descripción general del laboratorio

Azure Artifacts facilita la detección, instalación y publicación de paquetes NuGet, npm y Maven en Azure DevOps. Está profundamente integrado con otras características de Azure DevOps, como Build, haciendo que la gestión de paquetes forme parte natural de sus flujos de trabajo existentes.

## Objetivos

Después de completar este laboratorio, podrá:

- Crear y conectarse a un feed.
- Crear y publicar un paquete NuGet.
- Importar un paquete NuGet.
- Actualizar un paquete NuGet.

## Tiempo estimado: 35 minutos

## Instrucciones

### Ejercicio 0: Configurar los requisitos previos del laboratorio

En este ejercicio, configurará los requisitos previos del laboratorio.

#### Tarea 1: (omitir si ya se realizó) Crear y configurar el proyecto del equipo

En esta tarea, creará un proyecto de Azure DevOps **eShopOnWeb** que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del navegador abra su organización de Azure DevOps. Haga clic en **New Project**. Asigne al proyecto el nombre **eShopOnWeb** y deje los demás campos con sus valores predeterminados. Haga clic en **Create**.

   ![Captura de pantalla del panel para crear un proyecto nuevo.](images/create-project.png)

#### Tarea 2: (omitir si ya se realizó) Importar el repositorio Git de eShopOnWeb

En esta tarea, importará el repositorio Git de eShopOnWeb que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del navegador abra su organización de Azure DevOps y el proyecto **eShopOnWeb** que creó previamente. Haga clic en **Repos > Files**, luego en **Import a Repository**. Seleccione **Import**. En la ventana **Import a Git Repository**, pegue la siguiente URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> y haga clic en **Import**:

   ![Captura de pantalla del panel para importar el repositorio.](images/import-repo.png)

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

#### Tarea 4: Configurar la solución eShopOnWeb en Visual Studio

En esta tarea, configurará Visual Studio para prepararlo para el laboratorio.

1. Asegúrese de estar viendo el proyecto del equipo **eShopOnWeb** en el portal de Azure DevOps.

   > **Nota**: puede acceder directamente a la página del proyecto navegando a la URL [https://dev.azure.com/`<your-Azure-DevOps-account-name>`/eShopOnWeb](https://dev.azure.com/`<your-Azure-DevOps-account-name>`/eShopOnWeb), donde el marcador de posición `<your-Azure-DevOps-account-name>` representa el nombre de su organización de Azure DevOps.

1. En el menú vertical del lado izquierdo del panel **eShopOnWeb**, haga clic en **Repos**.
1. En el panel **Files**, haga clic en **Clone**, seleccione la flecha desplegable junto a **Clone in VS Code** y, en el menú desplegable, elija **Visual Studio**.
1. Si aparece un mensaje para confirmar, haga clic en **Open**.
1. Si se le solicita, inicie sesión con la cuenta de usuario que usó para configurar su organización de Azure DevOps.
1. Dentro de la interfaz de Visual Studio, en la ventana emergente **Azure DevOps**, acepte la ruta local predeterminada (C:\eShopOnWeb) y haga clic en **Clone**. Esto importará automáticamente el proyecto a Visual Studio.
1. Deje la ventana de Visual Studio abierta para usarla en el laboratorio.

### Ejercicio 1: Trabajar con Azure Artifacts

En este ejercicio, aprenderá a trabajar con Azure Artifacts siguiendo estos pasos:

- Crear y conectarse a un feed.
- Crear y publicar un paquete NuGet.
- Importar un paquete NuGet.
- Actualizar un paquete NuGet.

#### Tarea 1: Crear y conectarse a un feed

En esta tarea, creará y se conectará a un feed.

1. En la ventana del navegador que muestra la configuración del proyecto en el portal de Azure DevOps, en el panel de navegación vertical, seleccione **Artifacts**.
1. Cuando se muestre el centro **Artifacts**, haga clic en **+ Create feed** en la parte superior del panel.

   > **Nota**: este feed será una colección de paquetes NuGet disponibles para los usuarios dentro de la organización y se situará junto al feed público de NuGet como un elemento equivalente. El escenario de este laboratorio se centrará en el flujo de trabajo para usar Azure Artifacts, por lo que las decisiones arquitectónicas y de desarrollo son puramente ilustrativas. Este feed incluirá funcionalidad común que puede compartirse entre proyectos de esta organización.

1. En el panel **Create new feed**, en el cuadro de texto **Name**, escriba **`eShopOnWebShared`**. En la sección **Visibility**, seleccione **Specific people**; en la sección **Scope**, seleccione la opción **Project:eShopOnWeb**; deje el resto con sus valores predeterminados y haga clic en **Create**.

   > **Nota**: cualquier usuario que quiera conectarse a este feed de NuGet deberá configurar su entorno.

1. En el centro **Artifacts**, haga clic en **Connect to feed**.
1. En el panel **Connect to feed**, en la sección **NuGet**, seleccione **Visual Studio** y, en el panel **Visual Studio**, copie la URL **Source**. `https://pkgs.dev.azure.com/Azure-DevOps-Org-Name/_packaging/eShopOnWebShared/nuget/v3/index.json`
1. Vuelva a la ventana **Visual Studio**.
1. En la ventana de Visual Studio, haga clic en el encabezado del menú **Tools**; en el menú desplegable, seleccione **NuGet Package Manager** y, en el submenú, elija **Package Manager Settings**.
1. En el cuadro de diálogo **Options**, haga clic en **Package Sources** y luego en el signo más para agregar una nueva fuente de paquetes.
1. En la parte inferior del cuadro de diálogo, en el cuadro de texto **Name**, reemplace **Package source** por **eShopOnWebShared** y, en el cuadro **Source**, pegue la URL que copió en el portal de Azure DevOps.
1. Haga clic en **Update** y luego en **OK** para finalizar la adición.

   > **Nota**: Visual Studio ya está conectado al nuevo feed.

#### Tarea 2: Crear y publicar un paquete NuGet desarrollado internamente

En esta tarea, creará y publicará un paquete NuGet personalizado desarrollado internamente.

1. En la ventana de Visual Studio que usó para configurar la nueva fuente de paquetes, en el menú principal haga clic en **File**, en el menú desplegable haga clic en **New** y luego, en el submenú, haga clic en **Project**.

   > **Nota**: ahora crearemos un ensamblado compartido que se publicará como un paquete NuGet para que otros equipos puedan integrarlo y mantenerse actualizados sin tener que trabajar directamente con el código fuente del proyecto.

1. En el panel **Create a new project**, use el cuadro de búsqueda para localizar la plantilla **Class Library**, seleccione la plantilla de C# destinada a .NET o .NET Standard y haga clic en **Next**.
1. En la página **Class Library** del panel **Create a new project**, especifique la siguiente configuración y haga clic en **Create**:

   | Configuración | Valor |
   | ------------- | ----- |
   | Project name | **eShopOnWeb.Shared** |
   | Location | acepte el valor predeterminado |
   | Solution | **Create new solution** |
   | Solution name | **eShopOnWeb.Shared** |

   Marque la casilla **Place solution and project in the same directory**.

1. Haga clic en **Next**. Acepte **.NET 8** como opción de Framework.
1. Confirme la creación del proyecto pulsando el botón **Create**.
1. Dentro de la interfaz de Visual Studio, en el panel **Solution Explorer**, haga clic con el botón derecho en **Class1.cs**, seleccione **Delete** en el menú contextual y, cuando se le solicite confirmación, haga clic en **OK**.
1. Presione **Ctrl+Shift+B** o haga clic con el botón derecho en el proyecto **EShopOnWeb.Shared** y seleccione **Build** para compilar el proyecto.
1. Desde la estación de trabajo del laboratorio, abra el menú Inicio y busque **Windows PowerShell**. A continuación, en el menú desplegable, haga clic en **Open Windows PowerShell as administrator**.
1. En la ventana **Administrator: Windows PowerShell**, navegue a la carpeta eShopOnWeb.Shared ejecutando el siguiente comando:

   ```powershell
   cd c:\eShopOnWeb\eShopOnWeb.Shared
   ```

   > **Nota**: la carpeta **eShopOnWeb.Shared** es la ubicación del archivo **eShopOnWeb.Shared.csproj**. Si eligió una ubicación o un nombre de proyecto diferentes, navegue a esa ubicación en su lugar.

1. Ejecute lo siguiente para crear un archivo **.nupkg** a partir del proyecto (cambie el valor del marcador de posición `XXXXXX` por una cadena única):

   ```powershell
   dotnet pack .\eShopOnWeb.Shared.csproj -p:PackageId=eShopOnWeb-XXXXXX.Shared
   ```

   > **Nota**: el comando **dotnet pack** compila el proyecto y crea un paquete NuGet en la carpeta **bin\Release**. Si no tiene una carpeta **Release**, puede usar la carpeta **Debug** en su lugar.

   > **Nota**: ignore cualquier advertencia que aparezca en la ventana **Administrator: Windows PowerShell**.

   > **Nota**: **dotnet pack** crea un paquete mínimo basado en la información que puede identificar del proyecto. El argumento `-p:PackageId=eShopOnWeb-XXXXXX.Shared` le permite crear un paquete con un nombre específico en lugar de usar el nombre contenido en el proyecto. Por ejemplo, si sustituye la cadena `12345` por el marcador de posición `XXXXXX`, el nombre del paquete será **eShopOnWeb-12345.Shared.1.0.0.nupkg**. El número de versión se obtuvo del ensamblado.

1. En la ventana de PowerShell, ejecute el siguiente comando para abrir la carpeta **bin\Release**:

   ```powershell
   cd .\bin\Release
   ```

1. Ejecute lo siguiente para publicar el paquete en el feed **eShopOnWebShared**. Reemplace la fuente con la URL que copió anteriormente desde la URL **Source** de Visual Studio `https://pkgs.dev.azure.com/Azure-DevOps-Org-Name/_packaging/eShopOnWebShared/nuget/v3/index.json`

   ```powershell
   dotnet nuget push --source "https://pkgs.dev.azure.com/Azure-DevOps-Org-Name/_packaging/eShopOnWebShared/nuget/v3/index.json" --api-key az "eShopOnWeb-XXXXXX.Shared.1.0.0.nupkg"
   ```

   > **Importante**: si recibe un error de autorización (401 Unauthorized), deberá instalar el proveedor de credenciales para su sistema operativo para poder autenticarse con Azure DevOps. Puede encontrar las instrucciones de instalación en [Azure Artifacts Credential Provider](https://go.microsoft.com/fwlink/?linkid=2099625). Puede instalarlo ejecutando el siguiente comando en la ventana de PowerShell: `iex "& { $(irm https://aka.ms/install-artifacts-credprovider.ps1) } -AddNetfx"`

   > **Nota**: debe proporcionar una **API Key**, que puede ser cualquier cadena no vacía. Aquí estamos usando **az**. Cuando se le solicite, inicie sesión en su organización de Azure DevOps.

   > **Nota**: si la solicitud no aparece, o recibe la advertencia **warn : The plugin credential provider could not acquire credentials. Authentication may require manual action. Consider re-running the command with --interactive for `dotnet`, /p:NuGetInteractive=true for MSBuild or removing the -NonInteractive switch for NuGet"**, puede agregar el parámetro **--interactive** al comando.

1. Espere la confirmación de la operación correcta de carga del paquete.
1. Cambie a la ventana del navegador que muestra el portal de Azure DevOps y, en el panel de navegación vertical, seleccione **Artifacts**.
1. En el panel del centro **Artifacts**, haga clic en la lista desplegable en la parte superior izquierda y, en la lista de feeds, seleccione la entrada **eShopOnWebShared**.

   > **Nota**: el feed **eShopOnWebShared** debería incluir el paquete NuGet recién publicado.

1. Haga clic en el paquete NuGet para mostrar sus detalles.

#### Tarea 3: Importar un paquete NuGet de código abierto al feed de paquetes de Azure DevOps

Además de desarrollar sus propios paquetes, ¿por qué no usar la biblioteca de paquetes NuGet de código abierto (<https://www.nuget.org>) de .NET? Con varios millones de paquetes disponibles, siempre habrá algo útil para su aplicación.

En esta tarea, usaremos un paquete de ejemplo genérico "Newtonsoft.Json", pero puede usar el mismo enfoque para otros paquetes de la biblioteca.

1. Desde la misma ventana de PowerShell utilizada en la tarea anterior para cargar el nuevo paquete, vuelva a la carpeta **eShopOnWeb.Shared** (`cd ../..`) y ejecute el siguiente comando de **dotnet** para instalar el paquete de ejemplo:

   ```powershell
   dotnet add package Newtonsoft.Json
   ```

1. Revise la salida del proceso de instalación. Muestra los distintos feeds desde los que intentará descargar el paquete:

   ```powershell
   Feeds used:
     https://api.nuget.org/v3/registration5-gz-semver2/newtonsoft.json/index.json
     https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/eShopOnWeb/_packaging/eShopOnWebShared/nuget/v3/index.json
   ```

1. A continuación, se mostrará una salida adicional sobre el proceso de instalación en sí.

   ```powershell
   Determining projects to restore...
   Writing C:\Users\AppData\Local\Temp\tmpxnq5ql.tmp
   info : X.509 certificate chain validation will use the default trust store selected by .NET for code signing.
   info : X.509 certificate chain validation will use the default trust store selected by .NET for timestamping.
   info : Adding PackageReference for package 'Newtonsoft.Json' into project 'c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj'.
   info :   GET https://api.nuget.org/v3/registration5-gz-semver2/newtonsoft.json/index.json
   info :   OK https://api.nuget.org/v3/registration5-gz-semver2/newtonsoft.json/index.json 124ms
   info : Restoring packages for c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj...
   info :   GET https://api.nuget.org/v3/vulnerabilities/index.json
   info :   OK https://api.nuget.org/v3/vulnerabilities/index.json 84ms
   info :   GET https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/vulnerability.base.json
   info :   GET https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/2024.02.17.11.23.35/vulnerability.update.json
   info :   OK https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/vulnerability.base.json 14ms
   info :   OK https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/2024.02.17.11.23.35/vulnerability.update.json 30ms
   info : Package 'Newtonsoft.Json' is compatible with all the specified frameworks in project 'c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj'.
   info : PackageReference for package 'Newtonsoft.Json' version '13.0.3' added to file 'c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj'.
   info : Writing assets file to disk. Path: c:\eShopOnWeb\eShopOnWeb.Shared\obj\project.assets.json
   log  : Restored c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj (in 294 ms).
   ```

1. El paquete Newtonsoft.Json se instaló en Packages como **Newtonsoft.Json**. En **Solution Explorer** de Visual Studio, navegue hasta el proyecto **eShopOnWeb.Shared**, expanda **Dependencies** y observe **Newtonsoft.Json** bajo **Packages**. Haga clic en la flecha pequeña a la izquierda de **Packages** para abrir la lista de carpetas y archivos.

Cuando creó el feed de paquetes de Azure DevOps Artifacts, por diseño, permite **fuentes upstream**, como nuget.org en el ejemplo de dotnet que importa el paquete Newtonsoft.Json desde donde se aloja. Esta es una práctica común para evitar la duplicación de paquetes y garantizar que siempre se use la última versión.

1. Desde el portal de Azure DevOps, **actualice** la página del feed de paquetes de Artifacts. La lista de paquetes muestra tanto el paquete personalizado **eShopOnWeb.Shared** como el paquete público **Newtonsoft.Json**.
1. Desde la solución **eShopOnWeb.Shared** en Visual Studio, haga clic con el botón derecho en el proyecto **eShopOnWeb.Shared** y seleccione **Manage NuGet Packages** en el menú contextual.
1. En la ventana del NuGet Package Manager, valide que la **Package Source** esté establecida en **eShopOnWebShared**.
1. Haga clic en **Browse** y espere a que cargue la lista de paquetes NuGet.
1. Esta lista también mostrará tanto el paquete personalizado **eShopOnWeb.Shared** como el paquete público **Newtonsoft.Json**.

## Revisión

En este laboratorio, aprendió a trabajar con Azure Artifacts siguiendo estos pasos:

- Creó y conectó un feed.
- Creó y publicó un paquete NuGet.
- Importó un paquete NuGet desarrollado de forma personalizada.
