
Projeto público: https://entranaregua.lovable.app/entrar

## Como foi construído

Este projeto foi desenvolvido com **vibe coding**, usando o
[Lovable](https://lovable.dev) para geração do app e o **Claude
(Anthropic)** como engenheiro de prompt — refinando o escopo, as
regras de design e o comportamento do agente conversacional antes
de cada geração.

### Prompt utilizado

```markdown
Crie o MVP de um aplicativo de organização financeira pessoal
que funciona por conversa (chat), seguindo as especificações
detalhadas abaixo.

# Prompt — MVP de App de Finanças por Conversa (versão completa)

## 1. Persona e voz
Nome do app: [defina algo próprio — evite "FinChat", "MoneyBot",
"MoneyBuddy" e variações óbvias]
Fala como um amigo organizado que entende de dinheiro — não como
um robô de suporte, nem como um consultor formal de banco.

Exemplos do tom certo:
- "Anotado! Mercado, R$45."
- "Você já gastou R$620 esse mês — um pouco acima da sua média."
- "Antes de eu calcular isso, preciso saber mais uma coisa:"

Exemplos do tom errado (evitar):
- "Transação registrada com sucesso. ID: #4521."
- "Processando solicitação..."
- "Nós entendemos que finanças podem ser desafiadoras! 😊"

## 2. Regra de design — não seja genérico
Não use:
- fundo creme (#F4F1EA) com acento terracota (#D97757)
- cards com mesmo raio de borda e mesma sombra cinza em tudo
- rótulos em CAIXA ALTA espaçada acima de cada seção
- "→" no fim de botões
- gradientes decorativos sem função
- numeração 01/02/03 em conteúdo que não é sequência
- card branco centralizado com gradiente de fundo em telas de login

Antes de gerar qualquer tela, defina e documente:
- Paleta de 4 a 6 cores nomeadas (ex: "verde-saldo", "vermelho-alerta",
  "tinta-principal") — não cores genéricas tipo "primary/secondary"
- Uma direção visual concreta e um motivo para ela (ex: "extrato
  bancário redesenhado" → linhas horizontais finas, números
  tabulares, hierarquia por peso de fonte, não por cor)
- Tipografia: 1 fonte para números/dados (com números tabulares) e
  1 para texto corrido — ambas com peso e escala intencionais
- Motion: animação só em resposta a uma ação do usuário (nunca
  fade-in automático ao carregar a tela)

## 3. Modelo de dados (alto nível)
- **Transação**: valor, tipo (entrada/saída), categoria, data,
  descrição original (texto que o usuário digitou), status
  (confirmada/pendente de correção)
- **Categoria**: nome, tipo (receita/despesa), regra de
  classificação (palavras-chave associadas)
- **Meta**: nome, valor-alvo, prazo, valor já economizado/pago,
  tipo de origem (criada manualmente / gerada a partir de um plano
  do agente), status (ativa/concluída)
- **Plano de quitação/financiamento**: valor da dívida, taxa de
  juros (informada ou estimada), parcelas restantes, valor sugerido
  de pagamento extra, meta vinculada
- **Usuário**: e-mail, dados de autenticação, data de criação

## 4. Arquitetura conversacional — registro de entradas/saídas
Tela principal = chat. Fluxo:
1. Extrair valor, categoria provável e data do texto livre
2. Confirmar de forma breve e editável
   ("Anotado: Mercado, R$45, Alimentação — corrigir?")
3. Permitir correção por texto natural
   ("na verdade foi transporte" → atualiza categoria sem reabrir formulário)

### Casos-limite a tratar
- Valor sem "R$" ou "reais" explícito ("gastei 45 no mercado") →
  assumir moeda padrão do usuário
- Frases com múltiplas transações na mesma mensagem
  ("gastei 30 no uber e 20 no lanche") → dividir em duas transações
- Ausência de categoria clara → perguntar, nunca salvar como
  "Outros" silenciosamente
- Valores ambíguos ("gastei uns 40 e pouco") → registrar como
  aproximado e sinalizar visualmente na lista de lançamentos

## 5. Registro preciso de metas financeiras
- Extrair 3 campos obrigatórios antes de salvar: nome da meta,
  valor-alvo, prazo (se não informado, perguntar)
- Confirmar de forma breve antes de salvar
  ("Meta: Quitar dívida do cartão, alvo R$X, até [data] — confirma?")
- Nunca salvar meta com campo faltando — perguntar o que falta
  em vez de assumir um valor padrão
- Permitir editar meta existente por conversa
  ("mudei de ideia, quero economizar 500 em vez de 300")

## 6. Entendimento de contexto amplo (agente financeiro inteligente)
O assistente deve reconhecer intenções amplas, não só transações
isoladas — cobrindo dívida de cartão, financiamento e outras
intenções financeiras futuras, com o mesmo padrão de raciocínio.

### 6.1 Detectar o tipo de intenção
- Transação simples (entrada/saída)
- Dívida de cartão de crédito
- Financiamento (veículo, imóvel, outros bens)
- Meta de economia/investimento
- Dúvida financeira geral (sem ação imediata — ex: "isso é caro?")

### 6.2 Perguntas de acompanhamento por categoria
Perguntar uma de cada vez, só o necessário para calcular:

**Dívida de cartão**
- Qual o valor da dívida?
- Há quanto tempo está pagando?
- Já parcelou ou está no rotativo?

**Financiamento**
- Qual o valor financiado (ou saldo devedor atual)?
- Qual a taxa de juros do contrato, se souber?
- Quantas parcelas faltam e qual o valor de cada uma?
- Quer quitar antecipado, renegociar ou só entender o impacto atual?

### 6.3 Exemplo de diálogo completo
```
Usuário: quero quitar a dívida do meu cartão
App: Vamos montar um plano. Qual o valor total da dívida hoje?
Usuário: uns 3000
App: Entendi. Você já parcelou ou está no rotativo?
Usuário:
