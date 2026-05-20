# axiom-engines

> Registry de distribuição dos bundles do **engine Axiom** — consumido
> pelo launcher do StateDelta.

Cada *engine release* do Axiom é um bundle CJS único, minificado e
**auto-contido** (`engine-X.Y.Z.cjs`) — todas as dependências
inlinadas, zero `node_modules`. Os bundles são baixados em runtime pelo
launcher, no estilo de versionamento de engine do Minecraft (snapshot
imutável + validação por hash).

## Como funciona

- O bundle é produzido no monorepo `statedelta-axiom` (privado) via
  `pnpm bundle` — que também gera os metadados (`engine-X.Y.Z.json`:
  versão, tamanho, `sha256`, data) e roda um smoke test.
- Cada engine release é publicada aqui como um **GitHub Release**; o
  `.cjs` e o `.json` ficam como *assets* — **não** são commitados no
  git (artefatos binários ficam fora do histórico).
- O `engine-manifest.json` (na raiz, versionado no git) é o **índice**
  que o launcher consome.

## `engine-manifest.json`

```json
{
  "engine": "axiom",
  "latest": "0.3.0",
  "channels": { "0.3": "0.3.0" },
  "versions": [
    {
      "version": "0.3.0",
      "file": "engine-0.3.0.cjs",
      "size": 357847,
      "sha256": "…",
      "built": "2026-05-20T…",
      "url": "https://github.com/statedelta/axiom-engines/releases/download/v0.3.0/engine-0.3.0.cjs"
    }
  ]
}
```

- `versions[]` — toda versão exata publicada, imutável, com `sha256`.
- `channels` — ponteiros móveis `major.minor → última patch`. O
  launcher exibe os channels (UI simplificada); o patch é resolvido
  automaticamente.
- `latest` — atalho pra última versão publicada.

## Fluxo do launcher

1. Baixa o `engine-manifest.json`.
2. Resolve o channel escolhido (ex.: `0.3`) → versão exata (`0.3.0`).
3. Baixa o asset `.cjs` da release correspondente.
4. Valida o `sha256` contra o manifest.
5. Carrega o bundle e instancia o engine via `createRuntime`.

Resolvido o channel, o launcher **fixa a versão exata** no projeto/save
— a faixa é só pra escolher; o pin garante replay determinístico.

## Promover uma engine release

`changeset publish` do monorepo não toca aqui — promover uma versão a
engine release é um ato **curado**: nem toda versão do runtime vira
bundle (snapshots vs releases oficiais). Publicar:

```bash
gh release create v0.3.0 --repo statedelta/axiom-engines \
  engine-0.3.0.cjs engine-0.3.0.json \
  --title "Axiom Engine 0.3.0"
```

Depois adiciona-se a entry correspondente ao `engine-manifest.json`.
