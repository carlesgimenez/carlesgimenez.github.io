---
title: Protege tus aplicaciones con Azure Front Door + WAF
author: Carles Giménez
date: 2024-12-23 21:14:00 +0800
tags: [Azure Front Door, Azure WAF]
category: [Azure]
image: /assets/img/frontdoor/azure-front-door-waf.webp
---

A pesar de que Azure Front Door es un servicio global que ayuda a que las aplicaciones desplegadas en el Cloud tengan mayor disponibilidad, sean más escalabes y una menor latencia para los usuarios, también es un reto protegerse contra los atacantes.

## ¿Que es Azure WAF y como se integra en Front Door?

Aunque se configure Front Door para que sea el único punto de entrada a las aplicaciones de la organización, es importante protegerse frente a vulnerabilidades, y atacantes, por esto Azure proporciona un firewall de aplicaciones web (WAF) que puede integrarse tanto en Front Door como en Application Gateway y Azure Content Delivery Network (CDN).

Azure WAF supervisa y filtra activamente las peticiones entrantes a las aplicaciones mediante un conjunto de reglas, proporcionando una protección centralizada para todas las aplicaciones que pasen por Front Door. En este punto es muy importante tener en cuenta los dos niveles disponibles en Front Door (Estándar y Premium) y como va a afectar esto en el uso del WAF y en como va a protegerse la organización. En el caso de hacer uso del nivel Estándar solo se admiten reglas personalizadas para el WAF. Sim embargo, si se decide usar el nivel premium se dispondrá de las funcionalidades completas que permiten hacer uso de reglas que proporciona directamente Azure para protegerse aún mejor.

## Reglas y Directivas

Para hacer uso de WAF se tendrá que crear una directiva, y asociarla a uno o varios dominios de Front Door para comenzar a protegerlos. Esta directiva, consta de los dos tipos de reglas de seguridad que comentábamos anteriormente:

- Las reglas personalizadas son reglas que crearemos nosotros mismos y que ayudarán a cumplir con los requisitos de seguridad de la aplicación o incluso de la organización.
- Las reglas administradas son un conjunto de reglas preconfiguradas, administradas y periódicamente actualizadas por Azure que ayudan a que la organización se proteja frente a amenazas de seguridad comunes como inyección SQL, OWASP Top 10, u otros tipos de ataques.

### ¿Cuál es la prioridad de ejecución de los conjuntos de reglas?
Cuando estemos usando ambos tipos de reglas, las reglas personalizadas se procesan antes que los conjuntos de reglas administrados por Azure. Cada regla se comprende por una condición de coincidencia, un número de prioridad y una acción. La acción puede ser de cuatro tipos: PERMITIR, BLOQUEAR, REGISTRAR o REDIRIGIR.

- PERMITIR: Se permite el paso de la solicitud.
- BLOQUEAR: La solicitud se bloquea y se devuelve un código de respuesta.
- LOG: La solicitud se registra sin ninguna acción adicional.
- REDIRECT: La solicitud se redirige a una URL especificada.

### ¿Qué podemos hacer con el uso de reglas personalizadas?
- Restricción por IP: Podemos controlar el acceso a nuestras aplicaciones en función de un IP de origen o un rango de direcciones IP permitiendo o bloqueando las solicitudes.
- Filtrado geográfico: Bloquear o permitir solicitudes según la ubicación geográfica de la dirección IP de origen, lo que permite restricciones de acceso a países o regiones específicas.
- Control de acceso basado en los parámetros HTTP/HTTPS: Podemos crear una regla de coincidencia para bloquear solicitudes que contengan palabras clave o patrones específicos en la cadena de consulta o el cuerpo de la solicitud.
- Control de acceso basado en el método de solicitud: Se pueden hacer reglas para restringir métodos en las solicitudes HTTP como GET, PUT o HEAD.
- Limite de frecuencia: Limitar el número de solicitudes por unidad de tiempo, evitando solicitudes excesivas y mitigando ataques de denegación de servicio.

Aquí te dejo algunos ejemplos de [`reglas personalizadas`](https://learn.microsoft.com/es-es/azure/web-application-firewall/afds/waf-front-door-custom-rules).

## Modos del WAF, y cuando usar cada uno de ellos
Podemos configurar Azure WAF para que funcione de dos modos.

- Detección: En este caso se supervisan y se registran todas las solicitudes y alertas de amenazas, pero no se realiza ninguna otra opción, es decir, no se bloquean las peticiones mientras está configurado en este modo.
- Prevención: Bloquea las peticiones y los ataques detectados por las reglas, registrando eventos detallados para su análisis. El atacante recibe un error 403 y se cierra la conexión, mientras que las solicitudes bloqueadas quedan registradas para futuras revisiones.

En este punto nos podemos preguntar, ¿cómo es el enfoque adecuado para habilitar el WAF? o ¿cuándo usar un modo u otro? Tanto si es una aplicación en desarrollo como si es una aplicación en Producción, lo más recomendable al principio es comenzar con el modo detección y más adelante pasar al modo prevención. El objetivo principal de esto es detectar y testear como se comportan las reglas del WAF con la aplicación, y poder identificar falsos positivos (se puede estar detectando tráfico legítimo como una amenaza) y corregirlos o crear excepciones antes de pasar al modo prevención. De esta forma se minimiza el riesgo de que los usuarios finales se vean afectados.

Para todo este proceso es necesario habilitar el registro de diagnóstico en un Log Analytics Workspace por ejemplo, con el fin de consultar y analizar los registros del firewall y agregar las exclusiones que hayamos detectado.

## Supervisión y registro
En ambos modos comentados anteriormente es realmente importante que siempre monitoricemos que está haciendo este servicio y como se comporta, no solo para ver si las aplicaciones de la organización están sufriendo algún ataque, sino también para asegurarnos de que los usuarios que consumen estas aplicaciones pueden seguir utilizándolas con normalidad y no se bloquee ninguna petición que sea legítima.

Para esto es necesario integrar el WAF con Azure Monitor, y revisar la telemetría y los registro que dejan las peticiones sobre este servicio. De esta forma, conseguiremos hacer un seguimiento de las acciones que hace el WAF, y entenderemos y mejoraremos su rendimiento.

### ¿Cómo habilitar los registros?
Estos registros no están habilitados de forma predeterminada en Front Door y tenemos que habilitarlos de manera explícita para poder obtener los informes detallados de cada solicitud y amenaza que detecte WAF.
Hay varios tipos de registros que podemos usar para diferentes propósitos, pero en nuestro caso el que habilitaremos es Front Door WebApplicationFirewall Log para detectar los posibles ataques y detecciones de falsos positivos que podrían ser solicitudes legítimas que el WAF ha bloqueado y que las solicitudes coinciden con alguna de las reglas que hay configuradas.

![Desktop View](/assets/img/frontdoor/diagnostics-settings-frontdoor.png)
_Azure Front Door Diagnostic Setting_
{: .nolineno}

Una vez habilitado, podemos dirigirnos al log analytics workspace donde se estén volcando las trazas y ejecutar consultas personalizadas para ver y analizar las solicitudes bloqueadas u otro tipo de información que necesitemos revisar del comportamiento del WAF.

### ¿Cómo detectar y bloquear una IP que nos está atacando?

Cuando una dirección IP intenta acceder a rutas sospechosas o no autorizadas de una aplicación, es vital actuar rápido para proteger los recursos de la organización. A continuación, te muestro un proceso simple pero efectivo para detectar y bloquear esas IP maliciosas:

- Paso 1: Detectar la IP atacante en Log Analytics.

    El primer paso para identificar una IP que pueda estar atacando la aplicación es revisar los registros de WAF en Log Analytics. Podemos usar la siguiente consulta para ver todas las trazas que se han registrado (tanto las solicitudes permitidas como las bloqueadas).

    `AzureDiagnostics | where ResourceProvider == "MICROSOFT.CDN" and Category == "FrontDoorWebApplicationFirewallLog"`

    Entre los datos obtenidos, presta atención al campo _requestUri_s_, que muestra las rutas específicas que están siendo solicitadas detrás del dominio de la aplicacións. Si detectas patrones como intentos de acceder a rutas de WordPress, archivos de contraseñas o directorios no existentes, es una clara señal de un posible ataque.
    Por ejemplo, si ves registros de intentos de acceso a rutas como _/wp-admin_, _/etc/passwd_, o _/login.php_, podrías estar frente a un atacante que busca vulnerabilidades conocidas.

- Paso 2: Analizar el origen del tráfico.

    Identifica la IP o el rango de IPs desde donde provienen estos intentos. Tanto en los registros de Log Analytics como en Application Insights, encontrarás trazas y errores relacionados con estas solicitudes. Esto te ayudará a confirmar el comportamiento sospechoso antes de tomar acción.

- Paso 3: Bloquear la IP en WAF.

    Con la IP maliciosa identificada, el siguiente paso es bloquearla utilizando una regla de restricción de IP en el WAF. Así puedes hacerlo:

    - Accede al WAF en Azure Portal.
        - Ve a Azure Front Door y selecciona la instancia donde está configurado tu WAF.
        - Dirígete a las reglas personalizadas.
    - Crea una regla de bloqueo.
        - Configura una regla personalizada basada en dirección IP.
        - En la variable de coincidencia, selecciona _RemoteAddr_ (dirección IP del cliente original) o _SocketAddr_ (dirección IP visible desde WAF).
        - Añade la IP o rango de IPs maliciosas en formato CIDR.
    - Define la acción.
        - Establece la acción como _Block_ para denegar el acceso.
        - Guarda la regla y aplica los cambios.

> 💡Configurar reglas de restricción de IP proactivamente puede ayudarte a anticiparte a futuros intentos de ataque. Mantén el WAF de tu organización actualizado y revisa periódicamente los registros para mantener una defensa sólida en las aplicaciones implementadas.
{: .prompt-info }

### ¿Cómo crear una regla de exclusión en WAF?

Cuando el WAF de Azure bloquea solicitudes legítimas por error (falsos positivos), es crucial ajustar su configuración para evitar interrupciones en las aplicaciones de la organización. Estas situaciones suelen ocurrir debido al formato de una llamada, un encabezado específico o algún parámetro en particular que el WAF interpreta como malicioso. Afortunadamente, podemos configurar reglas de exclusión para resolver este problema.

- Paso 1: Analizar los eventos bloqueados.

    Lo primero es identificar las solicitudes bloqueadas que causan el problema. Esto se logra utilizando Log Analytics con la siguiente consulta:

    `AzureDiagnostics | where ResourceProvider == "MICROSOFT.CDN" and Category == "FrontDoorWebApplicationFirewallLog" | where action_s == "Block"`

    Con esta consulta, podrás encontrar detalles sobre las solicitudes bloqueadas, incluyendo los parámetros específicos que el WAF está evaluando como amenazas.

- Paso 2: Configurar una regla de exclusión.

    Una vez identificado el elemento que necesitas excluir, puedes ajustar la configuración del WAF para evitar que este sea evaluado. Por ejemplo, si el WAF detecta erróneamente un encabezado como una amenaza, puedes excluirlo:

    - Escenario: El encabezado _My-Header_ tiene un valor _1=1_, el cual es legítimo para tu aplicación.
    - Solución: Configura una exclusión para este valor usando _RequestHeaderValues_, el operador _contains_ y el selector _My-Header_. Esto detendrá la evaluación de todos los valores bajo _My-Header_.

A continuación, te muestro algunos escenarios comunes y cómo configurarlos:

| Atributo a excluir     | Variable   | Operador        | Selector | Ejemplo de solicitud | Se excluye | 
|:-----------------------| :--------- | :-------------- | :--------| :------------------- | ----------: |
| Encabezado             | RequestHeaderKeys  | Equals  | X-Scanner | `{"X-Scanner": "test"}` | X-Scanner |
| Cadena de consulta     | RequestArgKeys   | Equals   | /etc/passwd | `http://localhost/?/etc/passwd=test` | /etc/passwd |
| Cookie                 | RequestCookieNames | Contains | arg1 | `{"Cookie": "arg1=/etc/passwd"}` | /etc/passwd |

Aunque configurar exclusiones es la forma más recomendada para evitar bloqueos de peticiones legítimas, también existe la opción de deshabilitar reglas predeterminadas de Microsoft en el WAF si se identifica que es la causa del problema. Sin embargo, con esto hay que tener mucho cuidado ya que al desactivar una regla completa, podrías estar abriendo nuevas brechas de seguridad que expondrían tu aplicación a riesgos innecesarios. Por eso, siempre es mejor optar por las exclusiones específicas que resuelvan el problema sin comprometer la protección general de las aplicaciones que se estén construyendo.

> 💡Cada regla de exclusión debe ser configurada cuidadosamente para evitar vulnerabilidades. Al seguir este proceso, puedes garantizar que el WAF proteja tu aplicación sin bloquear tráfico legítimo.
{: .prompt-info }

## Precios

A pesar de que el objetivo en el cloud es siempre construir aplicaciones escalables y protegerlas de manera eficaz contra amenazas y vulnerabilidades, es importante tener en cuenta que los precios de integrar WAF con Front Door incluirán cargos fijos mensuales y cargos por procesamiento según las solicitudes. Cada una de las políticas tiene un cargo mensual y existen cargos adicionales por reglas personalizadas y conjuntos de reglas administradas configuradas dentro de la política. Si quieres obtener más información acerca de los costes consulta este [`enlace`](https://azure.microsoft.com/es-es/pricing/details/web-application-firewall/#pricing).