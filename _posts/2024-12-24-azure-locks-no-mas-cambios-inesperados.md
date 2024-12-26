---
title: Azure Locks, no más cambios inesperados ni eliminaciones accidentales
author: Carles Giménez
date: 2024-12-26 20:40:00 +0800
tags: [Locks]
category: [Azure]
image: /assets/img/locks/azure_locks.webp
---

En el día a día de la administración de recursos en Azure, es vital garantizar la estabilidad y seguridad de cada aplicación implementada de la organización. Los cambios accidentales o eliminaciones inesperadas pueden generar problemas significativos, desde interrupciones en el servicio hasta pérdidas de datos críticos. Azure Locks es una herramienta poderosa y gratuita que permite proteger los recursos de manera efectiva, asegurando que solo se realicen modificaciones planificadas cuando realmente sea necesario.

## ¿Qué es un bloqueo en Azure?

Un bloqueo en Azure es una configuración que restringe las operaciones administrativas sobre recursos en un nivel específico: suscripción, grupo de recursos o recurso individual. Existen dos tipos principales de bloqueos:

- Bloqueos de Solo lectura: Evitan cualquier acción administrativa que modifique el recurso, como agregar o eliminar elementos. Sin embargo, no afectan a las operaciones funcionales del recurso.

- Bloqueos de Eliminación: Impiden la eliminación de recursos, pero permiten otras acciones administrativas, como agregar o modificar elementos dentro del recurso.

Cuando un bloqueo se aplica en un nivel superior, como una suscripción o un grupo de recursos, este se hereda por todos los recursos de los niveles inferiores. Por ejemplo, un bloqueo de eliminación en un grupo de recursos evitará que cualquier recurso dentro de ese grupo sea eliminado. Esto es especialmente útil para proteger entornos de producción o recursos críticos.

Si aplicamos un bloqueo de solo lectura a una máquina virtual encendida dentro de un grupo de recursos, ningún usuario podrá cambiar sus configuraciones ni reiniciarla desde el portal de Azure. Sin embargo, la máquina seguirá ejecutándose sin problema.

Ahora bien, ¿Quién puede gestionar bloqueos?
Solo los propietarios de los recursos o los usuarios con el rol de "Administrador de acceso a usuarios" pueden crear o eliminar bloqueos. Esto asegura que su gestión esté limitada a personal autorizado.

## Consideraciones importantes antes de aplicar bloqueos

Aunque los bloqueos son una herramienta muy potente, es crucial considerar ciertos factores antes de implementarlos, ya que pueden generar resultados inesperados:

- Operaciones administrativas bloqueadas: Algunos bloqueos de solo lectura impiden acciones que podrían no parecer modificaciones. Por ejemplo, la visualización de claves en una cuenta de almacenamiento o el uso del Explorador de archivos en App Service.

- Herencia de bloqueos: Los bloqueos aplicados en niveles superiores afectan a todos los recursos descendientes. Si un grupo de recursos está bloqueado, todos los recursos dentro de él también lo estarán.

- Impacto en servicios como Azure Advisor: Un bloqueo de solo lectura sobre un servicio puede evitar que Azure Advisor almacene resultados de consultas.

- Restricciones en escalabilidad: Un App Service Plan con bloqueo de solo lectura no podrá ser escalado ni horizontal ni verticalmente hasta que el bloqueo sea eliminado.

- Inicio de maquinas virtuales: Un bloqueo de solo lectura en un grupo de recursos que contiene una máquina virtual impide que todos los usuarios inicien o reinicien la máquina virtual.

- Movimiento de recursos existentes: Un bloqueo de solo lectura en un grupo de recursos impide mover los recursos existentes dentro o fuera del grupo de recursos. Aún así, un recurso con bloqueo de solo lectura si que se podrá mover a otro grupo de recursos.

Por ello, es recomendable aplicar bloqueos solo una vez que los recursos estén configurados, organizados y en producción, o sobre servicios críticos que disponga la organización. En caso de necesitar hacer cambios sobre un servicio, primero se tendrá que eliminar el bloqueo, a continuación se hará la modificación y luego se volverá a restablecer el bloqueo, evitando de nuevo cambios inesperados o eliminaciones accidentales de recursos.

## Configurando bloqueos en Azure

Existen varias formas de configurar bloqueos en Azure: mediante el portal, Azure PowerShell, CLI, plantillas ARM, Azure Bicep o la API de REST. En este artículo nos vamos a enfocar en cómo configurarlos desde el portal y mediante Azure Bicep.

### Portal

1. Navega al recurso, grupo de recursos o suscripción donde deseas aplicar el bloqueo.

2. En el menú lateral izquierdo, selecciona "Bloqueos".

    ![Desktop View](/assets/img/locks//select-lock.png)
    _Seleccionar Bloqueo_
    {: .nolineno}

3. Haz clic en "Agregar" y elige el tipo de bloqueo:

    - Solo lectura (“Read-Only”).

    - Eliminar (“Delete”).

    ![Desktop View](/assets/img/locks/add-lock.png)
    _Añadir Bloqueo_
    {: .nolineno}

4. Asigna un nombre y una descripción al bloqueo, y guarda los cambios.

    ![Desktop View](/assets/img/locks/set-lock.png)
    _Configurar Bloqueo_
    {: .nolineno}

### Azure Bicep

En el ejemplo siguiente te muestro una plantilla en bicep donde se crea un App Service Plan con su App Service, y se añade un bloqueo a nivel de App Service.

```text
param hostingPlanName string
param location string = resourceGroup().location
var siteName = concat('ExampleSite', uniqueString(resourceGroup().id))
resource serverFarm 'Microsoft.Web/serverfarms@2020-12-01' = {
  name: hostingPlanName
  location: location
  sku: {
    tier: 'Free'
    name: 'f1'
    capacity: 0
  }
  properties: {
    targetWorkerCount: 1
  }
}
resource webSite 'Microsoft.Web/sites@2020-12-01' = {
  name: siteName
  location: location
  properties: {
    serverFarmId: serverFarm.name
  }
}
resource siteLock 'Microsoft.Authorization/locks@2016-09-01' = {
  name: 'siteLock'
  scope: webSite
  properties:{
    level: 'CanNotDelete'
    notes: 'Site should not be deleted.'
  }
}
```

> Este bloqueo lo puedes desplegar utilizando Azure CLI o PowerShell.
{: .prompt-info }

## Conclusión

Azure Locks es una herramienta esencial para fortalecer la gobernanza y protección de recursos en Azure. Su capacidad para prevenir modificaciones y eliminaciones accidentales asegura estabilidad y control en entornos críticos de la organización. Sin embargo, para maximizar su efectividad, es crucial aplicarlos con precaución y evaluar cómo impactarán en las operaciones diarias. Aprovecha la integración en herramientas como Azure Bicep para automatizar la protección de tus recursos y garantizar un entorno seguro y robusto.