# Nível 3 — ADVPL

## O que é ADVPL?

**ADVPL** — Advanced Protheus Language — é a linguagem de programação do Protheus. É baseada em **xBase**, a mesma família do Clipper e dBASE dos anos 80/90, o que explica muitas das escolhas que parecem estranhas pra quem vem do PHP.

```text
PHP                          ADVPL
tipagem dinâmica      →      tipagem dinâmica
funções               →      funções
arrays                →      arrays
orientação a objetos  →      orientação a objetos (limitada)
ponto e vírgula       →      sem ponto e vírgula
```

---

## 1. Variáveis

```clipper
// PHP
$nome  = "João";
$idade = 30;

// ADVPL
Local cNome  := "João"
Local nIdade := 30
```

Sem `$`. Sem ponto e vírgula. Com **prefixo de tipo** — convenção obrigatória no mercado:

```text
c  →  character (string)    cNome, cCodigo
n  →  numeric               nValor, nQtd
l  →  logical (booleano)    lAtivo, lOk
d  →  date                  dVencto, dEmissao
a  →  array                 aItens, aLista
o  →  objeto                oCliente, oConn
u  →  undefined/qualquer    uRetorno
```

---

## 2. Escopo de variáveis

```clipper
Local    // existe só dentro da função         → use sempre este
Private  // existe na função e nas que ela chamar
Public   // existe em todo o programa          → evite
Static   // persiste entre chamadas da função
```

---

## 3. Estrutura de uma função

```clipper
// PHP
function somaDois($n) {
    return $n + 2;
}

// ADVPL
Function SomaDois(nNumero)
    Local nResultado := 0
    nResultado := nNumero + 2
Return nResultado
```

> Sem chaves `{}`. Bloco termina com `Return`. Convenção de nome é **PascalCase**.

---

## 4. If, For, While

```clipper
// IF
If nIdade >= 18
    MsgInfo("Maior de idade")
Else
    MsgInfo("Menor de idade")
EndIf

// FOR
Local nI := 0
For nI := 1 To 10
    ConOut(cValToChar(nI))
Next nI

// WHILE
Local nI := 0
While nI < 10
    nI++
EndDo
```

---

## 5. Strings — o que muda do PHP

```clipper
// Concatenação — usa + (não ponto como no PHP)
Local cNome := "João" + " " + "Silva"

// Trim — muito usado, campos do banco vêm com espaços
Local cCod := AllTrim(A1_COD)

// Tamanho
Local nTam := Len(cNome)

// Substring
Local cTrecho := SubStr(cNome, 1, 4)  // "João"

// Maiúsculo / minúsculo
Upper(cNome)
Lower(cNome)
```

> ⚠️ **Atenção:** Campos no banco Protheus são `CHAR` com tamanho fixo. `A1_COD` tem sempre
> 6 caracteres — se o código for "001", o banco armazena "001   " com espaços no final.
> Sem `AllTrim()`, a comparação `A1_COD == "001"` retorna falso porque está comparando
> `"001   "` com `"001"`. **Sempre use `AllTrim()` ao comparar campos.**

---

## 6. Arrays

```clipper
// Criar
Local aLista := {}
Local aFixo  := {"João", "Maria", "Pedro"}

// Adicionar
aAdd(aLista, "João")
aAdd(aLista, "Maria")

// Acessar — índice começa em 1, não em 0
cNome := aLista[1]   // "João"

// Tamanho
nTam := Len(aLista)

// Loop
Local nI := 0
For nI := 1 To Len(aLista)
    ConOut(aLista[nI])
Next nI
```

> ⚠️ **Índice começa em 1**, não em 0. Erro clássico de quem vem do PHP.

---

## 7. Funções úteis do dia a dia

```clipper
ConOut("texto")          // imprime no console
MsgInfo("texto")         // popup de informação
MsgAlert("texto")        // popup de alerta
Alert("texto")           // popup simples

cValToChar(nValor)       // número → string
Val("123")               // string → número
DToC(dData)              // data   → string  "31/12/2024"
CToD("31/12/2024")       // string → data
Date()                   // data de hoje
Time()                   // hora atual

AllTrim(cStr)            // remove espaços dos dois lados
Upper(cStr)              // maiúsculo
Lower(cStr)              // minúsculo
Len(cStr)                // tamanho
```

---

## Exemplo completo

```clipper
Function OlaProtheus()
    Local cMensagem := ""
    Local nIdade    := 0

    cMensagem := "Olá, mundo ERP!"
    nIdade    := 25

    If nIdade >= 18
        MsgInfo(cMensagem + " - Maior de idade")
    Else
        MsgInfo(cMensagem + " - Menor de idade")
    EndIf

Return .T.
```

---

## Exercícios

### 1. Função com If/Else

```clipper
Function Apresentacao(cNome, nIdade)
    Local cFala := ""

    If nIdade >= 18
        cFala := cNome + " é maior de idade"
    Else
        cFala := cNome + " é menor de idade"
    EndIf

Return cFala

// Chamada
ConOut(Apresentacao("Pablo", 23))
```

### 2. Array com loop

```clipper
Function ListaProdutos(aLista)
    Local nI := 0

    If Len(aLista) == 0
        ConOut("Lista vazia")
        Return .F.
    EndIf

    For nI := 1 To Len(aLista)
        ConOut("Produto " + cValToChar(nI) + ": " + aLista[nI])
    Next nI

Return .T.

Function Teste()
    Local aProdutos := {"Soja", "Milho", "Trigo"}
    ListaProdutos(aProdutos)
Return .T.
```