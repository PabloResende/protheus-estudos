# Nível 2 — Modelo de Dados do Protheus

## A lógica de nomenclatura das tabelas

```text
S  A  1
│  │  └── Número sequencial da tabela
│  └───── Letra do GRUPO (módulo/categoria)
└──────── S = Standard (padrão TOTVS)
```

## O que cada letra de grupo significa

A letra do meio no nome da tabela define o assunto:

```text
A  →  Cadastros gerais     (clientes, fornecedores)
B  →  Estoque / Produtos   (produtos, saldos, armazéns)
C  →  Pedidos              (pedido de venda, pedido de compra)
D  →  Documentos           (itens de NF, movimentações)
E  →  Financeiro           (contas a receber, contas a pagar)
F  →  Fiscal / NF          (cabeçalho de NF entrada e saída)
G  →  Estrutura / MRP      (estrutura de produto)
H  →  Rastreabilidade      (rastro de lotes)
I  →  Patrimônio
J  →  Contabilidade auxiliar
K  →  Ordens de produção
L  →  Livros fiscais
M  →  Usuários e acessos
N  →  Contratos
X  →  Dicionário de dados  (campos, parâmetros, gatilhos)
```

Então quando ver uma tabela desconhecida, a letra do meio já te diz o assunto:
- `SK5` → algo relacionado a ordens de produção
- `SN1` → algo relacionado a contratos
- `SE3` → algo relacionado a financeiro

---

## O dicionário SX — coração do Protheus

```text
SX1  → Perguntas de parâmetros
SX2  → Empresas
SX3  → Campos (define estrutura de todas as tabelas)
SX5  → Tabelas genéricas (listas de valores)
SX6  → Parâmetros do sistema (MV_xxxx)
SX7  → Gatilhos entre campos
```

> O **SX3** define todos os campos de todas as tabelas.
> Quando um analista cria um campo customizado, ele registra no SX3.

> O **SX6** guarda os parâmetros — tudo no Protheus é controlado por parâmetro.

```sql
-- Ver parâmetros do sistema
SELECT X6_VAR, X6_DESCRI, X6_CONTEUD
FROM SX6010
WHERE D_E_L_E_T_ = ' '
AND X6_VAR LIKE 'MV_%'
```

---

## A lógica dos campos

```text
A  1  _  C O D
│  │     └──── Nome do campo
│  └────────── Número da tabela
└──────────── Letra do módulo
```

```sql
-- Tabela SA1 (Clientes)
A1_FILIAL    -- Filial
A1_COD       -- Código do cliente
A1_NOME      -- Nome
A1_CGC       -- CNPJ
A1_END       -- Endereço
A1_TEL       -- Telefone
A1_EMAIL     -- E-mail

-- Tabela SB1 (Produtos)
B1_FILIAL    -- Filial
B1_COD       -- Código do produto
B1_DESC      -- Descrição
B1_UM        -- Unidade de medida
B1_TIPO      -- Tipo do produto
B1_LOCPAD    -- Armazém padrão

-- Tabela SE1 (Contas a Receber)
E1_FILIAL    -- Filial
E1_NUM       -- Número do título
E1_CLIENTE   -- Código do cliente
E1_VALOR     -- Valor
E1_VENCTO    -- Vencimento
E1_SALDO     -- Saldo em aberto
E1_BAIXA     -- Data da baixa
```

---

## Campos presentes em TODAS as tabelas

```sql
D_E_L_E_T_   -- ' ' = ativo | '*' = deletado logicamente
R_E_C_N_O_   -- número único do registro (como um ID interno)
R_E_C_D_E_L_ -- controle de deleção interno
```

```sql
-- ❌ ERRADO — traz registros deletados
SELECT * FROM SA1010

-- ✅ CERTO
SELECT * FROM SA1010
WHERE D_E_L_E_T_ = ' '
AND A1_FILIAL = '01'
```

---

## Por que SA1010 e não SA1?

```text
S A 1 0 1 0
│ │ │ │ │ │
│ │ │ │ │ └── sufixo fixo (sempre 0)
│ │ │ │ └──── número da empresa (01, 02, 99...)
│ │ │ └────── sufixo fixo (sempre 0)
│ │ └──────── número da tabela
│ └────────── grupo/módulo
└──────────── Standard
```

| Contexto       | Nome usado  |
|----------------|-------------|
| ADVPL / Docs   | SA1         |
| SQL Server     | SA1010      |

> Empresa 99 teria `SA1990`, `SB1990`, etc.

---

## Principais tabelas por módulo

| Tabela  | Nome lógico                  | Módulo    |
|---------|------------------------------|-----------|
| SA1010  | Cadastro de Clientes         | SIGAFAT   |
| SA2010  | Cadastro de Fornecedores     | SIGACOM   |
| SB1010  | Cadastro de Produtos         | SIGAEST   |
| SB2010  | Saldo em Estoque             | SIGAEST   |
| SC5010  | Cabeçalho Pedido de Venda    | SIGAFAT   |
| SC6010  | Itens do Pedido de Venda     | SIGAFAT   |
| SD3010  | Movimentações de Estoque     | SIGAEST   |
| SE1010  | Contas a Receber             | SIGAFIN   |
| SE2010  | Contas a Pagar               | SIGAFIN   |
| SF2010  | Cabeçalho NF Saída           | SIGAFAT   |

---

## Por que não existem Foreign Keys no Protheus?

O Protheus foi criado nos anos 90, quando performance era prioridade absoluta.
Foreign Keys custam processamento — validação a cada insert/update.

Com milhões de registros e múltiplas filiais, isso inviabilizaria o sistema na época.

**O que isso implica na prática:**
- Você pode ter `C5_CLIENTE` apontando para um cliente que não existe
- Você pode ter registros órfãos no banco
- Todo relacionamento precisa ser validado pela aplicação ou pela sua query
- Todo JOIN precisa do filtro de filial e delete lógico manualmente

---

## Relacionamentos na prática

```sql
-- ✅ JOIN correto no Protheus — Pedido de Venda + Cliente
SELECT
    C5_FILIAL,
    C5_NUM,
    C5_CLIENTE,  -- chave estrangeira manual
    A1_NOME,
    C5_VALOR
FROM SC5010
INNER JOIN SA1010
    ON SA1010.A1_COD    = SC5010.C5_CLIENTE  -- relacionamento manual
    AND SA1010.A1_FILIAL = SC5010.C5_FILIAL   -- sempre filtrar filial no JOIN
    AND SA1010.D_E_L_E_T_ = ' '              -- sempre filtrar delete no JOIN
WHERE SC5010.D_E_L_E_T_ = ' '
AND SC5010.C5_FILIAL = '01'
```

> ⚠️ **Regra de ouro:** Em todo JOIN no Protheus, sempre inclua filial e delete lógico
> na condição ON — não só no WHERE.

---

## Tabelas mais usadas no Agronegócio

```text
SB1  → Produtos (sementes, defensivos, fertilizantes)
SB2  → Saldo em estoque por armazém
SD3  → Movimentações (entrada/saída de insumos)
SC5  → Pedidos de venda (grãos, commodities)
SE1  → Contas a receber (safra)
SE2  → Contas a pagar (fornecedores de insumo)
SA1  → Clientes (cooperativas, tradings)
SA2  → Fornecedores (distribuidores de insumo)
```