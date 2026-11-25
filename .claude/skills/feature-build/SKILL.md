---
name: feature-build
description: Workflow nativo de ci-templates para construir una unidad de una feature del harness (diseño + implementación). Lo despacha /feature:build; compone con harness-unit.
---

# feature-build (nativo · ci-templates)

Construyes **una unidad** en ci-templates: los workflows reutilizables (`.github/workflows/`) y las
composite actions (`.github/actions/`) que usan los monorepos de Traventia. Trabajas a tu
criterio, sin fases ni agentes.

## Contrato del «entre» (detalle en `harness-unit`)
Lee el spec, el contrato (si existe) y tu nodo del `progress.json` de harness. Respeta el DAG. Prosa
y comentarios en español; código en inglés. **El contrato es central**: si el real difiere o es
inviable, para y repórtalo; no parchees la costura.

## Reglas duras del repo
La ley local es `.github/instructions/base.instructions.md`. Las reglas que más se rompen:
- **Repo PÚBLICO**: nada de secretos, hostnames internos, ARNs, IPs, nombres de bucket ni rutas de
  infra. Todo llega por `secrets`/`vars`/`inputs`, y los ejemplos llevan placeholders.
- **Sin versionado**: los consumidores usan `@main`, así que el merge llega a todos los pipelines en
  su siguiente ejecución. Inputs y secrets nuevos son **opcionales**, con un default que mantenga el
  comportamiento actual. No renombres ni borres inputs, outputs o jobs que alguien consuma: búscalo
  antes en `../traventia-services`, `../god`, `../traventia-web`, `../us_moebiuspro` y
  `../fenix-next`.
- **La lógica operativa vive en `../ci-scripts`** (privado, se baja de S3). Si la unidad exige
  cambiarla, eso es otra unidad, y el orden de merge va en el resumen.
- Cada workflow o action tocado actualiza su `README.md` y, si aplica, `examples/`.
- **Nunca commitees** por tu cuenta.

## Skills de conocimiento
`commit-conventions` (scopes `workflow|<nombre>`, `actions|<nombre>`).

## Cómo trabajas
1. **Explora** los workflows y actions afectados, cómo los llaman los consumidores y qué scripts de
   `ci-scripts` invocan.
2. **Nota de diseño breve**, solo si la unidad lo merece, donde diga `harness-unit`.
3. **Implementa.** Si una variante (`-ovh`, `-selfhosted`, `-v2`…) comparte la lógica, decide de
   forma explícita si la tocas también y dilo.
4. **Verifica** en local, porque no hay suite de tests:
   - el YAML parsea (`python3 -c 'import yaml,sys; yaml.safe_load(open(sys.argv[1]))' <f>`);
   - `actionlint` y `shellcheck` si están instalados (si no, `npx`/`docker` cuando se pueda; si no
     se puede, dilo);
   - `bash -n` sobre los `run:` no triviales, extraídos a un fichero;
   - `grep` de secretos o literales de infra en el diff.
5. Pasa **`/code-review`** sobre el diff completo y arregla lo que proceda.
6. **Para** antes de commitear y resume: qué hiciste, ficheros tocados, qué verificaste y cómo, qué
   consumidores se ven afectados, el orden de merge respecto a `ci-scripts` y a los repos de
   producto, y los pasos manuales pendientes (secrets o vars de la org, runners).
