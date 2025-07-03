# Build Docker Images Only Action

Esta GitHub Action construye imágenes Docker para las aplicaciones especificadas, pero **no realiza el push** al registro. Utiliza el script `build_and_ecr_push_images.sh` de ci-scripts y establece la variable de entorno `NOPUSH=1` para evitar el push.

## Uso

```yaml
- name: Build Docker images only
  uses: Traventia/ci-templates/.github/actions/only-build@main
  with:
    apps: "app1,app2"
    ecr_registry: "123456789.dkr.ecr.eu-west-1.amazonaws.com"
    image_tag: "latest"
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    s3-bucket: "my-ci-scripts-bucket"
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

1. **Descarga scripts**: Descarga el archivo `ci-scripts.zip` desde S3.
2. **Construye imágenes**: Para cada aplicación, construye la imagen Docker.
3. **No pushea**: No realiza el push de las imágenes al registro (NOPUSH=1).

## Características especiales

- Útil para validar builds locales o en CI sin publicar imágenes.
- Compatible con los mismos parámetros que la acción `build-and-push`.

## Prerrequisitos

- Acceso a AWS y al bucket S3 con los scripts privados.
- Definir los secretos necesarios en el repositorio.

## Troubleshooting

- Verifica que la variable `NOPUSH` esté en 1 si no deseas pushear imágenes.
- Revisa los logs del job para detalles de errores de build.
