# Technical Notes - Fluxo Ideal 2025.10.2-pre

**Data de Release:** TBD
**Status:** [X] Pré-release [ ] Release Oficial
**Release Manager:** Claude Code
**DevOps Lead:** TBD

## 📦 Versões dos Componentes

### Microserviços
| Microserviço | Versão Anterior | Versão Atual | Mudanças |
|--------------|-----------------|--------------|----------|
| fluxoideal-agendamento | v0.7.360 | v0.7.360 | No changes |
| fluxoideal-atendimento | v0.7.760 | v0.7.760 | No changes |
| fluxoideal-central | v0.1.004 | v0.1.004 | No changes |
| fluxoideal-clientes | v0.7.477 | v0.7.477 | No changes |
| fluxoideal-documentos | v0.7.809 | v0.7.809 | No changes |
| fluxoideal-estabelecimento | v0.8.201 | v0.8.201 | No changes |
| fluxoideal-interacoes | v0.6.532 | v0.6.532 | No changes |
| fluxoideal-middleware | v0.96.421 | v0.96.421 | No changes |
| fluxoideal-notification-center | v0.6.303 | v0.6.303 | No changes |
| fluxoideal-websocket-server | v0.7.073 | v0.7.073 | No changes |
| site-bia | v0.5.010 | v0.5.010 | No changes |

### Databases
| Database | Versão Anterior | Versão Atual | Migrations | Status |
|----------|-----------------|--------------|------------|--------|
| postgres-agendamentos | 15 | 15 | - | No changes |
| postgres-atendimento | 15 | 15 | - | No changes |
| postgres-clientes | 15 | 15 | - | No changes |
| postgres-documentos | 15 | 15 | - | No changes |
| postgres-estabelecimento | 15 | 15 | - | No changes |
| postgres-interacoes | 15 | 15 | - | No changes |
| postgres-keycloak | 13 | 13 | - | No changes |
| postgres-mensageria | 16 | 16 | - | No changes |
| n8n-postgres | 13 | 13 | - | No changes |

### Cache e Storage
| Componente | Versão Anterior | Versão Atual | Status |
|------------|-----------------|--------------|--------|
| redis-cache | (sem tag; imagem por ID) | (sem tag; imagem por ID) | No changes |
| redis-documentos | 7 | 7 | No changes |
| redis-mensageria | (sem tag; imagem por ID) | (sem tag; imagem por ID) | No changes |
| s3-s3_minio-1 | RELEASE.2025-03-12T18-04-18Z | RELEASE.2025-03-12T18-04-18Z | No changes |

### Ferramentas de Administração e Desenvolvimento
| Ferramenta | Versão | Notas |
|------------|--------|-------|
| adminer | latest | Database management tool |
| portainer | ce:latest | Container management UI |
| redisinsight | latest | Redis GUI |
| n8n-n8n-1 | latest | Workflow automation |
| ghost | 5-alpine | CMS/Blog platform |

### Dependências Externas
| Dependência | Versão | Notas |
|-------------|--------|-------|
| keycloak-keycloak-1 | 24.0.1 | Autenticação e autorização |
| traefik | v2.11 | Load balancer e proxy reverso |

### Padrões de Configuração
- **Portas dos Microserviços:** Todos os microserviços fluxoideal-* utilizam a porta padrão **8000**
- **Comunicação Interna:** URLs seguem padrão `http://[nome-microservico]:8000/`

## 🏗️ Mudanças de Arquitetura
- **Nova Rota de Configuração:** Adicionado endpoint `/configsite` ao middleware para acesso às configurações do site via microserviço fluxoideal-estabelecimento
- **Roteamento do Middleware:** Arquivo `paths_prefixados.yaml` atualizado com nova entrada para rota de configuração

## 🗄️ Comandos de Database

### Migrations
```sql
-- Nenhuma migration nesta release (ainda)
```

### Grants e Permissões
```sql
-- Nenhuma permissão necessária nesta release
```

### Índices e Otimizações
```sql
-- Nenhum índice novo nesta release (ainda)
```

## 🔐 Permissões de Sistema

### Permissões de Arquivos (chmod)
```bash
# Nenhuma permissão específica necessária nesta release
```

### Permissões de Diretórios
```bash
# Estrutura padrão do git
```

## ⚙️ Configurações de Ambiente

### Variáveis de Ambiente Novas
```bash
# N/A
```

### Variáveis de Ambiente Modificadas
```bash
# N/A
```

### Arquivos de Configuração
- **paths_prefixados.yaml:** Arquivo de configuração do middleware que define roteamento de endpoints
  - **Localização:** Servidor (diretório do middleware)
  - **Ação necessária:** Atualizar arquivo e reiniciar middleware
  - **Mudanças:** Adicionada nova rota `/configsite`
  - **Conteúdo do arquivo:**
  ```yaml
  /clientes:
    target_url: http://fluxoideal-clientes:8000/clientes
  /profissionais:
    target_url: http://fluxoideal-estabelecimento:8000/profissionais
  /convenio:
    target_url: http://fluxoideal-estabelecimento:8000/convenio
  /board:
    target_url: http://fluxoideal-estabelecimento:8000/board
  /historico_conversa:
    target_url: http://fluxoideal-interacoes:8000/historico_conversa
  /interacoes:
    target_url: http://fluxoideal-interacoes:8000/interacoes
  /agendamento:
    target_url: http://fluxoideal-agendamento:8000/agendamento
  /bloqueio:
    target_url: http://fluxoideal-agendamento:8000/bloqueio
  /slots:
    target_url: http://fluxoideal-agendamento:8000/slots
  /atendimento:
    target_url: http://fluxoideal-atendimento:8000/atendimento
  /documentos:
    target_url: http://fluxoideal-documentos:8000/documentos
  /estabelecimento:
    target_url: http://fluxoideal-estabelecimento:8000/estabelecimento
  /cobranca:
    target_url: http://fluxoideal-atendimento:8000/cobranca
  /servicos:
    target_url: http://fluxoideal-atendimento:8000/servicos
  /pagamento:
    target_url: http://fluxoideal-atendimento:8000/pagamento
  /exames:
    target_url: http://fluxoideal-estabelecimento:8000/exames
  /templates:
    target_url: http://fluxoideal-estabelecimento:8000/templates
  /tipos:
    target_url: http://fluxoideal-estabelecimento:8000/tipos
  /chat:
    target_url: http://fluxoideal-websocket-server:8000/chat
  /notify:
    target_url: http://fluxoideal-notification-center:8000/notify
  /prontuario:
    target_url: http://fluxoideal-atendimento:8000/atendimento
  /autocomplete:
    target_url: http://fluxoideal-estabelecimento:8000/autocomplete
  /estrutura-atendimento:
    target_url: http://fluxoideal-estabelecimento:8000//estrutura-atendimento
  /atendimento-dashboard:
    target_url: http://fluxoideal-atendimento:8000/atendimento-dashboard
  /agendamento-dashboard:
    target_url: http://fluxoideal-agendamento:8000/agendamento-dashboard
  /configsite:
    target_url: http://fluxoideal-estabelecimento:8000/configsite
  ```

## 🚀 Procedimentos de Deploy

### Ordem de Deploy
1. [ ] Atualizar arquivo paths_prefixados.yaml no servidor
2. [ ] Reiniciar middleware
3. [ ] Verificar roteamento da nova rota /configsite

### Comandos de Deploy
```bash
# Passo 1: Atualizar arquivo paths_prefixados.yaml no servidor
# Substituir o conteúdo do arquivo paths_prefixados.yaml com as configurações documentadas na seção "Arquivos de Configuração"
# Localização: [diretório_do_middleware]/paths_prefixados.yaml

# Passo 2: Reiniciar middleware
docker-compose restart fluxoideal-middleware

# Passo 3: Verificar roteamento
curl -f http://[middleware-url]/configsite/health
```

### Health Checks
```bash
# Verificar saúde dos microserviços
curl -f http://fluxoideal-atendimento:8000/health
curl -f http://fluxoideal-estabelecimento:8000/health
```

## 🔄 Procedimentos de Rollback

### Pré-requisitos para Rollback
- Rollback para versão 2025.10.1 se necessário

### Passos de Rollback
1. **TBD** - Aguardando definição de mudanças

## 📊 Monitoramento Pós-Deploy

### Métricas Críticas
- [ ] Nenhuma métrica adicional nesta release

### Logs para Acompanhar
- Logs padrão dos microserviços

### Alertas Configurados
- Alertas padrão mantidos

## ⚠️ Riscos e Mitigações

| Risco | Probabilidade | Impacto | Mitigação |
|-------|---------------|---------|-----------|
| Mudança de escopo durante pré-release | Alta | Baixo | Normal para fase de pré-release |

## 📝 Notas de Segurança
- Nenhuma mudança de segurança nesta release (ainda)

## 🔗 Referências
- [CLAUDE.md](../../CLAUDE.md) - Documentação para Claude Code
- [README.md](../../README.md) - Documentação principal do projeto
- [Templates](../../releases/templates/) - Templates de documentação

---
*Este documento deve ser mantido atualizado durante todo o ciclo de vida da release.*
