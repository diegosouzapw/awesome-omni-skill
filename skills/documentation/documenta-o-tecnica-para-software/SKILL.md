---
name: Documentação tecnica para Software
description: Este agente auxilia na criação de documentação técnica detalhada para software, incluindo APIs, diagramas e especificações de requisitos.
---

# Documentação Técnica para Software

Para escrever uma documentação técnica é uma mistura de engenharia, ensino e design. Não se trata apenas de "explicar o código", mas de reduzir a fricção para que o usuário alcance seu objetivo o mais rápido possível.

Aqui estão as habilidades essenciais divididas em três pilares fundamentais:

---

## Habilidades Essenciais

### 1. Habilidades de Escrita e Comunicação

A clareza é a regra de ouro. Se o leitor precisar ler a mesma frase duas vezes para entender, a documentação falhou.

- **Capacidade de Síntese:** Transformar conceitos complexos em explicações diretas, sem "encher linguiça".
- **Voz e Tom Consistentes:** Manter um tom profissional, porém acessível. É importante saber quando ser instrutivo (tutoriais) e quando ser puramente informativo (referência de API).
- **Gramática e Estruturação:** Domínio do idioma e, crucialmente, o uso de **Markdown** para estruturar textos com hierarquia visual clara (títulos, listas e blocos de código).

### 2. Competência Técnica (Hard Skills)

Você não precisa ser o desenvolvedor que escreveu o código, mas precisa entender como ele funciona "sob o capô".

- **Leitura de Código:** Capacidade de ler linguagens como Python, JavaScript ou Java para extrair a lógica que precisa ser documentada.
- **Domínio de Ferramentas "Docs as Code":** Familiaridade com ferramentas como **Git/GitHub**, geradores de sites estáticos (Docusaurus, Hugo) e ferramentas de documentação de API (Swagger/OpenAPI).
- **Entendimento de Arquitetura:** Saber como o software se conecta a bancos de dados, APIs de terceiros e infraestrutura de nuvem.

### 3. Empatia e Experiência do Usuário (UX)

O documentador atua como um advogado do usuário final.

- **Públicos Distintos:** Saber adaptar a linguagem para diferentes perfis (um tom para o CTO que decide a compra e outro para o desenvolvedor que vai implementar o SDK).
- **Curadoria de Conteúdo:** Saber o que _não_ escrever. Documentação em excesso pode ser tão prejudicial quanto a falta dela.
- **Design de Informação:** Organizar a navegação de forma lógica. O usuário deve encontrar o que precisa em poucos cliques.

---

#### Tabela: Tipos de Documentação vs. Foco Principal

| Tipo                  | Objetivo                          | Público-alvo          |
| --------------------- | --------------------------------- | --------------------- |
| **Tutorial**          | Aprender fazendo (passo a passo)  | Iniciantes no produto |
| **Guia de Conceito**  | Entender o "porquê" e a teoria    | Arquitetos e Gestores |
| **Referência de API** | Consulta técnica rápida e precisa | Desenvolvedores       |
| **Troubleshooting**   | Resolver problemas e erros comuns | Suporte e Ops         |

---

## Roteiro de ferramentas e prática

Para avançarmos, preparei um **roteiro de ferramentas e práticas** que são o padrão de mercado atual (o modelo "Docs as Code"). Esse fluxo permite que você trate a documentação com o mesmo rigor que trata o código.

Aqui está um caminho prático para você começar:

---

### 1. O Stack Tecnológico (Ferramentas)

Se você quer profissionalizar sua escrita técnica, esqueça editores de texto comuns (como Word). O foco deve ser em ferramentas que se integram ao fluxo do desenvolvedor:

- **Linguagem de Marcação:** Aprenda **Markdown** a fundo. É o padrão universal para GitHub, GitLab e a maioria dos geradores de sites.
- **Geradores de Sites Estáticos (SSG):** Use o **Docusaurus** (baseado em React, muito usado pela Meta) ou o **MkDocs** (baseado em Python, extremamente simples e eficiente). Eles transformam seus arquivos Markdown em um site de documentação profissional.
- **Versionamento:** Utilize **Git**. Salve sua documentação no mesmo repositório do código para que as atualizações de funcionalidades e manuais aconteçam simultaneamente.
- **Diagramação:** Domine o **Mermaid.js**. Ele permite que você crie fluxogramas e diagramas de sequência usando apenas texto dentro do Markdown.

### 2. Estrutura de um Documento de "Classe Mundial"

Toda excelente documentação de funcionalidade deve seguir esta anatomia:

1. **Título e Sumário:** O que é e o que o usuário vai aprender.
2. **Pré-requisitos:** O que ele precisa ter instalado ou configurado antes.
3. **Quick Start (Início Rápido):** O menor caminho possível para ver algo funcionando.
4. **Exemplos de Código:** Blocos de código comentados e testáveis.
5. **Tabela de Parâmetros:** Descrição clara de entradas, saídas e tipos de dados.
6. **Erros Comuns:** Uma seção de "o que pode dar errado" economiza horas de suporte.

---

### 3. Incorporar a Técnica de Gherkin na Documentação

A técnica de **Gherkin** é um dos pilares da documentação moderna (especialmente no BDD - _Behavior-Driven Development_). Ela funciona como uma "ponte" entre o requisito de negócio e a implementação técnica, transformando conceitos abstratos em cenários concretos e testáveis.

Para incorporá-la à sua documentação de software, siga este guia estruturado:

---

#### 1. A Sintaxe Fundamental

O Gherkin baseia-se em uma estrutura de 3 palavras-chave principais que contam uma história:

- **Dado (Given):** Define o estado inicial ou as pré-condições (ex: o usuário está logado).
- **Quando (When):** Descreve a ação ou o evento gatilho (ex: o usuário clica no botão "Excluir").
- **Então (Then):** Define o resultado esperado ou a reação do sistema (ex: o registro desaparece da lista).

> **Dica:** Use **E (And)** ou **Mas (But)** para adicionar mais condições sem repetir as palavras-chave principais.

---

#### 2. Estrutura de um Arquivo de Funcionalidade (`.feature`)

Ao documentar, organize seus cenários dentro de uma "Funcionalidade" (Feature). No seu fluxo de trabalho com **C#** ou **Angular**, você pode salvar esses arquivos no repositório de documentação ou junto aos testes de aceitação (como o SpecFlow ou Cucumber).

```gherkin
Funcionalidade: Busca de CEP
  Como um usuário do sistema de logística
  Quero consultar um endereço a partir de um CEP
  Para agilizar o preenchimento de cadastros

  Cenário: Consulta de CEP válido
    Dado que eu estou na tela de cadastro de cliente
    Quando eu insiro o CEP "80010-000"
    Então o sistema deve autopreencher o logradouro com "Rua XV de Novembro"
    E a cidade deve ser definida como "Curitiba"

  Cenário: Consulta de CEP inexistente
    Dado que eu estou na tela de cadastro
    Quando eu digito o CEP "00000-000"
    Então o sistema deve exibir a mensagem "CEP não encontrado"

```

---

#### 3. Melhores Práticas para Documentação Técnica

Para que o Gherkin não seja apenas "mais um texto", ele precisa ser útil para o desenvolvedor e para o QA:

- **Abordagem Declarativa vs. Imperativa:** Evite descrever cliques específicos de interface (ex: "Clica no input, digita, aperta Enter"). Foque no **comportamento** (ex: "Quando o usuário pesquisa por..."). Isso evita que sua documentação "quebre" se você mudar um botão de lugar.
- **Use Esquema do Cenário (Scenario Outline):** Se você tem muitos dados para testar a mesma lógica (como diferentes tipos de CEP), use tabelas de exemplos:

```gherkin
Esquema do Cenário: Validar formatos de CEP
  Quando eu envio o CEP <codigo>
  Então o resultado deve ser <status>

  Exemplos:
    | codigo      | status      |
    | "80010000"  | "Sucesso"   |
    | "80010-000" | "Sucesso"   |
    | "123"       | "Inválido"  |

```

- **Vincule aos Requisitos:** Se estiver escrevendo uma **Especificação de Requisitos**, use o Gherkin para definir os **Critérios de Aceite**. Cada cenário Gherkin é, na prática, um critério que o software deve atender para ser considerado "pronto".

---

#### 4. Onde inserir na Documentação?

Se você usa ferramentas como **Docusaurus** ou **MkDocs**, você pode criar uma seção chamada "Cenários de Comportamento" ou "Critérios de Aceite" dentro da página da funcionalidade.

1. **Requisito:** Descrição em texto livre.
2. **User Case:** Fluxo principal.
3. **Cenários Gherkin:** Detalhamento dos casos de sucesso e erro.

#### 5. Exemplo prático

Para que o exemplo seja o mais útil possível para o seu dia a dia, transformei o temas **integração com APIs de consulta de CEP** — em uma documentação estruturada utilizando **Gherkin**.

Aqui está como uma **Especificação de Requisitos** moderna deve parecer:

---

## Funcionalidade: Integração com API de Consulta de CEP

**Objetivo:** Automatizar o preenchimento de endereços no sistema para reduzir erros de digitação e aumentar a velocidade do cadastro.

### 1. Contexto Técnico

- **Serviço:** Integração com API gratuita (ex: ViaCEP).
- **Entrada:** String de 8 dígitos.
- **Saída:** Objeto JSON com logradouro, bairro, localidade e UF.

### 2. Cenários de Comportamento (Gherkin)

```gherkin
# language: pt
Funcionalidade: Busca Automática de Endereço via CEP

  Contexto:
    Dado que o serviço de integração de CEP está ativo e disponível

  Cenário: Consulta de CEP válido com sucesso
    Dado que o usuário está na tela de "Cadastro de Endereço"
    Quando o usuário insere o CEP "80010010" no campo correspondente
    Então o sistema deve realizar uma chamada para a API de busca
    E os campos "Logradouro", "Bairro" e "Cidade" devem ser preenchidos automaticamente
    E o foco do cursor deve ser movido para o campo "Número"

  Esquema do Cenário: Validação de formatos de entrada
    Dado que o usuário insere um CEP no formato <entrada>
    Quando o campo perde o foco (onBlur)
    Então o sistema deve formatar o valor para <saida_esperada>

    Exemplos:
      | entrada      | saida_esperada |
      | "80010010"   | "80010-010"    |
      | "80.010-010" | "80010-010"    |

  Cenário: Tratamento de CEP inexistente
    Dado que o usuário insere um CEP que não consta na base nacional ("00000000")
    Quando a consulta é processada
    Então o sistema deve exibir a mensagem de alerta "CEP não encontrado"
    E permitir que o usuário preencha o endereço manualmente

  Cenário: Falha de conexão com a API
    Dado que o serviço de consulta de CEP está fora do ar
    Quando o usuário tenta realizar uma busca
    Então o sistema deve registrar o erro nos logs internos
    E habilitar a edição manual de todos os campos de endereço sem exibir erro técnico ao usuário

```

---

### Por que esta documentação é "excelente"?

1. **Elimina Ambiguidade:** O desenvolvedor sabe exatamente o que acontece se a API cair ou se o CEP não existir.
2. **Facilita o Teste:** O QA (ou você, ao escrever testes unitários em C# ou Angular) já tem os dados de entrada e os resultados esperados prontos.
3. **Documentação Viva:** Se você usar ferramentas como o **SpecFlow** (para .NET), esses mesmos textos em Gherkin podem se tornar os seus testes automatizados.

---
