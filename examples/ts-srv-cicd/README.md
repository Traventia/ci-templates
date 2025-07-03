# TypeScript Services CI/CD Workflow Examples

Este directorio contiene ejemplos de cómo usar el workflow `ts-srv-cicd.yml` en diferentes escenarios. El workflow proporciona un pipeline completo de CI/CD para servicios TypeScript, incluyendo identificación automática de aplicaciones cambiadas, testing, build y despliegue.

## 📋 Requisitos Previos

Antes de usar estos ejemplos, asegúrate de tener configurados los siguientes secrets en tu repositorio:

### 🔐 Secrets Obligatorios

| Secret                  | Descripción                         |
| ----------------------- | ----------------------------------- |
| `AWS_ACCESS_KEY_ID`     | AWS Access Key ID                   |
| `AWS_SECRET_ACCESS_KEY` | AWS Secret Access Key               |
| `AWS_CI_BUCKET_REGION`  | Región del bucket S3 con ci-scripts |
| `AWS_CI_BUCKET`         | Nombre del bucket S3 con ci-scripts |

### 🔐 Secrets Opcionales

| Secret                   | Descripción                      | Requerido para      |
| ------------------------ | -------------------------------- | ------------------- |
| `NPM_AUTH_TOKEN`         | Token de autenticación NPM       | Paquetes privados   |
| `EKS_PROD_USER_TOKEN`    | Token de usuario EKS Production  | Deploy a producción |
| `EKS_PROD_CA`            | Certificado CA EKS Production    | Deploy a producción |
| `EKS_PROD_URL`           | URL del servidor EKS Production  | Deploy a producción |
| `EKS_PROD_USER_NAME`     | Nombre de usuario EKS Production | Deploy a producción |
| `EKS_STAGING_USER_TOKEN` | Token de usuario EKS Staging     | Deploy a staging    |
| `EKS_STAGING_CA`         | Certificado CA EKS Staging       | Deploy a staging    |
| `EKS_STAGING_URL`        | URL del servidor EKS Staging     | Deploy a staging    |
| `EKS_STAGING_USER_NAME`  | Nombre de usuario EKS Staging    | Deploy a staging    |

## 📝 Ejemplos Disponibles

### 1. [`basic-usage.yml`](./basic-usage.yml) - Uso Básico

**Caso de uso**: Pipeline CI/CD estándar con configuración mínima.

- ✅ Identificación automática de aplicaciones cambiadas
- ✅ Tests unitarios e integración
- ✅ Build y push de imágenes Docker
- ✅ Despliegue automático

**Triggers**:

- Push a `main` o `develop`
- Pull requests a `main`

### 2. [`advanced-usage.yml`](./advanced-usage.yml) - Uso Avanzado

**Caso de uso**: Pipeline completo con múltiples entornos y deployment strategies.

**Características**:

- 🎯 Despliegue a producción desde `main`
- 🧪 Despliegue a staging desde `develop`
- ✅ Validación de PR sin despliegue
- 🎮 Despliegue manual con inputs personalizados

**Triggers**:

- Push a `main`, `develop`, `release/*`
- Pull requests
- Dispatch manual

### 3. [`test-only.yml`](./test-only.yml) - Solo Testing

**Caso de uso**: Validación de feature branches sin build ni deploy.

**Características**:

- ✅ Solo ejecuta tests unitarios e integración
- ❌ Omite build y deploy
- 🚀 Ideal para feature branches

**Triggers**:

- Push a `feature/*`, `bugfix/*`, `hotfix/*`
- Pull requests

### 4. [`specific-apps.yml`](./specific-apps.yml) - Aplicaciones Específicas

**Caso de uso**: Despliegue manual de aplicaciones específicas.

**Características**:

- 🎯 Permite especificar qué aplicaciones desplegar
- 🎮 Interfaz manual con inputs
- ⚡ Opción de saltar tests para deploy rápido

**Triggers**:

- Dispatch manual únicamente

### 5. [`build-only.yml`](./build-only.yml) - Solo Build

**Caso de uso**: Build de imágenes sin despliegue.

**Características**:

- 🏗️ Solo build y push de imágenes Docker
- ❌ Omite deployment
- 📅 Incluye builds nocturnos programados
- 📊 Ejemplo de uso de outputs del workflow

**Triggers**:

- Push a `main`, `develop`
- Tags `v*`
- Cron schedule (nightly)

## 🚀 Cómo Usar los Ejemplos

### 1. Copia el ejemplo que necesites

Copia el contenido del archivo de ejemplo a `.github/workflows/` en tu repositorio.

### 2. Configura los secrets

Asegúrate de tener todos los secrets necesarios configurados en tu repositorio.

### 3. Personaliza según tus necesidades

Modifica los ejemplos según tu estructura de proyecto:

```yaml
# Ejemplo de personalización
with:
  environment: "production" # tu entorno
  eks_namespace: "my-namespace" # tu namespace
  node_version: "18.x" # tu versión de Node.js
  apps: "my-app1,my-app2" # aplicaciones específicas
```

### 4. Estructura de proyecto esperada

El workflow espera que tu proyecto tenga:

```
tu-repo/
├── .github/workflows/          # Tus workflows
├── packages/                   # Aplicaciones TypeScript
│   ├── accommodation/
│   ├── booking/
│   ├── web-bff/
│   └── ...
├── Makefile                   # Con targets: install, test, test-i
└── package.json
```

## 🛠️ Aplicaciones Soportadas

El workflow puede identificar y manejar las siguientes aplicaciones:

- `accommodation`
- `product`
- `booking`
- `apolo`
- `web-bff`
- `hermes`
- `ticketing`
- `apis-accommodation`
- `system`
- `marketing`
- `sparrow`
- `web-metrics-and-logs`
- `apis-x`
- `moebius`

## 📊 Outputs del Workflow

El workflow proporciona la siguiente información:

| Output           | Descripción                                          |
| ---------------- | ---------------------------------------------------- |
| `apps_to_deploy` | Lista separada por comas de aplicaciones a desplegar |

Ejemplo de uso:

```yaml
jobs:
  deploy:
    uses: Traventia/ci-templates/.github/workflows/ts-srv-cicd.yml@main
    # ... configuración ...

  notify:
    needs: [deploy]
    runs-on: ubuntu-latest
    steps:
      - name: Show deployed apps
        run: echo "Deployed: ${{ needs.deploy.outputs.apps_to_deploy }}"
```

## 🎛️ Parámetros de Configuración

### Inputs Disponibles

| Input                    | Descripción                                          | Default      | Tipo    |
| ------------------------ | ---------------------------------------------------- | ------------ | ------- |
| `environment`            | Entorno a desplegar (production/staging)             | `production` | string  |
| `apps`                   | Apps específicas (si no se provee, se auto-detectan) | -            | string  |
| `eks_namespace`          | Namespace de Kubernetes                              | `default`    | string  |
| `skip_tests`             | Saltar tests unitarios                               | `false`      | boolean |
| `skip_integration_tests` | Saltar tests de integración                          | `false`      | boolean |
| `skip_build`             | Saltar build                                         | `false`      | boolean |
| `skip_deploy`            | Saltar deploy                                        | `false`      | boolean |
| `node_version`           | Versión de Node.js                                   | `20.x`       | string  |

### Casos de Uso Comunes

```yaml
# Solo testing
with:
  skip_build: true
  skip_deploy: true

# Solo build
with:
  skip_deploy: true

# Deploy específico
with:
  apps: 'accommodation,booking'
  environment: 'staging'

# Deploy rápido sin tests
with:
  skip_tests: true
  skip_integration_tests: true
```

## 🔍 Troubleshooting

### Problemas Comunes

1. **Error de autenticación AWS**

   - Verifica que los secrets AWS estén configurados correctamente
   - Asegúrate de que las credenciales tengan permisos para S3 y ECR

2. **No se identifican aplicaciones**

   - Verifica que existan cambios en las aplicaciones
   - Asegúrate de usar `fetch-depth: 0` en el checkout

3. **Error de deployment**

   - Verifica que los secrets de Kubernetes estén configurados
   - Asegúrate de que el namespace existe en el cluster

4. **Tests fallan**
   - Verifica que el `Makefile` tenga los targets correctos
   - Asegúrate de que las dependencias estén correctamente instaladas

### Debug

Para debuggear problemas, puedes:

1. Activar logs detallados:

```yaml
env:
  ACTIONS_STEP_DEBUG: true
  ACTIONS_RUNNER_DEBUG: true
```

2. Usar outputs para ver qué aplicaciones se identificaron:

```yaml
- name: Debug apps
  run: echo "Apps: ${{ needs.identify-apps.outputs.apps }}"
```

## 🤝 Contribuir

Si encuentras problemas o tienes sugerencias para mejorar estos ejemplos, por favor:

1. Abre un issue describiendo el problema
2. Propón mejoras en los ejemplos existentes
3. Sugiere nuevos casos de uso que deberían estar cubiertos

## 📚 Referencias

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Reusable Workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- [Workflow Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
