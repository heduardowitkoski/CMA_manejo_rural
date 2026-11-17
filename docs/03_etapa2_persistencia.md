# Etapa 2 — Salvar e carregar com TinyDB

**Objetivo:** fazer a área sobreviver ao fechamento do app. Ainda será **uma única área**, para simplificar.

**Resultado esperado:** o usuário marca os pontos, dá um nome, salva, fecha o app e, ao abrir de novo, a área e o polígono voltam.

**Tempo estimado:** 30–60 minutos.

---

## Passo 1 — Novos componentes

Mantenha tudo da Etapa 1 e acrescente:

| # | Componente | Onde fica | Propriedades |
|---|---|---|---|
| 1 | `TextBoxNomeArea` | dentro de `VerticalArrangement1`, antes do `LabelTitulo` | `Hint`: `Nome da área (ex.: Talhão Norte)`; `Width`: Fill parent |
| 2 | `ButtonSalvar` | dentro de `HorizontalArrangement1` | `Text`: `Salvar área` |
| 3 | `ButtonApagar` | dentro de `HorizontalArrangement1` | `Text`: `Apagar área` |
| 4 | `LabelStatus` | dentro de `VerticalArrangement1`, no final | `Text`: vazio; `FontSize`: 12; `TextColor`: Blue |
| 5 | `TinyDB1` | não visual | — |

O `TinyDB` é um banco local chave-valor. Ele continua existindo depois que o app fecha, mas é **por dispositivo**: não sincroniza entre celulares.

---

## Passo 2 — Formato do registro

Nesta etapa, vamos guardar **uma área** na tag `area_atual`:

```text
area_atual = [ nome, pontos, areaM2, atividades ]

exemplo:
[
  "Talhão Norte",
  [[-15.793889, -47.882778], [-15.793889, -47.881878], [-15.794789, -47.881878]],
  9591.9,
  []
]
```

A quarta posição (`atividades`) já existe, vazia, para facilitar a Etapa 4. Ainda não vamos usá-la.

---

## Passo 3 — Blocos

### 3.1 Ao abrir a tela

Substitua o `Screen1.Initialize` da Etapa 1 por:

```text
quando Screen1.Initialize
    set global areaSalva to TinyDB1.GetValue("area_atual", create empty list)

    se not is empty(global areaSalva) então
        set TextBoxNomeArea.Text to select list item(global areaSalva, 1)
        set global pontos to select list item(global areaSalva, 2)
        set LabelStatus.Text to join("Área carregada: ", formatarHa(select list item(global areaSalva, 3)), " ha")
    senão
        set global pontos to create empty list

    chamar atualizarMapa
```

Crie também a variável global:

| Bloco | Valor |
|---|---|
| `initialize global areaSalva to` | `create empty list` |

> `is empty` funciona para listas e textos. Para negar, use o bloco `not` (logic negate).

### 3.2 Salvar

```text
quando ButtonSalvar.Click
    se is empty(TextBoxNomeArea.Text) então
        chamar Notifier1.ShowAlert("Dê um nome para a área.")
    senão se length(global pontos) < 3 então
        chamar Notifier1.ShowAlert("Marque pelo menos 3 pontos no mapa.")
    senão
        set global areaSalva to make a list
            TextBoxNomeArea.Text
            global pontos
            calcularAreaM2(global pontos)
            create empty list
        chamar TinyDB1.StoreValue("area_atual", global areaSalva)
        set LabelStatus.Text to join("Salvo! Área: ", formatarHa(calcularAreaM2(global pontos)), " ha")
```

### 3.3 Apagar

```text
quando ButtonApagar.Click
    chamar TinyDB1.StoreValue("area_atual", create empty list)
    set TextBoxNomeArea.Text to ""
    set global pontos to create empty list
    set LabelStatus.Text to "Área apagada do armazenamento."
    chamar atualizarMapa
```

### 3.4 Limpar (só os pontos)

O `ButtonLimpar` continua como antes: limpa apenas `global pontos`. Se o usuário limpar e não salvar, o registro antigo permanece no `TinyDB`. Isso é intencional: “limpar” é diferente de “apagar”.

---

## Passo 4 — Testar

1. Marque 3 ou 4 pontos.
2. Digite um nome.
3. Toque em **Salvar área**. O `LabelStatus` deve mostrar a área.
4. Feche o app de verdade:
   - no Companion, use **Stop** ou feche o app pelo Android;
   - se estiver usando APK, feche pelos recentes.
5. Abra de novo. O nome, os pontos e o polígono devem voltar.
6. Toque em **Apagar área** e reabra: tudo deve estar vazio.

### Se não carregar

| Verificação | O que fazer |
|---|---|
| A tag está igual? | Use exatamente `area_atual` nos dois blocos |
| O `StoreValue` está gravando uma lista? | Use `make a list`, não `join` |
| O app foi realmente reiniciado? | No Companion, “Connect” de novo não reinicia; use Stop |
| Os dados do app foram limpos? | Limpar dados do app no Android apaga o TinyDB |

---

## O que você validou

- Persistência local (offline) funcionando.
- O formato de dados que será usado nas próximas etapas.
- Que o app não perde a área ao fechar.

Na próxima etapa, vamos sair da área única e criar a **lista de áreas** com edição.
