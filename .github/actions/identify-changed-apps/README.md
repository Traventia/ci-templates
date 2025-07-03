# Identify Changed Apps Action

Esta GitHub Action identifica qué aplicaciones han cambiado basándose en el historial de git y despliegues previos. Utiliza el script `identify_changed_apps.js` de ci-scripts.

## Uso

```yaml
- name: Identify changed apps
  id: identify
  uses: Traventia/ci-templates/.github/actions/identify-changed-apps@main
  with:
    environment: 'production'
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: ${{ secrets.AWS_CI_BUCKET_REGION }}
    s3-bucket: ${{ secrets.AWS_CI_BUCKET }}

- name: Use identified apps
  run: echo "Changed apps: ${{ steps.identify.outputs.changed_apps }}"
```

## Inputs

| Input                   | Descripción                                      | Requerido | Default      |
| ----------------------- | ------------------------------------------------ | --------- | ------------ |
| `environment`           | Entorno a verificar (production o staging)       | ❌        | `production` |
| `aws-access-key-id`     | AWS Access Key ID                                | ✅        |              |
| `aws-secret-access-key` | AWS Secret Access Key                            | ✅        |              |
| `aws-region`            | Región AWS donde está el bucket S3               | ❌        | `eu-west-1`  |
| `s3-bucket`             | Nombre del bucket S3 que contiene ci-scripts.zip | ✅        |              |

## Outputs

| Output         | Descripción                                        |
| -------------- | -------------------------------------------------- |
| `changed_apps` | Lista de aplicaciones cambiadas separadas por coma |

## Funcionamiento

1. **Descarga scripts**: Descarga el archivo `ci-scripts.zip` desde S3
2. **Configura Node.js**: Instala Node.js 18
3. **Ejecuta identificación**: Ejecuta `identify_changed_apps.js` con el entorno especificado
4. **Retorna resultado**: Devuelve la lista de aplicaciones cambiadas

## Aplicaciones válidas

El script puede identificar cambios en las siguientes aplicaciones:

- accommodation
- product
- booking
- apolo
- web-bff
- hermes
- ticketing
- apis-accommodation
- system
- marketing
- sparrow
- web-metrics-and-logs
- apis-x
- moebius

## Secrets requeridos

Asegúrate de configurar estos secrets en tu repositorio:

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_CI_BUCKET_REGION`
- `AWS_CI_BUCKET`
