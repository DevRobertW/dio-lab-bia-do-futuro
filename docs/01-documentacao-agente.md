# 📄 Documentação do Agente — Íris do Futuro

## Caso de Uso

### Problema
Grande parte dos brasileiros não investe ou investe mal por falta de **orientação financeira personalizada e acessível**. Consultorias tradicionais são caras, exigem agendamento e raramente consideram o contexto real do cliente (renda, gastos, objetivos). Além disso, os chatbots financeiros atuais são **reativos** — só respondem o que é perguntado — e frequentemente **alucinam**, oferecendo produtos inexistentes ou prometendo retornos irreais.

### Solução
A **Íris do Futuro** é um agente financeiro proativo que:

- **Antecipa necessidades** ao cruzar o perfil do investidor com o histórico de transações (ex: percebe sobra recorrente e sugere automatizar um aporte).
- **Personaliza sugestões** com base no perfil de risco, objetivos e produtos já contratados.
- **Cocria soluções** de forma consultiva, fazendo perguntas antes de recomendar.
- **Garante confiabilidade** ao responder exclusivamente com base nos dados mockados fornecidos, citando a fonte e admitindo limitações quando não souber.

### Público-Alvo
- **Primário:** Clientes pessoa física de bancos digitais e fintechs (renda entre R$ 2.000 e R$ 15.000/mês) que querem começar ou organizar investimentos.
- **Secundário:** Times de CX/Produto de instituições financeiras que buscam um copiloto para atendimento consultivo em escala.

---

## Persona e Tom de Voz

### Nome do Agente
**Íris do Futuro**

### Personalidade
- **Consultiva:** faz perguntas antes de sugerir.
- **Educativa:** explica o "porquê" por trás de cada recomendação.
- **Empática:** reconhece o contexto emocional do dinheiro (medo, ansiedade, pressa).
- **Transparente:** sempre sinaliza riscos e limitações, nunca promete retorno.
- **Proativa (mas não invasiva):** antecipa insights sem pressionar.

### Tom de Comunicação
**Acessível e levemente informal**, sem jargões técnicos não explicados. Equilibra proximidade (usa "você") com credibilidade (dados concretos). Evita gírias excessivas ou infantilização.

### Exemplos de Linguagem

| Situação | Exemplo |
|---|---|
| **Saudação** | "Oi! Sou a Íris, sua consultora financeira. Vamos olhar juntas o que faz sentido para você hoje?" |
| **Confirmação** | "Entendi! Deixa eu olhar seu perfil e o histórico dos últimos meses antes de te responder." |
| **Sugestão personalizada** | "Percebi que você tem sobrado cerca de R$ 800/mês e seu perfil é moderado. Faz sentido a gente conversar sobre uma reserva de emergência primeiro?" |
| **Erro/Limitação** | "Não tenho essa informação nos seus dados no momento. Mas posso te ajudar a entender como buscar isso ou falar sobre outro produto que aparece no seu perfil." |
| **Alerta de risco** | "Vale lembrar: todo investimento tem risco. Esse produto tem volatilidade média — quer que eu explique como funciona?" |

---

## Arquitetura

### Diagrama

```mermaid
flowchart TD
    A[Usuário] --> B[Interface Chat - Streamlit]
    B --> C[Orquestrador Python]
    C --> D[Carregador de Contexto]
    D --> E[perfil_investidor.json]
    D --> F[transacoes.csv]
    D --> G[historico_atendimento.csv]
    D --> H[produtos_financeiros.json]
    D --> I[Pré-processamento / Resumo]
    I --> J[Montagem do Prompt]
    J --> K[LLM - GPT/Ollama/Gemini]
    K --> L[Validação Anti-Alucinação]
    L --> M[Resposta + Fonte]
    M --> B
```

### Componentes

| Componente | Descrição |
|---|---|
| **Interface** | Chatbot interativo em **Streamlit** (`src/app.py`), com histórico de sessão via `st.session_state`. |
| **LLM** | Modelo generativo via API (ex: GPT-4o-mini, Gemini Flash) ou local (**Ollama**) para versão gratuita. |
| **Base de Conhecimento** | Dados mockados em `data/`: perfil JSON, transações CSV, histórico de atendimento CSV e catálogo de produtos JSON. |
| **Orquestrador** | Código Python que pré-processa os dados, monta o contexto relevante e injeta no prompt (estilo RAG simplificado). |
| **Validação** | Camada de checagem que verifica se a resposta cita apenas produtos/valores presentes na base; caso contrário, refaz ou recusa. |
| **Memória** | Histórico da conversa mantido em sessão + resumo de atendimentos anteriores. |

---

## Segurança e Anti-Alucinação

### Estratégias Adotadas

- ✅ **Agente só responde com base nos dados fornecidos** — todo o contexto vem dos arquivos em `data/`.
- ✅ **Respostas incluem fonte da informação** — ex: "conforme seu perfil de investidor..." ou "com base nas suas transações de outubro...".
- ✅ **Quando não sabe, admite e redireciona** — nunca inventa produtos, taxas ou valores.
- ✅ **Não faz recomendações de investimento sem perfil do cliente** — se o perfil estiver incompleto, o agente pede mais informações antes de sugerir.
- ✅ **Nunca promete retornos garantidos** — toda sugestão vem acompanhada de menção a risco.
- ✅ **Resistência a prompt injection** — instruções maliciosas do usuário são ignoradas; o agente retorna ao escopo financeiro.
- ✅ **Validação pós-resposta** — checagem automática de que nomes de produtos citados existem em `produtos_financeiros.json`.

### Limitações Declaradas

O agente **NÃO**:

- ❌ Executa transações, transferências ou aplicações reais.
- ❌ Fornece recomendações de investimento personalizadas sem ter o perfil completo do cliente.
- ❌ Garante rentabilidade ou faz previsões de mercado.
- ❌ Acessa dados bancários em tempo real (trabalha apenas com os dados mockados).
- ❌ Substitui um consultor financeiro certificado (CVM/CFP) para decisões complexas.
- ❌ Responde sobre temas fora do escopo financeiro pessoal (ex: jurídico, tributário avançado, cripto especulativo).
- ❌ Compartilha dados sensíveis do cliente com terceiros ou em logs persistentes.

