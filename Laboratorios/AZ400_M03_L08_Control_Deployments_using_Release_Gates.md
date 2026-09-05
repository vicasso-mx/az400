---
lab:
  title: "Controlar implementaciones mediante release gates"
  module: "Módulo 03: Diseñar e implementar una estrategia de publicación"
---

# Controlar implementaciones mediante release gates

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [navegador compatible con Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configure una organización de Azure DevOps:** si aún no tiene una organización de Azure DevOps que pueda usar para este laboratorio, créela siguiendo las instrucciones disponibles en [Create an organization or project collection](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifique una suscripción de Azure existente o cree una nueva.

- Verifique que tiene una cuenta de Microsoft o una cuenta de Microsoft Entra con el rol de Propietario en la suscripción de Azure y el rol de Administrador global en el inquilino de Microsoft Entra asociado a la suscripción de Azure. Para obtener más información, consulte [List Azure role assignments using the Azure portal](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) y [View and assign administrator roles in Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Descripción general del laboratorio

Este laboratorio cubre la configuración de las puertas de implementación (deployment gates) y explica cómo usarlas para controlar la ejecución de Azure Pipelines. Para ilustrar su implementación, configurará una definición de publicación con dos entornos para una aplicación web de Azure. Implementará en el entorno DevTest solo cuando no haya errores bloqueantes para la aplicación y marcará el entorno DevTest como completado solo cuando no haya alertas activas en Application Insights de Azure Monitor.

Una canalización de publicación especifica el proceso de publicación de extremo a extremo para una aplicación que se implementará en varios entornos. Las implementaciones en cada entorno están completamente automatizadas mediante trabajos y tareas. Lo ideal es que las nuevas actualizaciones de las aplicaciones no se expongan simultáneamente a todos los usuarios. Es una buena práctica exponerlas de forma gradual, es decir, publicarlas para un subconjunto de usuarios, supervisar su uso y luego exponerlas al resto según la experiencia del conjunto inicial de usuarios.

Las aprobaciones y los gates le permiten controlar el inicio y la finalización de las implementaciones en una publicación. Puede esperar a que los usuarios aprueben o rechacen las implementaciones manualmente con aprobaciones. Mediante release gates, puede especificar criterios de salud de la aplicación que deben cumplirse antes de promocionar la publicación al siguiente entorno. Antes o después de cualquier implementación del entorno, se evalúan automáticamente todas las puertas especificadas hasta que pasan o alcanzan el tiempo de espera definido y fallan.

Los gates se pueden agregar a un entorno en la definición de publicación desde los paneles de condiciones previas a la implementación o posteriores a la implementación. Se pueden agregar varios gates a las condiciones del entorno para garantizar que todas las entradas sean correctas para la publicación.

Como ejemplo:

- Los gates previos a la implementación garantizan que no haya problemas activos en el sistema de gestión de elementos de trabajo o de problemas antes de implementar una compilación en un entorno.
- Los gates posteriores a la implementación garantizan que no haya incidentes en el sistema de supervisión o gestión de incidentes de la aplicación después de la implementación, antes de promocionar la publicación al siguiente entorno.

Hay 4 tipos de gates incluidos de forma predeterminada en cada cuenta.

- Invoke Azure Function: ejecuta una Azure Function y garantiza que se complete correctamente.
- Query Azure Monitor alerts: supervisa las reglas de alerta configuradas de Azure Monitor para alertas activas.
- Invoke REST API: realiza una llamada a una REST API y continúa si devuelve una respuesta correcta.
- Query work items: garantiza que el número de elementos de trabajo coincidentes devueltos por una consulta se encuentre dentro de un umbral.

## Objetivos

Después de completar este laboratorio, podrá:

- Configurar canalizaciones de publicación.
- Configurar release gates.
- Probar release gates.

## Tiempo estimado: 75 minutos

## Instrucciones

### Ejercicio 0: Configure los requisitos previos del laboratorio

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
   - La carpeta **src** contiene el sitio web de .NET 8 que se usa en los escenarios del laboratorio.

1. Vaya a **Repos > Branches**.
1. Pase el cursor sobre la rama **main** y luego haga clic en los puntos suspensivos a la derecha de la columna.
1. Haga clic en **Set as default branch**.

#### Tarea 3: Configurar la canalización de CI como código con YAML en Azure DevOps

En esta tarea, agregará una definición de compilación YAML al proyecto existente.

1. Regrese al panel **Pipelines** del centro **Pipelines**.
1. En la ventana **Create your first Pipeline**, haga clic en **Create pipeline**.

   > **Nota**: usaremos el asistente para crear una nueva definición de canalización YAML basada en nuestro proyecto.

1. En el panel **Where is your code?**, haga clic en la opción **Azure Repos Git (YAML)**.
1. En el panel **Select a repository**, haga clic en **eShopOnWeb**.
1. En el panel **Configure your pipeline**, desplácese hacia abajo y seleccione **Existing Azure Pipelines YAML File**.
1. En el panel **Selecting an existing YAML File**, especifique los siguientes parámetros:
   - Branch: **main**
   - Path: **.ado/eshoponweb-ci.yml**
1. Haga clic en **Continue** para guardar estos ajustes.
1. En la pantalla **Review your Pipeline YAML**, haga clic en **Run** para iniciar el proceso de canalización de compilación.
1. Espere a que la canalización de compilación se complete correctamente. Ignore cualquier advertencia relacionada con el código fuente, ya que no son relevantes para este ejercicio del laboratorio.

   > **Nota**: cada tarea del archivo YAML está disponible para revisión, incluidas las advertencias y los errores.

1. La canalización tomará un nombre basado en el nombre del proyecto. Vamos a **renombrarla** para identificarla mejor. Vaya a **Pipelines > Pipelines** y haga clic en la canalización creada recientemente. Haga clic en los puntos suspensivos y seleccione la opción **Rename/move**. Asigne el nombre **`eshoponweb-ci`** y haga clic en **Save**.

### Ejercicio 1: Crear los recursos de Azure necesarios para la canalización de publicación

#### Tarea 1: Crear dos aplicaciones web de Azure

En esta tarea, creará dos aplicaciones web de Azure que representan los entornos **DevTest** y **Production**, en los que implementará la aplicación mediante Azure Pipelines.

1. Desde el equipo del laboratorio, inicie un navegador web, vaya al [**Azure Portal**](https://portal.azure.com) e inicie sesión con la cuenta de usuario que tiene el rol de Propietario en la suscripción de Azure que se usará en este laboratorio y tiene el rol de Administrador global en el inquilino de Microsoft Entra asociado a esa suscripción.
1. En Azure Portal, haga clic en el icono de **Cloud Shell**, ubicado directamente a la derecha del cuadro de búsqueda en la parte superior de la página.
1. Si se le solicita que seleccione **Bash** o **PowerShell**, elija **Bash**.

   > **Nota**: si es la primera vez que inicia **Cloud Shell** y aparece el mensaje **You have no storage mounted**, seleccione la suscripción que está usando en este laboratorio y haga clic en **Apply**.

1. En el símbolo del sistema **Bash** del panel de Cloud Shell, ejecute el siguiente comando para crear un grupo de recursos (reemplace el marcador de posición `<region>` por el nombre de la región de Azure que alojará las dos aplicaciones web, por ejemplo 'westeurope', 'centralus' o cualquier otra región disponible de su elección):

   > **Nota**: las ubicaciones posibles se pueden consultar ejecutando el siguiente comando y usando el valor **Name** en `<region>`: `az account list-locations -o table`

   ```bash
   REGION='<region>'
   RESOURCEGROUPNAME='az400m03l08-RG'
   az group create -n $RESOURCEGROUPNAME -l $REGION
   ```

1. Para crear un plan de servicio de aplicaciones:

   ```bash
   SERVICEPLANNAME='az400m03l08-sp1'
   az appservice plan create -g $RESOURCEGROUPNAME -n $SERVICEPLANNAME --sku S1
   ```

1. Cree dos aplicaciones web con nombres únicos.

   ```bash
   SUFFIX=$RANDOM$RANDOM
   az webapp create -g $RESOURCEGROUPNAME -p $SERVICEPLANNAME -n RGATES$SUFFIX-DevTest
   az webapp create -g $RESOURCEGROUPNAME -p $SERVICEPLANNAME -n RGATES$SUFFIX-Prod
   ```

   > **Nota**: anote el nombre de la aplicación web DevTest. Lo necesitará más adelante en este laboratorio.

   > **Nota**: si encuentra un error que dice "The subscription is not registered to use namespace 'Microsoft.Web'", ejecute el siguiente comando para registrar el proveedor de recursos Microsoft.Web: `az provider register --namespace Microsoft.Web` y espere a que termine el registro antes de crear las aplicaciones web.

1. Espere a que finalice el proceso de aprovisionamiento de los recursos de App Service y cierre el panel de **Cloud Shell**.

#### Tarea 2: Configurar un recurso de Application Insights

1. En Azure Portal, use el cuadro de texto **Search resources, services, and docs** en la parte superior de la página para buscar **`Application Insights`** y, en la lista de resultados, seleccione **Application Insights**.
1. En el panel de **Application Insights**, seleccione **+ Create**.
1. En el panel de **Application Insights**, en la pestaña **Basics**, especifique la siguiente configuración (deje los demás valores con sus valores predeterminados):

   | Configuración | Valor |
   | -------------- | ----- |
   | Resource group | **az400m03l08-RG** |
   | Name | el nombre de la aplicación web DevTest que registró en la tarea anterior |
   | Region | la misma región de Azure en la que implementó las aplicaciones web al principio de la tarea anterior |

1. Haga clic en **Review + create** y luego en **Create**.
1. Espere a que termine el proceso de aprovisionamiento.
1. En Azure Portal, navegue hasta el grupo de recursos **az400m03l08-RG** que creó en la tarea anterior.
1. En la lista de recursos, haga clic en la aplicación web **DevTest**.
1. En la página de la aplicación web **DevTest**, en el menú vertical de la izquierda, dentro de la sección **Monitoring**, haga clic en **Application Insights**.
1. En el panel **Application Insights**, haga clic en **Turn on Application Insights**.
1. En la sección **Change your resource**, haga clic en la opción **Select existing resource**; en la lista de recursos existentes, seleccione el recurso de Application Insights recién creado, haga clic en **Apply** y, cuando se le solicite confirmación, haga clic en **Yes**.
1. Espere a que el cambio surta efecto.

   > **Nota**: aquí creará alertas de monitorización que usará en la parte posterior de este laboratorio.

1. A partir de la misma opción de menú **Settings** / **Application Insights** dentro de la aplicación web, seleccione **View Application Insights Data**. Esto lo redirige al panel de Application Insights en Azure Portal.
1. En el panel del recurso de Application Insights, en la sección **Monitoring**, haga clic en **Alerts** y luego en **Create > Alert rule**.
1. En el panel **Create an Alert Rule**, en la sección **Condition**, haga clic en el enlace **See all signals**, escriba **Requests**. En la lista de resultados, seleccione **Failed Requests**.
1. En el panel **Create an Alert Rule**, en la sección **Condition**, deje **Threshold** en **Static**, valide los demás valores predeterminados como se indica a continuación:

   - Aggregation Type: Count
   - Operator: Greater Than
   - Unit: Count

1. En el cuadro de texto **Threshold value**, escriba **0** y haga clic en **Next:Actions**. No realice cambios en el panel de configuración **Actions** y defina los siguientes parámetros bajo la sección **Details**:

   | Configuración | Valor |
   | ------------- | ----- |
   | Severity | **2- Warning** |
   | Alert rule name | **RGATESDevTest_FailedRequests** |
   | Advanced Options: Automatically resolve alerts | **cleared** |

   > **Nota**: las reglas de alerta de métricas pueden tardar hasta 10 minutos en activarse.

   > **Nota**: puede crear varias reglas de alerta en distintas métricas, como disponibilidad < 99 %, tiempo de respuesta del servidor > 5 segundos o excepciones del servidor > 0.

1. Confirme la creación de la regla de alerta haciendo clic en **Review+Create** y vuelva a confirmar haciendo clic en **Create**. Espere a que la regla de alerta se cree correctamente.

### Ejercicio 2: Configurar la canalización de publicación

En este ejercicio, configurará una canalización de publicación.

#### Tarea 1: Configurar tareas de publicación

En esta tarea, configurará las tareas de publicación como parte de la canalización de publicación.

1. Desde el proyecto **eShopOnWeb** en el portal de Azure DevOps, en el panel de navegación vertical, seleccione **Pipelines** y luego, dentro de la sección **Pipelines**, haga clic en **Releases**.
   > **Nota**: si la sección **Releases** no está visible, probablemente se debe a que la opción **Disable creation of classic release pipelines** está activada en **Project Settings** > **Pipelines** > **Settings** > **General**.
1. Haga clic en **New Pipeline**.
1. En la ventana **Select a template**, elija **Azure App Service Deployment** (Implementa tu aplicación en Azure App Service. Elige Web App en Windows, Linux, contenedores, Function Apps o WebJobs) dentro de la lista **Featured**.
1. Haga clic en **Apply**.
1. En la ventana **Stage** que aparece, actualice el nombre predeterminado de la etapa "Stage 1" a **`DevTest`**. Cierre la ventana emergente con el botón **X**. Ahora se encuentra en el editor gráfico de la canalización de publicación, mostrando la etapa DevTest.
1. En la parte superior de la página, cambie el nombre de la canalización actual de **New release pipeline** a **`eshoponweb-cd`**.
1. Pase el cursor sobre la etapa DevTest y haga clic en el botón **Clone** para copiar la etapa DevTest a una etapa adicional. Asigne a esta etapa el nombre **`Production`**.

   > **Nota**: la canalización ahora contiene dos etapas llamadas **DevTest** y **Production**.

1. En la pestaña **Pipeline**, seleccione el rectángulo **Add an Artifact** y, en el campo **Source (build pipeline)**, seleccione **eshoponweb-ci**. Haga clic en **Add** para confirmar la selección del artefacto.
1. En el rectángulo **Artifacts**, observe el disparador de implementación continua (rayo). Haga clic en él para abrir la configuración de **Continuous deployment trigger**. Haga clic en **Disabled** para activar el interruptor y habilitarlo. Deje el resto de ajustes con sus valores predeterminados y cierre el panel **Continuous deployment trigger** haciendo clic en la X en la esquina superior derecha.
1. Dentro de la etapa **DevTest Environments**, haga clic en la etiqueta **1 job, 1 task** y revise las tareas de esta etapa.

   > **Nota**: el entorno DevTest tiene 1 tarea que, en consecuencia, publica el paquete del artefacto en Azure Web App.

1. En el panel **All pipelines > eshoponweb-cd**, asegúrese de que la etapa **DevTest** esté seleccionada. En la lista desplegable **Azure subscription**, seleccione su suscripción de Azure y haga clic en **Authorize**. Si se le solicita, autentíquese con la cuenta de usuario que tiene el rol de Propietario en la suscripción de Azure.
1. Confirme que el tipo de aplicación está establecido como "Web App on Windows". A continuación, en la lista desplegable **App Service name**, seleccione el nombre de la aplicación web **DevTest**.
1. Seleccione la tarea **Deploy Azure App Service**. En el campo **Package or Folder**, actualice el valor predeterminado "$(System.DefaultWorkingDirectory)/**/* .zip" por "$(System.DefaultWorkingDirectory)/**/Web.zip".

   > **Nota**: observe un signo de exclamación junto a la pestaña Tasks. Esto es esperable, ya que aún debemos configurar la configuración para la etapa Production.

1. Abra el panel **Application and Configuration Settings** e introduzca `-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development` en el cuadro **App settings**.

1. En el panel **All pipelines > eshoponweb-cd**, vaya a la pestaña **Pipeline** y, esta vez, dentro de la etapa **Production**, haga clic en la etiqueta **1 job, 1 task**. Al igual que en la etapa DevTest, complete la configuración de la canalización. En la pestaña **Tasks** / **Production Deployment process**, en la lista desplegable **Azure subscription**, seleccione la suscripción de Azure que usó para la etapa **DevTest Environment**, mostrada en **Available Azure Service connections**, ya que la conexión al servicio se creó antes al autorizar el uso de la suscripción.
1. En la lista desplegable **App Service name**, seleccione el nombre de la aplicación web **Prod**.
1. Seleccione la tarea **Deploy Azure App Service**. En el campo **Package or Folder**, actualice el valor predeterminado "$(System.DefaultWorkingDirectory)/**/* .zip" por "$(System.DefaultWorkingDirectory)/**/Web.zip".
1. Abra el panel **Application and Configuration Settings** e introduzca `-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development` en el cuadro **App settings**.
1. En el panel **All pipelines > eshoponweb-cd**, haga clic en **Save** y, en el cuadro de diálogo **Save**, haga clic en **OK**.

   Ahora ha configurado correctamente la canalización de publicación.

1. En la ventana del navegador que muestra el proyecto **eShopOnWeb**, en el panel de navegación vertical, en la sección **Pipelines**, haga clic en **Pipelines**.
1. En el panel **Pipelines**, haga clic en la entrada que representa la canalización de compilación **eshoponweb-ci** y luego haga clic en **Run Pipeline**.
1. En el panel **Run pipeline**, acepte la configuración predeterminada y haga clic en **Run** para desencadenar la canalización. **Espere a que termine la canalización de compilación**.

   > **Nota**: cuando la compilación se complete correctamente, la publicación se activará automáticamente y la aplicación se implementará en ambos entornos. Valide las acciones de publicación cuando la canalización de compilación haya terminado correctamente.

1. En el panel de navegación vertical, dentro de la sección **Pipelines**, haga clic en **Releases** y, en el panel **eshoponweb-cd**, haga clic en la entrada que representa la versión más reciente.
1. En el panel **eshoponweb-cd > Release-1**, siga el progreso de la publicación y verifique que la implementación en ambas aplicaciones web se haya completado correctamente.
1. Cambie a la interfaz de Azure Portal, navegue hasta el grupo de recursos **az400m03l08-RG**, en la lista de recursos haga clic en la aplicación web **DevTest**; en el panel de la aplicación web, haga clic en **Browse** y verifique que la página web (sitio de comercio electrónico) se cargue correctamente en una nueva pestaña del navegador.
1. Vuelva a Azure Portal y, esta vez, navegue hasta el grupo de recursos **az400m03l08-RG**; en la lista de recursos, haga clic en la aplicación web **Production**; en el panel de la aplicación web, haga clic en **Browse** y verifique que la página web se cargue correctamente en una nueva pestaña del navegador.
1. Cierre la pestaña del navegador que muestra el sitio web **EShopOnWeb**.

   > **Nota**: ahora ya tiene la aplicación con CI/CD configurado. En el siguiente ejercicio configuraremos Quality Gates como parte de una canalización de publicación más avanzada.

### Ejercicio 3: Configurar release gates

En este ejercicio, configurará Quality Gates en la canalización de publicación.

#### Tarea 1: Configurar gates previos a la implementación para aprobaciones

En esta tarea, configurará gates previos a la implementación.

1. Cambie a la ventana del navegador que muestra el portal de Azure DevOps y abra el proyecto **eShopOnWeb**. En el panel de navegación vertical, dentro de la sección **Pipelines**, haga clic en **Releases** y, en el panel **eshoponweb-cd**, haga clic en **Edit**.
1. En el panel **All pipelines > eshoponweb-cd**, en el borde izquierdo del rectángulo que representa la etapa **DevTest Environment**, haga clic en la forma ovalada que representa las **Pre-deployment conditions**.
1. En el panel **Pre-deployment conditions**, active el interruptor **Pre-deployment approvals** y, en el cuadro de texto **Approvers**, escriba y seleccione el nombre de su cuenta de Azure DevOps.

   > **Nota**: en un escenario real, esto debería ser un alias del nombre del equipo de DevOps en lugar de su propio nombre.

1. **Guarde** la configuración de aprobación previa y cierre la ventana emergente.
1. Haga clic en **Create Release** y confirme pulsando el botón **Create** en la ventana emergente.
1. Observe el mensaje verde de confirmación que indica que se ha creado "Release-2". Haga clic en el enlace "Release-2" para ir a sus detalles.
1. Observe que la etapa **DevTest** está en estado **Pending Approval**. Haga clic en el botón **Approve**. Esto vuelve a activar la etapa DevTest.

#### Tarea 2: Configurar gates posteriores a la implementación para Azure Monitor

En esta tarea, habilitará el gate posterior a la implementación para el entorno DevTest.

1. En el panel **All pipelines > eshoponweb-cd**, en el borde derecho del rectángulo que representa la etapa **DevTest Environment**, haga clic en la forma ovalada que representa las **Post-deployment conditions**.
1. En el panel **Post-deployment conditions**, active el interruptor **Gates**, haga clic en **+ Add** y, en el menú emergente, haga clic en **Query Azure Monitor Alerts**.
1. En el panel **Post-deployment conditions**, en la sección **Query Azure Monitor Alerts**, en la lista desplegable **Azure subscription**, seleccione la entrada de **service connection** que representa la conexión a su suscripción de Azure, y en la lista desplegable **Resource group**, seleccione la entrada **az400m03l08-RG**.
1. En el panel **Post-deployment conditions**, expanda la sección **Advanced** y configure las siguientes opciones:

   - Filter type: **None**
   - Severity: **Sev0, Sev1, Sev2, Sev3, Sev4**
   - Time Range: **Past Hour**
   - Alert State: **Acknowledged, New**
   - Monitor Condition: **Fired**

1. En el panel **Post-deployment conditions**, expanda la sección **Evaluation options** y configure las siguientes opciones:

   - Establezca el valor de **Time between re-evaluation of gates** en **5 Minutes**.
   - Establezca el valor de **Timeout after which gates fail** en **8 Minutes**.
   - Seleccione la opción **On successful gates, ask for approvals**.

   > **Nota**: el intervalo de muestreo y el tiempo de espera trabajan juntos para que los gates llamen a sus funciones a intervalos adecuados y rechacen la implementación si no tienen éxito durante el mismo intervalo de muestreo dentro del período de tiempo de espera.

1. Cierre el panel **Post-deployment conditions** haciendo clic en la X en la esquina superior derecha.
1. En el panel **eshoponweb-cd**, haga clic en **Save** y, en el cuadro de diálogo **Save**, haga clic en **OK**.

### Ejercicio 4: Probar release gates

En este ejercicio, probará los release gates actualizando la aplicación, lo que desencadenará una implementación.

#### Tarea 1: Actualizar e implementar la aplicación después de agregar release gates

En esta tarea, primero generará algunas alertas para la aplicación web DevTest y luego supervisará el proceso de publicación con los release gates habilitados.

1. Desde Azure Portal, navegue hasta el recurso **DevTest Web App** implementado antes.
1. Desde el panel Overview, observe el campo **URL** que muestra el hipervínculo de la aplicación web. Haga clic en este enlace, que lo redirige a la aplicación web eShopOnWeb en el navegador.
1. Para simular una **Failed Request**, agregue **/discount** a la URL, lo que dará como resultado un mensaje de error porque esa página no existe. Actualice esta página varias veces para generar varios eventos.
1. Desde Azure Portal, en el campo "Search resources, services and docs", escriba **`Application Insights`** y seleccione el recurso **DevTest-AppInsights** creado en el ejercicio anterior. A continuación, vaya a **Alerts**.
1. Debería haber al menos **1** alerta nueva en la lista de resultados, con **Severity 2**. Entre en **`Alerts`** para abrir el servicio de alertas de Azure Monitor.
1. Observe que debería haber al menos **1** Failed_Alert con **Severity 2 - Warning** en la lista. Se activó cuando validó la dirección URL del sitio web inexistente en el ejercicio anterior.

   > **Nota**: si aún no aparece ninguna alerta, espere unos minutos más.

1. Regrese al portal de Azure DevOps, abra el proyecto **eShopOnWeb**. Vaya a **Pipelines**, seleccione **Releases** y luego **eshoponweb-cd**.
1. Haga clic en el botón **Create Release**.
1. Espere a que la canalización de publicación se inicie y **apruebe** la acción de publicación de la etapa DevTest.
1. Espere a que la etapa de publicación DevTest se complete correctamente. Observe cómo **Post-deployment Gates** cambia al estado **Evaluation Gates**. Haga clic en el icono **Evaluation Gates**.
1. Para **Query Azure Monitor Alerts**, observe un estado inicial fallido.
1. Deje la canalización de publicación en estado pendiente durante los próximos 5 minutos. Cuando pasen los 5 minutos, observe que la segunda evaluación vuelve a fallar.
1. Este es un comportamiento esperado, ya que hay alertas de Application Insights activadas para la aplicación web DevTest.

   > **Nota**: como hay una alerta activada por la excepción, el gate **Query Azure Monitor** fallará. Esto, a su vez, impedirá la implementación en el entorno **Production**.

1. Espere un par de minutos y vuelva a validar el estado de los release gates. En unos minutos, después de que se hayan revisado los release gates iniciales y como la alerta inicial de Application Insights se activó con la acción "Fired", debería resultar en un release gate correcto que permita la implementación de la etapa de publicación Production.

   > **Nota**: si el gate falla, cierre la alerta.

   > [!IMPORTANT]
   > Recuerde eliminar los recursos creados en Azure Portal para evitar cargos innecesarios.

## Revisión

En este laboratorio, configuró canalizaciones de publicación y luego configuró y probó release gates.
