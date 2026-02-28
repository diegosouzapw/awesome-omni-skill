---
name: pca-skill-manager
description: "Acelera a criacao e manutencao de skills do Claude. Automatiza o scaffolding de pastas, validacao de metadados YAML e empacotamento ZIP correto."
---

# Gestor de Habilidades PCA

## Visão Geral
Esta habilidade atua como um "Meta-Assistente". Ela ajuda o Engenheiro de Habilidades a criar novas ferramentas para o projeto PCA Camocim sem erros manuais. Ela garante que todas as novas skills sigam o padrão de "Boneca Russa" e passem na validação estrita do Claude.

## Ferramentas (Scripts)
Esta skill utiliza scripts Python localizados em `scripts/` para gerir o ciclo de vida de outras skills:

1.  **Validar (Quick Validate):**
    Verifica se o `Skill.md` tem chaves inválidas (como `version` ou `dependencies` no lugar errado) e se a estrutura de pastas está correta.
    ```python
    python scripts/quick_validate.py "caminho/para/nova_skill"
    ```

2.  **Empacotar (Package Skill):**
    Cria o arquivo `.zip` final pronto para upload, ignorando arquivos desnecessários (`.DS_Store`, `__pycache__`) e garantindo que a pasta raiz esteja correta.
    ```python
    python scripts/package_skill.py "caminho/para/nova_skill"
    ```

3.  **Iniciar (Init Skill):**
    Cria o esqueleto de uma nova skill com os templates padrão.
    ```python
    python scripts/init_skill.py "nome-da-skill"
    ```

## Fluxo de Trabalho Recomendado

### Criar uma Nova Habilidade
Quando o usuário disser "Crie uma estrutura para a skill 'pca-analytics'":
1.  Execute `init_skill.py` para gerar a pasta e o `Skill.md` básico.
2.  Peça ao usuário as instruções específicas.
3.  Preencha o `Skill.md`.

### Finalizar uma Habilidade
Quando o usuário disser "Empacote a skill 'pca-analytics'":
1.  Execute `quick_validate.py` para garantir que não haverá rejeição no upload.
2.  Se passar, execute `package_skill.py`.
3.  Forneça o caminho do ZIP gerado.

## Referências
Consulte a pasta `references/` para padrões de saída e fluxos de trabalho ideais na criação de agentes.