---
name: admin-chat-ui-flow
description: Padroniza UX visual e fluxo da tela de chat admin com estilo da area cliente.
bundle: frontend
risk: caution
version: 1.0.0
---

# admin-chat-ui-flow

Skill para manter a tela de chat clara, consistente e responsiva.

## Inputs

- `chat_sections` (threads, meta, messages, composer)
- `interaction_states` (loading, empty, sending, error)
- `visual_tokens`

## Outputs

- `templates/pages/admin/chat.php`
- `public/assets/js/chat.js` (quando fluxo visual exigir)

## Criterios de aceite

- Fluxo de thread/sending permanece funcional.
- Componentes de chat seguem identidade green-glass.
- Interface continua usavel em viewport mobile.
