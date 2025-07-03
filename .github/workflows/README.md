# TypeScript Services CI/CD Workflow

Este workflow proporciona un pipeline completo de CI/CD para servicios TypeScript, incluyendo identificación automática de aplicaciones cambiadas, testing, build y despliegue.

## Uso

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  cicd:
    uses: Traventia/ci-templates/.github/workflows/ts-srv-cicd.yml@main
    with:
      environment: "production"
      node_version: "20.x"
      eks_namespace: "default"
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      AWS_CI_BUCKET_REGION: ${{ secrets.AWS_CI_BUCKET_REGION }}
      AWS_CI_BUCKET: ${{ secrets.AWS_CI_BUCKET }}
      NPM_AUTH_TOKEN: ${{ secrets.NPM_AUTH_TOKEN }}
      EKS_I2_USER_TOKEN: ${{ secrets.EKS_I2_USER_TOKEN }}
      EKS_I2_CA: ${{ secrets.EKS_I2_CA }}
      EKS_I2_URL: ${{ secrets.EKS_I2_URL }}
      EKS_I2_USER_NAME: ${{ secrets.EKS_I2_USER_NAME }}
```

## Inputs

| Input                    | Descripción                                                                        | Requerido | Default      |
| ------------------------ | ---------------------------------------------------------------------------------- | --------- | ------------ |
| `environment`            | Entorno a desplegar (production o staging)                                         | ❌        | `production` |
| `apps`                   | Lista de apps separadas por coma (si no se provee, se identifican automáticamente) | ❌        |              |
| `eks_namespace`          | Namespace de EKS para despliegue                                                   | ❌        | `default`    |
| `skip_tests`             | Saltar ejecución de tests unitarios                                                | ❌        | `false`      |
| `skip_integration_tests` | Saltar ejecución de tests de integración                                           | ❌        | `false`      |
| `skip_build`             | Saltar paso de build                                                               | ❌        | `false`      |
| `skip_deploy`            | Saltar paso de despliegue                                                          | ❌        | `false`      |
| `node_version`           | Versión de Node.js a usar                                                          | ❌        | `20.x`       |

## Secrets requeridos

### AWS (Obligatorios)

| Secret                  | Descripción                         |
| ----------------------- | ----------------------------------- |
| `AWS_ACCESS_KEY_ID`     | AWS Access Key ID                   |
| `AWS_SECRET_ACCESS_KEY` | AWS Secret Access Key               |
| `AWS_CI_BUCKET_REGION`  | Región del bucket S3 con ci-scripts |
| `AWS_CI_BUCKET`         | Nombre del bucket S3 con ci-scripts |

### Desarrollo (Opcionales)

| Secret           | Descripción                |
| ---------------- | -------------------------- |
| `NPM_AUTH_TOKEN` | Token de autenticación NPM |

### Kubernetes (Opcionales - requeridos para deploy)

| Secret              | Descripción                     |
| ------------------- | ------------------------------- |
| `EKS_I2_USER_TOKEN` | Token de usuario de Kubernetes  |
| `EKS_I2_CA`         | Certificado CA de Kubernetes    |
| `EKS_I2_URL`        | URL del servidor de Kubernetes  |
| `EKS_I2_USER_NAME`  | Nombre de usuario de Kubernetes |

## Outputs

| Output           | Descripción                                                    |
| ---------------- | -------------------------------------------------------------- |
| `apps_to_deploy` | Lista de aplicaciones que serán desplegadas separadas por coma |

## Pipeline del Workflow

### 1. Identificación de Apps (`identify-apps`)

- **Condición**: Solo se ejecuta si no se proporcionan apps específicas
- **Función**: Identifica automáticamente qué aplicaciones han cambiado
- **Salida**: Lista de aplicaciones cambiadas

### 2. Tests Unitarios (`unit-tests`)

- **Dependencias**: `identify-apps`
- **Condición**: `!skip_tests`
- **Proceso**:
  - Instala Node.js
  - Cachea node_modules
  - Ejecuta `make install`
  - Ejecuta `make test` con las apps identificadas
- **Ejecución**: En paralelo con integration-tests y build

### 3. Tests de Integración (`integration-tests`)

- **Dependencias**: `identify-apps`
- **Condición**: `!skip_integration_tests`
- **Proceso**:
  - Instala Node.js
  - Cachea node_modules
  - Ejecuta `make install`
  - Ejecuta `make install-infra-deps`
  - Ejecuta `make test-i` con las apps identificadas
- **Ejecución**: En paralelo con unit-tests y build

### 4. Build (`build`)

- **Dependencias**: `identify-apps`
- **Condición**: `!skip_build`
- **Proceso**:
  - Configura credenciales AWS
  - Login a Amazon ECR
  - Construye y pushea imágenes Docker
- **Ejecución**: En paralelo con tests

### 5. Despliegue (`deploy`)

- **Dependencias**: `identify-apps`, `unit-tests`, `integration-tests`, `build`
- **Condición**: `!skip_deploy` y todos los jobs previos exitosos
- **Proceso**:
  - Configura credenciales AWS
  - Login a Amazon ECR
  - Despliega a Kubernetes
  - Envía notificaciones

## Requisitos del proyecto

Para usar este workflow, tu proyecto debe tener:

### Makefile

El proyecto debe incluir un `Makefile` con los siguientes targets:

```makefile
# Instalar dependencias
install:
	npm ci

# Instalar dependencias de infraestructura (para tests de integración)
install-infra-deps:
	# Comandos para instalar dependencias adicionales

# Tests unitarios
test:
	npm test

# Tests de integración
test-i:
	npm run test:integration
```

### Estructura de aplicaciones

- Las aplicaciones deben estar organizadas en una estructura que permita la identificación automática
- Cada aplicación debe tener su propio Dockerfile o usar uno compartido
- Las aplicaciones deben estar configuradas para despliegue en Kubernetes

## Estrategias de caché

- **Node modules**: Se cachean automáticamente basándose en `package-lock.json`
- **Reutilización**: Los `make install` se reutilizan entre jobs cuando es posible

## Gestión de errores

- **Tests fallan**: El despliegue no se ejecuta
- **Build falla**: El despliegue no se ejecuta
- **Jobs en paralelo**: Un fallo en tests no afecta al build hasta el despliegue
- **Condiciones flexibles**: Se puede saltar cualquier paso usando los parámetros `skip_*`

## Notificaciones

Después del despliegue exitoso, se envían notificaciones a:

- **Apolo**: Sistema interno de tracking de despliegues
- **Microsoft Teams**: Canal de notificaciones del equipo

## Ejemplo avanzado

```yaml
name: Custom CI/CD

on:
  push:
    branches: [main]
  workflow_dispatch:
    inputs:
      apps:
        description: "Apps to deploy (comma-separated)"
        required: false
      environment:
        description: "Environment"
        default: "production"

jobs:
  cicd:
    uses: Traventia/ci-templates/.github/workflows/ts-srv-cicd.yml@main
    with:
      environment: ${{ github.event.inputs.environment || 'production' }}
      apps: ${{ github.event.inputs.apps }}
      node_version: "18.x"
      skip_tests: false
      skip_integration_tests: false
    secrets: inherit
```

## Troubleshooting

### Error en identificación de apps

- Verificar que el bucket S3 sea accesible
- Comprobar las credenciales AWS
- Revisar que el script `identify_changed_apps.js` esté en ci-scripts

### Error en tests

- Verificar que el `Makefile` tenga los targets correctos
- Comprobar que `package.json` tenga los scripts necesarios
- Revisar dependencias en `package-lock.json`

### Error en build

- Verificar acceso a ECR
- Comprobar que los Dockerfiles existan
- Revisar configuración de NPM_AUTH_TOKEN si es necesario

### Error en despliegue

- Verificar credenciales de Kubernetes
- Comprobar que los deployments existan en el cluster
- Revisar permisos del usuario de Kubernetes
