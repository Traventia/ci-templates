# TypeScript Services CI Examples

Este directorio contiene ejemplos de uso del workflow reutilizable `ts-srv-ci.yml` para diferentes escenarios de integración continua sin despliegue.

## Archivos de ejemplo

- `basic-usage.yml` - Uso básico del workflow
- `test-only.yml` - Solo ejecutar tests, sin build
- `build-only.yml` - Solo build, sin tests
- `specific-apps.yml` - Build de aplicaciones específicas
- `advanced-usage.yml` - Configuración avanzada con múltiples opciones

## Uso general

El workflow `ts-srv-ci` es ideal para:

- Pull requests donde solo necesitas validar el código
- Pipelines de CI que no requieren despliegue
- Validación de builds antes de merge
- Testing continuo sin publicación de imágenes
