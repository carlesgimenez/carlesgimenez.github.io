---
title: Cómo estimar costes de Azure en tus Pull Requests con Bicep y Azure DevOps
author: Carles Giménez
date: 2025-09-10 10:14:00 +0800
tags: [FinOps, Azure Bicep, Azure DevOps]
category: [Azure]
image: /assets/img/finops/ace.webp
---

Imagina esto: tu equipo despliega una nueva infraestructura en Azure, todo funciona perfecto... hasta que llega la factura. Y de repente, el coste es mucho más alto de lo que esperabas. ¿Te suena familiar?

Ese es uno de los mayores retos al trabajar en la nube: los costes suelen descubrirse tarde, cuando ya es demasiado difícil corregirlos.

La buena noticia es que no tiene por qué ser así. Con un enfoque FinOps podemos llevar la conversación sobre costes al mismo lugar donde hablamos de código: los Pull Request (PR).

En este artículo te enseño cómo hacerlo realidad con Bicep y Azure DevOps integrando estimaciones de costes automáticas directamente en tus PR. Así, cada vez que alguien proponga un cambio de infraestructura, no solo podrás revisarlo desde el punto de vista técnico, sino también económico.

## El reto y la solución: de InfraCost a Azure Cost Estimator (ACE)

Si alguna vez has trabajado con Terraform, seguro que conoces InfraCost: una herramienta genial que muestra estimaciones de costes directamente en los PR. El problema es que, si usas Bicep o ARM templates, esa opción simplemente no existe. Y sí, a mí también me frustró descubrirlo.

La buena noticia es que hay alternativa. Azure Cost Estimator (ACE), es un proyecto open source que se conecta a la API oficial de precios de Azure para calcular costes a partir de tus plantillas Bicep, ARM o incluso también Terraform.

¿Lo mejor de todo? Puedes integrarlo en tus pipelines de Azure DevOps para que cada vez que alguien abra un PR, aparezca automáticamente una estimación de coste en los comentarios. Así, tu equipo sabe desde el minuto uno cómo impacta cada cambio en la factura de Azure.

Esto es lo que llamamos “shift-left FinOps”: llevar la gestión de costes al inicio del ciclo de desarrollo, en el mismo sitio donde revisamos el código.

## ¿Qué necesitas para implementarlo? (Prerrequisitos)

Antes de entrar en materia, vamos a preparar el terreno. Piensa en esto como cuando cocinas: si no tienes los ingredientes listos, no puedes seguir la receta.

### Paso 1: Prepara tu entorno

Necesitas:

- Una suscripción de Azure con un Resource Group. Aquí es donde ACE hará las simulaciones de costes. Este Resource Group se usa únicamente como contexto para las simulaciones. ACE no crea ni modifica recursos reales, simplemente consulta la API de precios de Azure.

- Un proyecto en Azure DevOps con un repositorio Git que contenga tus plantillas Bicep (por ejemplo en la carpeta infra/).

- Una conexión de servicio en Azure DevOps hacia tu suscripción de Azure. Si no la tienes: Project Settings > Service connections > New connection.

- Permisos básicos en Azure DevOps para trabajar con repositorios y PR.

### Paso 2: Dale permisos al servicio de compilación

El pipeline va a necesitar publicar comentarios en tus PR. Para eso, tienes que darle permisos al Build Service (la cuenta que usa Azure DevOps para ejecutar pipelines).

Ve a:
Project Settings > Repositories > [tu repo] > Seguridad

Busca la cuenta Build Service y activa la opción “Contribute to pull requests”.

Es como darle un boli a tu pipeline para que deje notas en los PR. Si no das este permiso, el pipeline se ejecutará, pero los comentarios de costes no aparecerán en tus PR.

![Desktop View](/assets/img/finops/permissions_compilation_service.png)
_Compilation Service Permissions_
{: .nolineno}

### Paso 3: Agrega los archivos clave a tu repositorio

Aquí empieza la magia. Necesitas tres archivos principales:

1. El pipeline YAML: define el flujo de trabajo.

2. Un script PowerShell para detectar cambios en Bicep: genera los reportes de costes.

3. Un script PowerShell para publicar en PRs: convierte esos reportes en comentarios visibles.

La estructura de tu repositorio quedaría así:
```
infra/
├── modules/
│   ├── appServicePlan.bicep
│   ├── keyVault.bicep
│   ├── sqlDatabase.bicep
│   ├── storageAccount.bicep
├── main.bicep
pipelines/
├── azure-pipelines.yml
scripts/
├── bicep-diff.ps1
└── post-markdown.ps1
```

>💡 Nota: en este artículo no se incluyen los archivos .bicep de ejemplo porque ACE funciona con cualquier plantilla que ya tengas en tu repositorio. Lo importante es mantener un main.bicep como punto de entrada en la carpeta infra/. A partir de ahí, puedes organizar tus módulos (modules/) según tu propio diseño.
{: .prompt-info }

#### 3.1: El pipeline YAML – El motor que lo ejecuta todo

El pipeline es el guion: descarga ACE, ejecuta los scripts y publica los resultados.

```
trigger: none
pool:
  vmImage: ubuntu-latest
jobs:
  - job: RunNoCache
    displayName: Estimador Bicep
    steps:
      - checkout: self
        fetchDepth: 0
      - task: Bash@3
        inputs:
          targetType: 'inline'
          script: |
            wget https://github.com/TheCloudTheory/arm-estimator/releases/download/1.5.1/linux-x64.zip
            unzip linux-x64.zip
            chmod +x ./azure-cost-estimator
          workingDirectory: $(System.DefaultWorkingDirectory)
      - task: AzureCLI@2
        inputs:
          azureSubscription: 'NOMBRE_DE_TU_CONEXIÓN_DE_SERVICIO'
          scriptType: 'pscore'
          scriptLocation: 'scriptPath'
          scriptPath: '$(System.DefaultWorkingDirectory)/scripts/bicep-diff.ps1'
          addSpnToEnvironment: true
          workingDirectory: $(System.DefaultWorkingDirectory)
      - task: PowerShell@2
        condition: eq(variables['Build.Reason'], 'PullRequest')
        env:
          SYSTEM_ACCESSTOKEN: $(System.AccessToken)
        inputs:
          filePath: '$(System.DefaultWorkingDirectory)/scripts/post-markdown.ps1'
          workingDirectory: $(System.DefaultWorkingDirectory)
      - task: PublishBuildArtifacts@1
        inputs:
          PathtoPublish: '$(System.DefaultWorkingDirectory)/md-reports'
          ArtifactName: 'cost-reports'
```

👉 Cambia el valor de `NOMBRE_DE_TU_CONEXIÓN_DE_SERVICIO` por el nombre de tu conexión de servicio.

#### 3.2: Script PowerShell para detectar cambios en Bicep

Este script es el detective: busca qué archivos Bicep has cambiado y ejecuta ACE con las estimaciones. Puede generar reportes en Markdown, JSON y HTML.

En el script verás que se incluye `--currency EUR`. Puedes cambiar la moneda a USD, GBP, u otra divisa soportada por la API de Azure. Ten en cuenta que los precios se basan en catálogo estándar: no incluyen descuentos de Reservations, Savings Plans ni acuerdos EA, ni tampoco consumos variables como transacciones o transferencias de datos.

> 💡 Importante: aunque cambies un módulo dentro de infra/modules, ACE siempre analiza main.bicep. Esto garantiza que el cálculo de costes refleje el impacto total del despliegue completo, y no solo de un archivo aislado.
{: .prompt-info }

```
$files = (git diff origin/main --name-only -- infra | Where-Object { $_.EndsWith('.bicep') })

if (-not (Test-Path "md-reports")) {
    New-Item -Name "md-reports" -ItemType Directory -Force
}

$mainFile = "infra/main.bicep"

if ($files.count -ge 1) {
    if (Test-Path $mainFile) {
        
        function Get-ResourceDetailsFromOutput($output) {
            $resourceDetails = @{}
            $lines = $output -split "`n"
            $inEstimationTable = $false
            
            foreach ($line in $lines) {
                if ($line -match "Estimation" -or $line -match "Change type.*Resource name.*Resource type") {
                    $inEstimationTable = $true
                    continue
                }
                
                if ($line -match "^(Summary|Free Resources|Unsupported Resources)" -and $inEstimationTable) {
                    $inEstimationTable = $false
                    continue
                }
                
                if ($inEstimationTable -and $line -match '\|Create\s*\|([^|]+)\|([^|]+)\|([^|]+)\|([^|]*)\|([^|]*)\|') {
                    $resourceName = $matches[1].Trim()
                    $resourceType = $matches[2].Trim()
                    $location = $matches[3].Trim()
                    $totalCost = $matches[4].Trim()
                    $delta = $matches[5].Trim()
                    
                    $resourceDetails[$resourceName] = @{
                        Type = $resourceType
                        Location = $location
                        TotalCost = $totalCost
                        Delta = $delta
                    }
                }
            }
            
            return $resourceDetails
        }
        
        try {
            $aceOutput = ./azure-cost-estimator $mainFile "TU_ID_DE_SUSCRIPCIÓN" "TU_GRUPO_DE_RECURSOS" --currency EUR --outputFormat Table --generate-markdown-output --generateJsonOutput true --generateHtmlOutput true 2>&1
            Write-Host $aceOutput
            
            $textResourceDetails = Get-ResourceDetailsFromOutput ($aceOutput | Out-String)
            
            $aceJsonFiles = Get-ChildItem -Path ".\ace_*.json" -ErrorAction SilentlyContinue
            $aceMdFiles = Get-ChildItem -Path ".\ace_*.md" -ErrorAction SilentlyContinue
            
            if ($aceJsonFiles -or $aceMdFiles) {
                $consolidatedContent = "# 📊 Azure Cost Estimator - Reporte de Costes`n`n"
                
                $allResources = @()
                $totalCost = 0
                $totalResources = 0
                
                if ($aceJsonFiles) {
                    foreach ($jsonFile in $aceJsonFiles) {
                        try {
                            $jsonContent = Get-Content $jsonFile.FullName | ConvertFrom-Json
                            
                            if ($jsonContent.Resources) {
                                foreach ($resource in $jsonContent.Resources) {
                                    $resourceId = $resource.Id
                                    $resourceName = "Unknown"
                                    $resourceType = "Unknown"
                                    $location = "Unknown"
                                    
                                    if ($resourceId) {
                                        $idParts = $resourceId -split '/'
                                        $resourceName = $idParts[-1]
                                        
                                        $providersIndex = -1
                                        for ($i = 0; $i -lt $idParts.Length; $i++) {
                                            if ($idParts[$i] -eq "providers") {
                                                $providersIndex = $i
                                                break
                                            }
                                        }
                                        
                                        if ($providersIndex -ge 0 -and ($providersIndex + 3) -lt $idParts.Length) {
                                            $resourceType = "$($idParts[$providersIndex + 1])/$($idParts[$providersIndex + 2])"
                                            
                                            if (($providersIndex + 4) -lt $idParts.Length) {
                                                $resourceType += "/$($idParts[$providersIndex + 4])"
                                            }
                                        }
                                        
                                        # Intentar obtener location de la salida de texto más adelante
                                        $location = "Unknown"
                                    }
                                    
                                    $resourceCost = 0
                                    if ($resource.TotalCost -and $resource.TotalCost.Value) {
                                        $resourceCost = [double]$resource.TotalCost.Value
                                    } elseif ($resource.TotalCost) {
                                        $resourceCost = [double]$resource.TotalCost
                                    }
                                    
                                    if ($textResourceDetails.ContainsKey($resourceName)) {
                                        $textResource = $textResourceDetails[$resourceName]
                                        if ($textResource.Type -and $textResource.Type -ne "") {
                                            $resourceType = $textResource.Type
                                        }
                                        if ($textResource.Location -and $textResource.Location -ne "") {
                                            $location = $textResource.Location
                                        }
                                    }
                                    
                                    $allResources += [PSCustomObject]@{
                                        Name = $resourceName
                                        Type = $resourceType
                                        Cost = $resourceCost
                                        Location = $location
                                    }
                                    $totalCost += $resourceCost
                                }
                            }
                            
                            if ($jsonContent.TotalCost) {
                                if ($jsonContent.TotalCost.Value) {
                                    $totalCost = [double]$jsonContent.TotalCost.Value
                                } else {
                                    $totalCost = [double]$jsonContent.TotalCost
                                }
                            }
                        }
                        catch {
                            Write-Host "##[warning] Error procesando archivo JSON: $($jsonFile.Name) - $($_.Exception.Message)"
                        }
                    }
                }
                
                Write-Host "$($files.count) archivos Bicep encontrados con cambios"
                
                if ($allResources.Count -eq 0 -and $aceMdFiles) {
                    foreach ($mdFile in $aceMdFiles) {
                        $content = Get-Content -Path $mdFile.FullName -Raw
                        $consolidatedContent += $content
                        $consolidatedContent += "`n`n---`n`n"
                    }
                } else {
                    $totalResources = $allResources.Count
                    $roundedTotal = "{0:F2}" -f ([double]$totalCost)
                    
                    $consolidatedContent += "## 📋 Resumen Ejecutivo`n`n"
                    $consolidatedContent += "| 📊 Métrica | Valor |`n"
                    $consolidatedContent += "|------------|-------|`n"
                    $consolidatedContent += "| Recursos Analizados | $totalResources |`n"
                    $consolidatedContent += "| Recursos con Coste | $totalResources |`n"
                    $consolidatedContent += "| **💰 Coste Total Mensual** | **$roundedTotal EUR** |`n`n"
                    
                    $consolidatedContent += "## 💰 Estimación de Costes por Recurso`n`n"
                    $consolidatedContent += "| Nombre del Recurso | Tipo de Recurso | Región | Coste Mensual |`n"
                    $consolidatedContent += "|-------------------|-----------------|--------|---------------|`n"
                    
                    foreach ($resource in $allResources) {
                        $roundedCost = "{0:F2}" -f ([double]$resource.Cost)
                        $safeType = '`' + $resource.Type + '`'
                        $consolidatedContent += "| $($resource.Name) | $safeType | $($resource.Location) | $roundedCost EUR |`n"
                    }
                    
                    $consolidatedContent += "`n## 🎯 Resumen Final`n`n"
                    $consolidatedContent += "| 💼 Concepto | 💰 Importe |`n"
                    $consolidatedContent += "|-------------|------------|`n"
                    $roundedTotal = "{0:F2}" -f ([double]$totalCost)
                    $consolidatedContent += "| **Coste Total Mensual Estimado** | **$roundedTotal EUR** |`n"
                    $roundedAnnual = "{0:F2}" -f ([double]$totalCost * 12)
                    $consolidatedContent += "| **Coste Anual Estimado** | **$roundedAnnual EUR** |`n`n"

                    $consolidatedContent += "> 💡 **Nota**: Los costes mostrados son estimaciones basadas en precios estándar de Azure. Los costes reales pueden variar según el uso, la región y las ofertas aplicables.`n`n"
                }
                
                $consolidatedContent += "---`n"
                $consolidatedContent += "*Generado automáticamente por Azure Cost Estimator - $(Get-Date -Format "yyyy-MM-dd HH:mm:ss")*`n"
                
                $consolidatedContent | Out-File -FilePath "md-reports\ace_comparison.md" -Encoding UTF8
                
                Remove-Item -Path ".\ace_*.md" -Force -ErrorAction SilentlyContinue
                Remove-Item -Path ".\ace_*.json" -Force -ErrorAction SilentlyContinue
            } else {
                Write-Host "##[warning] No se generaron archivos de reporte ACE"
            }
        }
        catch {
            Write-Host "##[error] Error ejecutando azure-cost-estimator: $($_.Exception.Message)"
            exit 1
        }
    } else {
        Write-Host "##[error] Archivo infra/main.bicep no encontrado"
        exit 1
    }
    Write-Host "$($files.count) archivos Bicep encontrados con cambios"
}
else {
    Write-Host "No hay actualizaciones de Bicep o archivo main.bicep no encontrado"
}
```

👉 Recuerda reemplazar `TU_ID_DE_SUSCRIPCIÓN` y `TU_GRUPO_DE_RECURSOS` por tu ID de suscripción y grupo de recursos correspondiente.

#### 3.3: Script PowerShell para publicar en los Pull Request
Este es el mensajero: coge el reporte en Markdown y lo publica como comentario en el PR usando la API de Azure DevOps.

```
$statusCode = 1

$comparisonFile = Get-ChildItem -Path "md-reports" -Filter "ace_comparison.md" -ErrorAction SilentlyContinue

if (-not $comparisonFile) {
    Write-Host "No se encontró reporte de comparación ACE"
    exit 0
}

Write-Host "Procesando reporte consolidado: $($comparisonFile.FullName)"
$content = Get-Content -Path $comparisonFile.FullName -Raw

$bodyObject = @{
    comments = @(
        @{
            parentCommentId = 0
            content = $content
            commentType = 1
        }
    )
    status = $statusCode
}

$body = $bodyObject | ConvertTo-Json -Depth 3

Write-Debug $body

try {
    $url = "$($env:SYSTEM_TEAMFOUNDATIONCOLLECTIONURI)$env:SYSTEM_TEAMPROJECTID/_apis/git/repositories/$($env:BUILD_REPOSITORY_NAME)/pullRequests/$($env:SYSTEM_PULLREQUEST_PULLREQUESTID)/threads?api-version=7.0"
    Write-Host "URL: $url"
    Write-Host "Publicando reporte consolidado de costes"
    
    $response = Invoke-RestMethod -Uri $url -Method POST -Headers @{Authorization = "Bearer $env:SYSTEM_ACCESSTOKEN"} -Body $body -ContentType application/json
    
    if ($response -ne $null) {
        Write-Host "*******************Éxito - Reporte consolidado publicado*********************************"
    }
}
catch {
    Write-Error $_
    Write-Error $_.Exception.Message
    exit 1
}
```

### Paso 4: Configura la política de validación de compilación

Ahora, con el fichero yaml tendrás que crear un pipeline y asegurarte de que este se ejecute siempre cuando alguien abra un PR hacia tu rama principal.

En Azure DevOps ve a:
Project Settings > Repositories > Policies > [rama main]
y agrega una build policy apuntando a tu pipeline YAML.

>🔧 También puedes aplicar esta policy a otras ramas protegidas como `develop`, `qa` o `release`. Así tendrás visibilidad de costes en cualquier fase del ciclo de vida.
{: .prompt-info }

![Desktop View](/assets/img/finops/pipeline_policy.png)
_Pipeline Policy_
{: .nolineno}

## Escenario en acción

Haz un cambio sencillo, por ejemplo cambiar el SKU de una base de datos en un archivo Bicep. Crea un PR hacia main y observa lo que pasa:

1. El pipeline se ejecuta.

2. ACE calcula el coste de los recursos que vas a desplegar.

3. Aparece un comentario automático en tu PR con el reporte.

Y listo. Ahora tu equipo no solo revisa el código, también el impacto en la factura de Azure en tiempo real.

![Desktop View](/assets/img/finops/ace_result.png)
_ACE Result_
{: .nolineno}

## Conclusión

Uno de los grandes problemas de muchas empresas en la nube es que los costes se disparan sin que nadie lo note a tiempo. La infraestructura funciona, pero la factura llega después… y ahí empiezan los sustos.

Herramientas como Azure Cost Estimator (ACE) integradas en tus pipelines cambian por completo este escenario: te dan visibilidad de los costes en el mismo momento en que decides cómo será tu infraestructura. Eso significa menos sorpresas, mejores decisiones y un control real desde el inicio.

Al final, no se trata solo de escribir buen código o diseñar arquitecturas escalables, sino de asegurarnos de que la nube también sea sostenible económicamente. Y con soluciones como esta, das un paso firme en esa dirección.