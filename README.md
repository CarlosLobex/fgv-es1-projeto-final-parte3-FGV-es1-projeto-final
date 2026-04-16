
[![Open in Codespaces](https://classroom.github.com/assets/launch-codespace-2972f46106e565e64193e422d61a12cf1da4916b45550586e14ef0a7c637dd04.svg)](https://classroom.github.com/open-in-codespaces?assignment_repo_id=23048544)
# Projeto Final ES1 - Parte 3

## Desenvolvimento Iterativo-Incremental

**Nota sobre adaptação didática**: Esta parte está organizada em 3 iterações correspondentes às semanas 7, 8 e 9. Devido ao tempo limitado, apenas a Iteração 1 terá implementação de código. As Iterações 2 e 3 focarão em análise e propostas de solução, permitindo que você demonstre compreensão dos conceitos sem a pressão de múltiplas implementações simultâneas.

---

## Iteração 1 (Semana 7): Princípios SOLID

### Tarefa 1: Análise SOLID no Código Existente (1.5 pontos)

Analise o código atual do backend (`routes/`, `models/`) identificando:

**a) Pontos Positivos (0.75 pontos)**
- Identifique 3 trechos de código que **seguem** princípios SOLID
- Para cada um, explique qual princípio é respeitado e por quê

**b) Oportunidades de Melhoria (0.75 pontos)**
- Identifique 2 trechos que **violam** princípios SOLID
- Para cada um, explique qual princípio é violado e como poderia ser melhorado

Documente em arquivo `ANALISE_SOLID.md` com exemplos de código.

**Entrega**: Arquivo `ANALISE_SOLID.md`

---

### Tarefa 2: Implementação com SOLID (2.0 pontos)

Implemente **1 funcionalidade** das escolhidas na Parte 1, aplicando princípios SOLID:

**Requisitos de implementação:**

1. **Single Responsibility Principle (SRP)**: Separe responsabilidades em classes/módulos distintos
2. **Dependency Inversion Principle (DIP)**: Use abstrações, evite dependências diretas de implementações concretas
3. **Open/Closed Principle (OCP)**: Código deve permitir extensão sem modificação

**Documentação Obrigatória:**

Crie arquivo `IMPLEMENTACAO_SOLID.md` explicando:
- Qual funcionalidade foi implementada
- Como cada princípio SOLID foi aplicado
- Trechos de código que demonstram a aplicação dos princípios

**Entrega**: 
- Código funcional (commits no repositório)
- Arquivo `IMPLEMENTACAO_SOLID.md`

---

## Iteração 2 (Semana 8): Padrões de Projeto

### Tarefa 3: Identificação de Padrões Existentes (1.0 ponto)

Analise o código e identifique padrões de projeto já presentes (mesmo que parcialmente).

Documente em `PADROES_EXISTENTES.md`:
- Quais padrões você identificou
- Onde estão aplicados (módulos/arquivos)
- Se a implementação está completa ou poderia ser melhorada

**Entrega**: Arquivo `PADROES_EXISTENTES.md`

---

### Tarefa 4: Proposta de Aplicação de Padrões de Projeto (2.5 pontos)

Escolha **3 padrões de projeto** e proponha como aplicá-los nas funcionalidades do sistema.

**Sugestões de padrões úteis para este projeto:**
- **Criacionais**: Factory, Singleton
- **Estruturais**: Adapter, Decorator, Facade
- **Comportamentais**: Strategy, Observer, Template Method

**Para cada padrão (2.5 pontos divididos entre os 3):**

**a) Justificativa e Contexto**
- Em qual funcionalidade o padrão seria aplicado?
- Qual problema ele resolve?
- Por que este padrão é adequado?

**b) Proposta de Solução**
- Descreva detalhadamente como o padrão seria implementado
- Quais classes/módulos seriam criados?
- Como eles interagiriam?
- Inclua diagrama de classes UML mostrando a estrutura do padrão aplicado ao contexto

**c) Exemplo de Código (pseudo-código aceito)**
- Mostre trechos-chave da implementação proposta
- Pode ser pseudo-código ou código JavaScript simplificado
- Foque nas partes que demonstram o padrão

Documente em arquivo `PADROES_PROPOSTOS.md` com diagramas e explicações.

**Requisitos de entrega para diagramas:**
- Arquivo de imagem para cada diagrama
- **SE usar Mermaid ou PlantUML**: arquivo fonte (`.mmd` ou `.puml`)
- **SE usar draw.io ou Lucidchart**: link compartilhável do projeto
- **SE usar ferramenta de modelagem (Modelio, ArgoUML, etc.)**: arquivo do projeto

**Entrega**:
- Arquivo `PADROES_PROPOSTOS.md` com diagramas e explicações detalhadas
- Arquivos fonte ou links dos diagramas

---

## Iteração 3 (Semana 9): Arquitetura

### Tarefa 5: Análise Arquitetural (1.0 ponto)

Analise a arquitetura atual do sistema ESM Forum.

Documente em arquivo `ARQUITETURA.md`:

**a) Identificação da Arquitetura (0.5 pontos)**
- Qual(is) estilo(s) arquitetural(is) o sistema atual segue?
- Identifique as camadas existentes (apresentação, negócio, dados)
- Como o frontend e backend se comunicam?

**b) Diagrama Arquitetural (0.5 pontos)**
- Crie um diagrama mostrando os principais componentes
- Indique as camadas e suas responsabilidades
- Mostre o fluxo de dados

**Requisitos de entrega para diagrama:**
- Arquivo de imagem do diagrama
- **SE usar Mermaid ou PlantUML**: arquivo fonte (`.mmd` ou `.puml`)
- **SE usar draw.io ou Lucidchart**: link compartilhável do projeto
- **SE usar ferramenta de modelagem (Modelio, ArgoUML, etc.)**: arquivo do projeto

**Entrega**: Arquivo `ARQUITETURA.md` com diagrama e arquivos fonte/links

---

### Tarefa 6: Proposta de Organização Arquitetural (2.0 pontos)

Proponha como organizar o código seguindo boas práticas arquiteturais para as funcionalidades implementadas/propostas.

**a) Proposta de Separação em Camadas (1.0 ponto)**

Descreva como você organizaria o código em camadas bem definidas:
- **Camada de Apresentação** (API routes): O que ficaria aqui?
- **Camada de Negócio** (lógica de aplicação): Quais regras e operações?
- **Camada de Dados** (acesso ao banco): Como seria estruturado?

Para cada camada:
- Liste as responsabilidades específicas
- Dê exemplos de classes/módulos que fariam parte
- Explique como as camadas se comunicariam

**b) Proposta de Aplicação do Padrão MVC (1.0 ponto)**

Proponha como aplicar MVC no backend para pelo menos 2 funcionalidades:
- **Models**: Que dados seriam modelados? Que operações?
- **Views**: Como as respostas JSON seriam formatadas?
- **Controllers**: Que lógica de controle seria necessária?

Inclua:
- Diagrama mostrando a estrutura MVC proposta
- Descrição detalhada de como cada componente funciona
- Exemplo de fluxo completo (requisição → resposta)

Documente em `PROPOSTA_ARQUITETURA.md`.

**Requisitos de entrega para diagramas:**
- Arquivo de imagem para cada diagrama
- **SE usar Mermaid ou PlantUML**: arquivo fonte (`.mmd` ou `.puml`)
- **SE usar draw.io ou Lucidchart**: link compartilhável do projeto
- **SE usar ferramenta de modelagem (Modelio, ArgoUML, etc.)**: arquivo do projeto

**Entrega**:
- Arquivo `PROPOSTA_ARQUITETURA.md` com diagramas e explicações detalhadas
- Arquivos fonte ou links dos diagramas

---

## Pontuação Total da Parte 3: 10.0 pontos

### Iteração 1 - SOLID
| Tarefa | Nota |
|--------|--------|
| Análise SOLID | 1.5 |
| Implementação com SOLID | 2.0 |
| **Subtotal Iteração 1** | **3.5** |

### Iteração 2 - Padrões
| Tarefa | Nota |
|--------|--------|
| Identificação de Padrões | 1.0 |
| Proposta de Padrões | 2.5 |
| **Subtotal Iteração 2** | **3.5** |

### Iteração 3 - Arquitetura
| Tarefa | Nota |
|--------|--------|
| Análise Arquitetural | 1.0 |
| Proposta Arquitetural | 2.0 |
| **Subtotal Iteração 3** | **3.0** |

### **TOTAL PARTE 3** | **10.0**

---

## Critérios de Qualidade

**Para implementação (Tarefa 2):**
- **Funcionalidade**: Código deve executar sem erros
- **Aplicação dos Conceitos**: Princípios SOLID corretamente implementados
- **Qualidade do Código**: Código limpo, bem organizado e documentado
- **Documentação**: Explicações claras com exemplos

**Para propostas (Tarefas 4 e 6):**
- **Completude**: Propostas devem cobrir todos os aspectos solicitados
- **Viabilidade**: Soluções devem ser realistas e aplicáveis
- **Clareza**: Explicações detalhadas e bem estruturadas
- **Fundamentação**: Decisões baseadas nos conceitos estudados
- **Qualidade dos Diagramas**: Diagramas claros e informativos

---

## Prazo de Entrega

**Até o final da Semana 9**

Submeta:
- Código da funcionalidade implementada (Tarefa 2) nos repositórios
- Todos os arquivos markdown solicitados
- Todos os diagramas (imagens + fontes/links quando aplicável)

---

## Notas Importantes

1. **Commits regulares**: Para a implementação (Tarefa 2), faça commits frequentes mostrando a evolução do trabalho
2. **Mensagens descritivas**: Use mensagens de commit claras
3. **Documentação rica**: Nas propostas, quanto mais detalhada a explicação, melhor
4. **Diagramas informativos**: Use diagramas para complementar explicações textuais
5. **Pseudo-código é válido**: Não precisa ser código JavaScript perfeito se for apenas para ilustrar a ideia

**Dica**: Mesmo sem implementar todas as funcionalidades, você pode demonstrar profundo entendimento através de propostas bem elaboradas!
