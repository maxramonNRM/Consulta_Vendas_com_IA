# Consulta_Vendas_com_IA

Agente em Python que converte **perguntas em linguagem natural** em **queries SQL** (text-to-SQL com LLM), executa a consulta em um banco SQLite em **modo somente leitura** e mostra o resultado.

> Projeto de estudo, feito com uma base de vendas 100% fictícia.

## O problema

Responder perguntas sobre uma base de vendas ("qual produto vendeu mais?", "qual o top 3 de produtos em cada UF?") normalmente exige escrever SQL. A ideia aqui é deixar a pessoa perguntar em português e o agente cuidar da query.

## Como funciona

1. A pessoa digita a pergunta (`input()` do Python).
2. A pergunta é enviada ao modelo (API da Anthropic) junto com a descrição da tabela (`SCHEMA`).
3. O modelo devolve **somente o texto de uma query SQL**. Ele não acessa o banco.
4. O Python valida a query (só `SELECT` ou `WITH`) e a executa no SQLite.
5. O banco calcula e devolve as linhas, que são exibidas com Pandas.

Quem faz as contas é o banco, não o modelo. O modelo apenas traduz a pergunta para SQL.

## Tecnologias

Python, Pandas, SQLite, API da Anthropic (`claude-sonnet-5-5`), python-dotenv.

## Base de dados

`vendas_ficticias.csv`: 1.500 vendas fictícias entre 01/01/2025 e 30/09/2026.

| Coluna | Descrição |
|---|---|
| id_venda | Identificador da venda |
| data | Data (YYYY-MM-DD) |
| uf, cidade | Localização da venda |
| categoria, produto | Produto vendido |
| quantidade | Unidades vendidas |
| preco_unitario | Preço por unidade |
| valor_total | quantidade x preco_unitario |
| vendedor | Nome do vendedor |
| canal | Loja física, E-commerce ou Marketplace |

O banco `vendas.db` é criado automaticamente a partir do CSV na primeira execução.

## Como rodar

1. Instale as dependências:

   ```
   pip install pandas anthropic python-dotenv
   ```

2. Crie uma chave de API no console da Anthropic (a API é paga por uso e exige créditos).

3. Crie um arquivo `.env` na pasta do projeto com **uma única linha**, sem aspas e sem espaços:

   ```
   ANTHROPIC_API_KEY=sua-chave-aqui
   ```

   O `.env` está no `.gitignore` e **nunca deve ser enviado ao GitHub**.

4. Abra o notebook e execute a célula do agente. Digite uma pergunta quando o campo `Pergunta:` aparecer (vazio ou `sair` encerra).

## Exemplo

**Pergunta:** qual o produto mais vendido?

**SQL gerado:**

```sql
SELECT produto, SUM(quantidade) AS total_vendido
FROM vendas
GROUP BY produto
ORDER BY total_vendido DESC
LIMIT 1;
```

**Resultado:**

| produto | total_vendido |
|---|---|
| Bola de Futebol | 235 |

Outras perguntas para testar:

- Qual UF teve o maior valor total de vendas?
- Qual a média de valor por venda em cada canal?
- Qual o top 3 de produtos em cada UF por quantidade vendida?

## Segurança

- A conexão com o banco é **somente leitura** (`mode=ro`).
- O código só executa queries que começam com `SELECT` ou `WITH`.
- A chave de API fica no `.env`, fora do versionamento.

## Erros encontrados e como foram resolvidos

| Problema | Causa | Solução |
|---|---|---|
| `Could not resolve authentication method` | O arquivo de chave estava com o nome `Key.env`, e o `load_dotenv()` procura `.env` | Renomear o arquivo para `.env` (ou apontar o nome no `load_dotenv`) |
| Query cortada no meio (`... ORDER B`) e erro de sintaxe | O `max_tokens=300` era baixo e a resposta foi interrompida | Aumentar o limite para 2000 e verificar o `stop_reason` antes de executar |

## Limitações

- O modelo só conhece as colunas descritas no `SCHEMA`; perguntas sobre dados que não estão na tabela podem gerar queries erradas.
- Perguntas ambíguas (por exemplo, "mais vendido" por quantidade ou por valor) podem ser interpretadas de formas diferentes. Conferir a query gerada é importante.
- Não usa RAG: o foco é text-to-SQL sobre dados estruturados.

## Próximos passos

- Enviar o resultado de volta ao modelo para ele responder em frase.
- Testar mais perguntas e documentar em quais o modelo erra.
- Montar uma versão no n8n.
