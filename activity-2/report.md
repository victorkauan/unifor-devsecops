# Relatório de Execução — Pipeline DevSecOps com VAmPI

## Resumo Executivo

Este relatório documenta a configuração e execução de um pipeline de segurança automatizado usando Jenkins, Semgrep (SAST) e cdxgen (SCA) sobre a aplicação VAmPI, com os resultados enviados para Defect Dojo e Dependency Track respectivamente. Toda a infraestrutura foi provisionada via Docker Compose.

---

## 1. Infraestrutura — Docker Compose

Todos os serviços foram iniciados com um único comando a partir do diretório `activity-2/`:

```bash
docker-compose up -d --build
```

Os serviços provisionados foram: Jenkins, Defect Dojo (nginx + uwsgi + celery + postgres + redis) e Dependency Track (API server + frontend).

---

## 2. Configuração do Defect Dojo

### 2.1 Acesso inicial

Acesso via browser em `http://localhost:8085` com as credenciais padrão `admin / admin123`.

### 2.2 Obtenção da API Key

1. Menu do usuário (canto superior direito) → **API v2**
2. Copiar o token exibido na página

---

## 3. Configuração do Dependency Track

### 3.1 Acesso inicial

Acesso via browser em `http://localhost:8082` com as credenciais padrão `admin / admin`. O sistema solicita troca de senha no primeiro acesso.

### 3.2 Obtenção da API Key

1. **Administration** → **Access Management** → **Teams** → **Automation**
2. Copiar o API Key exibido (ou gerar um novo)

### 3.3 Criação do projeto VAmPI

1. Menu **Projects** → **Create Project**
2. Nome: `VAmPI`, tipo: `Application`
3. Após salvar, o UUID do projeto aparece na URL: `http://localhost:8082/projects/{uuid}/`

---

## 4. Configuração do Jenkins

### 4.1 Acesso inicial e desbloqueio

Acesso via browser em `http://localhost:9090`. Na primeira vez, o Jenkins solicita uma senha de desbloqueio obtida com:

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

### 4.2 Cadastro de credenciais

Em **Manage Jenkins** → **Credentials** → **System** → **Global credentials** → **Add Credentials**, foram cadastradas 3 credenciais do tipo *Secret text*:

| ID | Descrição |
|----|-----------|
| `defectdojo-api-key` | Token de API do Defect Dojo |
| `dtrack-api-key` | API Key do Dependency Track |
| `dtrack-project-uuid` | UUID do projeto VAmPI no Dependency Track |

### 4.3 Criação do pipeline

1. **New Item** → nome: `vampi-pipeline` → tipo: **Pipeline** → OK
2. Em **Pipeline** → **Definition**: `Pipeline script`
3. Colar o conteúdo do `Jenkinsfile`
4. Salvar

---

## 5. Execução do Pipeline

O pipeline foi disparado via **Build Now** no painel do job `vampi-pipeline`.

### 5.1 Estágio: Checkout VAmPI

O Jenkins clonou o repositório `https://github.com/erev0s/VAmPI.git` (branch `master`).

### 5.2 Estágio: SAST — Semgrep

O Semgrep foi executado com `--config=auto` e identificou **4 findings** de severidade High em 24 arquivos.

```
Findings: 4 (4 blocking)
Rules run: 328
Targets scanned: 24
```

### 5.3 Estágio: Upload SAST → Defect Dojo

O relatório JSON foi enviado para o Defect Dojo via API. A resposta confirmou a criação do produto, engagement e importação dos findings.

### 5.4 Estágio: SCA — cdxgen

O cdxgen gerou o SBOM no formato CycloneDX com **12 componentes** identificados (PyPI e GitHub Actions).

### 5.5 Estágio: Upload SCA → Dependency Track

O SBOM foi enviado via API e o pipeline aguardou o processamento com polling:

```
BOM upload token: <uuid>
Processing: true
Processing: false
BOM processing complete.
```

---

## 6. Resultados no Defect Dojo

### 6.1 Findings importados

Após a execução do pipeline, os 4 findings do Semgrep aparecem no produto `VAmPI`, engagement `VAmPI - SAST Pipeline`.

### 6.2 Detalhe de um finding

---

## 7. Resultados no Dependency Track

### 7.1 Componentes identificados

O projeto `VAmPI` no Dependency Track exibe os 12 componentes detectados pelo cdxgen a partir do `requirements.txt` e dos workflows do GitHub.

### 7.2 Vulnerabilidades encontradas

Após o download e processamento da base NVD, as vulnerabilidades (CVEs) associadas aos componentes são exibidas na aba **Vulnerabilities**.

---

## 8. Conclusão

O pipeline de segurança foi configurado e executado com sucesso, integrando análise estática (SAST) e análise de composição de software (SCA) em um fluxo automatizado. Os resultados foram consolidados nas plataformas Defect Dojo e Dependency Track, permitindo rastreabilidade e gestão das vulnerabilidades encontradas na aplicação VAmPI.
