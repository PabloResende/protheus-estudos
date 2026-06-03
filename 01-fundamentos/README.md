O que é ERP? 

ERP significa Enterprise Resource Planning - responsável por centralizar todos os ambientes de uma empresa em um só lugar, em uma só base de dados, em tempo real, isso garante que não tenham duplicações, ou quaisquer atrasos no fluxo de trabalho da empresa.

Sem ERP o financeiro usuaria o Excel, o estoque usuaria um sistema próprio, vendas outro, etc.

O que é TOTVS Protheus? 

É o ERP mais usado do Brasil, principalmente no agronegócio, varejo, construção civil, indústria, etc.

Como o Protheus é construído?

Protheus
├── Módulos
│   ├── SIGACOM → Compras
│   ├── SIGAFAT → Faturamento/Vendas
│   ├── SIGAEST → Estoque
│   ├── SIGAFIN → Financeiro
│   ├── SIGACONT → Contabilidade
│   ├── SIGAGPE → RH/Folha
│   └── SIGAAGR → Agronegócio
│
├── Linguagem → ADVPL
├── Banco de Dados → SQL Server (principal), Oracle, PostgreSQL
└── Servidor → Protheus Application Server (AppServer)

Como funciona por dentro:

[Usuário - SmartClient]
        ↓
[AppServer - executa ADVPL]
        ↓
[DBAccess - camada de banco]
        ↓
[SQL Server - dados]

O SmartCliente é como um navegador para o usuário, onde ele o acessa e todo o processamento é feito no servidor.

O conceito mais importante do Protheus - Filial(Branch)
 
Toda empresa do Protheus tem pelo menos uma filial, todos os dados são separados por elas. Exemplo:

Empresa: 99
├── Filial 01 → Matriz São Paulo
├── Filial 02 → Filial Goiânia
└── Filial 03 → Filial Cuiabá

Isso aparece em todas as tabelas do banco:

-- Toda tabela tem esses campos
D_E_L_E_T_  -- registro deletado logicamente
R_E_C_N_O_  -- número do registro
R_E_C_D_E_L_-- controle interno

-- E os campos de empresa/filial
SA1.A1_FILIAL -- filial do cliente

Obs: diferente de um PHP da vida, aqui se marca como deletado, ao invés de realmente deletar.Todo SELECT no Protheus tem filtro implícito no D_E_L_E_T_. Guarda isso.

Exemplo de fluxo de venda com o Protheus:
Cliente faz pedido
        ↓
Pedido de Venda (SC5/SC6) → SIGAFAT
        ↓
Aprovação de crédito → SIGAFIN
        ↓
Separação no estoque → SIGAEST
        ↓
Emissão NF-e (SF2/SD2) → SIGAFAT
        ↓
Contas a Receber (SE1) → SIGAFIN
        ↓
Baixa do estoque (SD3) → SIGAEST
        ↓
Lançamento contábil (CT2) → SIGACONT

Cada seta dessa é uma tabela diferente no SQL Server. Quando você customiza o Protheus, está manipulando esse fluxo.


