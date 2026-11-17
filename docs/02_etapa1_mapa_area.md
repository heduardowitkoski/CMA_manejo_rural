# Etapa 1 — Mapa, polígono e cálculo de área

**Objetivo:** validar o núcleo do projeto em **uma única tela**, sem banco de dados, sem GPS e sem atividades. Se esta etapa funcionar, o risco técnico acabou.

**Resultado esperado:** o usuário toca no mapa, os pontos viram um polígono e o app mostra a área em hectares.

**Tempo estimado:** 1–2 horas.

---

## Notação dos blocos

Os trechos deste guia são **pseudocódigo**, não a sintaxe exata do App Inventor. A correspondência é:

| No guia | No App Inventor |
|---|---|
| `set x to y` | bloco `set ... to ...` |
| `se ... então ... senão ...` | `if ... then ... else ...` |
| `para cada p em lista` | `for each item in list` |
| `para i de 1 até n` | `for each number from 1 to n` |
| `procedimento ... retorna ...` | `to ... return ...` |
| `local x = ...` | `initialize local x to ...` |
| `chamar ...` | `call ...` |
| `select list item(lista, i)` | bloco `select list item` |
| `make a list(...)` | bloco `make a list` |
| `create empty list` | bloco `create empty list` |
| `join(...)` | bloco `join` |
| `format as decimal(n, 2)` | bloco `format as decimal` |

Os nomes dos **componentes** (`Map1`, `Polygon1`, `global pontos`) são os que você deve usar. Os nomes dos **blocos** podem aparecer traduzidos se o App Inventor estiver em português.

---

## Passo 1 — Criar o projeto

1. Acesse <https://ai2.appinventor.mit.edu>.
2. Clique em **Create Apps** e faça login com uma conta Google.
3. **Projects → Start new project**.
4. Nome sugerido: `ManejoRuralMVP`.
5. Instale o **MIT AI2 Companion** no celular e conecte por QR Code (celular e computador na mesma rede Wi-Fi).

> Dica: teste no celular desde o início. O emulador do App Inventor não representa bem o mapa e o GPS.

---

## Passo 2 — Montar a tela no Designer

### 2.1 Propriedades da Screen1

| Propriedade | Valor |
|---|---|
| `Title` | `Manejo Rural — MVP` |
| `AlignHorizontal` | `Center` |
| `Scrollable` | `false` |
| `Sizing` (se aparecer) | `Responsive` |

### 2.2 Componentes

Adicione na ordem abaixo. Os componentes de mapa (`Polygon`, `LineString`, `Marker`) **não ficam soltos na tela**: arraste cada um para dentro do `Map1` no visualizador. Na árvore de componentes eles aparecem como filhos de `Map1`.

> Se o componente `Map` não aparecer na paleta, selecione `Screen1`, abra **Project Properties → Blocks Toolkit** e escolha **All**. O toolkit “Beginner” esconde componentes de mapa.

| # | Componente | Onde fica | Propriedades principais |
|---|---|---|---|
| 1 | `Map1` | direto na Screen1 | `Width`: Fill parent; `Height`: 55 percent; `CenterFromString`: `-15.793889,-47.882778`; `ZoomLevel`: 16; `MapType`: Roads; `ShowZoom`: true; `ShowUser`: false; `ShowScale`: true |
| 2 | `Polygon1` | dentro de `Map1` | `Visible`: false; `FillColor`: Green; `FillOpacity`: 0.25; `StrokeColor`: Dark Green; `StrokeWidth`: 3; `Title`: `Área de manejo` |
| 3 | `LineString1` | dentro de `Map1` | `Visible`: false; `StrokeColor`: Orange; `StrokeWidth`: 3 |
| 4 | `Marker1` | dentro de `Map1` | `Visible`: false; `FillColor`: Red; `Title`: `Ponto` |
| 5 | `ScrollArrangement1` | direto na Screen1 | `Width`: Fill parent; `Height`: 45 percent |
| 6 | `VerticalArrangement1` | dentro de `ScrollArrangement1` | `Width`: Fill parent; `Height`: Automatic; `AlignHorizontal`: Center |
| 7 | `LabelTitulo` | dentro de `VerticalArrangement1` | `Text`: `1) Toque no mapa para marcar os vértices`; `FontSize`: 16; `FontBold`: true |
| 8 | `LabelPontos` | idem | `Text`: `Pontos: 0` |
| 9 | `LabelArea` | idem | `Text`: `Área: —`; `FontSize`: 18; `FontBold`: true |
| 10 | `HorizontalArrangement1` | idem | `Width`: Fill parent; `AlignHorizontal`: Center |
| 11 | `ButtonDesfazer` | dentro de `HorizontalArrangement1` | `Text`: `Desfazer último` |
| 12 | `ButtonLimpar` | dentro de `HorizontalArrangement1` | `Text`: `Limpar tudo` |
| 13 | `LabelDica` | dentro de `VerticalArrangement1` | `Text`: `Mínimo de 3 pontos. A ordem dos toques define o polígono.`; `FontSize`: 12; `TextColor`: Gray |
| 14 | `Notifier1` | não visual | — |

> **Por que 55% + 45%?** O mapa fica com a maior parte da tela e os controles ficam embaixo, sem precisar rolar. Se o seu celular for pequeno, use 50% + 50%.

### 2.3 Nomes importam

Use exatamente os nomes da tabela (`Map1`, `Polygon1`, `global pontos` etc.). Os blocos deste guia se referem a esses nomes. Se o App Inventor estiver em português, os **blocos** aparecem traduzidos, mas os nomes dos componentes continuam iguais.

---

## Passo 3 — Criar as variáveis e procedimentos

No editor de blocos (**Blocks**), crie:

### 3.1 Variável global

| Bloco | Valor |
|---|---|
| `initialize global pontos to` | `create empty list` |

`global pontos` vai guardar uma lista de listas no formato `[[lat1, lon1], [lat2, lon2], ...]`.

### 3.2 Procedimentos auxiliares

Crie os procedimentos abaixo em **Procedures → to do ... return ...** (para os que retornam valor) ou **to ...** (para os que não retornam).

#### `latitudeDoPonto(p)`

```text
procedimento latitudeDoPonto (p)
    retornar  select list item (p, 1)
```

#### `longitudeDoPonto(p)`

```text
procedimento longitudeDoPonto (p)
    retornar  select list item (p, 2)
```

#### `ultimoPonto()`

```text
procedimento ultimoPonto ()
    retornar  select list item (global pontos, length(global pontos))
```

#### `formatarHa(areaM2)`

```text
procedimento formatarHa (areaM2)
    retornar  format as decimal (areaM2 / 10000, 2)
```

> `format as decimal` devolve, por exemplo, `0.96`. Se quiser vírgula, envolva com `replace all text` trocando `.` por `,`.

#### `calcularAreaM2(listaPontos)`

Este é o coração do app. A fórmula é a **de Gauss / shoelace**, aplicada a coordenadas projetadas localmente:

- latitude vira `y` (metros): `lat * 110540`;
- longitude vira `x` (metros): `lon * 111320 * cos(latitude média)`;
- a área é `|Σ(x_i * y_j − x_j * y_i)| / 2`, fechando o polígono do último ponto para o primeiro.

```text
procedimento calcularAreaM2 (listaPontos)
    local n = length(listaPontos)
    se n < 3 então
        retornar 0

    local somaLat = 0
    para cada p em listaPontos
        set somaLat to somaLat + latitudeDoPonto(p)

    local latMedia = somaLat / n
    local fatorLon = 111320 * cos(latMedia)   // no App Inventor, cos usa GRAUS
    local fatorLat = 110540
    local soma = 0

    para i de 1 até n
        local p1 = select list item (listaPontos, i)
        local j = 0
        se i = n então
            set j to 1
        senão
            set j to i + 1
        local p2 = select list item (listaPontos, j)

        local x1 = longitudeDoPonto(p1) * fatorLon
        local y1 = latitudeDoPonto(p1) * fatorLat
        local x2 = longitudeDoPonto(p2) * fatorLon
        local y2 = latitudeDoPonto(p2) * fatorLat

        set soma to soma + (x1 * y2 - x2 * y1)

    retornar  abs(soma) / 2
```

**Atenção:** no App Inventor, `cos` recebe **graus**, não radianos. Não multiplique por π/180.

> A fórmula de Gauss/shoelace é equivalente a somar as áreas dos triângulos que formam o polígono. É a maneira prática de implementar a “triangulação” pedida no enunciado sem precisar desenhar cada triângulo.

#### `atualizarMapa()`

```text
procedimento atualizarMapa ()
    set LabelPontos.Text to join("Pontos: ", length(global pontos))

    se length(global pontos) >= 3 então
        set Polygon1.Points to global pontos
        set Polygon1.Visible to true
        set LineString1.Visible to false
        set LabelArea.Text to join("Área: ", formatarHa(calcularAreaM2(global pontos)), " ha")
    senão
        set Polygon1.Visible to false

        se length(global pontos) = 2 então
            set LineString1.Points to global pontos
            set LineString1.Visible to true
        senão
            set LineString1.Visible to false

        set LabelArea.Text to "Área: marque pelo menos 3 pontos"

    se length(global pontos) > 0 então
        set Marker1.SetLocation(latitudeDoPonto(ultimoPonto()), longitudeDoPonto(ultimoPonto()))
        set Marker1.Visible to true
    senão
        set Marker1.Visible to false
```

> O `LineString1` serve só para mostrar uma linha quando há 2 pontos. O `Polygon1` só pode ser exibido com 3 ou mais.

---

## Passo 4 — Blocos dos eventos

```text
quando Screen1.Initialize
    set global pontos to create empty list
    chamar atualizarMapa
```

```text
quando Map1.TapAtPoint (latitude, longitude)
    adicionar itens à lista (global pontos)
        item: make a list (latitude, longitude)
    chamar atualizarMapa
```

```text
quando ButtonDesfazer.Click
    se length(global pontos) > 0 então
        remove list item (global pontos, length(global pontos))
        chamar atualizarMapa
```

```text
quando ButtonLimpar.Click
    set global pontos to create empty list
    chamar atualizarMapa
```

Pronto. Não precisa de mais nada nesta etapa.

---

## Passo 5 — Testar

### 5.1 Teste visual

1. Abra o app no celular.
2. Toque em 4 pontos formando um quadrilátero.
3. Devem aparecer:
   - 1 marcador vermelho no último ponto;
   - uma linha laranja enquanto houver 2 pontos;
   - um polígono verde com 3 ou mais pontos;
   - a área em hectares.
4. Toque em **Desfazer último**: o último ponto deve sumir e a área deve ser recalculada.
5. Toque em **Limpar tudo**: o polígono e o marcador devem sumir.

### 5.2 Teste numérico

Use estes quatro pontos (região de Brasília). A ordem importa: A → B → C → D.

| Ponto | Latitude | Longitude |
|---|---|---|
| A | -15.793889 | -47.882778 |
| B | -15.793889 | -47.881878 |
| C | -15.794789 | -47.881878 |
| D | -15.794789 | -47.882778 |

O resultado esperado é aproximadamente **0,96 ha** (cerca de 9.600 m²). O valor exato depende da projeção, mas deve ficar entre 0,94 e 0,98 ha.

> **Teste de sanidade:** um quadrado de 100 m × 100 m tem 10.000 m² = 1 ha. Se o seu resultado der 0, 1 ou 100, algo está errado na fórmula ou na ordem dos pontos.

### 5.3 Teste de ordem

Se você tocar os pontos em “zigue-zague” (A → C → B → D), o polígono se cruza e a área sai errada. Isso é esperado: a fórmula shoelace exige um polígono simples, sem auto-interseção. No MVP, a ordem dos toques é responsabilidade do usuário. No relatório, registre essa limitação.

---

## Problemas comuns nesta etapa

| Sintoma | Causa provável | Solução |
|---|---|---|
| O mapa fica cinza | Sem internet ou tiles não carregaram | Conecte o celular à internet e aguarde |
| O polígono não aparece | `Visible` está false, há menos de 3 pontos ou `Points` não é lista de listas | Confira o `atualizarMapa` e o formato `[lat, lon]` |
| A área dá 0 | Pontos repetidos, todos colineares ou lista vazia | Use pontos diferentes e não alinhados |
| A área dá um valor absurdo | Ordem dos pontos cruzada ou fórmula sem fechar o polígono | Confira o `j = 1` quando `i = n` |
| O toque não adiciona ponto | Evento `TapAtPoint` não conectado ou mapa não inicializado | Reconecte o bloco e teste no celular |
| O app fecha ao abrir | Componente com nome errado ou bloco incompleto | Use **Debug** no Companion e leia a mensagem |

---

## O que você acabou de validar

- O App Inventor consegue capturar coordenadas do mapa.
- O polígono é desenhado corretamente.
- A fórmula de área funciona.
- O fluxo de “marcar → visualizar → calcular” está pronto.

Na próxima etapa, vamos salvar isso no `TinyDB` para não perder os dados ao fechar o app.
