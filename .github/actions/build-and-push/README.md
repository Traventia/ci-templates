# Build and Push Action

Esta GitHub Action construye y pushea imágenes Docker para aplicaciones especificadas. Utiliza el script `build_and_ecr_push_images.sh` de ci-scripts.

## Uso

```yaml
- name: Build and push Docker images
  uses: Traventia/ci-templates/.github/actions/build-and-push@main
  with:
    apps: "accommodation,booking,product"
    ecr_registry: ${{ steps.login-ecr.outputs.registry }}
    image_tag: ${{ github.sha }}
    npm_token: ${{ secrets.NPM_AUTH_TOKEN }}
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: ${{ secrets.AWS_CI_BUCKET_REGION }}
    s3-bucket: ${{ secrets.AWS_CI_BUCKET }}
```

## Inputs

| Input                   | Descripción                                      | Requerido | Default     |
| ----------------------- | ------------------------------------------------ | --------- | ----------- |
| `apps`                  | Lista de aplicaciones separadas por coma         | ✅        |             |
| `ecr_registry`          | URL del registro ECR                             | ✅        |             |
| `image_tag`             | Tag de la imagen Docker                          | ✅        |             |
| `npm_token`             | Token de autenticación NPM                       | ❌        |             |
| `aws-access-key-id`     | AWS Access Key ID                                | ✅        |             |
| `aws-secret-access-key` | AWS Secret Access Key                            | ✅        |             |
| `aws-region`            | Región AWS donde está el bucket S3               | ❌        | `eu-west-1` |
| `s3-bucket`             | Nombre del bucket S3 que contiene ci-scripts.zip | ✅        |             |
| `is_bc`                 | Si es un build BC (1 para true, 0 para false)    | ❌        | `0`         |

## Funcionamiento

1. **Descarga scripts**: Descarga el archivo `ci-scripts.zip` desde S3
2. **Construye imágenes**: Para cada aplicación:
   - Verifica si la imagen ya existe en ECR
   - Si no existe, construye la imagen Docker
   - Pushea la imagen al registro ECR
3. **Optimización**: Evita reconstruir imágenes que ya existen

## Características especiales

- **Verificación de existencia**: Comprueba si la imagen ya existe antes de construir
- **Soporte para Hermes**: Maneja el Dockerfile especial para la aplicación Hermes
- **Soporte BC**: Permite construir versiones BC de las aplicaciones
- **NPM Token**: Incluye soporte para autenticación NPM privada

## Variables de entorno utilizadas

- `ECR_REGISTRY`: URL del registro ECR
- `IMAGE_TAG`: Tag de la imagen
- `NPM_TOKEN`: Token NPM (opcional)
- `IS_BC`: Flag para builds BC

## Secrets requeridos

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_CI_BUCKET_REGION`
- `AWS_CI_BUCKET`
- `NPM_AUTH_TOKEN` (opcional)
