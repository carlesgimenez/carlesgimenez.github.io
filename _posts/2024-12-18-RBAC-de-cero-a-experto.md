---
title: Entendiendo Azure RBAC en 5 minutos
author: Carles Giménez
date: 2024-12-18 13:09:00 +0800
tags: [RBAC]
category: [Azure]
image: /assets/img/rbac/azure-rbac.webp
---

Una de las funciones más importantes como Cloud Engineer en el día a día es administrar accesos sobre cada una de las suscripciones, grupos de recursos y servicios que tenga la organización. Para esto, Azure proporciona un sistema de autorización basado en roles (RBAC), el cual ayudará a administrar y securizar los accesos a los diferentes scopes del tenant.

## ¿Cómo funciona RBAC y que puedes hacer?

El concepto principal respecto a RBAC y que tienes que entender es que se trata de como aplicar permisos (roles), a entidades (usuario, grupos, etc) sobre algún scope (ámbito). Algunos ejemplos podrían ser:

- Permitir que un usuario administre las máquinas virtuales de una suscripción y que otro usuario administre las redes virtuales.
- Permitir que un grupo de DBA administre bases de datos SQL en una suscripción.
- Permitir que un usuario administre todos los recursos de un grupo de recursos, como las máquinas virtuales, los sitios web y las subredes.
- Permitir que una aplicación acceda a todos los recursos de un grupo de recursos.

## ¿Que son las entidades?

Una entidad es un objeto dentro de un tenant que representa a un usuario, un grupo, una entidad de servicio o una identidad administrada, el cual solicita acceso a recursos de Azure. Podemos asignar un rol a cualquiera de esas entidades.

![Desktop View](/assets/img/rbac/rbac-security-principal.png){}
_RBAC Security Principal_
{: .nolineno}

## ¿Que son los roles?

Para entender fácilmente que es un rol, piensa que es como una recopilación de permisos, en los cuales vas a definir que acciones se pueden realizar. Por ejemplo, lectura, escritura y eliminación. Hay que tener en cuenta que en Azure disponemos de roles generales como el rol de propietario, y de roles específicos sobre ámbitos concretos, como por ejemplo lector de máquina virtual, entre otros.

![Desktop View](/assets/img/rbac/rbac-role-definition.png){}
_RBAC Role Definition_
{: .nolineno}

En Azure se incluyen diversos roles integrados (Propietario, Colaborador, Lector, etc) que podremos asignar a usuarios, grupos, entidades de servicio e identidades administradas. Si estos roles no cumplen con las necesidades de la organización podemos crear nuestros propios [`roles personalizados`](https://learn.microsoft.com/es-es/azure/role-based-access-control/custom-roles-portal), cumpliendo así con necesidades muy especificas de acceso. Por ejemplo, imagina que hay un equipo de usuarios que necesita administrar máquinas virtuales, pero no debe tener acceso a la configuración de red, para solucionar esto crearíamos un rol personalizado que otorgue permisos para iniciar, detener y reiniciar máquinas virtuales, pero sin permisos para modificar configuraciones de red o acceder a recursos de red.

Para asignar los distintos roles (ya sean integrados o personalizados) se puede usar el [`portal de Azure`](https://learn.microsoft.com/es-es/azure/role-based-access-control/role-assignments-portal), el [`CLI`](https://learn.microsoft.com/es-es/azure/role-based-access-control/role-assignments-cli) de Azure, [`PowerShell`](https://learn.microsoft.com/es-es/azure/role-based-access-control/role-assignments-powershell) o incluso [`Azure Bicep`](https://learn.microsoft.com/es-es/azure/role-based-access-control/quickstart-role-assignments-bicep?tabs=CLI) u otros lenguajes de IaC.

## ¿Que son los scope (ámbito)?

Un scope es un conjunto de recursos sobre el cual se les puede aplicar un acceso. Dentro de Azure tenemos 4 scopes: Grupos de administración, suscripciones, grupos de recursos y servicios.

![Desktop View](/assets/img/rbac/rbac-scope.png){}
_RBAC Scope_
{: .nolineno}

Cada permiso que se asigne sobre un scope superior se hereda hacia los scope inferiores, aplicando los accesos e incluso directivas (Azure Policy) que hayamos configurado en nivel jerárquico. ¿Qué quiere decir esto? Que podemos asignar roles sobre cualquiera de estos ámbitos y que según donde los asignemos se determina el grado de amplitud de ese rol, pasando de los niveles superiores a los inferiores.

## Funcionamiento de la lógica de evaluación de RBAC

Azure RBAC utiliza un proceso claro para decidir si alguien tiene acceso a un recurso en Azure. A continuación, te explico los pasos de forma simple:

1. Inicio sin acceso
 - Por defecto, nadie tiene acceso a nada. Se comienza con "acceso denegado".

2. Revisión de asignaciones
    - Azure revisa todas las asignaciones de roles y denegaciones relacionadas con el recurso.

3. Evaluar denegaciones explícitas
    - Primero, se revisan las asignaciones de denegación.
        - Si hay una regla que dice "denegar acceso", el acceso se bloquea de inmediato.
        - Si no hay una denegación explícita, se continúa con los roles asignados.

4. Evaluar roles asignados
    - Azure verifica si el usuario (o su grupo) tiene algún rol asignado que permita realizar la acción solicitada en el recurso.
        - Si no hay un rol que coincida, no se permite el acceso.

5. Revisión de condiciones
    - Si el usuario tiene un rol que coincide, se revisan si hay condiciones adicionales.
        - Si no hay condiciones, se permite el acceso.
        - Si hay condiciones, Azure verifica si se cumplen.

6. Decisión final
    - Si se cumplen las condiciones, el acceso es permitido.
    - Si no se cumplen, el acceso es denegado.

Este proceso asegura que solo los usuarios con los permisos correctos y que cumplan con ciertas condiciones específicas, puedan acceder a los recursos.

![Desktop View](/assets/img/rbac/evaluation-logic.png){}
_RBAC Evaluation Logic_
{: .nolineno}

## Recomendaciones para usar RBAC correctamente

Teniendo en cuenta que el uso adecuado de RBAC determina en parte los accesos de forma segura a los diferentes servicios de una organización, es importante seguir estas buenas prácticas:

- Solo conceder acceso a lo que los usuarios necesitan: En lugar de otorgar permisos a los usuarios sin control sobre ámbitos amplios como grupos de recursos o suscripciones, proporcionar permisos para realizar determinadas acciones sobre un ámbito concreto. Se le debe conceder a los usuarios el privilegio mínimo para que realicen su trabajo.

- Limitar el número de propietarios de las suscripciones: Tener un máximo de 3 propietarios por suscripción, de tal forma que se minimicen los riesgos ante las cuentas que puedan verse comprometidas y que tengan acceso completo a los recursos en Azure de la organización.

- Uso de Microsoft Entra Privileged Identity Management (PIM): Con el fin de proteger las cuentas con privilegios frente a ataques, es recomendable usar esta característica que tiene Azure, la cual reducirá el tiempo de exposición de permisos proporcionando solo acceso Just in Time al usuario y al recurso donde necesita acceder.

- Asignar roles a grupos, no a usuarios: Con el objetivo de administrar y controlar de forma más sencilla los permisos que se otorgan hay que evitar asignar roles directamente a usuarios, se deben asignar esos roles a grupos. Esto también nos servirá para minimizar el número de asignaciones sobre los ámbitos y no llegar al límite.

- Asignar roles mediante el identificador único del rol en lugar del nombre: Existen dos casuísticas donde se asigna un determinado rol y el nombre del rol cambia, esto puede ser porque al crear un rol personalizado hemos cambiado nosotros el nombre o porque había un rol nuevo en preview y al liberarse este cambia de nombre. Si usamos scripts u otros tipos de automatizaciones para asignar roles es recomendable usar el id del rol, el cual nunca cambiará, al contrario que el nombre.

## Conclusión

El control de acceso basado en roles (RBAC) es un punto muy importante dentro de las tareas que hace un Cloud Engineer, y una herramienta esencial para gestionar y securizar los accesos en Azure. Entendiendo bien los conceptos explicados en este artículo podrás gestionar de forma eficiente los accesos de tu organización. Si quieres saber más acerca de RBAC, aquí te dejo la [`documentación oficial`](https://learn.microsoft.com/es-es/azure/role-based-access-control/overview ).
