# FIAP | CP05 - Security Pipeline (Cloud / DevOps)

[![Security Pipeline](https://github.com/GabrielMachadoCP/fiap-cp05-security-pipeline/actions/workflows/security.yml/badge.svg?branch=main)](https://github.com/GabrielMachadoCP/fiap-cp05-security-pipeline/actions/workflows/security.yml)

> **Checkpoint 05 - Engenharia de Software (FIAP, 4ESPY)**  
> Pipeline de referência para validação automatizada de segredos, código vulnerável e imagens Docker. Este README documenta os testes e descreve as **15 capturas de tela** organizadas no relatório PDF do trabalho.

## Integrantes

| Nome completo | RM |
| --- | --- |
| Gabriel Machado Carrara Pimentel | 99880 |
| Lourenzo Ramos | 99951 |
| Vitor Hugo Rodrigues | 97758 |

## Objetivo do projeto

Após a simulação de um vazamento de chave de API em um repositório, foi desenvolvido um pipeline de segurança que executa verificações a cada **push** e **pull request**. O pipeline possui três jobs:

| Job obrigatório | Ferramenta | O que valida | Ação quando encontra um problema |
| --- | --- | --- | --- |
| \`Gitleaks - Segredos\` | [Gitleaks](https://github.com/gitleaks/gitleaks) | Segredos e credenciais em arquivos e histórico Git | Reprova o job |
| \`Semgrep - Codigo\` | [Semgrep](https://semgrep.dev/) | Regras de análise estática, incluindo uso inseguro de \`eval()\` | Reprova o job |
| \`Trivy - Docker\` | [Trivy](https://trivy.dev/) | CVEs de severidade crítica da imagem Docker | Reprova o job |

\`\`\`mermaid
flowchart TD
    A[Push ou Pull Request] --> B[GitHub Actions]
    B --> C[Gitleaks: segredos]
    B --> D[Semgrep: código]
    B --> E[Docker build + Trivy: CVEs]
    C --> F{Checks obrigatórios aprovados?}
    D --> F
    E --> F
    F -->|Sim| G[PR elegível para merge]
    F -->|Não| H[Merge bloqueado]
\`\`\`

## Estrutura do repositório

\`\`\`text
fiap-cp05-security-pipeline/
├── .github/workflows/security.yml  # Pipeline com 3 jobs
├── .gitleaks.toml                   # Regra de demonstração para segredo fictício
├── .semgrep.yml                     # Regra de análise estática
├── .dockerignore
├── Dockerfile                       # Imagem Nginx para o site
├── index.html                       # Página web de demonstração
├── README.md
└── GUIA_CP05.md
\`\`\`

Os arquivos \`teste-segredo.txt\` e \`teste-inseguro.py\` foram criados **somente nas branches de teste**, e não foram incorporados à \`main\`.

## Evidências técnicas

### 1. Repositório inicial e pipeline aprovado

O repositório possui o site HTML, Dockerfile e configurações das três ferramentas. A [execução inicial do pipeline na main](https://github.com/GabrielMachadoCP/fiap-cp05-security-pipeline/actions/runs/38492564223) foi **aprovada nos três jobs**, registrando o comportamento esperado em um estado sem sabotagem.

### 2. Sabotagem do Gitleaks

Foi criada a branch [\`teste-gitleaks\`](https://github.com/GabrielMachadoCP/fiap-cp05-security-pipeline/tree/teste-gitleaks), contendo um arquivo de texto com uma **credencial totalmente fictícia** para disparar uma regra de detecção personalizada do Gitleaks.

- [Falha no push da branch](https://github.com/GabrielMachadoCP/fiap-cp05-security-pipeline/actions/runs/38492920850).
- [Pull Request #1 - teste de segurança Gitleaks](https://github.com/GabrielMachadoCP/fiap-cp05-security-pipeline/pull/1).
- [Execução do Pull Request com Gitleaks reprovado](https://github.com/GabrielMachadoCP/fiap-cp05-security-pipeline/actions/runs/38494385284).

**Resultado:** Gitleaks falhou como esperado, enquanto Semgrep e Trivy foram aprovados. A regra da branch principal tornou o check reprovado impeditivo para o merge.

### 3. Sabotagem do Semgrep

Foi criada a branch [\`teste-semgrep\`](https://github.com/GabrielMachadoCP/fiap-cp05-security-pipeline/tree/teste-semgrep) contendo \`teste-inseguro.py\`, um exemplo propositalmente inseguro de uso de \`eval(entrada)\`. Trata-se de um **teste estático**, sem execução de entrada maliciosa.

- [Falha no push da branch](https://github.com/GabrielMachadoCP/fiap-cp05-security-pipeline/actions/runs/38493190613).
- [Pull Request #2 - teste de segurança Semgrep](https://github.com/GabrielMachadoCP/fiap-cp05-security-pipeline/pull/2).
- [Execução do Pull Request com Semgrep reprovado](https://github.com/GabrielMachadoCP/fiap-cp05-security-pipeline/actions/runs/38494580474).

**Resultado:** Semgrep reprovou a análise do código, enquanto Gitleaks e Trivy passaram. O merge foi bloqueado pelo status check obrigatório.

### 4. Proteção da branch principal

A [regra \`CP05 - Protecao da main\`](https://github.com/GabrielMachadoCP/fiap-cp05-security-pipeline/rules/25041682) está **ativa** e exige:

- Pull Request para incorporar alterações na branch \`main\`;
- aprovação de \`Gitleaks - Segredos\`, \`Semgrep - Codigo\` e \`Trivy - Docker\`;
- bloqueio de force push e exclusão da branch.

Os PRs de sabotagem permaneceram abertos e **não devem ser mesclados**.

### 5. Docker e Trivy

O [Dockerfile](https://github.com/GabrielMachadoCP/fiap-cp05-security-pipeline/blob/main/Dockerfile) usa Nginx Alpine para servir o site estático. O job Trivy constrói a imagem no próprio GitHub Actions e examina possíveis CVEs críticos:

\`\`\`bash
docker build -t cp05-site:ci .
\`\`\`

Também foi realizado um teste opcional no **Docker Desktop**, com a imagem local \`fiap-cp05:latest\` e acesso ao site em \`http://localhost:8080\`. O build e a verificação de CVEs do pipeline foram aprovados nas execuções documentadas.

Para reproduzir localmente:

\`\`\`bash
docker build -t fiap-cp05 .
docker run -d --name cp05-web -p 8080:80 fiap-cp05
# Abrir http://localhost:8080 no navegador
docker stop cp05-web
docker rm cp05-web
\`\`\`

## Índice das 15 capturas de tela do relatório

As **capturas originais estão incorporadas ao relatório PDF da entrega**. A tabela abaixo explica brevemente o que cada print comprova e, quando disponível, leva à evidência correspondente no próprio GitHub.

| Print | Evidência | O que foi realizado | Conferência on-line |
| :---: | --- | --- | --- |
| 01 | Estrutura do repositório | Arquivos do site, Dockerfile e pipeline publicados. | [Código](https://github.com/GabrielMachadoCP/fiap-cp05-security-pipeline) |
| 02 | Pipeline aprovado | Gitleaks, Semgrep e Trivy finalizados com sucesso na main. | [Execução inicial](https://github.com/GabrielMachadoCP/fiap-cp05-security-pipeline/actions/runs/38492564223) |
| 03 | Branch Gitleaks | Branch isolada criada para o teste de segredo fictício. | [Branch](https://github.com/GabrielMachadoCP/fiap-cp05-security-pipeline/tree/teste-gitleaks) |
| 04 | Segredo fictício | Arquivo de demonstração adicionado fora da main. | [Arquivo de teste](https://github.com/GabrielMachadoCP/fiap-cp05-security-pipeline/blob/teste-gitleaks/teste-segredo.txt) |
| 05 | Falha de segurança Gitleaks | Commit acionou a barreira e reprovou a execução. | [Run reprovada](https://github.com/GabrielMachadoCP/fiap-cp05-security-pipeline/actions/runs/38492920850) |
| 06 | Criação do PR Gitleaks | Alteração insegura foi proposta para main sem mesclagem. | [PR #1](https://github.com/GabrielMachadoCP/fiap-cp05-security-pipeline/pull/1) |
| 07 | Bloqueio do PR Gitleaks | Checks required do Gitleaks falharam e o merge ficou indisponível. | [PR #1 - checks](https://github.com/GabrielMachadoCP/fiap-cp05-security-pipeline/pull/1/checks) |
| 08 | Código inseguro Semgrep | \`eval(entrada)\` usado apenas para provocar uma detecção estática. | [Arquivo de teste](https://github.com/GabrielMachadoCP/fiap-cp05-security-pipeline/blob/teste-semgrep/teste-inseguro.py) |
| 09 | Branch Semgrep | Código de sabotagem separado da main. | [Branch](https://github.com/GabrielMachadoCP/fiap-cp05-security-pipeline/tree/teste-semgrep) |
| 10 | Falha Semgrep | Análise estática terminou com erro esperado. | [Run reprovada](https://github.com/GabrielMachadoCP/fiap-cp05-security-pipeline/actions/runs/38493190613) |
| 11 | Criação do PR Semgrep | Pull Request usado para acionar as verificações obrigatórias. | [PR #2](https://github.com/GabrielMachadoCP/fiap-cp05-security-pipeline/pull/2) |
| 12 | Bloqueio do PR Semgrep | Semgrep falhou e impediu a mesclagem na branch protegida. | [PR #2 - checks](https://github.com/GabrielMachadoCP/fiap-cp05-security-pipeline/pull/2/checks) |
| 13 | Ruleset da main | Proteção ativa com três checks e exigência de PR. | [Regra de proteção](https://github.com/GabrielMachadoCP/fiap-cp05-security-pipeline/rules/25041682) |
| 14 | Imagem no Docker Desktop | Imagem \`fiap-cp05:latest\` construída e listada localmente. | Print no relatório PDF |
| 15 | Site local em execução | Interface HTML servida por Nginx em localhost:8080. | Print no relatório PDF |

## Conclusão

O CP05 apresenta um pipeline que detecta riscos antes da incorporação de alterações na branch principal. As duas sabotagens produziram as reprovações esperadas, o ruleset bloqueou os PRs inseguros e o job Trivy validou a construção e a análise de uma imagem Docker.

> **Segurança dos testes:** o segredo Gitleaks é falso; o código com \`eval()\` existe apenas para análise estática. As branches de teste não devem ser mescladas na main.
