# Etapa 4 — Atividades da área

**Objetivo:** registrar, listar, editar e excluir atividades dentro de uma área, com custo, receita, avaliação e totais.

**Resultado esperado:**

- `Screen3` mostra o nome da área, o resumo (hectares e quantidade de atividades) e a lista de atividades.
- Formulário para adicionar/editar atividade.
- Totais de custo, receita e saldo.
- Dados salvos no `TinyDB` junto com a área.

**Tempo estimado:** 2–3 horas.

---

## Passo 1 — Componentes da Screen3

| # | Componente | Onde fica | Propriedades |
|---|---|---|---|
| 1 | `LabelAreaNome` | topo da Screen3 | `Text`: vazio; `FontSize`: 20; `FontBold`: true; `TextAlignment`: Center |
| 2 | `LabelResumo` | abaixo | `Text`: vazio; `FontSize`: 14; `TextAlignment`: Center |
| 3 | `LabelTotais` | abaixo | `Text`: vazio; `FontSize`: 13; `TextAlignment`: Center; `TextColor`: Dark Gray |
| 4 | `ListViewAtividades` | abaixo | `Width`: Fill parent; `Height`: 28 percent |
| 5 | `ScrollArrangementForm` | abaixo | `Width`: Fill parent; `Height`: 55 percent |
| 6 | `VerticalArrangementForm` | dentro do scroll | `Width`: Fill parent; `Height`: Automatic |
| 7 | `LabelFormTitulo` | dentro | `Text`: `Nova atividade`; `FontBold`: true |
| 8 | `TextBoxDescricao` | dentro | `Hint`: `Descrição (ex.: adubação, colheita)`; `Width`: Fill parent |
| 9 | `TextBoxHoras` | dentro | `Hint`: `Tempo estimado (horas)`; `NumbersOnly`: true |
| 10 | `TextBoxCusto` | dentro | `Hint`: `Custo operacional (R$)`; `NumbersOnly`: true |
| 11 | `TextBoxReceita` | dentro | `Hint`: `Receita esperada (R$)`; `NumbersOnly`: true |
| 12 | `SpinnerAvaliacao` | dentro | `ElementsFromString`: `Positiva,Neutra,Negativa`; `SelectionIndex`: 2 |
| 13 | `TextBoxObservacoes` | dentro | `Hint`: `Observações`; `MultiLine`: true; `Width`: Fill parent |
| 14 | `HorizontalArrangementAcoes` | dentro | `Width`: Fill parent; `AlignHorizontal`: Center |
| 15 | `ButtonSalvarAtividade` | dentro do arrangement | `Text`: `Salvar atividade` |
| 16 | `ButtonNovaAtividade` | idem | `Text`: `Nova` |
| 17 | `ButtonExcluirAtividade` | idem | `Text`: `Excluir`; `Visible`: false; `BackgroundColor`: Red |
| 18 | `ButtonVoltar` | dentro do form | `Text`: `Voltar para a área` |
| 19 | `LabelStatusAtividade` | dentro do form | `Text`: vazio; `FontSize`: 12; `TextColor`: Blue |
| 20 | `TinyDB1` | não visual | — |
| 21 | `Clock1` | não visual | — |
| 22 | `Notifier1` | não visual | — |

> O `NumbersOnly` do App Inventor aceita números com ponto decimal e sinal negativo. Para o MVP, use valores positivos.

---

## Passo 2 — Modelo de dados da atividade

```text
atividade = [
  descricao,      // texto
  horas,          // número
  custo,          // número
  receita,        // número
  avaliacao,      // 1 = positiva, 0 = neutra, -1 = negativa
  observacoes,    // texto
  data            // texto "dd/MM/yyyy HH:mm"
]
```

A área continua sendo:

```text
area = [ nome, pontos, areaM2, atividades ]
```

Então uma atividade fica em `area[4]`, que é uma lista de listas.

---

## Passo 3 — Variáveis globais da Screen3

| Bloco | Valor |
|---|---|
| `initialize global areas to` | `create empty list` |
| `initialize global areaIndex to` | `0` |
| `initialize global areaAtual to` | `create empty list` |
| `initialize global atividades to` | `create empty list` |
| `initialize global atividadeIndex to` | `0` |
| `initialize global textos to` | `create empty list` |
| `initialize global totalCusto to` | `0` |
| `initialize global totalReceita to` | `0` |

---

## Passo 4 — Procedimentos auxiliares

Copie para a `Screen3` os procedimentos `formatarHa` e `calcularAreaM2` da Etapa 1 (ou recrie). A `Screen3` precisa de `formatarHa` para mostrar a área.

### `numeroOuZero(texto)`

```text
procedimento numeroOuZero (texto) retorna number
    se is empty(texto) então
        retornar 0
    retornar texto + 0
```

> O `+ 0` converte texto numérico em número. Como os campos têm `NumbersOnly`, o usuário não consegue digitar letras.

### `textoAvaliacao(valor)`

```text
procedimento textoAvaliacao (valor) retorna text
    se valor = 1 então
        retornar "positiva"
    senão se valor = -1 então
        retornar "negativa"
    senão
        retornar "neutra"
```

### `valorAvaliacao()`

```text
procedimento valorAvaliacao () retorna number
    se SpinnerAvaliacao.SelectionIndex = 1 então
        retornar 1
    senão se SpinnerAvaliacao.SelectionIndex = 3 então
        retornar -1
    senão
        retornar 0
```

### `indiceAvaliacao(valor)`

```text
procedimento indiceAvaliacao (valor) retorna number
    se valor = 1 então
        retornar 1
    senão se valor = -1 então
        retornar 3
    senão
        retornar 2
```

---

## Passo 5 — Blocos de inicialização

```text
quando Screen3.Initialize
    set global areas to TinyDB1.GetValue("areas", create empty list)
    set global areaIndex to TinyDB1.GetValue("areaIndex", 0)

    se global areaIndex >= 1 e global areaIndex <= length(global areas) então
        set global areaAtual to select list item(global areas, global areaIndex)
        set global atividades to select list item(global areaAtual, 4)
        set LabelAreaNome.Text to select list item(global areaAtual, 1)
        chamar limparFormulario
        chamar atualizarTela
    senão
        chamar Notifier1.ShowAlert("Área inválida. Volte para a lista.")
        chamar open another screen ("Screen1")
```

### `atualizarTela()`

```text
procedimento atualizarTela ()
    set global atividades to select list item(global areaAtual, 4)
    set LabelAreaNome.Text to select list item(global areaAtual, 1)
    set LabelResumo.Text to join(
        formatarHa(select list item(global areaAtual, 3)),
        " ha · ",
        length(global atividades),
        " atividades"
    )
    chamar atualizarListaAtividades
    chamar atualizarTotais
```

### `atualizarListaAtividades()`

```text
procedimento atualizarListaAtividades ()
    set global textos to create empty list

    para cada a em global atividades
        adicionar itens à lista (global textos)
            item: join(
                select list item(a, 1),
                " · ",
                select list item(a, 2),
                " h · R$ ",
                format as decimal(select list item(a, 3), 2),
                " → R$ ",
                format as decimal(select list item(a, 4), 2),
                " · ",
                textoAvaliacao(select list item(a, 5))
            )

    set ListViewAtividades.Elements to global textos
```

### `atualizarTotais()`

```text
procedimento atualizarTotais ()
    set global totalCusto to 0
    set global totalReceita to 0

    para cada a em global atividades
        set global totalCusto to global totalCusto + select list item(a, 3)
        set global totalReceita to global totalReceita + select list item(a, 4)

    set LabelTotais.Text to join(
        "Custo: R$ ", format as decimal(global totalCusto, 2),
        " · Receita: R$ ", format as decimal(global totalReceita, 2),
        " · Saldo: R$ ", format as decimal(global totalReceita - global totalCusto, 2)
    )
```

### `limparFormulario()`

```text
procedimento limparFormulario ()
    set global atividadeIndex to 0
    set TextBoxDescricao.Text to ""
    set TextBoxHoras.Text to ""
    set TextBoxCusto.Text to ""
    set TextBoxReceita.Text to ""
    set SpinnerAvaliacao.SelectionIndex to 2
    set TextBoxObservacoes.Text to ""
    set LabelFormTitulo.Text to "Nova atividade"
    set ButtonExcluirAtividade.Visible to false
    set LabelStatusAtividade.Text to ""
```

### `carregarAtividade(indice)`

```text
procedimento carregarAtividade (indice)
    set global atividadeIndex to indice
    local a = select list item(global atividades, indice)

    set TextBoxDescricao.Text to select list item(a, 1)
    set TextBoxHoras.Text to select list item(a, 2)
    set TextBoxCusto.Text to select list item(a, 3)
    set TextBoxReceita.Text to select list item(a, 4)
    set SpinnerAvaliacao.SelectionIndex to indiceAvaliacao(select list item(a, 5))
    set TextBoxObservacoes.Text to select list item(a, 6)
    set LabelFormTitulo.Text to "Editando atividade"
    set ButtonExcluirAtividade.Visible to true
    set LabelStatusAtividade.Text to ""
```

---

## Passo 6 — Salvar atividade

```text
quando ButtonSalvarAtividade.Click
    se is empty(TextBoxDescricao.Text) então
        chamar Notifier1.ShowAlert("Descreva a atividade.")
    senão
        local novaAtividade = make a list
            TextBoxDescricao.Text
            numeroOuZero(TextBoxHoras.Text)
            numeroOuZero(TextBoxCusto.Text)
            numeroOuZero(TextBoxReceita.Text)
            valorAvaliacao()
            TextBoxObservacoes.Text
            Clock1.FormatDateTime(Clock1.Now, "dd/MM/yyyy HH:mm")

        se global atividadeIndex = 0 então
            adicionar itens à lista (global atividades)
                item: novaAtividade
        senão
            set global atividades to replace list item(global atividades, global atividadeIndex, novaAtividade)

        set global areaAtual to replace list item(global areaAtual, 4, global atividades)
        set global areas to replace list item(global areas, global areaIndex, global areaAtual)
        chamar TinyDB1.StoreValue("areas", global areas)

        set LabelStatusAtividade.Text to "Atividade salva."
        chamar limparFormulario
        chamar atualizarTela
```

**Ponto crítico:** a atividade não é salva “solta”. Ela é salva dentro da área (`posição 4`) e a área é salva dentro da lista `areas`. Se você esquecer um dos `replace list item`, a atividade aparece na tela mas não persiste.

---

## Passo 7 — Selecionar, nova e excluir

```text
quando ListViewAtividades.AfterPicking
    chamar carregarAtividade(ListViewAtividades.SelectionIndex)
```

```text
quando ButtonNovaAtividade.Click
    chamar limparFormulario
```

```text
quando ButtonExcluirAtividade.Click
    se global atividadeIndex >= 1 então
        remove list item (global atividades, global atividadeIndex)
        set global areaAtual to replace list item(global areaAtual, 4, global atividades)
        set global areas to replace list item(global areas, global areaIndex, global areaAtual)
        chamar TinyDB1.StoreValue("areas", global areas)
        chamar limparFormulario
        chamar atualizarTela
        set LabelStatusAtividade.Text to "Atividade excluída."
```

```text
quando ButtonVoltar.Click
    chamar open another screen ("Screen2")
```

---

## Passo 8 — Ligar a Screen2 à Screen3

Na `Screen2`, adicione um botão:

| Componente | Propriedades |
|---|---|
| `ButtonAtividades` | `Text`: `Atividades da área` |

E o bloco:

```text
quando ButtonAtividades.Click
    se chamar salvarArea() então
        chamar open another screen ("Screen3")
```

> O `salvarArea` valida e salva a área antes de abrir as atividades. Se a área for nova, ele cria o registro e grava o `areaIndex`; a `Screen3` consegue encontrá-la.

Se você não quiser salvar automaticamente, use um `ShowChooseDialog` perguntando “Salvar antes de abrir as atividades?”. Para o MVP, salvar direto é mais simples.

---

## Passo 9 — Testar

1. Crie uma área com 3 pontos.
2. Toque em **Atividades da área**.
3. Adicione:
   - “Adubação”, 2 h, custo 500, receita 0, avaliação Positiva, obs. “Uréia”.
   - “Colheita”, 4 h, custo 800, receita 3000, avaliação Positiva.
4. A lista deve mostrar as duas.
5. Os totais devem mostrar:
   - custo: 1300,00;
   - receita: 3000,00;
   - saldo: 1700,00.
6. Toque em “Adubação”: o formulário deve carregar os dados. Mude o custo para 600 e salve. O total de custo deve ir para 1400,00.
7. Exclua “Colheita”: a lista e os totais devem atualizar.
8. Volte para a área, salve, abra as atividades de novo: a atividade “Adubação” deve continuar.
9. Feche e reabra o app: tudo deve continuar.

### Problemas comuns

| Sintoma | Causa | Solução |
|---|---|---|
| A atividade some ao salvar a área | `salvarArea` recriou a lista de atividades vazia | Preserve `select list item(global areaAtual, 4)` |
| A atividade aparece, mas some ao reabrir | Faltou `replace list item` em `areas` ou o `StoreValue` | Confira os três níveis: atividades → área → areas |
| Totais errados | Campos vazios ou texto | Use `numeroOuZero` e `NumbersOnly` |
| Spinner começa na opção errada | `SelectionIndex` não foi ajustado | Use `indiceAvaliacao` |
| Abre a atividade errada | `SelectionIndex` do ListView é 1-based | Use direto, sem somar 1 |
| “Área inválida” | `areaIndex` não foi salvo antes de abrir a Screen3 | Salve a área antes de navegar |

---

## O que você validou

- CRUD completo de atividades.
- Estrutura de listas aninhadas (área → atividades).
- Cálculo de totais.
- Persistência de tudo no TinyDB.

O MVP funcional está pronto. A próxima etapa acrescenta GPS, coordenadas manuais e a primeira sincronização remota.
