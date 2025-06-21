
# Documentación de CI/CD para Azure Synapse Analytics con Azure DevOps

## Descripción General

Esta documentación describe la implementación de un proceso de Integración y Despliegue Continuo (CI/CD) para Azure Synapse Analytics, utilizando como repositorio y motor de automatización a Azure DevOps.

## Arquitectura General

- Repositorio Git: Azure DevOps Repos
- Entornos: `Development`, `QA`, `Production`
- Pipelines: 
  - CI: Validación y empaquetado del código
  - CD: Despliegue automático hacia ambientes de QA/Producción
- Recursos Desplegados:
  - Notebooks
  - SQL Scripts
  - Pipelines de Synapse
  - Linked Services
  - Datasets

## Estructura del Repositorio

```plaintext
/ (root)
├── synapse/
│   ├── notebooks/
│   ├── pipelines/
│   ├── datasets/
│   ├── linkedServices/
│   └── sqlScripts/
├── .azure-pipelines/
│   └── synapse-deploy.yml
├── README.md
```

## Pipeline CI/CD en Azure DevOps

### 1. Pipeline CI (Integración Continua)

- Ubicación: `.azure-pipelines/synapse-ci.yml`
- Objetivos:
  - Validar sintaxis de notebooks y pipelines
  - Ejecutar pruebas unitarias (opcional)
  - Generar artefacto zip de Synapse workspace

#### Ejemplo de tareas clave:
```yaml
- task: Synapse workspace validation
- task: PublishBuildArtifacts@1
  inputs:
    pathToPublish: '$(Build.ArtifactStagingDirectory)'
    artifactName: 'synapse_artifacts'
```

### 2. Pipeline CD (Despliegue Continuo)

- Ubicación: `.azure-pipelines/synapse-deploy.yml`
- Entornos de destino:
  - QA
  - Producción
- Tareas:
  - Descargar artefacto generado
  - Autenticarse contra el workspace destino
  - Aplicar el contenido usando ARM templates o Synapse REST API

#### Ejemplo de pasos principales:
```yaml
- download: current
  artifact: synapse_artifacts

- task: AzureCLI@2
  inputs:
    azureSubscription: 'ServiceConnectionToSynapse'
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
      az synapse workspace import --workspace-name $(SynapseWorkspace) --file synapse_artifacts.zip
```

## Variables y Conexiones

- `$(SynapseWorkspace)`: Nombre del workspace destino
- `ServiceConnectionToSynapse`: Conexión de servicio en Azure DevOps con permisos RBAC
- Se recomienda usar Azure Key Vault para manejar secretos de conexión

## Buenas Prácticas

- Validar cambios en ramas feature antes de merge a `main`
- Ejecutar CD solo desde ramas `main` o `release/*`
- Controlar despliegue a producción con aprobaciones manuales
- Usar control de versiones para notebooks y pipelines

## Recursos

- [Documentación oficial de Synapse CI/CD](https://learn.microsoft.com/en-us/azure/synapse-analytics/cicd/continuous-integration-deployment)
- [Plantillas ARM para Synapse](https://github.com/Azure/azure-quickstart-templates)
- [Azure DevOps YAML Schema](https://learn.microsoft.com/en-us/azure/devops/pipelines/yaml-schema)

## Mantenimiento

- Revisar y actualizar tokens y conexiones cada 90 días
- Verificar permisos RBAC en workspaces destino
- Auditar artefactos y actividad de despliegue mensualmente
