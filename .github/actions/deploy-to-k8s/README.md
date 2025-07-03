# Deploy to Kubernetes Action

Esta GitHub Action despliega aplicaciones en un cluster de Kubernetes. Utiliza los scripts `deploy_apps.sh` y `notify_deploy.sh` de ci-scripts.

## Uso

```yaml
- name: Deploy to Kubernetes
  uses: Traventia/ci-templates/.github/actions/deploy-to-k8s@main
  with:
    apps: "accommodation,booking,product"
    ecr_registry: ${{ steps.login-ecr.outputs.registry }}
    image_tag: ${{ github.sha }}
    k8s_user_token: ${{ secrets.EKS_I2_USER_TOKEN }}
    k8s_ca: ${{ secrets.EKS_I2_CA }}
    k8s_server_url: ${{ secrets.EKS_I2_URL }}
    k8s_user_name: ${{ secrets.EKS_I2_USER_NAME }}
    k8s_namespace: "default"
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: ${{ secrets.AWS_CI_BUCKET_REGION }}
    s3-bucket: ${{ secrets.AWS_CI_BUCKET }}
    deploy_env: "production"
```

## Inputs

| Input                   | Descripción                                        | Requerido | Default      |
| ----------------------- | -------------------------------------------------- | --------- | ------------ |
| `apps`                  | Lista de aplicaciones separadas por coma           | ✅        |              |
| `ecr_registry`          | URL del registro ECR                               | ✅        |              |
| `image_tag`             | Tag de la imagen Docker                            | ✅        |              |
| `k8s_user_token`        | Token de usuario de Kubernetes                     | ✅        |              |
| `k8s_ca`                | Certificado CA de Kubernetes                       | ✅        |              |
| `k8s_server_url`        | URL del servidor de Kubernetes                     | ✅        |              |
| `k8s_user_name`         | Nombre de usuario de Kubernetes                    | ✅        |              |
| `k8s_namespace`         | Namespace de Kubernetes                            | ✅        |              |
| `aws-access-key-id`     | AWS Access Key ID                                  | ✅        |              |
| `aws-secret-access-key` | AWS Secret Access Key                              | ✅        |              |
| `aws-region`            | Región AWS donde está el bucket S3                 | ❌        | `eu-west-1`  |
| `s3-bucket`             | Nombre del bucket S3 que contiene ci-scripts.zip   | ✅        |              |
| `deploy_env`            | Entorno de despliegue                              | ❌        | `production` |
| `is_bc`                 | Si es un despliegue BC (1 para true, 0 para false) | ❌        | `0`          |

## Funcionamiento

1. **Descarga scripts**: Descarga el archivo `ci-scripts.zip` desde S3
2. **Configura Node.js**: Instala Node.js 18
3. **Instala kubectl**: Descarga e instala la herramienta kubectl
4. **Despliega aplicaciones**:
   - Configura credenciales de Kubernetes
   - Actualiza los deployments con las nuevas imágenes
5. **Notifica despliegue**: Si el despliegue es exitoso:
   - Notifica a Apolo (sistema interno)
   - Envía mensaje a Microsoft Teams

## Proceso de despliegue

### Configuración de Kubernetes

- Configura el cluster usando las credenciales proporcionadas
- Establece el contexto de kubectl
- Autentica con el cluster

### Despliegue

- Para cada aplicación, actualiza el deployment correspondiente
- Modifica la imagen del contenedor con el nuevo tag
- Verifica que el despliegue sea exitoso

### Notificaciones

- **Apolo**: Registra el despliegue en el sistema interno
- **Teams**: Envía mensaje con detalles del despliegue

## Variables de entorno utilizadas

- `ECR_REGISTRY`: URL del registro ECR
- `IMAGE_TAG`: Tag de la imagen
- `K8S_USER_TOKEN`: Token de Kubernetes (base64)
- `K8S_I2_CA`: Certificado CA (base64)
- `K8S_SERVER_URL`: URL del servidor
- `K8S_CLUSTER_NAME`: Nombre del cluster
- `K8S_USER_NAME`: Nombre de usuario
- `K8S_NS`: Namespace
- `DEPLOY_ENV`: Entorno de despliegue
- `IS_BC`: Flag para despliegues BC
- `USER_CHANGE`: Usuario que realizó el cambio
- `GITHUB_CONTEXT`: Contexto de GitHub (JSON)

## Secrets requeridos

### AWS

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_CI_BUCKET_REGION`
- `AWS_CI_BUCKET`

### Kubernetes

- `EKS_I2_USER_TOKEN`
- `EKS_I2_CA`
- `EKS_I2_URL`
- `EKS_I2_USER_NAME`

## Gestión de errores

- Si el despliegue falla, la action termina con error y no envía notificaciones
- Cada paso del proceso se valida antes de continuar
- Los logs proporcionan información detallada del proceso
