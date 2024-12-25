---
title: Que es FinOps y porque tienes que adoptarlo en tu organización
author: Carles Giménez
date: 2024-12-24 20:40:00 +0800
tags: [FinOps]
category: [Azure]
image: /assets/img/finops/azure_finops.webp
---

A pesar de que muchas empresas migran al cloud y de que este ofrece muchas ventajas respecto a las arquitecturas OnPremise como escalabilidad o pago por uso, siempre continúa habiendo preocupación en las organizaciones acerca de los costes y cómo controlarlos de forma eficiente. Tener que equilibrar la demanda y rendimiento de las aplicaciones desplegadas, y a la misma vez tener un control sobre los costes de los proyectos de la organización es algo fundamental, y a la vez un desafío. Es por esto que surge FinOps.

## De CapEx a OpEx: una transformación financiera

Tradicionalmente, las empresas invertían en infraestructura con un modelo de gastos de capital (CapEx). Esto implicaba adquisiciones de gran envergadura, como servidores, que luego se depreciaban a lo largo de varios años. Este modelo garantizaba un control estricto de los costes, ya que las adquisiciones eran esporádicas y fácilmente presupuestables.

Sin embargo, con la transición al cloud, los gastos pasan a ser operativos (OpEx). En este modelo, los recursos son consumidos bajo demanda, lo que introduce costes variables y ajustables según el uso real. Para los CTOs y responsables de ingeniería, esta transición representa un cambio significativo en la planificación financiera a largo plazo. Mientras que CapEx permite prever inversiones a futuro con relativa estabilidad, OpEx requiere un enfoque continuo de monitorización y ajustes para evitar gastos descontrolados y maximizar el retorno de inversión. La falta de control y visibilidad puede llevar a gastos innecesarios si no se implementan procesos adecuados para gestionar este nuevo modelo financiero.

## La importancia de adoptar perfiles FinOps

FinOps, o "operaciones financieras en la nube", no es simplemente una herramienta para reducir costes, sino una disciplina que combina personas, procesos y tecnología para maximizar el valor de la inversión en la nube. Incluir perfiles FinOps en tu organización permite:

- Fomentar la rendición de cuentas: Cada equipo o proyecto debe ser responsable de sus gastos en la nube.

- Impulsar la colaboración entre departamentos: Finanzas, tecnología y unidades de negocio trabajan juntos para optimizar el gasto.

- Tomar decisiones informadas: FinOps proporciona visibilidad clara sobre los costes y su relación con el rendimiento y los ingresos.

## Recomendaciones para bajar la factura de Azure

Adoptar FinOps no solo se trata de reducir costes, sino de asegurar que cada euro invertido genere valor. Aquí hay algunas estrategias específicas que puedes implementar en Azure para lograrlo:

1. Azure Hybrid Benefits: Aprovecha las licencias existentes de Windows Server y SQL Server para reducir costes en la nube.

2. Azure Advisor: Recibe recomendaciones personalizadas para optimizar los recursos y ajustar los costes basándote en patrones de uso.

3. Azure Reservations: Reserva recursos con anticipación para obtener descuentos significativos.

4. Identificación de recursos huérfanos: Elimina recursos que no estén en uso, como discos o IPs no asociadas a máquinas virtuales.

5. Redimensionar recursos: Ajusta los recursos sobredimensionados para adaptarlos a las necesidades reales del negocio.

6. Diseño eficiente: Implementa recomendaciones basadas en Azure Well-Architected Framework para optimizar cargas de trabajo.

7. Configurar autoescalado: Permite que las aplicaciones aumenten o disminuyan su capacidad según la demanda real.

8. Apagado automático: Programa el apagado de recursos fuera de horario laboral para evitar costes innecesarios.

9. Políticas claras: Establece quién puede crear suscripciones y realizar compras, asegurando el control de los gastos.

10. Usar la calculadora de Azure para estimar costes, configurar presupuestos y alertas: Estima el coste de cada proyecto y utiliza Azure Cost Manager para monitorizar el gasto en tiempo real y evitar desviaciones.

## Y si no hemos migrado aún a la nube, ¿qué podemos hacer para ahorrar costes desde el primer momento?

- Dimensionar antes de migrar: Muchas organizaciones sobreaprovisionan hardware para gestionar picos de carga, lo cual es un hábito que puede trasladarse a la nube. Realizar una evaluación precisa de las cargas de trabajo antes de migrar es clave para evitar el sobredimensionamiento.

- Adoptar un enfoque incremental: Migra solo lo necesario inicialmente y ajusta según nuevos escenarios o picos puntuales de trabajo.

- Optimizar durante la planificación: Utiliza herramientas y prácticas para prever el uso de recursos y seleccionar configuraciones que equilibren costes y rendimiento desde el día uno.

## Cómo FinOps impulsa el crecimiento empresarial

El objetivo final de FinOps no es solo ahorrar dinero, sino garantizar que el gasto en la nube impulse el crecimiento del negocio. Esto implica:

- Evaluar el valor del gasto: Identificar si el aumento de costes está generando ingresos proporcionales o mayores. 

- Evitar el desperdicio: Reducir los gastos innecesarios sin comprometer la innovación. Un caso común sería eliminar recursos de desarrollo que están inactivos después de las pruebas.

- Fomentar la innovación: Reasignar ahorros hacia proyectos que generen diferenciación competitiva, como soluciones basadas en IA. 

## ¿Como realizar un assessment FinOps de tu organización?

Un assessment de FinOps es una práctica recomendada para evaluar el estado actual de la gestión financiera en la nube de tu organización y planificar mejoras. Este [`assessment`](https://learn.microsoft.com/es-es/assessments/ad1c0f6b-396b-44a4-924b-7a4c778a13d3/) que se encuentra en la documentación oficial de Microsoft, te permite analizar tu madurez en FinOps y obtener recomendaciones.

### ¿Cómo funciona?

El assessment sigue un enfoque estructurado y fácil de usar:

- Responder preguntas clave: Evalúa diferentes áreas, como la visibilidad de costos, las prácticas de optimización y la colaboración entre equipos.

- Identificar áreas de mejora: El sistema identifica puntos críticos en los que tu organización puede mejorar.

- Obtener un informe personalizado: Recibirás un informe detallado con recomendaciones prácticas alineadas a tus necesidades.

### Beneficios de realizar un assessment

- Diagnóstico preciso: Entiende cómo se gestionan los costes actualmente y dónde hay oportunidades de mejora.

- Planificación estratégica: Define un roadmap claro para implementar prácticas FinOps efectivas.

- Alineación organizacional: Fomenta una visión compartida entre finanzas, tecnología y unidades de negocio.

Realizar un assessment es una inversión inicial que ayuda a estructurar tu estrategia FinOps desde la base, asegurando que todas las áreas estén alineadas con los objetivos del negocio.

## Conclusión

FinOps es fundamental para cualquier organización que desee maximizar el valor de su inversión en el cloud. Con las herramientas y prácticas adecuadas, puedes transformar los costes variables en una ventaja competitiva, equilibrando agilidad con control financiero. Adopta FinOps cuanto antes y comienza a alinear el gasto de la nube con los objetivos estratégicos del negocio.