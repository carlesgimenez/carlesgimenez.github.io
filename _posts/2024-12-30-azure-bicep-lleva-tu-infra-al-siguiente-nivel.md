---
title: Azure Bicep, lleva tu infraestructura al siguiente nivel
author: Carles Giménez
date: 2024-12-30 20:40:00 +0800
tags: [IaC]
category: [Azure Bicep]
image: /assets/img/bicep/azure_bicep.webp
---

Una de las prácticas fundamentales dentro de la metodología DevOps es el uso de IaC (Infrastructure as Code). Si este término te resulta nuevo, no te preocupes: en términos simples, se trata de definir, implementar y administrar infraestructura de manera programática, eliminando la necesidad de realizar configuraciones manuales en tus servicios de la nube. Azure, al igual que otros proveedores cloud, soporta tanto lenguajes imperativos (como Bash o PowerShell) como declarativos (Azure Bicep, ARM Templates, Terraform, etc) que permiten desplegar infraestructura de forma sencilla y rápida.

En este articulo te contaré porque Azure Bicep puede ser tu opción preferida antes que otros lenguajes, y buenas prácticas para usarlo de forma correcta y eficiente.

## ¿Porque deberias usar Azure Bicep antes que otras opciones?

Azure Bicep se ha ido consolidado como una herramienta declarativa y robusta para la implementación de infraestructura en Azure, destacando frente a otras opciones como ARM Templates o herramientas de terceros (Terraform, Ansible). A continuación, vamos a ver las principales razones por las que Bicep puede ser tu elección preferida cuando uses un lenguaje de IaC:

- Simplicidad sin comprometer la potencia:
    A diferencia de ARM Templates, cuya sintaxis JSON puede resultar densa y propensa a errores, Bicep utiliza una sintaxis más sencilla y legible. Esto reduce significativamente la curva de aprendizaje y facilita la colaboración entre equipos.

- Optimización del ciclo de vida del desarrollo:
    Azure Bicep se integra directamente con el flujo de trabajo de DevOps. Sus capacidades nativas permiten escribir, probar y desplegar infraestructura en Azure rápidamente, reduciendo el tiempo necesario para pasar de la fase de desarrollo a la producción.

- Reutilización y modularidad:
    Con Bicep, puedes crear módulos reutilizables que encapsulan configuraciones comunes de recursos. Esto es especialmente útil para organizaciones que necesitan implementar entornos similares de manera repetitiva.

- Interoperabilidad y migración desde ARM Templates:
    Si ya trabajas con ARM Templates, Azure Bicep te permite migrar fácilmente tu configuración existente gracias a herramientas como bicep decompile. Esto te asegura una transición sin fricciones y sin perder el trabajo previo.

- Soporte y evolución continua:
    Microsoft respalda Azure Bicep como su solución declarativa principal, asegurando un desarrollo continuo, soporte técnico y compatibilidad con todos los servicios nuevos de Azure desde el día uno. Además de que si solo trabajas con Azure como proveedor cloud Azure Bicep es la mejor opción.

    ![Desktop View](/assets/img/bicep/comparacion-arm-bicep.png)
    _Comparación de ARM con Azure Bicep_
    {: .nolineno}

## ¿Que ocurre si tengo plantillas ARM existenes?

Si ya estás utilizando ARM Templates para administrar tu infraestructura en Azure, no necesitas preocuparte por empezar desde cero al adoptar Azure Bicep. Microsoft ofrece herramientas que facilitan esta transición, como el comando `bicep decompile`, que permite convertir plantillas ARM existentes en código Bicep de forma automática. Esto elimina el esfuerzo manual de reescribir configuraciones complejas, permitiendo a las organizaciones aprovechar las ventajas de Bicep de inmediato.

El proceso de migración sigue un flujo de trabajo sencillo:

1. Exportar los recursos existentes desde Azure si no tienes ya tus plantillas ARM. Esto puede hacerse directamente desde el portal de Azure utilizando la opción de "Exportar plantilla".
2. Decompilar las plantillas ARM a Bicep utilizando el comando bicep decompile. Este paso transforma la plantilla JSON en un archivo .bicep más legible y modular.
3. Validar y ajustar el archivo Bicep. Aunque la decompilación automatiza gran parte del trabajo, es recomendable revisar el archivo resultante para optimizarlo o adaptarlo a tus estándares organizativos.
4. Probar y desplegar. Una vez validado el archivo Bicep, puedes probarlo para asegurarte de que despliega los recursos correctamente y luego integrarlo en tus procesos de implementación automatica como Azure DevOps.

    ![Desktop View](/assets/img/bicep/arm_a_bicep_flujo.png)
    _Flujo de trabajo para migrar recursos a Azure Bicep_
    {: .nolineno}

Si ya tienes automatizaciones o integraciones basadas en ARM Templates, puedes continuar utilizándolas sin interrupciones mientras migras progresivamente a Bicep. Por otro lado, si utilizas herramientas de terceros como Terraform y estás satisfecho con sus capacidades, no hay una necesidad urgente de migrar a Bicep. Microsoft colabora estrechamente con HashiCorp para garantizar que Terraform funcione de manera óptima en Azure. Sin embargo, Bicep es la opción recomendada si tu infraestructura está exclusivamente en Azure y deseas beneficiarte de una integración nativa y un lenguaje diseñado específicamente para este ecosistema.

## ¿Que ocurre si ya tengo recursos desplegados en Azure y no los tengo en un lenguaje de IaC?

Es común que, al comenzar a utilizar Azure, se creen recursos directamente desde el portal sin utilizar un lenguaje de IaC. Aunque este enfoque puede parecer práctico al principio, con el tiempo puede dificultar la administración, el control de cambios y el gobierno de los recursos. Por esta razón, es fundamental que todos los recursos existentes se incorporen a un lenguaje de IaC como Azure Bicep.

Si bien puede parecer un esfuerzo considerable migrar recursos ya desplegados a IaC, los beneficios a largo plazo son innumerables:

- Automatización y consistencia: Evita errores humanos en configuraciones manuales y garantiza que los entornos sean consistentes.
- Trazabilidad y control: Permite gestionar los cambios en los recursos de manera centralizada, con sistemas de control de versiones como Git.
- Ahorro de tiempo en el futuro: Simplifica la creación de nuevos entornos, la replicación de configuraciones y la implementación de cambios.
- Mejora del gobierno: Facilita el cumplimiento de políticas organizativas y normativas, asegurando una administración estructurada de los recursos en Azure.

### Cómo comenzar a migrar recursos existentes a IaC

Si no tienes claro qué propiedades deben incluirse en cada recurso que tengas creado o que significa cada una de ellas, Azure proporciona una extensa documentación oficial con ejemplos y plantillas predefinidas. Estas plantillas detallan las configuraciones y propiedades de los recursos en diferentes escenarios. Puedes reutilizar y personalizar estas plantillas para crear tus propios archivos Bicep.

Para crear las plantillas, podrias hacer uso del paso 1 y 2 del punto anterior, o directamente usar esta nueva opción que está en la [`versión preliminar del portal de Azure`](https://rc.portal.azure.com), la cual permite exportar directamente tus recursos a Bicep o incluso Terraform:

![Desktop View](/assets/img/bicep/export_to_bicep.png)
_Exportar directamente a Azure Bicep los recursos_
{: .nolineno}

Es cierto que documentar todos los recursos existentes en IaC puede llevar tiempo, pero este esfuerzo inicial se ve ampliamente recompensado con las ventajas en administración, escalabilidad y cumplimiento a futuro. La transición a IaC con Bicep no solo alinea la infraestructura con las mejores prácticas de DevOps, sino que también permite que los equipos trabajen de manera más eficiente y organizada.

## Mejores practicas para el uso de Azure Bicep

Implementar buenas prácticas al utilizar Azure Bicep es esencial para garantizar que tus archivos sean eficientes, fáciles de entender y mantener. A continuación, te daré recomendaciones clave para optimizar el uso de Azure Bicep:

1. Uso de parámetros

    - Nombres Claros y Descriptivos: Asigna a los parámetros nombres que reflejen claramente su propósito, facilitando la comprensión del código.

    ```text
    @description('Storage Account Name')  
    param storageAccountName string
    ```

    - Parámetros para Configuraciones Variables: Utiliza parámetros para aquellas configuraciones que cambian entre implementaciones, y variables o valores fijos para las que permanecen constantes.

    ```text
    @description('Environment where the resources will be implemented')
    @allowed([
      'dev'
      'pre'
      'pro'
    ])
    param environment string
    ```

    - Valores Predeterminados Seguros: Establece valores por defecto que sean seguros y económicos, como planes de tarifa de bajo costo, para evitar gastos innecesarios en entornos de prueba.

    ```text
    @description('Storage Account Sku Name')
    param StorageAccountSkuName string = 'Standard_LRS'  
    ```

    - Uso del Decorador @allowed: Define valores permitidos para un parámetro.

    ```text
    @description('Resource Location')
    @allowed([
      'westeurope'
      'centralus'
    ])
    param location string = 'westeurope'
    ```

    - Descripciones Informativas: Proporciona descripciones útiles para cada parámetro, ofreciendo información esencial sobre los valores esperados.

    ```text
    @description('Virtual Machine Size')  
    param vmSize string
    ```

    - Comentarios en el Código: Utiliza comentarios (//) para añadir notas aclaratorias dentro de los archivos Bicep, mejorando la legibilidad.

    - Ubicación de Declaraciones de Parámetros: Coloca las declaraciones de parámetros al inicio del archivo para facilitar la lectura y organización del código.

    - Restricciones de Longitud: Especifica longitudes mínimas y máximas para parámetros que controlan nombres, evitando errores durante la implementación.

    ```text
    @description('Storage Account Name')
    @minLength(3)
    @maxLength(24)
    param storageAccountName string  
    ```

2. Gestión de variables

    - Definición Implícita de Tipos: No es necesario declarar el tipo de datos al definir una variable; Bicep lo deduce automáticamente.

    ```text
    var resourceGroup = 'myResourceGroup'  
    ```

    - Uso de Funciones en Variables: Aprovecha las funciones integradas de Bicep para definir variables de manera dinámica y eficiente.

    ```text
    var uniqueName = toLower('${uniqueString(resourceGroup().id)}-app')  
    ```

    - Referencia a Variables: Una vez definidas, utiliza el nombre de la variable para acceder a su valor en el archivo Bicep.

    ```text
    var vmName = 'myVM'  
    resource myVm 'Microsoft.Compute/virtualMachines@2021-07-01' = {  
      name: vmName  
    }  
    ```


3. Nomenclatura de recursos

    - Convención de Nombres en CamelCase: Utiliza una notación estándar como CamelCase para las variables, recursos y parámetros. Por ejemplo, miRecursoEjemplo en lugar de nombres que mezclen estilos o sean inconsistentes. Esto mejora la legibilidad y facilita la colaboración.

    ```text
    var appServicePlanName = 'myAppServicePlan'  
    ```

    - Consistencia en los Nombres: Sigue un esquema de nomenclatura uniforme para los recursos. Por ejemplo, en lugar de asignar nombres arbitrarios, utiliza un formato como nombre_proyecto-entorno-tipo_recurso (ejemplo: web-pro-as). Esto facilita la identificación de recursos relacionados en implementaciones grandes.

    ```text
    var resourceName = 'web-pro-as'  
    ```

    - Evita Espacios y Caracteres Especiales: Usa guiones (-) o guiones bajos (_) en lugar de espacios y evita caracteres especiales para garantizar que los nombres sean compatibles con todas las herramientas de Azure.

    - Generación de Nombres Únicos: Cuando necesites asegurarte de que un nombre es único, utiliza la función uniqueString() en combinación con información específica, como el ID del grupo de recursos o el nombre del proyecto. Esto es especialmente útil para servicios globales como Storage Accounts, donde los nombres deben ser únicos en todo Azure.

    ```text
    var uniqueStorageName = uniqueString(resourceGroup().id)  
    ```

    - Documentación de Convenciones: Si trabajas en un equipo, documenta las reglas de nomenclatura para que todos las sigan, garantizando coherencia en cada uno de los proyectos de la organización.

4. Definición de recursos

    - Evita Configuraciones Hardcoded: No coloques valores estáticos (hardcoded) en las propiedades de los recursos. En su lugar, utiliza parámetros o variables, lo que permite una mayor flexibilidad y reutilización del código.

    ```text
    param location string = 'eastus'  
    resource myStorage 'Microsoft.Storage/storageAccounts@2021-04-01' = {  
      location: location  
    }  
    ```

    - Utiliza Dependencias Implícitas: Bicep detecta automáticamente las dependencias entre recursos según las referencias que utilices. No es necesario declarar explícitamente las dependencias en la mayoría de los casos, lo que simplifica el código.

    ```text
    resource myVnet 'Microsoft.Network/virtualNetworks@2021-02-01' = {  
      name: 'myVnet'  
    }  
    resource mySubnet 'Microsoft.Network/virtualNetworks/subnets@2021-02-01' = {  
      parent: myVnet  
      name: 'mySubnet'  
    }
    ```

    - Modulariza los Recursos: Divide las configuraciones en módulos reutilizables para gestionar recursos relacionados. Esto es útil cuando necesitas implementar patrones repetitivos, como un grupo de recursos que incluye una base de datos, una aplicación y un almacenamiento.

    ```text
    module network './modules/network.bicep' = {  
      name: 'networkModule'  
      params: {  
        location: location
        environment: environment 
        project: project
      }  
    }  
    ```

    - Agrupación de Etiquetas (tags): Define etiquetas comunes a nivel de parámetros y aplícalas a todos los recursos. Esto facilita la gestión y permite un control centralizado de costes, entornos y responsabilidades.

    ```text
    param tags object = {  
      environment: 'dev'  
      department: 'IT'  
    }  
    resource myStorage 'Microsoft.Storage/storageAccounts@2021-04-01' = {  
      tags: tags  
    }  
    ```

    - Validación Antes de Implementar: Antes de desplegar, utiliza el comando az deployment group validate para previsualizar los cambios que se aplicarán. Esto reduce errores en implementaciones y te asegura de que las configuraciones son correctas.

    ```text
    az deployment group validate --resource-group myGroup --template-file main.bicep  
    ```

    - Integra tus plantillas de Bicep en herramientas como Azure DevOps: Automatiza cambios en los recursos de forma rápida y programada, añadiendo validaciones por entorno.

    - Aprovecha las Plantillas Prediseñadas: Consulta la biblioteca oficial de plantillas de Microsoft para inspirarte o acelerar el desarrollo. Muchas configuraciones comunes están disponibles como punto de partida.

## Conclusión

El uso de IaC no solo agiliza la creación y modificación de recursos, sino que también mejora significativamente el gobierno y la consistencia en los entornos. Mediante la adopción de un lenguaje de IaC como Azure Bicep, los equipos pueden garantizar que los cambios en la infraestructura sean rastreables, reproducibles y sometidos a procesos de revisión y aprobación, lo que refuerza la seguridad y el cumplimiento de las politicas de la organización. Además, el uso de IaC permite automatizar implementaciones, reducir errores humanos y minimizar la dependencia de intervenciones manuales, salvo en casos muy excepcionales o urgentes.
