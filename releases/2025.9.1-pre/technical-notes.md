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

-- Criação de nova tabela: estrutura_atendimento
CREATE TABLE estrutura_atendimento (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    estrutura JSONB NOT NULL,
    estabelecimento_id UUID NOT NULL,
    CONSTRAINT fk_estrutura_atendimento_estabelecimento
        FOREIGN KEY (estabelecimento_id)
        REFERENCES estabelecimento(id)
);

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

-- Criar a tabela prontuario_adendo
CREATE TABLE prontuario_adendo (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    atendimento_id UUID REFERENCES atendimentos(id),
    cliente_id UUID NOT NULL,
    usuario TEXT NOT NULL,
    dados JSONB NOT NULL
);

-- Criar tabela autocomplete
CREATE TABLE autocomplete (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW() NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW() NOT NULL,
    autocompletefield_id UUID NOT NULL,
    profissional_id UUID NOT NULL,
    text TEXT NOT NULL,
    category TEXT,
    frequency INTEGER DEFAULT 0 NOT NULL,
    is_active BOOLEAN DEFAULT true NOT NULL,

    -- Foreign key para profissionais
    CONSTRAINT fk_autocomplete_profissional
        FOREIGN KEY (profissional_id)
        REFERENCES profissionais(id)
        ON DELETE CASCADE
);

-- Atualizar campo fim dos atendimentos finalizados
UPDATE atendimentos SET fim = updated_at WHERE status = 'Finalizado';

-- Database: postgres-documentos
-- Migration: Ajuste na estrutura_tela da tabela documentos_dados
-- Descrição: Ajustar o campo estrutura_tela para identificar as colunas dos grupos

-- ⚠️ IMPORTANTE: Este é um ajuste manual no campo JSONB estrutura_tela
-- A estrutura_tela deve ser atualizada para incluir informação de colunas nos grupos
-- Cada grupo no JSONB deve conter a propriedade 'columns' identificando o layout das colunas
-- Exemplo de estrutura esperada:
-- {
--   "grupos": [
--     {
--       "id": "grupo_id",
--       "nome": "Nome do Grupo",
--       "columns": 2,  // <-- Propriedade que identifica número de colunas
--       "campos": [...]
--     }
--   ]
-- }

-- Não há comando SQL automático para esta migration
-- Requer ajuste manual ou script de migração de dados específico

-- Database: postgres-documentos
-- Migration: Alteração de coluna documentos_gerados para arquivos_gerados
-- Descrição: Remover coluna documentos_gerados e adicionar coluna arquivos_gerados do tipo JSONB

-- SQL de Upgrade (aplicar alterações):

-- Remover coluna documentos_gerados
ALTER TABLE documentos_dados DROP COLUMN documentos_gerados;

-- Adicionar coluna arquivos_gerados do tipo JSONB
ALTER TABLE documentos_dados ADD COLUMN arquivos_gerados JSONB;
```

### Grants e Permissões
```sql
-- Nenhuma permissão necessária nesta release
```

### Índices e Otimizações
```sql
-- Database: postgres-atendimento
-- Índices para tabela prontuario_adendo
CREATE INDEX idx_prontuario_adendo_atendimento_id ON prontuario_adendo(atendimento_id);
CREATE INDEX idx_prontuario_adendo_cliente_id ON prontuario_adendo(cliente_id);
CREATE INDEX idx_prontuario_adendo_created_at ON prontuario_adendo(created_at);
CREATE INDEX idx_prontuario_adendo_updated_at ON prontuario_adendo(updated_at);
CREATE INDEX idx_prontuario_adendo_usuario ON prontuario_adendo(usuario);

-- Índices para tabela autocomplete
-- Índice composto principal para a query mais comum (filtro + ordenação)
CREATE INDEX idx_autocomplete_main_query
    ON autocomplete(profissional_id, autocompletefield_id, is_active, frequency DESC);

-- Índice para busca de texto (ilike/pattern matching)
CREATE INDEX idx_autocomplete_text_trgm
    ON autocomplete USING gin(text gin_trgm_ops);

-- Índice simples para autocompletefield_id (queries de filtragem)
CREATE INDEX idx_autocomplete_field
    ON autocomplete(autocompletefield_id);

-- Índice para profissional_id (foreign key lookup)
CREATE INDEX idx_autocomplete_profissional
    ON autocomplete(profissional_id);

-- Índice para ordenação por frequency quando filtrado por is_active
CREATE INDEX idx_autocomplete_active_frequency
    ON autocomplete(is_active, frequency DESC)
    WHERE is_active = true;

-- Habilitar extensão pg_trgm para busca de texto eficiente (se não estiver habilitada)
CREATE EXTENSION IF NOT EXISTS pg_trgm;

-- Função e trigger para updated_at automático
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$ language 'plpgsql';

CREATE TRIGGER update_prontuario_adendo_updated_at
    BEFORE UPDATE ON prontuario_adendo
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();

-- Trigger para atualizar updated_at automaticamente na tabela autocomplete
CREATE OR REPLACE FUNCTION update_autocomplete_updated_at()
RETURNS TRIGGER AS $
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_autocomplete_updated_at
    BEFORE UPDATE ON autocomplete
    FOR EACH ROW
    EXECUTE FUNCTION update_autocomplete_updated_at();

-- Comentários para documentação
COMMENT ON TABLE autocomplete IS 'Armazena sugestões de autocomplete por profissional e campo';
COMMENT ON COLUMN autocomplete.autocompletefield_id IS 'ID do campo de formulário para categorização';
COMMENT ON COLUMN autocomplete.frequency IS 'Contador de uso para ordenação por popularidade';
COMMENT ON COLUMN autocomplete.is_active IS 'Flag para habilitar/desabilitar sugestão';
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

- **paths_prefixados.yaml:** Arquivo de configuração do middleware que define roteamento de endpoints
  - **Localização:** Servidor (diretório do middleware)
  - **Ação necessária:** Atualizar arquivo e reiniciar middleware
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
  ```

## 🚀 Procedimentos de Deploy

### Ordem de Deploy
1. [X] Database migrations (postgres-estabelecimento)
2. [ ] Database migrations (postgres-atendimento - tabela auditoria e autocomplete)
3. [ ] Ajuste manual na estrutura_tela da tabela documentos_dados (postgres-documentos)
4. [ ] Renomear containers (integracos → interacoes)
5. [ ] Criar rede Docker "estabelecimento-net" se não existir
6. [ ] Atualizar variáveis de ambiente do fluxoideal-atendimento
7. [ ] Reconectar container fluxoideal-atendimento à rede estabelecimento-net
8. [ ] Atualizar arquivo paths_prefixados.yaml no servidor
9. [ ] Reiniciar middleware
10. [ ] Restart dos microserviços afetados
11. [ ] Verificar conectividade entre microserviços

### Comandos de Deploy
```bash
# Passo 1: Executar migration no postgres-estabelecimento
# (Comandos SQL já documentados na seção Migrations)

# Passo 2: Executar migration no postgres-atendimento
# (Comandos SQL já documentados na seção Migrations)

# Passo 3: Ajustar estrutura_tela da tabela documentos_dados (postgres-documentos)
# ⚠️ IMPORTANTE: Ajuste manual necessário no campo JSONB estrutura_tela
# Adicionar propriedade 'columns' em cada grupo para identificar número de colunas
# Este passo requer script customizado ou ajuste manual conforme documentado nas Migrations

# Passo 4: Renomear containers
docker rename fluxoideal-integracos fluxoideal-interacoes || echo "Container já renomeado"
docker rename postgres-integracos postgres-interacoes || echo "Container já renomeado"

# Passo 5: Criar rede Docker se necessário
docker network create estabelecimento-net --driver bridge || echo "Rede já existe"

# Passo 6: Atualizar container fluxoideal-atendimento
# Adicionar variável de ambiente ESTABELECIMENTO_URL=http://fluxoideal-estabelecimento:8000/
# Conectar à rede estabelecimento-net

# Passo 8: Atualizar arquivo paths_prefixados.yaml no servidor
# Substituir o conteúdo do arquivo paths_prefixados.yaml com as configurações documentadas na seção "Arquivos de Configuração"
# Localização: [diretório_do_middleware]/paths_prefixados.yaml

# Passo 9: Reiniciar middleware
docker-compose restart fluxoideal-middleware

# Passo 10: Restart dos serviços
docker-compose restart fluxoideal-atendimento
docker-compose restart fluxoideal-interacoes

# Passo 11: Verificar integração
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