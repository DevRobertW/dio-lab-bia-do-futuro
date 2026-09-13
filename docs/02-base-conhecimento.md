Claro. Fiz os ajustes de precisão e mantive o documento como **um único `docs/02-base-conhecimento.md`**, já pronto para substituir o anterior. Também removi a linguagem de rascunho e transformei a sugestão do Hugging Face em uma seção de evolução futura.

# Base de Conhecimento — Íris

> Este documento é complementar ao [`01-documentacao-agente.md`](./01-documentacao-agente.md). Enquanto aquele define **persona, tom e arquitetura** do agente, este detalha exclusivamente a **camada de dados**: quais fontes são utilizadas, como são processadas, como são transformadas em contexto para o LLM e como o risco de alucinação é mitigado.

---

## ⚠️ Princípio Fundamental

> **A Íris deve preferir admitir que não sabe a inventar uma informação.**

O LLM é responsável por **interpretar, explicar e conversar** com o usuário.

A Base de Conhecimento é responsável por fornecer os **fatos e o contexto autorizado**.

Essa separação torna o comportamento da Íris mais previsível, auditável e confiável. O modelo não deve ser tratado como fonte de dados do cliente ou do catálogo de produtos.

---

# Bloco 1 — Objetivo e Dados

## 1.1 Objetivo da Base de Conhecimento

A Base de Conhecimento da **Íris** fornece ao agente informações estruturadas e confiáveis sobre:

* Perfil financeiro do cliente;
* Comportamento financeiro observado;
* Histórico de atendimento;
* Produtos financeiros disponíveis;
* Objetivos e características relevantes para adequação.

O objetivo é evitar que o LLM precise interpretar diretamente grandes volumes de dados brutos.

Antes de chegar ao modelo, os dados passam por etapas de:

1. Carregamento;
2. Validação;
3. Normalização;
4. Pré-processamento;
5. Agregação;
6. Seleção de informações relevantes;
7. Montagem do contexto.

A arquitetura utiliza **quatro fontes principais de informação** e uma **camada de regras**:

| Camada            | Função                                                   |
| ----------------- | -------------------------------------------------------- |
| **Perfil**        | Representa características declaradas do cliente         |
| **Comportamento** | Representa padrões observados nas transações             |
| **Memória**       | Representa interações anteriores                         |
| **Produtos**      | Representa o catálogo autorizado                         |
| **Regras**        | Determinam critérios de filtragem, adequação e segurança |

Com isso, a Íris consegue produzir respostas personalizadas sem depender exclusivamente do conhecimento interno do LLM.

---

## 1.2 Dados Utilizados

| Arquivo                     | Formato | Papel no Agente                                                       | Confiabilidade |
| --------------------------- | ------- | --------------------------------------------------------------------- | -------------- |
| `perfil_investidor.json`    | JSON    | Informações estruturadas sobre perfil de risco, objetivos e horizonte | Alta           |
| `produtos_financeiros.json` | JSON    | Catálogo autorizado de produtos e suas características                | Alta           |
| `transacoes.csv`            | CSV     | Análise do comportamento financeiro e identificação de padrões        | Alta           |
| `historico_atendimento.csv` | CSV     | Memória de interações e identificação de assuntos pendentes           | Média/Alta     |

### `perfil_investidor.json`

Representa o perfil financeiro e de investimento do cliente.

Pode conter informações como:

* Perfil de risco;
* Objetivos financeiros;
* Horizonte de investimento;
* Preferências;
* Experiência com investimentos;
* Valor disponível para investir.

**Uso principal:** personalização e avaliação de adequação das sugestões.

O perfil estruturado é a principal referência para determinar o nível de risco que pode ser considerado nas sugestões.

---

### `produtos_financeiros.json`

Representa o catálogo de produtos financeiros que a Íris pode apresentar ao cliente.

Pode conter informações como:

* Nome;
* Tipo;
* Nível de risco;
* Prazo;
* Liquidez;
* Rentabilidade indicativa, quando disponível no dataset;
* Aporte mínimo;
* Objetivo;
* Perfil recomendado;
* Tags de utilização.

O arquivo é a **fonte autorizada para informações sobre produtos**.

A Íris não deve inventar:

* Produtos;
* Taxas;
* Rentabilidades;
* Prazos;
* Liquidez;
* Valores mínimos;
* Características que não estejam presentes no catálogo.

---

### `transacoes.csv`

Contém o histórico financeiro fictício utilizado para identificar padrões de comportamento.

Pode ser utilizado para calcular:

* Total de entradas;
* Total de saídas;
* Saldo estimado do período;
* Média mensal de entradas;
* Média mensal de despesas;
* Principais categorias de gastos;
* Aportes;
* Frequência dos aportes;
* Tendências de gastos;
* Possíveis sobras recorrentes.

As transações **não são enviadas integralmente ao LLM**.

Antes disso, são processadas e transformadas em um resumo financeiro compacto.

---

### `historico_atendimento.csv`

Representa a memória das interações anteriores.

É utilizado para:

* Retomar assuntos pendentes;
* Evitar perguntas repetidas;
* Contextualizar novas conversas;
* Identificar decisões ou dúvidas anteriores;
* Apoiar acompanhamentos proativos.

O campo `resolvido` permite diferenciar assuntos encerrados de assuntos que ainda podem exigir acompanhamento.

> **Observação:** o histórico é considerado confiável como registro conversacional, mas não substitui informações estruturadas e atuais do perfil ou dos produtos.

---

# Bloco 2 — Adaptações nos Dados

## 2.1 Normalização de Categorias

Os dados mockados originais foram mantidos em sua estrutura principal, recebendo adaptações necessárias para melhorar a análise e a integração com o agente.

As categorias de `transacoes.csv` foram padronizadas para evitar duplicidade semântica.

Exemplo:

```text
Mercado
Supermercado
Feira
```

podem ser normalizados para:

```text
Alimentação
```

Essa normalização permite realizar agregações mais confiáveis.

Sem ela, uma análise poderia interpretar:

```text
Mercado       → R$ 500
Supermercado → R$ 400
Feira         → R$ 200
```

como três grupos diferentes, quando na prática todos representam uma mesma categoria de consumo.

---

## 2.2 Campo `resolvido`

Foi adicionado o campo booleano `resolvido` ao `historico_atendimento.csv`.

Exemplo:

```text
resolvido = true
```

indica que o assunto foi encerrado.

Já:

```text
resolvido = false
```

indica que o assunto permanece pendente.

Essa informação permite que a Íris identifique temas que podem ser retomados posteriormente.

---

## 2.3 Campo `indicado_para`

Foi adicionado o campo `indicado_para` ao `produtos_financeiros.json`.

Esse campo utiliza tags estruturadas para facilitar a filtragem dos produtos.

Exemplos:

```text
reserva_emergencia
curto_prazo
medio_prazo
longo_prazo
diversificacao
liquidez
aposentadoria
```

O uso dessas tags reduz a dependência de buscas textuais e permite um processo de seleção mais previsível.

---

## 2.4 Dados Fictícios

Todos os dados utilizados no projeto são fictícios ou mockados.

Nenhum dado bancário real deve ser utilizado no repositório.

Não devem ser armazenados:

* Senhas;
* Tokens;
* Chaves de API;
* Dados bancários reais;
* Números de cartão;
* CPF real;
* Credenciais;
* Informações financeiras reais de terceiros.

---

# Bloco 3 — Hierarquia das Fontes

## 3.1 Ordem de Prioridade

Em caso de conflito entre informações, a Íris deve considerar a seguinte ordem de prioridade:

1. `perfil_investidor.json`;
2. `produtos_financeiros.json`;
3. `transacoes.csv`;
4. `historico_atendimento.csv`;
5. Conhecimento geral do LLM.

O conhecimento geral do LLM **não pode ser utilizado para preencher informações específicas ausentes da base**.

---

## 3.2 Regras por Fonte

### Perfil

Utilizado como referência para:

* Perfil de risco;
* Objetivos;
* Horizonte;
* Características declaradas.

### Produtos

Utilizado como referência para:

* Nome;
* Características;
* Elegibilidade;
* Risco;
* Prazo;
* Liquidez;
* Demais informações presentes no catálogo.

### Transações

Utilizadas como referência para:

* Comportamento financeiro;
* Gastos;
* Entradas;
* Aportes;
* Padrões observados.

### Histórico

Utilizado principalmente como:

* Memória conversacional;
* Contexto de interações anteriores;
* Identificação de pendências.

---

## 3.3 Regra para Conflitos

Caso duas fontes apresentem informações diferentes, a fonte de maior prioridade deve prevalecer.

Exemplo:

```text
perfil_investidor.json
Perfil: Moderado

historico_atendimento.csv
Cliente afirmou anteriormente ser Conservador
```

Nesse caso, o perfil estruturado atual deve ser utilizado como referência.

Se o conflito puder alterar significativamente uma recomendação, a Íris deve solicitar confirmação ao usuário em vez de assumir qual informação está correta.

---

# Bloco 4 — Estratégia de Integração

## 4.1 Fluxo Geral

A integração segue o fluxo:

```text
Dados brutos
     ↓
Carregamento
     ↓
Validação
     ↓
Pré-processamento
     ↓
Normalização
     ↓
Agregação
     ↓
Seleção de contexto
     ↓
Montagem do prompt
     ↓
LLM
     ↓
Validação da resposta
     ↓
Usuário
```

---

## 4.2 Carregamento

Os arquivos são carregados no início da sessão e mantidos em memória utilizando o mecanismo de estado do Streamlit.

O objetivo é evitar leituras repetidas do disco durante cada interação.

### Arquivos JSON

Os arquivos:

```text
perfil_investidor.json
produtos_financeiros.json
```

são carregados utilizando `json.load()`.

### Arquivos CSV

Os arquivos:

```text
transacoes.csv
historico_atendimento.csv
```

são carregados utilizando `pandas.read_csv()`.

Após o carregamento, os dados passam por validações e pré-processamento.

---

# Bloco 5 — Pré-processamento

## 5.1 Transações

O processamento de `transacoes.csv` pode incluir:

* Conversão e validação de datas;
* Padronização das categorias;
* Separação entre entradas e saídas;
* Agrupamento por categoria;
* Agrupamento por período;
* Cálculo de médias;
* Identificação de aportes;
* Identificação de padrões de gastos.

O resultado é transformado em um resumo financeiro.

---

## 5.2 Cálculo do Saldo Estimado

Quando calculado exclusivamente a partir das transações disponíveis, o saldo representa uma **estimativa baseada no período analisado**, e não necessariamente o saldo bancário atual.

A fórmula básica é:

```text
Saldo estimado do período =
Total de entradas - Total de saídas
```

Esse indicador deve ser apresentado como uma estimativa sempre que não houver uma fonte de saldo bancário real.

---

## 5.3 Histórico de Atendimento

O `historico_atendimento.csv` pode passar por:

* Ordenação cronológica;
* Identificação dos atendimentos mais recentes;
* Separação entre assuntos resolvidos e pendentes;
* Seleção dos registros relevantes para a sessão.

O objetivo é evitar enviar todo o histórico ao LLM.

---

## 5.4 Produtos

Os produtos passam por filtros antes de serem apresentados ao LLM.

Os principais critérios são:

* Perfil de risco;
* Horizonte;
* Objetivo;
* Intenção da pergunta;
* Características do produto;
* Tags `indicado_para`.

---

# Bloco 6 — Resumo Financeiro

A Íris não precisa enviar todas as transações históricas ao LLM.

Em vez disso, é criado um resumo financeiro.

Um resumo dos últimos 90 dias pode conter:

```text
Período analisado: últimos 90 dias

Entradas médias mensais: R$ X
Despesas médias mensais: R$ Y
Saldo estimado do período: R$ Z
Aporte médio mensal: R$ W

Principais categorias:
- Alimentação
- Transporte
- Moradia

Frequência de aportes:
- X aportes no período
```

Os valores devem ser calculados diretamente a partir dos dados disponíveis e não podem ser inventados pelo LLM.

---

# Bloco 7 — Estratégia de Contexto

A Íris utiliza uma estratégia híbrida de injeção de contexto.

| Dado                        | Onde entra          | Frequência          |
| --------------------------- | ------------------- | ------------------- |
| `perfil_investidor.json`    | Contexto fixo       | Uma vez por sessão  |
| Resumo de `transacoes.csv`  | Contexto financeiro | Uma vez por sessão  |
| Histórico relevante         | Memória resumida    | Uma vez por sessão  |
| `produtos_financeiros.json` | Contexto dinâmico   | Conforme a pergunta |

Essa estratégia evita o envio desnecessário de informações ao modelo.

---

## 7.1 Contexto Fixo

O contexto fixo pode conter:

* Perfil do cliente;
* Objetivos;
* Horizonte;
* Resumo financeiro;
* Memória relevante.

---

## 7.2 Contexto Dinâmico

O contexto dinâmico é montado de acordo com a intenção da pergunta.

Por exemplo, uma pergunta sobre reserva de emergência não precisa carregar todo o catálogo de produtos.

A Íris deve selecionar somente os produtos potencialmente relevantes para aquela intenção.

---

# Bloco 8 — Seleção Dinâmica de Produtos

## 8.1 Fluxo de Seleção

```text
Pergunta do usuário
        ↓
Identificação da intenção
        ↓
Perfil de risco
        ↓
Objetivo financeiro
        ↓
Horizonte de investimento
        ↓
Características desejadas
        ↓
Tags "indicado_para"
        ↓
Produtos elegíveis
        ↓
LLM
```

---

## 8.2 Perfil de Risco

O produto deve ser compatível com o perfil disponível do cliente.

Exemplo:

```text
Cliente: Moderado

Produto:
Perfil recomendado: Moderado

Resultado:
Elegível
```

---

## 8.3 Horizonte

O prazo do produto deve ser compatível com o horizonte declarado pelo cliente.

Exemplo:

```text
Cliente:
Horizonte: Longo prazo

Produto:
Indicado para: longo_prazo

Resultado:
Compatível
```

---

## 8.4 Objetivo

O objetivo financeiro também deve ser considerado.

Exemplo:

```text
Objetivo:
Reserva de emergência

Tags relevantes:
reserva_emergencia
liquidez
curto_prazo
```

---

## 8.5 Intenção da Pergunta

A intenção do usuário também influencia a seleção.

Exemplo:

```text
"Quero montar uma reserva de emergência."
```

deve priorizar produtos relacionados a:

```text
reserva_emergencia
liquidez
curto_prazo
```

Enquanto uma pergunta sobre diversificação pode priorizar:

```text
diversificacao
medio_prazo
longo_prazo
```

---

# Bloco 9 — Regras de Interpretação

A Íris deve diferenciar três níveis de informação.

## 9.1 Fato

Informação diretamente presente ou calculada a partir da base.

Exemplo:

> "A média de despesas nos últimos 90 dias foi de R$ X."

---

## 9.2 Inferência

Conclusão obtida a partir dos dados.

Exemplo:

> "Os dados indicam uma possível sobra financeira recorrente."

A inferência não deve ser apresentada como certeza absoluta.

---

## 9.3 Recomendação

Sugestão construída considerando os dados disponíveis.

Exemplo:

> "Considerando seu perfil moderado e o objetivo informado, podemos avaliar opções compatíveis com esse cenário."

A recomendação deve considerar:

* Perfil;
* Objetivo;
* Horizonte;
* Dados financeiros relevantes;
* Riscos conhecidos;
* Produtos disponíveis.

---

# Bloco 10 — Ausência de Dados

Quando uma informação necessária não estiver disponível, a Íris deve admitir a limitação.

### Exemplo

Usuário:

> "Quanto tenho investido atualmente?"

Se esse valor não estiver disponível:

> "Não encontrei esse valor nos dados disponíveis para esta sessão."

A Íris pode então oferecer uma alternativa:

> "Posso analisar os aportes registrados ou explicar quais informações seriam necessárias para calcular esse valor."

A agente **não deve estimar um valor simplesmente para preencher a lacuna**.

---

# Bloco 11 — Conflitos e Inconsistências

Quando os dados forem conflitantes, a Íris deve:

1. Identificar a inconsistência;
2. Aplicar a hierarquia das fontes;
3. Evitar assumir informações não confirmadas;
4. Solicitar confirmação quando o conflito for relevante.

Exemplo:

```text
Perfil estruturado:
Moderado

Histórico:
Cliente mencionou ser Conservador
```

A informação estruturada deve prevalecer.

Porém, se a diferença alterar significativamente uma recomendação, a Íris deve confirmar o perfil atual antes de avançar.

---

# Bloco 12 — Memória de Atendimento

A memória existe para melhorar a continuidade da experiência.

A Íris pode utilizá-la para:

* Retomar assuntos pendentes;
* Evitar perguntas repetidas;
* Relembrar objetivos mencionados;
* Acompanhar decisões anteriores;
* Identificar oportunidades de acompanhamento.

### Exemplo

Histórico:

```text
Assunto: Reserva de emergência
Status: Não resolvido
```

A Íris pode iniciar uma conversa relacionada dizendo:

> "Na nossa conversa anterior, você estava avaliando sua reserva de emergência. Quer continuar daquele ponto?"

A memória deve ser utilizada como **contexto conversacional**, e não como substituto para dados financeiros atuais.

---

# Bloco 13 — Proatividade

A Íris pode identificar padrões relevantes e transformar essas observações em oportunidades de conversa.

Exemplo:

```text
Entradas médias: R$ 4.500
Despesas médias: R$ 3.700
Sobra estimada: R$ 800
Aportes registrados: R$ 0
```

A Íris pode dizer:

> "Percebi uma possível sobra recorrente no seu histórico. Se quiser, podemos analisar como esse valor poderia contribuir para um dos seus objetivos."

A agente não deve assumir automaticamente que toda sobra deve ser investida.

---

## 13.1 Limites da Proatividade

A Íris não deve:

* Pressionar o usuário;
* Criar urgência artificial;
* Usar medo para incentivar decisões;
* Assumir que toda sobra deve ser investida;
* Fazer recomendações incompatíveis com o perfil;
* Tratar um padrão isolado como comportamento permanente;
* Transformar uma observação em recomendação automática.

A proatividade deve funcionar como uma **abertura para conversa**, não como uma ordem.

---

# Bloco 14 — Validação Anti-Alucinação

A resposta gerada pelo LLM pode passar por uma camada de validação antes de ser apresentada ao usuário.

Entre as verificações possíveis:

* Produto citado existe no catálogo;
* Valores mencionados estão presentes ou foram calculados a partir dos dados;
* Taxas citadas existem na fonte;
* Rentabilidades citadas existem na fonte;
* Recomendações respeitam o perfil disponível;
* Não foram criados produtos inexistentes;
* Não foram inventados dados do cliente;
* Não existem promessas de rentabilidade garantida;
* Informações ausentes não foram apresentadas como fatos.

---

## 14.1 Tratamento de Falhas

Caso uma inconsistência seja identificada, o sistema pode:

### Opção 1 — Regeneração

Solicitar ao LLM uma nova resposta utilizando regras reforçadas.

### Opção 2 — Resposta segura

Informar ao usuário que a informação não está disponível ou não pôde ser validada.

A prioridade deve ser **confiabilidade sobre completude**.

---

# Bloco 15 — Exemplo de Contexto Montado

Um contexto simplificado enviado ao LLM pode assumir a seguinte estrutura:

```text
[PERFIL DO CLIENTE]

Perfil de risco: Moderado
Objetivo principal: Formação de patrimônio
Horizonte: Longo prazo
Experiência: Intermediária


[RESUMO FINANCEIRO — ÚLTIMOS 90 DIAS]

Entradas médias mensais: R$ X
Despesas médias mensais: R$ Y
Saldo estimado: R$ Z
Aporte médio mensal: R$ W

Principais categorias:
- Alimentação
- Transporte
- Moradia


[MEMÓRIA DE ATENDIMENTO]

Último assunto: Reserva de emergência
Status: Não resolvido


[PRODUTOS ELEGÍVEIS]

Produto: Produto A
Risco: Moderado
Horizonte: Longo prazo
Indicado para:
- diversificacao
- longo_prazo


[REGRAS]

- Utilize somente as informações fornecidas neste contexto.
- Não invente produtos.
- Não invente taxas ou rentabilidades.
- Não invente dados do cliente.
- Diferencie fatos de inferências.
- Considere o perfil antes de sugerir produtos.
- Explique riscos relevantes.
- Se os dados forem insuficientes, informe a limitação.
```

Os valores apresentados nesse exemplo são apenas placeholders e não representam dados reais do dataset.

---

# Bloco 16 — Fluxo Completo de Consulta

Uma interação típica segue o seguinte fluxo:

```text
┌─────────────────────────┐
│   Pergunta do usuário   │
└────────────┬────────────┘
             ↓
┌─────────────────────────┐
│ Identificação da        │
│ intenção                │
└────────────┬────────────┘
             ↓
┌─────────────────────────┐
│ Recuperação do contexto │
│ relevante               │
└────────────┬────────────┘
             ↓
┌─────────────────────────┐
│ Perfil + objetivo +     │
│ horizonte               │
└────────────┬────────────┘
             ↓
┌─────────────────────────┐
│ Filtragem de produtos   │
│ quando aplicável        │
└────────────┬────────────┘
             ↓
┌─────────────────────────┐
│ Montagem do contexto    │
└────────────┬────────────┘
             ↓
┌─────────────────────────┐
│ Geração pelo LLM        │
└────────────┬────────────┘
             ↓
┌─────────────────────────┐
│ Validação               │
│ anti-alucinação         │
└────────────┬────────────┘
             ↓
┌─────────────────────────┐
│ Resposta ao usuário     │
└─────────────────────────┘
```

---

# Bloco 17 — Segurança e Privacidade

A Base de Conhecimento utiliza dados fictícios e deve seguir princípios de minimização de dados.

### Práticas adotadas

* Não armazenar credenciais no código;
* Não armazenar chaves de API no Git;
* Utilizar variáveis de ambiente para secrets;
* Não registrar dados financeiros desnecessários em logs;
* Não utilizar dados bancários reais;
* Não enviar informações desnecessárias ao LLM;
* Limitar o contexto ao necessário para cada pergunta.

---

# Bloco 18 — Limitações

A Base de Conhecimento possui limitações importantes.

### Dados mockados

Os dados não representam uma conta bancária real.

### Ausência de dados em tempo real

A Íris não consulta, nesta versão:

* Conta bancária real;
* Saldo bancário atual;
* Cotações em tempo real;
* Taxas de mercado atualizadas;
* APIs de instituições financeiras.

### Catálogo limitado

A Íris só pode apresentar produtos presentes no catálogo autorizado.

### Histórico limitado

A memória depende dos registros existentes em `historico_atendimento.csv`.

### Inferências

Os padrões identificados nas transações representam análises sobre o dataset disponível e não necessariamente refletem a situação financeira atual do cliente.

---

# Bloco 19 — Possíveis Expansões

A arquitetura foi projetada para permitir evolução futura.

Possíveis extensões incluem:

* Banco de dados PostgreSQL;
* Integração com APIs financeiras;
* Atualização automática do catálogo de produtos;
* Memória persistente;
* Busca semântica;
* Embeddings e banco vetorial;
* RAG baseado em documentos;
* Monitoramento de qualidade das respostas;
* Avaliação automática de alucinações;
* Sistema de feedback do usuário.

Também pode ser adicionada uma camada complementar de conhecimento geral sobre atendimento bancário.

Datasets públicos, como os disponíveis no [Hugging Face](https://huggingface.co/datasets), podem ser utilizados futuramente para ampliar a capacidade de responder perguntas frequentes, desde que sejam adequados ao escopo do projeto e às regras de licenciamento.

Essa camada de conhecimento deve ser mantida separada dos dados específicos do cliente.

---

# Bloco 20 — Princípio de Arquitetura

A arquitetura da Íris segue uma separação clara de responsabilidades:

```text
┌─────────────────────────────┐
│      BASE DE CONHECIMENTO   │
│                             │
│ Perfil                      │
│ Transações                  │
│ Histórico                   │
│ Produtos                    │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│      PROCESSAMENTO          │
│                             │
│ Normalização                │
│ Agregação                   │
│ Filtragem                   │
│ Seleção de contexto         │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│            LLM              │
│                             │
│ Interpretação               │
│ Raciocínio                  │
│ Comunicação                 │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│         VALIDAÇÃO           │
│                             │
│ Anti-alucinação             │
│ Consistência                │
│ Adequação                   │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│          USUÁRIO            │
└─────────────────────────────┘
```

O LLM é responsável por **interpretar e comunicar**.

A Base de Conhecimento é responsável por **fornecer os dados autorizados**.

A camada de processamento é responsável por **transformar dados brutos em contexto útil**.

A camada de validação é responsável por **reduzir o risco de respostas inconsistentes ou alucinadas**.

---

## Conclusão

A Base de Conhecimento da Íris foi projetada para manter uma separação clara entre **dados, processamento e geração de linguagem**.

O princípio central é simples:

> **A Íris deve preferir admitir que não sabe a inventar uma informação.**

Essa abordagem permite que o agente seja proativo e personalizado sem abrir mão de transparência, rastreabilidade e controle sobre as informações utilizadas nas respostas.
