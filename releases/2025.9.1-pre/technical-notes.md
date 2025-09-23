# Technical Notes - Fluxo Ideal 2025.9.1-pre

**Data de Release:** 15/09/2025
**Status:** [X] Pré-release [ ] Release Oficial
**Release Manager:** Claude Code
**DevOps Lead:** TBD

## 📦 Versões dos Componentes

### Microserviços
| Microserviço | Versão Anterior | Versão Atual | Mudanças |
|--------------|-----------------|--------------|----------|
| fluxoideal-agendamento | v0.7.344 | v0.7.314 | Minor (em teste) |
| fluxoideal-atendimento | v0.7.673 | v0.7.671 | Patch (em teste) |
| fluxoideal-central | v0.5.051 | v0.5.051 | No changes |
| fluxoideal-clientes | v0.7.440 | v0.7.440 | No changes |
| fluxoideal-documentos | v0.7.788 | v0.7.770 | Minor (em teste) |
| fluxoideal-estabelecimento | v0.8.165 | v0.8.166 | Patch (em teste) |
| fluxoideal-interacoes | v0.6.520 | v0.6.520 | Container renamed (integracos→interacoes) |
| fluxoideal-middleware | v0.6.421 | v0.96.410 | Major (em teste) |
| fluxoideal-notification-center | v0.6.303 | v0.6.303 | No changes |
| fluxoideal-websocket-server | v0.7.073 | v0.7.073 | No changes |

### Databases
| Database | Versão | Migrations | Status |
|----------|--------|------------|--------|
| postgres-agendamentos | 11.10-bca9 | - | No changes |
| postgres-atendimento | 15 | migration_create_auditoria_table | Updated (em teste) |
| postgres-clientes | 15 | - | No changes |
| postgres-documentos | 15 | - | No changes |
| postgres-estabelecimento | 11.10-bca9 | 15 + migration_add_padrao_columns | Updated (em teste) |
| postgres-interacoes | 11.10-bca9 | - | Container renamed (integracos→interacoes) |
| postgres-keycloak | 15 | 13 | Version change (em teste) |
| postgres-mensageria | 15 | 16 | Version upgrade (em teste) |

### Cache e Storage
| Componente | Versão Anterior | Versão Atual | Status |
|------------|-----------------|--------------|--------|
| redis-cache | 6.2-alpine | - | No changes (em teste) |
| redis-documentos | 7 | 7 | No changes |
| redis-mensageria | 7 | 7 | No changes |
| s3-s3-23-minio-1 | RELEASE.2025-03-12T18-04-18Z | RELEASE.2025-03-12T18-04-18Z | No changes |

### Dependências Externas
| Dependência | Versão | Notas |
|-------------|--------|-------|
| keycloak-keycloak-1 | 24.0.1 | Autenticação e autorização |
| traefik | v2.11 | Load balancer e proxy reverso |

### Padrões de Configuração
- **Portas dos Microserviços:** Todos os microserviços fluxoideal-* utilizam a porta padrão **8000**
- **Comunicação Interna:** URLs seguem padrão `http://[nome-microservico]:8000/`

## 🏗️ Mudanças de Arquitetura
- Estabelecida estrutura base para projeto guarda-chuva
- Preparação para arquitetura de microserviços
- Definição de padrões de documentação de releases
- **Comunicação entre Microserviços:** Adicionada integração entre fluxoideal-atendimento e fluxoideal-estabelecimento
- **Rede de Containers:** Container fluxoideal-atendimento agora conectado à rede "estabelecimento-net" para comunicação interna
- **Padronização de Nomenclatura:** Renomeação de containers para correção ortográfica:
  - `fluxoideal-integracos` → `fluxoideal-interacoes`
  - `postgres-integracos` → `postgres-interacoes`
- **Sistema de Auditoria:** Implementação de tabela de auditoria para rastreamento de alterações em atendimentos com campos JSONB para dados antes/depois das mudanças

## 🗄️ Comandos de Database

### Migrations
```sql
-- Database: postgres-estabelecimento
-- Migration: migration_add_padrao_columns
-- Descrição: Adiciona colunas 'padrao' às tabelas tipo_servico e tipo_agendamento

ALTER TABLE tipo_servico
ADD COLUMN padrao boolean NOT NULL DEFAULT false;

ALTER TABLE tipo_agendamento
ADD COLUMN padrao boolean NOT NULL DEFAULT false;

ALTER TABLE convenio
ADD COLUMN cor_rgb TEXT NULL;

ALTER TABLE convenio
ADD COLUMN cor_css TEXT NULL;

-- Criação de nova tabela: estabelecimento
CREATE TABLE estabelecimento (
    id UUID NOT NULL PRIMARY KEY DEFAULT gen_random_uuid(),
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    nome VARCHAR NOT NULL,
    slogan VARCHAR,
    logo_grande BYTEA,
    logo_pequeno BYTEA
);

-- ⚠️ IMPORTANTE: Após criar a tabela, É OBRIGATÓRIO inserir ao menos 1 registro
-- Exemplo:
-- INSERT INTO estabelecimento (nome) VALUES ('Nome do Estabelecimento');

-- Database: postgres-atendimento
-- Migration: migration_create_auditoria_table
-- Descrição: Criação de tabela para auditoria de atendimentos

CREATE TABLE atendimento_auditoria (
    id UUID NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT now() NOT NULL,
    usuario TEXT NOT NULL,
    nome TEXT NOT NULL,
    atendimento_id UUID NOT NULL,
    tipo_acao TEXT NOT NULL,
    tabelas TEXT NOT NULL,
    dados_de JSONB NOT NULL,
    dados_para JSONB,
    PRIMARY KEY (id),
    FOREIGN KEY(atendimento_id) REFERENCES atendimentos (id)
);
```

### Grants e Permissões
```sql
-- Nenhuma permissão necessária nesta release
```

### Índices e Otimizações
```sql
-- Nenhum índice criado nesta release
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
# Microserviço: fluxoideal-atendimento
# Descrição: URL do microserviço de estabelecimento para comunicação interna
ESTABELECIMENTO_URL=http://fluxoideal-estabelecimento:8000/
```

### Variáveis de Ambiente Modificadas
```bash
# N/A
```

### Arquivos de Configuração
- **Docker Compose:** Atualização necessária para fluxoideal-atendimento (adição de variável de ambiente e rede)
- **docker-compose.yml:** Seção do fluxoideal-atendimento deve incluir:
  ```yaml
  environment:
    - ESTABELECIMENTO_URL=http://fluxoideal-estabelecimento:8000/
  networks:
    - estabelecimento-net
  ```

## 🚀 Procedimentos de Deploy

### Ordem de Deploy
1. [X] Database migrations (postgres-estabelecimento)
2. [ ] Database migrations (postgres-atendimento - tabela auditoria)
3. [ ] Renomear containers (integracos → interacoes)
4. [ ] Criar rede Docker "estabelecimento-net" se não existir
5. [ ] Atualizar variáveis de ambiente do fluxoideal-atendimento
6. [ ] Reconectar container fluxoideal-atendimento à rede estabelecimento-net
7. [ ] Restart dos microserviços afetados
8. [ ] Verificar conectividade entre microserviços

### Comandos de Deploy
```bash
# Passo 1: Executar migration no postgres-estabelecimento
# (Comandos SQL já documentados na seção Migrations)

# Passo 2: Executar migration no postgres-atendimento
# (Comandos SQL já documentados na seção Migrations)

# Passo 3: Renomear containers
docker rename fluxoideal-integracos fluxoideal-interacoes || echo "Container já renomeado"
docker rename postgres-integracos postgres-interacoes || echo "Container já renomeado"

# Passo 4: Criar rede Docker se necessário
docker network create estabelecimento-net --driver bridge || echo "Rede já existe"

# Passo 5: Atualizar container fluxoideal-atendimento
# Adicionar variável de ambiente ESTABELECIMENTO_URL=http://fluxoideal-estabelecimento:8000/
# Conectar à rede estabelecimento-net

# Passo 6: Restart dos serviços
docker-compose restart fluxoideal-atendimento
docker-compose restart fluxoideal-interacoes

# Passo 7: Verificar integração
curl -f http://fluxoideal-atendimento:8000/health
```

### Health Checks
```bash
# Verificar saúde do microserviço fluxoideal-atendimento
curl -f http://fluxoideal-atendimento:8000/health

# Verificar conectividade com fluxoideal-estabelecimento
curl -f http://fluxoideal-estabelecimento:8000/health

# Verificar conectividade entre microserviços na rede estabelecimento-net
docker exec fluxoideal-atendimento ping -c 3 fluxoideal-estabelecimento

# Verificar database postgres-estabelecimento
pg_isready -h postgres-estabelecimento -U [usuario]
```

## 🔄 Procedimentos de Rollback

### Pré-requisitos para Rollback
- N/A (Release inicial de documentação)

### Passos de Rollback
1. **N/A** - Release apenas de documentação

## 📊 Monitoramento Pós-Deploy

### Métricas Críticas
- [ ] Nenhuma métrica estabelecida ainda

### Logs para Acompanhar
- Nenhum log gerado ainda

### Alertas Configurados
- Nenhum alerta configurado ainda

## ⚠️ Riscos e Mitigações

| Risco | Probabilidade | Impacto | Mitigação |
|-------|---------------|---------|-----------|
| Mudança de escopo durante pré-release | Alta | Baixo | Normal para fase de pré-release |

## 📝 Notas de Segurança
- Repositório público no GitHub
- Sem dados sensíveis no código

## 🔗 Referências
- [CLAUDE.md](../../CLAUDE.md) - Documentação para Claude Code
- [README.md](../../README.md) - Documentação principal do projeto
- [Templates](../../releases/templates/) - Templates de documentação

---
*Este documento deve ser mantido atualizado durante todo o ciclo de vida da release.*