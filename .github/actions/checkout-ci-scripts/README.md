# Checkout CI Scripts Action

Esta GitHub Action descarga el archivo `ci-scripts.zip` desde un bucket de S3, lo descomprime y coloca el contenido en la carpeta `ci/scripts` del workspace actual. **Incluye caching ultra-rápido** usando S3 metadata para evitar descargas innecesarias.

## Uso

```yaml
- name: Checkout CI Scripts
  uses: ./.github/actions/checkout-ci-scripts
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: ${{ secrets.AWS_CI_BUCKET_REGION }}
    s3-bucket: ${{ secrets.AWS_CI_BUCKET }}
```

## Inputs

| Input                   | Descripción                                      | Requerido |
| ----------------------- | ------------------------------------------------ | --------- |
| `aws-access-key-id`     | AWS Access Key ID                                | ✅        |
| `aws-secret-access-key` | AWS Secret Access Key                            | ✅        |
| `aws-region`            | Región AWS donde está el bucket S3               | ✅        |
| `s3-bucket`             | Nombre del bucket S3 que contiene ci-scripts.zip | ✅        |

## ⚡ Funcionalidad de Caching Ultra-Rápido

### Cómo funciona:

1. **HEAD request a S3**: Obtiene solo la metadata del objeto (ultra-rápido ~100-200ms)
2. **Extrae hash**: Lee el `content-hash` de la metadata del ZIP
3. **Busca en cache**: Usa GitHub Actions cache con la key `ci-scripts-{hash}`
4. **Cache hit**: Si encuentra los scripts en cache, los reutiliza directamente
5. **Cache miss**: Si no están en cache, descarga el ZIP, lo extrae y guarda en cache

### Beneficios:

- ⚡ **Ultra velocidad**: HEAD request ~200ms vs GET file ~1-2s
- 💰 **Ahorro de costos**: Menos transferencias desde S3
- 🔄 **Eficiencia**: Reutilización entre runners del mismo repositorio
- 🎯 **Precisión**: Solo actualiza cuando el contenido realmente cambia
- 🏗️ **Menos archivos**: Solo el ZIP, no archivos adicionales de hash

## Qué hace

### Flujo con cache:

1. **Configura credenciales AWS**: Autentica con AWS usando las credenciales proporcionadas
2. **Obtiene hash remoto**: Descarga el hash del contenido actual desde S3
3. **Verifica cache**: Busca si ya existen scripts con ese hash en cache
4. **Si cache hit**:
   - ✅ Reutiliza los scripts cacheados
   - ✅ Verifica que estén disponibles
5. **Si cache miss**:
   - 📥 Descarga `ci-scripts.zip` desde S3
   - 📦 Extrae el contenido en `ci/`
   - 🗑️ Limpia archivos temporales
   - 💾 Guarda en cache para futuros usos
6. **Establece permisos**: Da permisos de ejecución a todos los archivos `.sh`

## Estructura resultante

Después de ejecutar esta action, tendrás:

```
ci/
└── scripts/
    ├── build_and_ecr_push_images.sh
    ├── deploy_apps.sh
    ├── identify_changed_apps.js
    ├── notify_deploy_apolo.js
    ├── notify_deploy_teams.js
    ├── notify_deploy.sh
    └── tools/
        └── message_to_teams.js
```

## Ejemplo completo

```yaml
name: Deploy with CI Scripts

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Checkout CI Scripts
        uses: ./.github/actions/checkout-ci-scripts
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_CI_BUCKET_REGION }}
          s3-bucket: ${{ secrets.AWS_CI_BUCKET }}

      - name: Use CI Scripts
        run: |
          # Los scripts están disponibles en ci/scripts/
          bash ci/scripts/build_and_ecr_push_images.sh "app1,app2"

## Logs de ejemplo

### Cache Hit:
```

✅ Cache hit - using cached ci-scripts
Contents of cached ci/scripts:
total 48
-rwxr-xr-x 1 runner docker 1234 Jul 3 10:30 build_and_ecr_push_images.sh
-rwxr-xr-x 1 runner docker 2345 Jul 3 10:30 deploy_apps.sh
🚀 CI Scripts ready (from cache)

```

### Cache Miss:
```

Cache miss - downloading ci-scripts.zip from S3 bucket: my-ci-bucket
File downloaded successfully
Extracting ci-scripts.zip to ci/ directory
✅ ci/scripts directory created successfully
🚀 CI Scripts ready (downloaded and cached)

```

```

## Secrets requeridos

Asegúrate de configurar estos secrets en tu repositorio:

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_CI_BUCKET_REGION`
- `AWS_CI_BUCKET`

## Troubleshooting

### Cache no funciona

- Verifica que el archivo `scripts-hash.txt` esté disponible en S3
- El hash debe cambiar cuando los scripts cambien
- GitHub Actions cache tiene límites de tamaño y retención

### Error de permisos

- Asegúrate de que las credenciales AWS tengan permisos de lectura en el bucket S3
- Verifica que el bucket y la región sean correctos
