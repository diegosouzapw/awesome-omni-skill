---
name: openclaw-config-validator
description: Validate, analyze, and explain OpenClaw configuration files. Use when users need to check config.json for errors, understand what a config field does, compare configs, or safely modify OpenClaw configuration. Triggers on config validation requests, schema questions, or config editing tasks.
---

# OpenClaw Config Validator

Skill para validar, analisar e explicar configurações do OpenClaw.

## Fonte da Verdade

O schema oficial está em:
- **Runtime** (validação): `~/.nvm/versions/node/v24.13.0/lib/node_modules/openclaw/dist/config/zod-schema*.js`
- **Labels**: `dist/config/schema.js`
- **Repo** (referência): https://github.com/openclaw/openclaw

> **Nota**: A validação sempre usa o schema local (versão instalada) para garantir precisão.

## Scripts Disponíveis

### schema-analyzer.js
```bash
node scripts/schema-analyzer.js analyze <config.json>    # Analisa estrutura
node scripts/schema-analyzer.js validate <config.json>   # Valida schema
node scripts/schema-analyzer.js labels                   # Lista todos os labels
node scripts/schema-analyzer.js explain <field.path>     # Explica um campo
```

### config-diff.js
```bash
node scripts/config-diff.js report <config.json>   # Relatório completo
node scripts/config-diff.js diff <config.json>     # Diferenças do padrão
```

### check-update.js
```bash
node scripts/check-update.js check-update   # Verifica campos novos no GitHub
```
Verifica se há campos novos no schema do repositório comparado à versão local instalada.

### auto-fix.js
```bash
node scripts/auto-fix.js fix <config.json> --dry-run   # Preview de correções
node scripts/auto-fix.js fix <config.json> --apply     # Aplica correções
```
Corrige automaticamente:
- `"true"` → `true` (strings para boolean)
- `"123"` → `123` (strings para number)
- Enums inválidos → sugere valor correto
- Detecta campos renomeados/obsoletos

### migrate.js
```bash
node scripts/migrate.js migrate <config.json> --from=0.16.0 --to=0.17.0
node scripts/migrate.js migrate <config.json> --from=0.16.0 --to=0.17.0 --output=config-v17.json
```
Migra config entre versões do OpenClaw:
- Detecta breaking changes
- Renomeia campos automaticamente
- Adiciona novos campos com valores padrão
- Gera relatório de mudanças

## Validações Automáticas

- Seções desconhecidas (possivelmente obsoletas)
- Canais sem tipo definido
- Gateway remoto sem token
- Campos de segurança expostos
- Valores de enum inválidos

## Como Usar

**Para validar uma config:**
1. Rode `schema-analyzer.js validate` no arquivo
2. Analise erros e warnings
3. Use `auto-fix.js --dry-run` para ver correções possíveis
4. Aplique correções com `auto-fix.js --apply`
5. Re-valide

**Para migrar entre versões:**
1. Use `check-update.js` para ver novos campos disponíveis
2. Use `migrate.js --from=X --to=Y --dry-run` para preview
3. Aplique com `--output` para não sobrescrever o original

**Para entender um campo:**
1. Use `schema-analyzer.js explain <campo>`
2. Ou consulte os labels em schema.js

**Para modificar configs com segurança:**
1. Valide a config atual primeiro
2. Faça backup
3. Use auto-fix para correções seguras
4. Aplique mudanças incrementais
5. Re-valide após cada mudança

## Referências

- [Schema Reference](references/schema-reference.md) - Documentação completa
- [Roadmap](ROADMAP.md) - Melhorias futuras planejadas
