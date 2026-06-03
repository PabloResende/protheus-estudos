# Nível 1 — Fundamentos ERP e Protheus

## O que é ERP?

**ERP** significa **Enterprise Resource Planning** — sistema responsável por centralizar todos os processos de uma empresa em um só lugar, em uma só base de dados, em tempo real.

Isso garante que não haja duplicações, inconsistências ou atrasos no fluxo de trabalho.

**Sem ERP:**
- Financeiro usa Excel
- Estoque usa sistema próprio
- Vendas usa outro sistema
- → Dados inconsistentes, retrabalho, erro humano

**Com ERP:**
- Tudo no mesmo sistema
- Uma venda já baixa o estoque e gera o financeiro automaticamente
- → Dados integrados, rastreabilidade, controle total

---

## O que é TOTVS Protheus?

É o **ERP mais usado no Brasil**. Domina principalmente:

- 🌱 Agronegócio
- 🏭 Indústria
- 🛒 Varejo
- 🏗️ Construção Civil
- 🧾 Serviços

> **+40% do mercado brasileiro de ERP médio/grande**
> Milhares de empresas precisando de analistas — escassez de profissionais qualificados.

---

## Como o Protheus é estruturado

```text
Protheus
├── Módulos
│   ├── SIGACOM  → Compras
│   ├── SIGAFAT  → Faturamento/Vendas
│   ├── SIGAEST  → Estoque
│   ├── SIGAFIN  → Financeiro
│   ├── SIGACONT → Contabilidade
│   ├── SIGAGPE  → RH/Folha
│   └── SIGAAGR  → Agronegócio
│
├── Linguagem       → ADVPL
├── Banco de Dados  → SQL Server (principal), Oracle, PostgreSQL
└── Servidor        → Protheus Application Server (AppServer)
```

---

## Como funciona por dentro

```text
[Usuário - SmartClient]
          ↓
[AppServer - executa ADVPL]
          ↓
[DBAccess - camada de banco]
          ↓
[SQL Server - dados]
```

> O **SmartClient** funciona como um navegador — o usuário acessa por ele, mas todo o processamento acontece no servidor.

---

## O conceito mais importante — Filial (Branch)

Toda empresa no Protheus tem **pelo menos uma filial**. Todos os dados são separados por filial.

```text
Empresa: 99
├── Filial 01 → Matriz São Paulo
├── Filial 02 → Filial Goiânia
└── Filial 03 → Filial Cuiabá
```

Isso aparece em **todas** as tabelas do banco:

```sql
-- Campos presentes em todas as tabelas
D_E_L_E_T_   -- ' ' = ativo | '*' = deletado logicamente
R_E_C_N_O_   -- número único do registro (como um ID interno)
R_E_C_D_E_L_ -- controle de deleção interno

-- Campo de filial (exemplo na SA1)
A1_FILIAL    -- filial do cliente
```

> ⚠️ **Atenção:** No Protheus **nunca se deleta um registro de verdade**.
> O registro é marcado com `*` no campo `D_E_L_E_T_`.
> Todo SELECT precisa filtrar esse campo — sempre.

---

## Fluxo real de uma venda no Protheus

```text
Cliente faz pedido
        ↓
Pedido de Venda (SC5/SC6) ───── SIGAFAT
        ↓
Aprovação de crédito ─────────── SIGAFIN
        ↓
Separação no estoque ─────────── SIGAEST
        ↓
Emissão NF-e (SF2/SD2) ──────── SIGAFAT
        ↓
Contas a Receber (SE1) ──────── SIGAFIN
        ↓
Baixa do estoque (SD3) ──────── SIGAEST
        ↓
Lançamento contábil (CT2) ───── SIGACONT
```

> Cada etapa desse fluxo corresponde a uma **tabela diferente no SQL Server**.
> Customizar o Protheus significa manipular esse fluxo.