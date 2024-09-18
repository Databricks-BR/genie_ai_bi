




## Exercício 03.00 - Preparação

1. Conecte este notebook ao seu SQL Warehouse
2. Preencha o parâmetro no topo da página com o nome do database que será utilizado neste laborátio
3. Execute a célula abaixo para ativar este database

***Caso não tenha feito ainda, carregue os dados conforme descrito no [Lab 01 - Importando os dados](https://github.com/Databricks-BR/genie_ai_bi/blob/main/01_LAB_importando_dados/README.md)***


## Exercício 03.01 - Criar AI/BI Genie

Vamos começar criando uma Genie para fazer nossas perguntas. Para isso, vamos seguir os passos abaixo:

1. No menu principal (à esquerda), clique em `New` > `Genie space`

<img src="https://raw.githubusercontent.com/Databricks-BR/genie_ai_bi/main/images/genie_01.png">

2. Configure sua Genie
    - Crie um nome para a sua Genie, por exemplo `<suas iniciais> Genie de Vendas`
    - Selecione seu SQL Warehouse
    - Selecione as seguintes tabelas:
        - vendas
        - estoque
        - dim_medicamento
        - dim_loja
    - Clique em `Save`

<img src="https://raw.githubusercontent.com/Databricks-BR/genie_ai_bi/main/images/genie_02.png" width=800>


## Exercício 03.02 - Fazer perguntas ao AI/BI Genie

* Qual o faturamento em out/22?
* Agora, quebre por produto
* Mantenha somente os 10 produtos com maior faturamento
* Monte um gráfico de barras


## Exercício 03.03 - Fazer perguntas avançadas

* Qual o total de produtos vendidos em genéricos?
* Qual o valor total vendido de ansiolíticos?
* Quais produtos tiveram uma proporção de vendas por estoque maior que 0.8 em Outubro de 2022?


## Exercício 03.04 - Usando comentários

Faça a pergunta:
* Qual o valor total de venda por loja? Exiba o nome da loja
* Use a célula abaixo para adicionar um comentário na tabela `dim_loja`
* Faça novamente a pergunta anterior


ALTER TABLE dim_loja ALTER COLUMN nlj COMMENT 'Nome da loja'


## Exercício 03.05 - Usando chaves primárias

* Use a célula abaixo para adicionar as chaves primárias e estrangeiras nas tabelas `dim_loja` e `vendas`
* Faça novamente a pergunta anterior



ALTER TABLE dim_loja ADD CONSTRAINT pk_dim_loja PRIMARY KEY (cod);
ALTER TABLE vendas ADD CONSTRAINT fk_venda_dim_loja FOREIGN KEY (id_loja) REFERENCES dim_loja(cod);


## Exercício 03.06 - Usando instruções

Faça a pergunta:
* Calcule a quantidade de itens vendidos para prescrição

Adicione a instrução:
  - `* para calcular indicadores sobre prescrição use categoria_regulatoria <> 'GENÉRICO'`
Faça novamente a pergunta anterior

## Exercício 03.07 - Usando exemplos de queries

- Faça a pergunta:
  - Calcule a quantidade de itens vendidos por janela móvel de 3 meses 
- Adicione o exemplo de query abaixo:
  - `SELECT window.end AS dt_venda, SUM(vl_venda) FROM vendas GROUP BY WINDOW(dt_venda, '90 days', '1 day') `
- Faça novamente a pergunta anterior

## Exercício 03.08 - Usando funções

- Faça a pergunta:
  - Qual o lucro projetado do AAS?
- Crie a função da célula abaixo
- Adicione esta função a sua Genie
- Faça novamente a pergunta anterior





CREATE OR REPLACE FUNCTION calc_lucro(medicamento STRING)
  RETURNS TABLE(nome_medicamento STRING, lucro_projetado DOUBLE)
  COMMENT 'Use esta função para calcular o lucro projetado de um medicamento'
  RETURN 
    SELECT
      m.nome_medicamento,
      sum(case when m.categoria_regulatoria == 'GENÉRICO' then 1 else 0.5 end * v.vl_venda) / sum(v.qt_venda) as lucro_projetado
    FROM vendas v
    LEFT JOIN dim_medicamento m
    ON v.id_produto = m.id_produto
    WHERE m.nome_medicamento = calc_lucro.medicamento
    GROUP BY ALL    

  
