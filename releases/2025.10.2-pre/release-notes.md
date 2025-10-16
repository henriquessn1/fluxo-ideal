# Release Notes - Fluxo Ideal 2025.10.2-pre

**Data de Release:** TBD
**Status:** [X] Pré-release [ ] Release Oficial
**Release Manager:** Claude Code

## Resumo Executivo
Release com melhorias de configuração e roteamento de endpoints do middleware.

## 🎯 Novas Funcionalidades
- **Endpoint de Configuração do Site:** Nova rota `/configsite` disponível no middleware para gerenciamento de configurações do site através do microserviço de estabelecimento

## 🔧 Melhorias
- **Roteamento do Middleware:** Configuração atualizada do arquivo `paths_prefixados.yaml` com a nova rota para configuração de site

## 🐛 Correções
- TBD - Aguardando definição

## 📊 Impactos para Usuários

### Mudanças na Interface
- Nenhuma mudança definida ainda

### Mudanças no Comportamento
- **Nova Rota de API:** Disponível endpoint `/configsite` para acesso às configurações do site

### Ações Necessárias
- [ ] Após o deploy, atualizar aplicações que consomem a API para utilizar a nova rota `/configsite` quando necessário
- [ ] Verificar permissões de acesso à nova rota `/configsite`

## 🔄 Migrações e Compatibilidade
- **Compatibilidade com versões anteriores:** Sim
- **Necessária migração de dados:** Não
- **Tempo estimado de indisponibilidade:** ~30 segundos (restart do middleware)

## 📝 Notas Adicionais
Esta release adiciona uma nova rota de configuração sem impactar funcionalidades existentes. O middleware precisa ser reiniciado para aplicar a nova configuração de roteamento.

## 🔗 Links Úteis
- [Documentação Técnica](technical-notes.md)
- [Templates de Release](../../templates/)
- [Release Anterior - 2025.10.1](../2025.10.1/release-notes.md)

---
*Para informações técnicas detalhadas, consulte o [Technical Notes](technical-notes.md) desta release.*
