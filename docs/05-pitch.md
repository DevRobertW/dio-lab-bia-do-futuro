# Pitch — Versão Escrita

> Este documento substitui a gravação em vídeo do pitch por uma **apresentação escrita em formato de artigo**. A estrutura é a mesma (problema, solução, diferenciais, impacto), mas adaptada para leitura.

## 💠 Íris: consultoria financeira com IA que não inventa informação

### O problema

Mais de 60% dos brasileiros adultos não investem — não por falta de dinheiro, mas por falta de orientação personalizada. Consultorias tradicionais cobram caro e exigem agendamento. Chatbots de banco só respondem o que é perguntado, e quando vão além, **inventam**: oferecem produtos inexistentes, prometem retornos irreais. No setor financeiro, alucinação não é bug — é risco regulatório.

### A solução: Íris

A Íris é uma **consultora financeira digital com IA Generativa** que vai além do chatbot reativo. Ela:

- **Antecipa necessidades** — retoma pendências reais do histórico do cliente.
- **Personaliza sugestões** — respeita perfil de risco, objetivo e horizonte.
- **Cocria soluções** — faz perguntas antes de recomendar.
- **Não inventa** — responde apenas com base em dados estruturados e admite quando não sabe.

### Como funciona

A arquitetura separa **dados, processamento e linguagem**:

1. Dados vêm de arquivos JSON/CSV (perfil, transações, histórico, produtos)
2. Um orquestrador Python pré-processa, filtra e monta o contexto
3. O LLM (llama3.1 via Ollama) interpreta e conversa
4. Uma camada de validação pós-resposta bloqueia alucinações

O LLM **não é a fonte de verdade** — ele é a camada de comunicação. Toda informação factual vem da base estruturada.

### Diferenciais

**1. Separação entre fato, inferência e recomendação**
A Íris nunca apresenta um padrão estatístico como certeza. Ela diz *"os dados indicam"* — não *"você sempre sobra X"*.

**2. Filtragem por `aceita_risco`**
Se o perfil diz "moderado" mas `aceita_risco: false`, ela só sugere produtos de baixo risco — sem exceção.

**3. Validação anti-alucinação pós-LLM**
Depois que o modelo responde, uma camada verifica se os produtos citados existem, se os valores estão na base, se há promessas de retorno garantido. Se falhar, a resposta é bloqueada.

### Resultados

Testei a Íris em 8 cenários críticos:
- Proatividade e retomada de pendência
- Fato, inferência e recomendação
- Produto inexistente e promessa de retorno
- Fora de escopo e prompt injection

**Resultado: 7,5 de 8 aprovados — 100% em anti-alucinação e robustez a prompt injection.**

### Impacto

A Íris roda **100% local** com modelos abertos como o llama3.1 — sem custo de API, sem enviar dados sensíveis para fora. Também escala para GPT-4 ou Gemini quando a instituição quiser.

O impacto é claro: **democratizar a consultoria financeira personalizada** para quem hoje não tem acesso — com **segurança, transparência e sem invenção de informação**.

A Íris não substitui um consultor humano. Ela **amplifica** o alcance dele — atendendo milhares de clientes ao mesmo tempo com a mesma qualidade de atenção.

Esse é o futuro do atendimento financeiro: **proativo, personalizado e confiável**.

---

## 🎬 Roteiro Original (para gravação futura)

> O restante deste documento contém o roteiro original do pitch falado, para referência e uso futuro caso o autor decida gravar.
