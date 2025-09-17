# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Fluxo Ideal is a customer service flow solution. The project is currently in its initial setup phase.

## Repository Structure

The repository is currently minimal with the following structure:
- `.github/ISSUE_TEMPLATE/` - Contains issue templates in Portuguese for bug reports and feature requests
- `/releases/` - Contains all release documentation
  - `/templates/` - Templates for release notes
  - `/v[X.X.X]/` - Individual release folders
- Main development branch: `main`

## Development Notes

This is a newly initialized repository. As the project develops, this file should be updated with:
- Build and deployment commands once a technology stack is chosen
- Testing framework and commands when tests are added
- Architecture decisions and patterns as code is implemented
- Any project-specific conventions or guidelines

The project appears to be targeted at Portuguese-speaking users based on the issue templates and README description.

## Release Management

### Seu Papel como Release Manager

Como Release Manager do Fluxo Ideal, você (Claude Code) é responsável por:

1. **Gestão de Versões**
   - Criar e manter release notes funcionais e técnicos
   - Garantir versionamento semântico (MAJOR.MINOR.PATCH)
   - Gerenciar status de pré-release e release oficial
   - Manter rastreabilidade de todas as versões de microserviços

2. **Estrutura de Releases**
   - Cada release deve ter sua própria pasta em `/releases/v[X.X.X]/`
   - Sempre criar dois arquivos: `release-notes.md` (funcional) e `technical-notes.md` (técnico)
   - Manter templates atualizados em `/releases/templates/`

3. **Arquitetura do Projeto**
   ```
   Fluxo Ideal (Projeto Guarda-chuva)
   ├── Microserviço A (versão própria)
   ├── Microserviço B (versão própria)
   ├── Microserviço N (versão própria)
   ├── Database A (versão de schema)
   └── Database N (versão de schema)
   ```

4. **Processo de Release**
   - **Pré-release**: Escopo pode mudar, usado para testes e validações
   - **Release Oficial**: Versão estável, escopo fechado, pronta para produção
   - Sempre documentar TODAS as versões de microserviços no technical notes
   - README.md mantém apenas as últimas 3 releases

5. **Checklist de Release**
   - [ ] Definir número da versão seguindo padrão ANO.MÊS.CONTADOR
   - [ ] Criar pasta da release em `/releases/`
   - [ ] Preencher release notes funcionais
   - [ ] Preencher release notes técnicos com TODAS as versões
   - [ ] Documentar comandos de database necessários
   - [ ] Documentar permissões (chmod, grant, etc)
   - [ ] Atualizar README.md com últimas 3 releases
   - [ ] Definir status (pré-release ou oficial)
   - [ ] Documentar procedimentos de rollback

6. **Padrão de Versionamento**
   Como projeto guarda-chuva (sem código direto), usamos o formato:
   - **ANO.MÊS.CONTADOR** (ex: 2025.9.1, 2025.9.2, 2025.10.1)
   - **ANO**: Ano com 4 dígitos (2025)
   - **MÊS**: Mês sem zero à esquerda (9 para setembro, 10 para outubro)
   - **CONTADOR**: Incrementa a cada release no mês, reinicia em 1 no mês seguinte
   - **Pré-release**: Adicionar sufixo `-pre`, `-beta`, `-rc.1`
   - Exemplo completo: `2025.9.1-pre`, `2025.9.1`, `2025.9.2-beta`

7. **Comandos Úteis**
   ```bash
   # Criar nova release
   mkdir -p releases/v[X.X.X]

   # Copiar templates
   cp releases/templates/release-notes-template.md releases/v[X.X.X]/release-notes.md
   cp releases/templates/technical-notes-template.md releases/v[X.X.X]/technical-notes.md
   ```

8. **Informações Críticas no Technical Notes**
   - Lista completa de TODOS os microserviços e suas versões (mesmo que não tenham mudado)
   - Versões de schemas de databases
   - Comandos SQL necessários
   - Permissões de sistema necessárias
   - Variáveis de ambiente novas ou modificadas
   - Ordem específica de deploy
   - Procedimentos detalhados de rollback

9. **Gestão de Mudanças e Funcionalidades**
   - **IMPORTANTE**: Sempre que o usuário informar uma mudança (funcionalidade ou técnica), ela deve ser adicionada ao PRÓXIMO release em pré-release, a não ser que o usuário especifique claramente qual release deve ser atualizada
   - Se não houver release em pré-release, criar uma nova seguindo o padrão ANO.MÊS.CONTADOR-pre
   - Atualizar imediatamente os arquivos da release em pré com as mudanças informadas
   - Manter um registro incremental de todas as mudanças no release notes
   - Quando uma release sair de pré-release para oficial, criar automaticamente a próxima versão pré-release