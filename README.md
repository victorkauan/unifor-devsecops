# Trabalho Final — Disciplina DevSecOps

Repositório do trabalho final da disciplina **DevSecOps**, parte do curso de pós-graduação **Engenharia de Software com Foco em DevOps** da **Universidade de Fortaleza (UNIFOR)**.

**Professor:** Cristiano Santos — Application Security Specialist, OWASP Chapter Leader Fortaleza

---

## Sobre a Disciplina

A disciplina aborda a integração de práticas de segurança ao ciclo de vida de desenvolvimento de software, cobrindo:

- Fundamentos de DevSecOps e Shift Left
- Modelagem de Ameaças (Threat Modeling / STRIDE)
- Análise Estática de Código (SAST) com Semgrep
- Cadeia de Suprimentos de Software (SCA) com CycloneDX / cdxgen
- Análise Dinâmica (DAST)
- Gestão de Vulnerabilidades com Defect Dojo e Dependency Track

---

## Atividades

### Atividade 1 — Modelagem de Ameaças

Modelagem de ameaças do fluxo de login de uma aplicação convencional em nuvem utilizando a metodologia **STRIDE** e a ferramenta **OWASP Threat Dragon**.

→ [`activity-1/`](./activity-1/)

---

### Atividade 2 — Pipeline CI/CD com Scans de Segurança

Pipeline de CI/CD no Jenkins com scans automatizados de SAST (Semgrep → Defect Dojo) e SCA (cdxgen → Dependency Track) sobre a aplicação vulnerável **VAmPI**, com toda a infraestrutura provisionada via Docker Compose.

→ [`activity-2/`](./activity-2/)