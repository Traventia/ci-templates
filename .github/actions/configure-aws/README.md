# configure-aws

Único punto del repositorio donde se configuran credenciales de AWS. Autentica
por **OIDC** si le pasas un rol, y con **clave estática** si no.

Existe para que el paso de la clave a OIDC sea un cambio de entrada y no una
reescritura: las dos formas conviven mientras los repositorios consumidores
migran uno a uno.

> Este repositorio es **público**. El ARN del rol no se escribe aquí: llega como
> input desde el workflow que llama. Guárdalo como variable de organización
> (por ejemplo `AWS_CI_ROLE_ARN`) y pásalo desde cada consumidor.

## Inputs

| input | obligatorio | descripción |
| --- | --- | --- |
| `role-to-assume` | no | ARN del rol a asumir por OIDC. Si viene, manda sobre la clave. |
| `aws-access-key-id` | no | Clave de acceso. Solo se usa si `role-to-assume` viene vacío. |
| `aws-secret-access-key` | no | Secreto de la clave. |
| `aws-region` | no | Por defecto `eu-west-1`. |

Hay que pasar **una de las dos cosas**. Si no llega ninguna, la acción falla con
un mensaje que lo dice, en vez de dejar que el error salte tres pasos más tarde
como un `Unable to locate credentials`.

## Los dos permisos que hay que declarar

Al usar OIDC, el job necesita `id-token: write`. Y aquí está el detalle que más
se olvida: **en un workflow reutilizable el permiso lo concede quien llama**. La
plantilla no puede ampliarse a sí misma, así que declararlo solo en
`ci-templates` no sirve de nada.

Además, **declarar `permissions` sustituye el conjunto por defecto en lugar de
ampliarlo**. Si pones únicamente `id-token: write`, pierdes `contents: read` y
`actions/checkout` deja de funcionar. Hay que escribir el bloque completo:

```yaml
jobs:
  deploy:
    permissions:
      id-token: write
      contents: read
    uses: Traventia/ci-templates/.github/workflows/ts-srv-lambda-cicd.yml@main
    with:
      aws_role_to_assume: ${{ vars.AWS_CI_ROLE_ARN }}
```

La acción comprueba esto antes de intentar nada: si pasas un rol y el job no
puede pedir un token OIDC, falla señalando el permiso que falta en vez de
devolver el error genérico de STS, que no menciona los permisos por ningún lado.

## Seguir con la clave

No hay que hacer nada. Un consumidor que no pase `aws_role_to_assume` sigue
usando `AWS_ACCESS_KEY_ID` y `AWS_SECRET_ACCESS_KEY` igual que antes.

## Ver quién ha migrado

Al asumir el rol, la acción pone un `role-session-name` con el nombre del
repositorio y el id de la ejecución. Eso llega a CloudTrail, así que se puede
comprobar qué repositorios han pasado ya a OIDC mirando los eventos, sin
preguntarle a nadie.
