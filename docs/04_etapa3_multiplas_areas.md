# Etapa 3 — Várias áreas: lista + editor

**Objetivo:** sair da área única e permitir cadastrar, listar, editar e excluir várias unidades de manejo.

**Resultado esperado:**

- `Screen1`: lista de áreas com nome, hectares e quantidade de atividades.
- `Screen2`: editor da área (mapa, pontos, nome, salvar, excluir).
- Navegação: lista → editor → lista.

**Tempo estimado:** 2–3 horas.

---

## Passo 1 — Reorganizar as telas

No App Inventor, adicione duas telas:

- **Screen2** — título `Editor de área`;
- **Screen3** — título `Atividades` (será usada na Etapa 4).

Depois:

1. Na `Screen1`, selecione os componentes do editor (`Map1`, `Polygon1`, `LineString1`, `Marker1`, `TextBoxNomeArea`, `LabelPontos`, `LabelArea`, botões etc.) e tente **Ctrl+C**.
2. Abra a `Screen2` e use **Ctrl+V**.
3. Se a cópia entre telas não funcionar na sua versão, recrie os componentes usando as tabelas das Etapas 1 e 2. São poucos.

> Os blocos também podem ser copiados entre telas com Ctrl+C/Ctrl+V. Se não funcionar, recrie seguindo este documento.

A `Screen1` deve ficar apenas com a lista. A `Screen2` fica com o mapa e o editor.

---

## Passo 2 — Componentes da Screen1 (lista)

| # | Componente | Propriedades |
|---|---|---|
| 1 | `LabelTitulo` | `Text`: `Minhas áreas de manejo`; `FontSize`: 20; `FontBold`: true |
| 2 | `ButtonNovaArea` | `Text`: `＋ Nova área`; `Width`: Fill parent |
| 3 | `ListViewAreas` | `Width`: Fill parent; `Height`: 70 percent; `TextSize`: 16 |
| 4 | `LabelVazio` | `Text`: `Nenhuma área cadastrada. Toque em "Nova área" para começar.`; `Visible`: true; `FontSize`: 14; `TextColor`: Gray |
| 5 | `TinyDB1` | não visual |
| 6 | `Notifier1` | não visual |

A `Screen1` não precisa de mapa. O mapa fica na `Screen2`.

---

## Passo 3 — Componentes da Screen2 (editor)

Além dos componentes da Etapa 2 (`Map1`, `Polygon1`, `LineString1`, `Marker1`, `TextBoxNomeArea`, `LabelPontos`, `LabelArea`, `ButtonDesfazer`, `ButtonLimpar`, `ButtonSalvar`, `LabelStatus`, `TinyDB1`, `Notifier1`), acrescente:

| Componente | Propriedades |
|---|---|
| `ButtonExcluirArea` | `Text`: `Excluir área`; `Visible`: false; `BackgroundColor`: Red |
| `ButtonVoltar` | `Text`: `Voltar para a lista` |

Coloque `ButtonExcluirArea` e `ButtonVoltar` no final do `VerticalArrangement1`.

> A `Screen2` também precisa de um `TinyDB1` e de um `Notifier1` próprios. Componentes não visuais não são compartilhados entre telas; os **dados** do TinyDB são compartilhados, mas cada tela precisa do seu componente para acessá-los.

---

## Passo 4 — Modelo de dados

Agora o TinyDB guarda uma lista de áreas na tag `areas`:

```text
areas = [
  [ nome, pontos, areaM2, atividades ],
  [ nome, pontos, areaM2, atividades ],
  ...
]
```

E a área selecionada fica na tag `areaIndex`:

- `0` = nova área;
- `1..n` = índice da área na lista `areas`.

> O índice do App Inventor começa em **1**. O `ListView.SelectionIndex` também começa em 1, então ele casa diretamente com a posição da área na lista.

---

## Passo 5 — Blocos da Screen1

### 5.1 Variável global

| Bloco | Valor |
|---|---|
| `initialize global areas to` | `create empty list` |
| `initialize global textos to` | `create empty list` |

### 5.2 Ao abrir

```text
quando Screen1.Initialize
    set global areas to TinyDB1.GetValue("areas", create empty list)
    chamar atualizarLista
```

### 5.3 Atualizar a lista

```text
procedimento atualizarLista ()
    se length(global areas) = 0 então
        set ListViewAreas.Visible to false
        set LabelVazio.Visible to true
    senão
        set ListViewAreas.Visible to true
        set LabelVazio.Visible to false
        set global textos to create empty list

        para cada area em global areas
            adicionar itens à lista (global textos)
                item: join(
                    select list item(area, 1),
                    " — ",
                    formatarHa(select list item(area, 3)),
                    " ha — ",
                    length(select list item(area, 4)),
                    " atividades"
                )

        set ListViewAreas.Elements to global textos
```

> `formatarHa` é o procedimento criado na Etapa 1. Copie-o para a `Screen1` ou recrie.

### 5.4 Nova área

```text
quando ButtonNovaArea.Click
    chamar TinyDB1.StoreValue("areaIndex", 0)
    chamar open another screen ("Screen2")
```

### 5.5 Selecionar uma área

```text
quando ListViewAreas.AfterPicking
    chamar TinyDB1.StoreValue("areaIndex", ListViewAreas.SelectionIndex)
    chamar open another screen ("Screen2")
```

---

## Passo 6 — Blocos da Screen2

> **Antes de começar:** a `Screen2` precisa dos procedimentos e eventos da Etapa 1 que ficaram na tela antiga:
>
> - `latitudeDoPonto`, `longitudeDoPonto`, `ultimoPonto`, `calcularAreaM2`, `formatarHa`, `atualizarMapa`;
> - `Map1.TapAtPoint`, `ButtonDesfazer.Click`, `ButtonLimpar.Click`.
>
> A `Screen1` (lista) também precisa de `formatarHa` para montar o texto da lista. Copie o procedimento para as duas telas ou recrie.

### 6.1 Variáveis globais

| Bloco | Valor |
|---|---|
| `initialize global areas to` | `create empty list` |
| `initialize global areaIndex to` | `0` |
| `initialize global areaAtual to` | `create empty list` |
| `initialize global pontos to` | `create empty list` |

### 6.2 Ao abrir

```text
quando Screen2.Initialize
    set global areas to TinyDB1.GetValue("areas", create empty list)
    set global areaIndex to TinyDB1.GetValue("areaIndex", 0)

    se global areaIndex >= 1 e global areaIndex <= length(global areas) então
        set global areaAtual to select list item(global areas, global areaIndex)
        set TextBoxNomeArea.Text to select list item(global areaAtual, 1)
        set global pontos to select list item(global areaAtual, 2)
        set ButtonExcluirArea.Visible to true
        set LabelStatus.Text to join("Editando: ", select list item(global areaAtual, 1))
    senão
        set global areaIndex to 0
        set global areaAtual to create empty list
        set global pontos to create empty list
        set TextBoxNomeArea.Text to ""
        set ButtonExcluirArea.Visible to false
        set LabelStatus.Text to "Nova área"

    chamar atualizarMapa
```

> A condição `global areaIndex >= 1 e global areaIndex <= length(global areas)` usa o bloco `and`. Se o índice estiver fora da lista (por exemplo, uma área foi excluída), o app trata como nova área em vez de quebrar.

### 6.3 Procedimento `salvarArea`

Este procedimento valida, grava e devolve `true`/`false`. Ele será reutilizado na Etapa 4 pelo botão “Atividades”.

```text
procedimento salvarArea () retorna boolean
    se is empty(TextBoxNomeArea.Text) então
        chamar Notifier1.ShowAlert("Dê um nome para a área.")
        retornar false

    se length(global pontos) < 3 então
        chamar Notifier1.ShowAlert("Marque pelo menos 3 pontos no mapa.")
        retornar false

    local areaM2 = calcularAreaM2(global pontos)
    local atividades = create empty list

    se global areaIndex >= 1 então
        set atividades to select list item(global areaAtual, 4)

    local novoRegistro = make a list
        TextBoxNomeArea.Text
        global pontos
        areaM2
        atividades

    se global areaIndex = 0 então
        adicionar itens à lista (global areas)
            item: novoRegistro
        set global areaIndex to length(global areas)
    senão
        set global areas to replace list item(global areas, global areaIndex, novoRegistro)

    set global areaAtual to novoRegistro
    chamar TinyDB1.StoreValue("areas", global areas)
    chamar TinyDB1.StoreValue("areaIndex", global areaIndex)
    set LabelStatus.Text to join("Salvo! ", formatarHa(areaM2), " ha")
    retornar true
```

**Ponto crítico:** ao editar uma área, o código preserva a lista de atividades (`select list item(global areaAtual, 4)`). Sem isso, salvar a área apagaria todas as atividades dela.

### 6.4 Botões

```text
quando ButtonSalvar.Click
    chamar salvarArea()
```

```text
quando ButtonVoltar.Click
    chamar open another screen ("Screen1")
```

```text
quando ButtonExcluirArea.Click
    chamar Notifier1.ShowChooseDialog(
        "Excluir esta área e todas as atividades?",
        "Confirmar",
        "Excluir",
        "Cancelar",
        true
    )
```

```text
quando Notifier1.AfterChoosing (choice)
    se choice = "Excluir" então
        se global areaIndex >= 1 então
            remove list item (global areas, global areaIndex)
            chamar TinyDB1.StoreValue("areas", global areas)
            chamar TinyDB1.StoreValue("areaIndex", 0)
            chamar open another screen ("Screen1")
```

> O evento `AfterChoosing` do `Notifier` é disparado por qualquer `ShowChooseDialog`. Nesta tela só existe esse, então não há conflito.

---

## Passo 7 — Testar

1. Abra o app: a lista deve mostrar “Nenhuma área cadastrada”.
2. Toque em **Nova área**, marque 3 pontos, dê o nome “Talhão A” e salve.
3. Volte para a lista: deve aparecer `Talhão A — 0.96 ha — 0 atividades`.
4. Crie uma segunda área “Talhão B”.
5. Toque em “Talhão A”: o editor deve abrir com o nome e o polígono carregados.
6. Mude o nome para “Talhão A (editado)” e salve. A lista deve atualizar.
7. Exclua “Talhão B” e confirme. A lista deve ficar só com “Talhão A (editado)”.
8. Feche e reabra o app: a lista deve continuar.

### Problemas comuns

| Sintoma | Causa | Solução |
|---|---|---|
| A lista mostra “Item 1, Item 2” | `Elements` não foi definido | Confira o `atualizarLista` |
| Abre sempre a área errada | `areaIndex` não foi gravado antes de abrir a tela | Confira os blocos de `NovaArea` e `AfterPicking` |
| “Nova área” sobrescreve a existente | `areaIndex` ficou com valor antigo | Garanta `StoreValue("areaIndex", 0)` antes de abrir |
| Ao editar, as atividades somem | O `salvarArea` não preservou a posição 4 | Use o `select list item(global areaAtual, 4)` |
| “Screen not found” | Nome da tela diferente | Use exatamente `Screen2` e `Screen1` |

---

## O que você validou

- CRUD de áreas.
- Navegação entre telas.
- Persistência de uma lista de registros no TinyDB.
- Preservação de dados aninhados (atividades) ao editar.

Na próxima etapa, vamos criar a tela de atividades.
