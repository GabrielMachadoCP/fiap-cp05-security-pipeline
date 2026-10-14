# FIAP - CP05: Security Pipeline

Exemplo didatico com site estatico em Nginx, Dockerfile e um workflow GitHub Actions com tres jobs independentes:

- **Gitleaks**: procura segredos reais pelas regras padrao e permite demonstrar um segredo ficticio por uma regra local.
- **Semgrep**: procura `eval(...)` em arquivos Python e JavaScript/TypeScript para uma demonstracao deterministica. Para uso real, acrescente regras adicionais.
- **Trivy**: examina a imagem Docker em busca de CVEs criticos.

## Como usar

1. Suba todos os arquivos para a raiz de um repositorio GitHub (preservando `.github/workflows/security.yml` e os nomes que comecam com ponto).
2. Em **Actions > Security Pipeline**, abra a execucao disparada pelo push.
3. Confira o resultado dos tres jobs. Se Trivy encontrar vulnerabilidades na imagem, atualize a imagem-base e teste de novo.
4. Crie dois branches separados a partir da `main`, para fazer cada sabotagem sem contaminar o historico da `main`.
5. Capture os logs de falha do Gitleaks e do Semgrep e nao mescle esses branches na `main`.
6. Consulte `GUIA_CP05.md` para orientacoes detalhadas.

Nao use nenhum segredo de verdade, nem mesmo para testar o pipeline.
