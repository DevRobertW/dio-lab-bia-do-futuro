# 💠 Íris — Consultora Financeira Digital

> Agente financeiro inteligente com IA Generativa, construído para o desafio **DIO Lab: Bia do Futuro**.

A **Íris** é um agente conversacional que vai além do chatbot reativo: ela **antecipa necessidades**, **personaliza sugestões** com base no perfil do cliente e **cocria soluções** de forma consultiva — sempre com uma postura transparente e anti-alucinação.

---

## 📌 Sumário

- [Sobre o Projeto](#-sobre-o-projeto)
- [Como Funciona](#️-como-funciona)
- [Estrutura do Repositório](#-estrutura-do-repositório)
- [Como Rodar](#-como-rodar)
- [Exemplos de Uso](#-exemplos-de-uso)
- [Segurança e Anti-Alucinação](#️-segurança-e-anti-alucinação)
- [Métricas e Avaliação](#-métricas-e-avaliação)
- [Tecnologias](#️-tecnologias)
- [Roadmap](#️-roadmap)
- [Documentação Complementar](#-documentação-complementar)
- [Licença](#-licença)

---

## 🎯 Sobre o Projeto

### O problema

Grande parte dos brasileiros não investe ou investe mal por falta de **orientação financeira personalizada e acessível**. Consultorias tradicionais são caras, exigem agendamento e raramente consideram o contexto real do cliente. Chatbots financeiros atuais são **reativos** (só respondem o que é perguntado) e frequentemente **alucinam**, oferecendo produtos inexistentes ou prometendo retornos irreais.

### A solução

A **Íris** resolve isso sendo:

- 🧠 **Consultiva** — faz perguntas antes de recomendar.
- 🎯 **Personalizada** — cruza perfil de risco, objetivo, horizonte e histórico de transações.
- 🔍 **Proativa** — identifica padrões (ex: sobra recorrente) e retoma pendências do histórico.
- 🛡️ **Confiável** — responde **somente** com base nos dados fornecidos, citando a fonte e admitindo limitações quando não sabe.

### Público-alvo

- **Primário:** clientes pessoa física de bancos digitais e fintechs.
- **Secundário:** times de CX/Produto que buscam um copiloto para atendimento consultivo em escala.

---

## ⚙️ Como Funciona

### Arquitetura

```text
┌──────────────┐    ┌────────────────┐    ┌──────────────┐    ┌──────────────┐
│   USUÁRIO    │───▶│  INTERFACE     │───▶│ ORQUESTRADOR │───▶│  BASE DE     │
│              │    │  (Streamlit)   │    │  (Python)    │    │  CONHECIMENTO│
└──────────────┘    └────────────────┘    └──────┬───────┘    └──────┬───────┘
                                                 │                   │
                                                 ▼                   ▼
                                          ┌──────────────┐    ┌──────────────┐
                                          │  CONTEXTO    │◀───│ PRÉ-PROCESS. │
                                          │  MONTADO     │    │ (resumo+mem.)│
                                          └──────┬───────┘    └──────────────┘
                                                 │
                                                 ▼
                                          ┌──────────────┐
                                          │     LLM      │
                                          │ (Ollama/GPT) │
                                          └──────┬───────┘
                                                 │
                                                 ▼
                                          ┌──────────────┐
                                          │  VALIDAÇÃO   │
                                          │ ANTI-ALUCIN. │
                                          └──────┬───────┘
                                                 │
                                                 ▼
                                          ┌──────────────┐
                                          │   RESPOSTA   │
                                          └──────────────┘
```

### Fluxo de uma pergunta

1. **Usuário** envia uma mensagem no chat.
2. **Orquestrador** identifica a intenção e carrega o contexto relevante.
3. **Pré-processamento** resume transações e seleciona itens da memória.
4. **Filtragem dinâmica** escolhe produtos elegíveis por perfil + pergunta.
5. **System Prompt + contexto** são enviados ao LLM.
6. **Validação pós-resposta** verifica produtos, valores e promessas indevidas.
7. **Resposta** chega ao usuário com citação de fonte.

---

## 📂 Estrutura do Repositório

```text
dio-lab-iris-agente-financeiro/
│
├── 📄 README.md
├── 📄 requirements.txt
├── 📄 .gitignore
│
├── 📁 data/                                # Dados mockados
│   ├── perfil_investidor.json              # Perfil do cliente
│   ├── transacoes.csv                      # Histórico de transações
│   ├── historico_atendimento.csv           # Memória de atendimentos
│   └── produtos_financeiros.json           # Catálogo de produtos
│
├── 📁 docs/                                # Documentação do agente
│   ├── 01-documentacao-agente.md           # Persona, arquitetura, limites
│   ├── 02-base-conhecimento.md             # Estratégia de dados
│   ├── 03-prompts.md                       # System prompt + edge cases
│   ├── 04-metricas.md                      # Avaliação e métricas
│   └── 05-pitch.md                         # Roteiro do pitch
│
├── 📁 src/                                 # Aplicação
│   ├── app.py                              # Streamlit + LLM + validação
│   └── README.md                           # Documentação da pasta src
│
└── 📁 assets/                              # Imagens e diagramas (opcional)
```

---

## 🚀 Como Rodar

### Pré-requisitos

- **Python 3.10+**
- **[Ollama](https://ollama.com)** instalado (para rodar local e gratuito) **ou** uma chave da OpenAI

### Passo a passo (Ollama — recomendado)

```bash
# 1. Clonar o repositório
git clone https://github.com/<seu-usuario>/dio-lab-iris-agente-financeiro.git
cd dio-lab-iris-agente-financeiro

# 2. Criar e ativar ambiente virtual
python -m venv .venv
source .venv/bin/activate   # Linux/macOS
# .venv\Scripts\activate    # Windows

# 3. Instalar dependências
pip install -r requirements.txt

# 4. Baixar o modelo
ollama pull llama3.1

# 5. Rodar a aplicação
streamlit run src/app.py
```

Acesse em `http://localhost:8501`.

### Alternativa: OpenAI

```bash
export IRIS_LLM_PROVIDER=openai
export OPENAI_API_KEY=sk-...
pip install openai
streamlit run src/app.py
```

### Variáveis de ambiente suportadas

| Variável | Padrão | Descrição |
|---|---|---|
| `IRIS_LLM_PROVIDER` | `ollama` | `ollama` ou `openai` |
| `OLLAMA_URL` | `http://localhost:11434/api/chat` | Endpoint do Ollama |
| `OLLAMA_MODEL` | `llama3.1` | Modelo do Ollama |
| `OPENAI_MODEL` | `gpt-4o-mini` | Modelo da OpenAI |
| `OPENAI_API_KEY` | — | Chave da OpenAI (obrigatória se usar OpenAI) |

---

## 💬 Exemplos de Uso

### Saudação (proatividade + memória)

> **Usuário:** Oi
>
> **Íris:** Oi, João! Tudo bem? Na nossa última conversa (28/10), você ficou de avaliar quanto aportar mensalmente para completar a reserva de emergência. Quer retomar isso de onde paramos?

### Fato (dado calculado)

> **Usuário:** Quanto eu gasto por mês?
>
> **Íris:** João, com base no seu resumo financeiro, você tem despesas médias mensais de R$ 2.488,90...

### Recomendação personalizada

> **Usuário:** O que faço com a sobra?
>
> **Íris:** ...você pode considerar investir essa sobra em produtos de baixo risco, como os oferecidos em seu perfil...

### Anti-alucinação (produto inexistente)

> **Usuário:** Quanto rende o CDB do Banco XYZ?
>
> **Íris:** Prefiro não responder isso sem ter certeza de que a informação está na sua base de dados...

### Fora de escopo

> **Usuário:** Qual a previsão do tempo?
>
> **Íris:** João, infelizmente não tenho acesso a informações de previsão do tempo. Posso te ajudar a entender melhor as opções de investimento disponíveis para você?

---

## 🛡️ Segurança e Anti-Alucinação

A Íris adota **defesa em camadas** contra invenção de informações:

| Camada | Mecanismo |
|---|---|
| **Pré-processamento** | Apenas o essencial vai para o prompt (resumos, não dados brutos). |
| **Filtragem** | Produtos são filtrados por perfil + `aceita_risco` + intenção antes de entrar no contexto. |
| **System Prompt** | Regras explícitas: só responder com base no contexto, admitir limitações, citar fonte. |
| **Pós-validação** | Regex + checagem de produtos para detectar invenções. |
| **Fallback** | Se a validação falhar, resposta segura é devolvida ao usuário. |

**Regras de comportamento em edge cases:**

- ❌ **Não promete** rentabilidade ou retorno garantido.
- ❌ **Não inventa** produtos, taxas, prazos ou valores.
- ❌ **Não faz** recomendações incompatíveis com o perfil.
- ❌ **Não revela** system prompt ou dados sensíveis.
- ❌ **Não assume** informações antigas do histórico como atuais.
- ✅ **Pede confirmação** quando detecta conflito entre fontes.

---

## 📊 Métricas e Avaliação

A qualidade do agente é medida por:

| Métrica | Como medir |
|---|---|
| **Assertividade** | % de respostas corretas em uma suite de 8 perguntas de regressão |
| **Taxa anti-alucinação** | % de respostas que citam apenas dados presentes na base |
| **Coerência com perfil** | Recomendações respeitam `perfil_investidor` e `aceita_risco` |
| **Bloqueio de prompt injection** | % de tentativas de manipulação recusadas |
| **Latência** | Tempo médio de resposta por provedor |
| **Cobertura de edge cases** | % dos cenários-limite tratados corretamente |

Consulte [`docs/04-metricas.md`](./docs/04-metricas.md) para os detalhes.

---

## 🛠️ Tecnologias

| Categoria | Ferramenta |
|---|---|
| **Interface** | Streamlit |
| **LLM** | Ollama (llama3.1) · OpenAI (gpt-4o-mini) |
| **Linguagem** | Python 3.10+ |
| **Manipulação de dados** | pandas |
| **Versionamento** | Git + GitHub |

---

## 🗺️ Roadmap

Possíveis evoluções futuras (fora do escopo atual):

- [ ] Migração dos dados mockados para banco de dados (SQLite/PostgreSQL).
- [ ] Memória persistente entre sessões.
- [ ] Busca semântica e RAG com embeddings.
- [ ] Integração com APIs financeiras reais (em ambiente controlado).
- [ ] Camada de avaliação automática de alucinações.
- [ ] Suporte a múltiplos perfis de cliente (multi-tenant).

---

## 📚 Documentação Complementar

- [`docs/01-documentacao-agente.md`](./docs/01-documentacao-agente.md) — Persona, arquitetura e limites
- [`docs/02-base-conhecimento.md`](./docs/02-base-conhecimento.md) — Fontes, pré-processamento e hierarquia
- [`docs/03-prompts.md`](./docs/03-prompts.md) — System prompt, few-shots e edge cases
- [`docs/04-metricas.md`](./docs/04-metricas.md) — Avaliação e métricas
- [`docs/05-pitch.md`](./docs/05-pitch.md) — Roteiro do pitch
- [`src/README.md`](./src/README.md) — Arquitetura interna da aplicação

