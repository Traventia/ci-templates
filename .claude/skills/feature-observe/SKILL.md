---
name: feature-observe
description: Dónde mirar tras el merge de una unidad de ci-templates. La carga /feature:observe para comprobar en SOLO LECTURA que el cambio ha llegado y funciona.
---

# feature-observe (ci-templates)

## Cómo llega a «prod»
No es un servicio: no tiene Datadog, k8s ni `traventia-gitops` propios, ni CI propia. El merge a
`main` **ya es el despliegue**: los consumidores referencian `@main` y la siguiente ejecución de sus
pipelines usa la versión nueva. No hay espera de build.

## Qué comprobar
- **Consumo real**: las siguientes ejecuciones de los pipelines consumidores que usan el workflow o
  la action tocados, con `gh run list --repo Traventia/<repo> --limit 5` y
  `gh run view <id> --log`. Consumidores habituales: `traventia-services` (la variante
  `ts-srv-cicd-matrix-recheck-split-runners-ovh`, entre otras), `god`, `traventia-web`,
  `us_moebiuspro` y `fenix-next`.
- **Que los jobs tocados terminan bien** (identify-apps, tests, build-and-push, deploy-to-k8s) en el
  runner esperado (`runs-on` y la etiqueta del runner en el log) y que loguean lo esperado.
- **Efectos de deploy** (si tocaste build o deploy): imagen nueva en el registry y rollout en el
  cluster, vistos con `/feature:observe` del repo de producto o con Datadog en solo lectura.
- **Canario: `traventia-services`**. Es el que más plantillas usa y corre en OVH en cada push. Su
  primera ejecución tras el merge es la que valida el cambio; los demás consumidores, después.

## Nunca
- Relanzar pipelines de deploy (`gh run rerun`, `workflow_dispatch`) ni empujar commits vacíos para
  forzar una ejecución.
- Escribir en el registry, S3, EKS u OVH, ni tocar los secrets o vars de la org.
