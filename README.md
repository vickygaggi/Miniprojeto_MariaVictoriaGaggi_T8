# Mini-Projeto: Análise Exploratória de Dados (AED) - Base Varejo

**Aluna:** Maria Victoria Gaggi
**Turma:** T8
**Curso:** Análise de Dados com Python [T8]
**Mini-Projeto Avaliativo:** Módulo 1 - Semana 07

---

## 1. Objetivo

Realizar a leitura, o diagnóstico de qualidade, a limpeza e a análise exploratória dos dados de transações de uma rede de supermercados (base **Varejo**), transformando dados brutos em informações compreensíveis para quem não conhece Python. O projeto reúne leitura de CSV com `pandas`, diagnóstico de qualidade, padronização de texto, funções reutilizáveis, filtro, estatísticas básicas, agrupamentos e exportação da base tratada.

## 2. Origem e versão da base

- **Fonte:** base **Varejo** indicada no enunciado do projeto e disponível no Kaggle: <https://www.kaggle.com/datasets/namespaiva/base-varejo/data>.
- **Dicionário de dados:** Projeto III, FATEC Rubens Lara, Prof. Jobel Corrêa (23/03/2023).
- **Arquivo utilizado:** cópia `Base Varejo.csv` (separador `;`), com **830.000 linhas e 14 colunas brutas** (4 delas vazias).

**Observação sobre a versão:** o dicionário de dados original descreve um recorte de 50.000 compras de 1.000 clientes entre 2010 e 2022. A cópia utilizada neste projeto contém **18.471 compras** (`CO_ID` distintos) de **1.000 clientes**, entre **04/01/2019 e 08/12/2022**. Todas as análises se baseiam exclusivamente nos dados desta cópia.

**O que representa cada linha:** um **item adquirido em uma compra** realizada por um cliente. Por isso, o projeto diferencia *registros* (linhas), *compras* (`CO_ID` distintos) e *clientes* (`CL_ID` distintos).

## 3. Significado das colunas

| Coluna | Significado |
|---|---|
| `DATA` | Data da compra |
| `CO_ID` | Número da compra (número da nota fiscal) |
| `CL_ID` | Número do cliente |
| `CL_GENERO` | Sexo biológico informado pelo cliente |
| `CL_EC` | Estado civil: 1 casado ou união estável, 2 divorciado, 3 separado, 4 solteiro, 5 viúvo |
| `CL_FHL` | Número de filhos do cliente |
| `CL_SEG` | Segmentação econômica do cliente (classe A, B ou C) |
| `PR_ID` | Código do produto (SKU) |
| `PR_CAT` | Categoria do produto |
| `PR_NOME` | Nome do produto |

A base original também traz 4 colunas vazias (`Unnamed: 10` a `Unnamed: 13`), removidas na limpeza.

## 4. Decisões de limpeza

| Problema encontrado | Quantidade | Decisão | Justificativa |
|---|---|---|---|
| Colunas `Unnamed` vazias | 4 colunas (100% nulas) | Removidas | Não possuem informação de negócio |
| Linhas duplicadas | 96.553 (11,63%) | Removidas | Idênticas em todas as colunas (data, compra, cliente e produto) |
| `#N/D` em `PR_CAT` e `PR_NOME` | 3.650 linhas, todas do `PR_ID` 107 (3.228 após remover duplicadas) | Preenchido com `NAO INFORMADO`; linhas **mantidas** | Preserva data, cliente e compra válidos e não inventa uma categoria |
| `REFRIGERANTE LIMaO` (letra minúscula) | 7.320 linhas | Padronizado para `REFRIGERANTE LIMAO` | Mantém o mesmo formato em toda a coluna |
| `DATA` lida como texto | 1 coluna | Convertida para o tipo data | Permite análises por período |
| Valores nulos (`NaN`) nas colunas de negócio | 0 | Nada a tratar | Verificado no diagnóstico |
| Textos vazios ou só com espaços | 0 | Nada a tratar | Verificado no diagnóstico |

**Limitação sobre as duplicadas:** como a base não possui coluna de quantidade, parte das linhas repetidas pode representar o mesmo produto comprado mais de uma vez na mesma compra. Foram removidas apenas linhas idênticas em todas as colunas; compras diferentes do mesmo cliente **não** foram tratadas como duplicadas.

**Comparação antes e depois:**

| Indicador | Antes | Depois |
|---|---|---|
| Registros | 830.000 | 733.447 |
| Colunas | 14 | 10 |
| Valores nulos (`NaN`) | 3.320.000 | 0 |
| Linhas duplicadas | 96.553 | 0 |
| Linhas com `#N/D` | 3.650 | 0 |
| Linhas com `NAO INFORMADO` | 0 | 3.228 |
| Compras distintas | 18.471 | 18.471 |
| Clientes distintos | 1.000 | 1.000 |
| Produtos distintos | 229 | 229 |

## 5. Explicação do processo

A **leitura** dos dados ocorre na função `carregar_base_varejo()` (seção 1.1 do notebook), que lê o CSV com `pd.read_csv()` usando o separador `;`. A **transformação** foi construída passo a passo na Sprint 2 e reunida na função `tratar_dados()` (seção 3.1), que trabalha sobre uma cópia da base e devolve o DataFrame tratado. A **gravação** acontece na seção 5.1, com `to_csv(index=False, sep=';')`, gerando o arquivo `varejo_limpo.csv`.

Uma decisão de limpeza que afetou a qualidade da análise foi **preencher o `#N/D` com `NAO INFORMADO` em vez de remover as linhas**: as 3.228 linhas afetadas continuam contando como itens e compras válidas, e a categoria desconhecida aparece de forma explícita (0,4% dos registros em 2021) em vez de ser escondida. Já a padronização de `REFRIGERANTE LIMaO` evita que um mesmo produto seja tratado de forma diferente nos agrupamentos. A remoção das duplicadas reduziu os registros, mas manteve as 18.471 compras e os 1.000 clientes, o que indica que eram linhas repetidas dentro de compras que continuam existindo.

## 6. Conclusões

1. **Volume de dados e recorte temporal:** o projeto processou 830.000 registros brutos. Após a limpeza, a base tratada ficou com 733.447 registros. O ano de 2021 foi o recorte de maior atividade, com 216.813 itens adquiridos em 5.505 compras distintas por 995 clientes.
2. **Perfil familiar dos clientes:** entre os clientes ativos em 2021, a média é de 1,14 filhos por cliente (`CL_FHL`), com mediana 0 e máximo 4. A maioria (52,5%, ou 522 clientes) não possui filhos. A análise usa o cliente como unidade (uma linha por cliente).
3. **Concentração por categoria de produto:** `ALIMENTOS` lidera o volume de itens em 2021, com 52,5% do total (113.924 registros), seguida por `HIGIENE` (18,8%) e `LIMPEZA` (17,4%).
4. **Engajamento por segmento:** a classe `B` concentra o maior número de clientes (642) e de compras (3.510), mas o engajamento individual é homogêneo entre as classes A, B e C: entre 5,47 e 5,67 compras por cliente em 2021 e cerca de 39 itens por compra. A classe `B` aparece na frente por ter mais clientes, não porque cada cliente compre mais.
5. **Recomendação de negócio:** como `ALIMENTOS` representa mais da metade dos itens vendidos, vale testar ações de exposição cruzada (*cross-selling*) entre alimentos e categorias complementares como limpeza e higiene. Trata-se de uma hipótese a validar, pois a base não permite medir quais produtos são comprados juntos.
6. **Limitação da análise:** a base não possui valores monetários (faturamento) nem quantidades por item, então as métricas se restringem a contagens de registros, compras e clientes.

## 7. Como executar o projeto

**Ambiente:** o notebook foi desenvolvido para o **Google Colab** (usa `google.colab.files` para upload e download de arquivos). Módulos utilizados: `pandas` e `os`.

**Passo a passo:**

1. Abra o notebook `Miniprojeto_MariaVictoriaGaggi_T8.ipynb` no Google Colab (Arquivo → Fazer upload de notebook).
2. **Arquivo de entrada:** a base original está na pasta `data/` do repositório (`data/Base Varejo.csv`). O código procura o arquivo `Base Varejo.csv` no ambiente do Colab; se ele não estiver presente, o notebook abre automaticamente uma janela de upload. Selecione `Base Varejo.csv` (baixado da pasta `data/` do repositório, do Kaggle ou da cópia indicada pela professora).
3. Execute todas as células em ordem: **Ambiente de execução → Executar tudo**.
4. Ao final, o arquivo **`varejo_limpo.csv`** é gerado no ambiente do Colab (painel de arquivos à esquerda) e baixado automaticamente para o computador. A versão tratada entregue neste repositório está em `data/varejo_limpo.csv`.

**Observações sobre o arquivo de saída (`varejo_limpo.csv`):**

- 733.447 registros e 10 colunas, sem a coluna de índice.
- Separador `;` (o mesmo da base original). Para lê-lo: `pd.read_csv('varejo_limpo.csv', sep=';')`.
- A coluna `DATA` é gravada no formato `aaaa-mm-dd`.

## 8. Estrutura do repositório

```
Miniprojeto_MariaVictoriaGaggi_T8/
├── Miniprojeto_MariaVictoriaGaggi_T8.ipynb
├── README.md
└── data/
    ├── Base Varejo.csv
    └── varejo_limpo.csv
```

| Arquivo | Descrição |
|---|---|
| `Miniprojeto_MariaVictoriaGaggi_T8.ipynb` | Notebook com todo o projeto (Sprints 1 a 5), comentado |
| `README.md` | Documentação do projeto |
| `data/Base Varejo.csv` | Base original (entrada): 830.000 linhas e 14 colunas, separador `;` |
| `data/varejo_limpo.csv` | Base tratada (saída): 733.447 linhas e 10 colunas, exportada com `index=False` |
