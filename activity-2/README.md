[← Voltar ao início](../README.md)

# Atividade 2 — Pipeline CI/CD com Scans de Segurança

## Descrição

Criação de um pipeline de CI/CD no Jenkins que executa scans de segurança automatizados sobre o projeto **VAmPI** (Vulnerable REST API), uma aplicação Python/Flask intencionalmente vulnerável usada para prática de testes de segurança.

**Ferramenta de CI/CD:** Jenkins
**Aplicação-alvo:** [VAmPI](https://github.com/erev0s/VAmPI)

---

## Scans Executados

| Tipo | Ferramenta | Destino dos Resultados |
|------|-----------|------------------------|
| SAST | Semgrep | Defect Dojo |
| SCA | cdxgen (CycloneDX) | Dependency Track |

---

## Arquitetura

Todos os serviços são executados via Docker Compose em uma rede isolada (`devsecops`):

| Serviço | Descrição | Porta |
|---------|-----------|-------|
| Jenkins | Servidor de CI/CD | 9090 |
| Defect Dojo | Plataforma de gestão de vulnerabilidades (SAST) | 8085 |
| Dependency Track | Análise de composição de software (SCA) | 8081 (API), 8082 (UI) |

---

## Estágios do Pipeline

```
Checkout VAmPI → SAST (Semgrep) → Upload → DefectDojo → SCA (cdxgen) → Upload → Dependency Track
```

1. **Checkout VAmPI** — clona o repositório da aplicação-alvo diretamente do GitHub
2. **SAST — Semgrep** — analisa o código em busca de vulnerabilidades estáticas (`--config=auto`)
3. **Upload SAST → Defect Dojo** — envia o relatório JSON do Semgrep via API REST
4. **SCA — cdxgen** — gera o SBOM (Software Bill of Materials) no formato CycloneDX
5. **Upload SCA → Dependency Track** — envia o SBOM via API e aguarda o processamento

---

## Resultados

### SAST — Defect Dojo

- Findings de severidade High encontrados pelo Semgrep no código do VAmPI
- Importados automaticamente no produto `VAmPI`, engagement `VAmPI - SAST Pipeline`

### SCA — Dependency Track

- **54 componentes** identificados no SBOM, incluindo dependências transitivas (Flask, SQLAlchemy, PyJWT, connexion, etc.)
- Vulnerabilidades (CVEs) encontradas via análise NVD + OSS Index

---

## Arquivos

| Arquivo | Descrição |
|---------|-----------|
| `docker-compose.yml` | Definição de todos os serviços |
| `Jenkinsfile` | Pipeline declarativo com os 5 estágios |
| `jenkins/Dockerfile` | Imagem customizada do Jenkins com Semgrep e cdxgen |
| `.env.example` | Variáveis de ambiente necessárias (sem credenciais) |
| [`report.md`](./report.md) | Relatório de execução com evidências |
