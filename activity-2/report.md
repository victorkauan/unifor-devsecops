# Relatório de Execução — Pipeline DevSecOps com VAmPI

## Resumo Executivo

Este relatório documenta a configuração e execução de um pipeline de segurança automatizado usando Jenkins, Semgrep (SAST) e cyclonedx-py (SCA) sobre a aplicação VAmPI, com os resultados enviados para Defect Dojo e Dependency Track respectivamente. Toda a infraestrutura foi provisionada via Docker Compose.

---

## 1. Infraestrutura — Docker Compose

Todos os serviços foram iniciados com um único comando a partir do diretório `activity-2/`:

```bash
docker-compose up -d --build
```

Os serviços provisionados foram: Jenkins, Defect Dojo (nginx + uwsgi + celery + postgres + redis) e Dependency Track (API server + frontend).

![Docker Compose — containers rodando](./prints/docker-compose-containers.png)

---

## 2. Configuração do Defect Dojo

### 2.1 Acesso inicial

Acesso via browser em `http://localhost:8085` com as credenciais padrão `admin / admin123`.

![Defect Dojo — tela de login](./prints/defectdojo-login.png)

### 2.2 Obtenção da API Key

1. Menu do usuário (canto superior direito) → **API v2**
2. Copiar o token exibido na página

![Defect Dojo — página de API Key](./prints/defectdojo-api-key.png)

---

## 3. Configuração do Dependency Track

### 3.1 Acesso inicial

Acesso via browser em `http://localhost:8082` com as credenciais padrão `admin / admin`. O sistema solicita troca de senha no primeiro acesso.

![Dependency Track — tela de login](./prints/dtrack-login.png)

### 3.2 Obtenção da API Key

1. **Administration** → **Access Management** → **Teams** → **Automation**
2. Copiar o API Key exibido (ou gerar um novo)

![Dependency Track — API Key (1)](./prints/dtrack-api-key-1.png)
![Dependency Track — API Key (2)](./prints/dtrack-api-key-2.png)

### 3.3 Criação do projeto VAmPI

1. Menu **Projects** → **Create Project**
2. Nome: `VAmPI`, tipo: `Application`
3. Após salvar, o UUID do projeto aparece na URL: `http://localhost:8082/projects/{uuid}/`

![Dependency Track — criação do projeto VAmPI](./prints/dtrack-project-create.png)
![Dependency Track — UUID do projeto na URL](./prints/dtrack-project-uuid.png)

---

## 4. Configuração do Jenkins

### 4.1 Acesso inicial e desbloqueio

Acesso via browser em `http://localhost:9090`. Na primeira vez, o Jenkins solicita uma senha de desbloqueio obtida com:

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

![Jenkins — tela de desbloqueio](./prints/jenkins-unlock.png)

### 4.2 Cadastro de credenciais

Em **Manage Jenkins** → **Credentials** → **System** → **Global credentials** → **Add Credentials**, foram cadastradas 3 credenciais do tipo *Secret text*:

| ID | Descrição |
|----|-----------|
| `defectdojo-api-key` | Token de API do Defect Dojo |
| `dtrack-api-key` | API Key do Dependency Track |
| `dtrack-project-uuid` | UUID do projeto VAmPI no Dependency Track |

![Jenkins — credenciais (1)](./prints/jenkins-credentials-1.png)
![Jenkins — credenciais (2)](./prints/jenkins-credentials-2.png)
![Jenkins — credenciais (3)](./prints/jenkins-credentials-3.png)
![Jenkins — credenciais (4)](./prints/jenkins-credentials-4.png)
![Jenkins — credenciais (5)](./prints/jenkins-credentials-5.png)

### 4.3 Criação do pipeline

1. **New Item** → nome: `vampi-pipeline` → tipo: **Pipeline** → OK
2. Em **Pipeline** → **Definition**: `Pipeline script`
3. Colar o conteúdo do `Jenkinsfile`
4. Salvar

![Jenkins — criação do pipeline (1)](./prints/jenkins-pipeline-create-1.png)
![Jenkins — criação do pipeline (2)](./prints/jenkins-pipeline-create-2.png)

---

## 5. Execução do Pipeline

O pipeline foi disparado via **Build Now** no painel do job `vampi-pipeline`.

![Pipeline — visão geral com todos os estágios](./prints/pipeline-overview.png)

### 5.1 Estágio: Checkout VAmPI

O Jenkins clonou o repositório `https://github.com/erev0s/VAmPI.git` (branch `master`).

![Pipeline — log do estágio Checkout VAmPI](./prints/pipeline-checkout.png)

### 5.2 Estágio: SAST — Semgrep

O Semgrep foi executado com `--config=auto` e rulesets adicionais (`p/owasp-top-ten`, `p/python`, `p/flask`, `p/jwt`), identificando findings de severidade High em 24 arquivos.

![Pipeline — log do estágio SAST com resumo do Semgrep](./prints/pipeline-sast-semgrep.png)

### 5.3 Estágio: Upload SAST → Defect Dojo

O relatório JSON foi enviado para o Defect Dojo via API. A resposta confirmou a criação do produto, engagement e importação dos findings.

![Pipeline — log do estágio com resposta do Defect Dojo](./prints/pipeline-upload-defectdojo.png)

### 5.4 Estágio: SCA — cyclonedx-py

O cyclonedx-py gerou o SBOM no formato CycloneDX a partir de um ambiente virtual isolado com as dependências do VAmPI instaladas.

![Pipeline — log do estágio SCA](./prints/pipeline-sca-cyclonedx.png)

### 5.5 Estágio: Upload SCA → Dependency Track

O SBOM foi enviado via API e o pipeline aguardou o processamento com polling:

```
BOM upload token: <uuid>
Processing: true
Processing: false
BOM processing complete.
```

![Pipeline — upload para Dependency Track (1)](./prints/pipeline-upload-dtrack-1.png)
![Pipeline — upload para Dependency Track (2)](./prints/pipeline-upload-dtrack-2.png)

---

## 6. Resultados no Defect Dojo

### 6.1 Findings importados

Após a execução do pipeline, os findings do Semgrep aparecem no produto `VAmPI`, engagement `VAmPI - SAST Pipeline`.

![Defect Dojo — lista de findings (1)](./prints/defectdojo-findings-1.png)
![Defect Dojo — lista de findings (2)](./prints/defectdojo-findings-2.png)
![Defect Dojo — lista de findings (3)](./prints/defectdojo-findings-3.png)

### 6.2 Detalhe de um finding

![Defect Dojo — detalhe de finding](./prints/defectdojo-finding-detail.png)

---

## 7. Resultados no Dependency Track

Visão geral do projeto `VAmPI` no Dependency Track com métricas de componentes e vulnerabilidades.

![Dependency Track — visão geral do projeto VAmPI](./prints/dtrack-project-overview.png)

### 7.1 Componentes identificados

O projeto exibe os 54 componentes detectados pelo cyclonedx-py a partir do ambiente virtual com as dependências do VAmPI instaladas, incluindo dependências transitivas.

![Dependency Track — componentes do projeto VAmPI](./prints/dtrack-components.png)

### 7.2 Vulnerabilidades encontradas

Após o processamento da base NVD e OSS Index, as vulnerabilidades (CVEs) associadas aos componentes são exibidas na aba **Vulnerabilities**.

![Dependency Track — vulnerabilidades (1)](./prints/dtrack-vulnerabilities-1.png)
![Dependency Track — vulnerabilidades (2)](./prints/dtrack-vulnerabilities-2.png)
![Dependency Track — vulnerabilidades (3)](./prints/dtrack-vulnerabilities-3.png)

---

## 8. Conclusão

O pipeline de segurança foi configurado e executado com sucesso, integrando análise estática (SAST) e análise de composição de software (SCA) em um fluxo automatizado. Os resultados foram consolidados nas plataformas Defect Dojo e Dependency Track, permitindo rastreabilidade e gestão das vulnerabilidades encontradas na aplicação VAmPI.
