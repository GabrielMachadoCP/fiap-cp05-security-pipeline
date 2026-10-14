# Guia de execucao e evidencias - CP05

## Repositorio

Criar o repositorio publico no GitHub com `index.html`, `Dockerfile`, `.gitleaks.toml`, `.semgrep.yml`, `.dockerignore`, `README.md` e `.github/workflows/security.yml`.

## Evidencia 1: fluxo normal

Fazer upload dos arquivos para a `main`, abrir **Actions**, selecionar **Security Pipeline**, entrar na execucao do push e capturar a tela mostrando os tres jobs. Se algum falhar, consultar o log antes de prosseguir.

## Evidencia 2: Gitleaks

A partir da `main`, criar um branch `teste-gitleaks`. Nele, adicionar um arquivo `teste-segredo.txt` com uma atribuicao de `CP05_DEMO_SECRET` a um texto ficticio que siga a regra adicional de `.gitleaks.toml`. Confirmar o commit neste branch. Abrir Actions para localizar a execucao reprovada e tirar print do job Gitleaks. Nao mesclar o branch na `main`. Excluir o branch de teste ao terminar.

## Evidencia 3: Semgrep

A partir da `main`, criar o branch `teste-semgrep`. Criar um arquivo `teste-inseguro.py` contendo uma funcao que usa `eval(entrada)`; o codigo nao deve ser executado. Confirmar o commit no branch. Abrir Actions e registrar o job Semgrep reprovado, com o nome da regra e a linha detectada. Nao mesclar o branch na `main`. Excluir o branch de teste.

## Evidencia 4: Trivy (adicional)

Na execucao normal, abrir o job Trivy e capturar o resultado da verificacao da imagem. Somente marcar o job como aprovado se nao houver CVEs criticos dentro do escopo selecionado.

## Protecao da `main`

Em Settings > Rules > Rulesets (ou Settings > Branches, dependendo da interface/permissoes), criar uma regra para a `main` exigindo Pull Request e verificacoes obrigatorias `Gitleaks - Segredos`, `Semgrep - Codigo` e `Trivy - Docker`. Isso impede merge com checks reprovados. Falha no Actions, sozinha, nao impede um push direto sem protecao de branch.

## Sugestao de PDF

1. Capa: FIAP, disciplina, CP05, participantes, turma e RMs.
2. Link para o repositorio e descricao da estrutura.
3. Print do workflow completo e explicacao dos tres jobs.
4. Print do Gitleaks reprovando segredo falso; explicacao.
5. Print do Semgrep reprovando eval; explicacao.
6. Print da analise Trivy, com observacao sobre severidade CRITICAL.
7. Conclusao e link para a execucao no GitHub Actions.
