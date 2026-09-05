---
lab:
    title: 'Compartir conocimiento del equipo usando Azure Project Wiki'
    module: 'Módulo 08: Implementar retroalimentación continua'
---

# Compartir conocimiento del equipo usando Azure Project Wiki

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [explorador compatible con Azure DevOps.](https://docs.microsoft.com/azure/devops/server/compatibility)

- **Configurar una organización de Azure DevOps:** si aún no tiene una organización de Azure DevOps que pueda usar para este laboratorio, cree una siguiendo las instrucciones disponibles en [Requisitos previos del laboratorio AZ-400](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

- **Configurar el proyecto de ejemplo eShopOnWeb:** si aún no tiene el proyecto de ejemplo eShopOnWeb que pueda usar para este laboratorio, cree uno siguiendo las instrucciones disponibles en [Requisitos previos del laboratorio AZ-400](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

## Información general del laboratorio

En este laboratorio, creará y configurará una wiki en Azure DevOps, incluida la administración de contenido markdown y la creación de un diagrama Mermaid.

## Objetivos

Después de completar este laboratorio, podrá:

- Crear una wiki en un Azure Project.
- Agregar y editar markdown.
- Crear un diagrama Mermaid.

## Tiempo estimado: 45 minutos

## Instrucciones

### Ejercicio 1: Publicar código como wiki

En este ejercicio, recorrerá el proceso de publicar un repositorio de Azure DevOps como wiki y administrar la wiki publicada.

> **Nota**: El contenido que mantiene en un repositorio Git puede publicarse en una wiki de Azure DevOps. Por ejemplo, el contenido escrito para respaldar un software development kit, documentación de producto o archivos README puede publicarse directamente en una wiki. Tiene la opción de publicar varias wikis dentro del mismo proyecto de equipo de Azure DevOps.

#### Tarea 1: Publicar una rama de un repositorio de Azure DevOps como wiki

En esta tarea, publicará una rama de un repositorio de Azure DevOps como wiki.

> **Nota**: Si la wiki publicada corresponde a una versión de producto, puede publicar nuevas ramas a medida que publique nuevas versiones del producto.

1. En el menú vertical del lado izquierdo, haga clic en **Repos**. En la sección superior del panel **Files**, asegúrese de tener seleccionado el repo **eShopOnWeb**; elíjalo desde la lista desplegable superior con el icono de Git. En la lista desplegable de ramas, ubicada encima de "Files" con el icono de rama, seleccione **main** y revise el contenido de la rama main.

1. A la izquierda del panel **Files**, en la lista de la jerarquía de carpetas y archivos del repo, expanda la carpeta **src** y navegue a la subcarpeta **Web -> wwwroot -> images**. En la subcarpeta **Images**, localice la entrada **brand.png**, mantenga el puntero del mouse sobre su extremo derecho para mostrar el símbolo de puntos suspensivos verticales, tres puntos, que representa el menú **More**, y haga clic en **Download** para descargar el archivo **brand.png** en la carpeta local **Downloads** del equipo de laboratorio.

    > **Nota**: Usará esta imagen en el siguiente ejercicio de este laboratorio.

1. Almacenaremos los archivos de origen de la Wiki en una carpeta separada dentro de la estructura de carpetas actual de Repos. Desde **Repos**, seleccione **Files**. Observe el título del repo **eShopOnWeb** en la parte superior de la estructura de carpetas. **Seleccione los puntos suspensivos (3 puntos)**, elija **New / Folder** y proporcione **`Documents`** como título para el nombre de la nueva carpeta. Como un repo no permite crear una carpeta vacía, proporcione **`READ.ME`** como nombre del nuevo archivo.

1. Confirme la creación de la carpeta y del archivo presionando el botón **Create**.

1. El archivo READ.ME se abrirá en el modo de vista integrado. Como esto se almacena "como código", debe hacer **Commit** de los cambios haciendo clic en el botón **Commit**. En la ventana Commit, confirme una vez más presionando **Commit**.

1. En el menú vertical de Azure DevOps del lado izquierdo, haga clic en **Overview**. En la sección **Overview**, seleccione **Wiki** y seleccione **Publish code as wiki*.

1. En el panel **Publish code as wiki**, especifique la siguiente configuración y haga clic en **Publish**.

    | Configuración | Valor |
    | ------- | ----- |
    | Repository | **eShopOnWeb** |
    | Branch | **main** |
    | Folder | **/Documents** |
    | Wiki name | **`eShopOnWeb (Documents)`** |

    > **Nota**: Esto abrirá automáticamente la sección Wiki y publicará **el editor**, donde puede proporcionar un título de página de Wiki, además de agregar el contenido real. Observe que se le recomienda usar formato MarkDown, pero puede usar la cinta de opciones para apoyarse con parte de la sintaxis de diseño de MarkDown.

1. En el campo **Title** de la Wiki Page, escriba "`Welcome to our Online Retail Store!`".

1. En el cuerpo de la Wiki Page, pegue el siguiente texto:

    ```text
    ##Welcome to Our Online Retail Store!
    At our online retail store, we offer a **wide range of products** to meet the **needs of our customers**. Our selection includes everything from *clothing and accessories to electronics, home decor, and more*.
    
    We pride ourselves on providing a seamless shopping experience for our customers. Our website offers the following benefits:
    1. user-friendly,
    1. and easy to navigate, 
    1. allowing you to find what you're looking for,
    1. quickly and easily. 
    
    We also offer a range of **_payment and shipping options_** to make your shopping experience as convenient as possible.
    
    ### about the team
    Our team is dedicated to providing exceptional customer service. If you have any questions or concerns, our knowledgeable and friendly support team is always available to assist you. We also offer a hassle-free return policy, so if you're not completely satisfied with your purchase, you can easily return it for a refund or exchange.
    
    ### Physical Stores
    |Location|Area|Hours|
    |--|--|--|
    | New Orleans | Home and DIY  |07.30am-09.30pm  |
    | Seattle | Gardening | 10.00am-08.30pm  |
    | New York | Furniture Specialists  | 10.00am-09.00pm |
    
    ## Our Store Qualities
    - We're committed to providing high-quality products
    - Our products are offered at affordable prices 
    - We work with reputable suppliers and manufacturers 
    - We ensure that our products meet our strict standards for quality and durability. 
    - Plus, we regularly offer sales and discounts to help you save even more.
    
    #Summary
    Thank you for choosing our online retail store for your shopping needs. We look forward to serving you!
    ```

1. Este texto de ejemplo le ofrece una visión general de varias características comunes de sintaxis MarkDown, que van desde títulos y subtítulos (##), negrita (**), cursiva (*), cómo crear tablas y más.

1. Cuando termine, presione el botón **Save** en la esquina superior derecha.

1. **Actualice** el explorador, o seleccione cualquier otra opción del portal de DevOps y vuelva a la sección Wiki. Observe que ahora se le presenta la Wiki **EshopOnWeb (Documents)**, y que **Welcome to our Online Retail Store** aparece como **HomePage** de la Wiki.

#### Tarea 2: Administrar el contenido de una wiki publicada

En esta tarea, administrará el contenido de la wiki que publicó en la tarea anterior.

1. En el menú vertical del lado izquierdo, haga clic en **Repos**. Asegúrese de que el menú desplegable en la sección superior del panel **Files** muestre el repo **eShopOnWeb** y la rama **main**. En la jerarquía de carpetas del repo, seleccione la carpeta **Documents** y seleccione el archivo **Welcome-to-our-Online-Retail-Store!.md**.

1. Observe cómo el formato MarkDown es visible aquí como texto sin formato, lo que le permite continuar editando el contenido del archivo desde esta ubicación.

> **Nota**: Como los archivos de origen de la Wiki se tratan como código fuente, recuerde que todas las prácticas del control de código fuente tradicional, como Clone, Pull Requests, Approvals y más, ahora también pueden aplicarse a las páginas Wiki.

### Ejercicio 2: Crear y administrar una project wiki

En este ejercicio, recorrerá el proceso de crear y administrar una project wiki.

> **Nota**: Puede crear y administrar una wiki de forma independiente de los repos existentes.

#### Tarea 1: Crear una project wiki que incluya un diagrama Mermaid y una imagen

En esta tarea, creará una project wiki y le agregará un diagrama Mermaid y una imagen.

1. En el equipo de laboratorio, en el portal de Azure DevOps que muestra el **Wiki pane** del proyecto **EShopOnweb**, con el contenido de la wiki **eShopOnWeb (Documents)** seleccionado, en la parte superior del panel, haga clic en el encabezado de lista desplegable **eShopOnWeb (Documents)**, el icono de flecha hacia abajo, y en la lista desplegable seleccione **Create new project wiki**.

1. En el cuadro de texto **Page title**, escriba **`Project Design`**.

1. Coloque el cursor en el cuerpo de la página, haga clic en el icono más a la izquierda de la barra de herramientas que representa la configuración de encabezado y, en la lista desplegable, haga clic en **Header 1**. Esto agregará automáticamente el carácter numeral (**#**) al principio de la línea.

1. Directamente después del carácter **#** recién agregado, escriba **`Authentication and Authorization`** y presione la tecla **Enter**.

1. Haga clic en el icono más a la izquierda de la barra de herramientas que representa la configuración de encabezado y, en la lista desplegable, haga clic en **Header 2**. Esto agregará automáticamente el carácter numeral doble (**##**) al principio de la línea.

1. Directamente después de los caracteres **##** recién agregados, escriba **`Azure DevOps OAuth 2.0 Authorization Flow`** y presione la tecla **Enter**.

1. **Copie y pegue** el siguiente código para insertar un diagrama mermaid en la wiki.

    ```text
    ::: mermaid
    sequenceDiagram
     participant U as User
     participant A as Your app
     participant D as Azure DevOps
     U->>A: Use your app
     A->>D: Request authorization for user
     D-->>U: Request authorization
     U->>D: Grant authorization
     D-->>A: Send authorization code
     A->>D: Get access token
     D-->>A: Send access token
     A->>D: Call REST API with access token
     D-->>A: Respond to REST API
     A-->>U: Relay REST API response
    :::
    ```

    > **Nota**: Para obtener detalles sobre la sintaxis Mermaid, consulte [About Mermaid](https://mermaid-js.github.io/mermaid/#/).

1. A la derecha del panel del editor, en el panel de vista previa, haga clic en **Load diagram** y revise el resultado.

    > **Nota**: La salida debe parecerse al diagrama de flujo que ilustra cómo [Authorize access to REST APIs with OAuth 2.0](https://docs.microsoft.com/azure/devops/integrate/get-started/authentication/oauth).

1. En la esquina superior derecha del panel del editor, haga clic en el símbolo de intercalación hacia abajo junto al botón **Save** y, en el menú desplegable, haga clic en **Save with revision message**.

1. En el cuadro de diálogo **Save page**, escriba **`Authentication and authorization section with the OAuth 2.0 Mermaid diagram`** y haga clic en **Save**.

1. En el panel del editor de **Project Design**, coloque el cursor al final del elemento Mermaid que agregó anteriormente en esta tarea, presione la tecla **Enter** para agregar una línea adicional, haga clic en el icono más a la izquierda de la barra de herramientas que representa la configuración de encabezado y, en la lista desplegable, haga clic en **Header 2**. Esto agregará automáticamente el carácter numeral doble (**##**) al principio de la línea.

1. Directamente después de los caracteres **##** recién agregados, escriba **`User Interface`** y presione la tecla **Enter**.

1. En el panel del editor de **Project Design**, en la barra de herramientas, haga clic en el icono de clip que representa la acción **Insert a file**. En el cuadro de diálogo **Open**, navegue a la carpeta **Downloads**, seleccione el archivo **Brand.png** que descargó en el ejercicio anterior y haga clic en **Open**.

1. De vuelta en el panel del editor de **Project Design**, revise el panel de vista previa y compruebe que la imagen se muestra correctamente.

1. En la esquina superior derecha del panel del editor, haga clic en el símbolo de intercalación hacia abajo junto al botón **Save** y, en el menú desplegable, haga clic en **Save with revision message**.

1. En el cuadro de diálogo **Save page**, escriba **`User Interface section with the eShopOnWeb image`** y haga clic en **Save**.

1. De vuelta en el panel del editor, en la esquina superior derecha, haga clic en **Close**.

#### Tarea 2: Administrar una project wiki

En esta tarea, administrará la project wiki recién creada.

> **Nota**: Comenzará revirtiendo el cambio más reciente en la página wiki.

1. En el equipo de laboratorio, en el portal de Azure DevOps que muestra el **Wiki pane** del proyecto **eShopOnWeb**, con el contenido de la wiki **Project Design** seleccionado, en la esquina superior derecha, haga clic en el símbolo de puntos suspensivos verticales y, en el menú desplegable, haga clic en **View revisions**.

1. En el panel **Revisions**, haga clic en la entrada que representa el cambio más reciente.

1. En el panel resultante, revise la comparación entre la versión anterior y la versión actual del documento, haga clic en **Revert**; cuando se le solicite confirmación, haga clic en **Revert** nuevamente y luego haga clic en **Browse Page**.

1. De vuelta en el panel **Project Design**, compruebe que el cambio se revirtió correctamente.

    > **Nota**: Ahora agregará otra página a la project wiki y la establecerá como página principal de la wiki.

1. En el panel **Project Design**, en la esquina inferior izquierda, haga clic en **+ New page**.

1. En el panel del editor de páginas, en el cuadro de texto **Page title**, escriba **`Project Design Overview`**, haga clic en **Save** y, a continuación, haga clic en **Close**.

1. De vuelta en el panel que lista las páginas dentro de la project wiki **Project Design**, ubique la entrada **Project Design Overview**, selecciónela con el puntero del mouse, arrástrela y suéltela encima de la entrada de página **Project Design**.

1. Confirme los cambios presionando el botón **Move** en la ventana que aparece.

1. Compruebe que la entrada **Project Design Overview** aparece como página de nivel superior con el icono de inicio que la designa como página principal de la wiki.

## Revisión

En este laboratorio, creó y configuró una Wiki en Azure DevOps, incluida la administración de contenido markdown y la creación de un diagrama Mermaid.
