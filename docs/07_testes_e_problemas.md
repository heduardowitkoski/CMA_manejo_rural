# Testes, problemas comuns e limitações

## 1. Checklist de testes por etapa

### Etapa 1 — Mapa e área

- [ ] O mapa abre centralizado.
- [ ] Um toque adiciona um ponto (marcador vermelho).
- [ ] Dois pontos mostram uma linha laranja.
- [ ] Três pontos mostram um polígono verde.
- [ ] A área aparece em hectares.
- [ ] Quatro pontos em ordem formam um quadrilátero.
- [ ] “Desfazer último” remove o último ponto e recalcula.
- [ ] “Limpar tudo” remove tudo.
- [ ] O teste de 100 m × 100 m dá aproximadamente 1 ha.

### Etapa 2 — TinyDB

- [ ] Salvar mostra mensagem de sucesso.
- [ ] Fechar e reabrir o app restaura nome, pontos e polígono.
- [ ] “Apagar área” limpa o registro.
- [ ] “Limpar tudo” não apaga o registro salvo (só os pontos da tela).

### Etapa 3 — Múltiplas áreas

- [ ] A lista começa vazia com a mensagem de “Nenhuma área”.
- [ ] Criar duas áreas mostra as duas na lista.
- [ ] Tocar em uma área abre o editor com os dados certos.
- [ ] Editar o nome e salvar atualiza a lista.
- [ ] Excluir uma área remove só ela.
- [ ] Fechar e reabrir mantém a lista.
- [ ] Editar uma área não apaga as atividades dela.

### Etapa 4 — Atividades

- [ ] Adicionar atividade mostra na lista.
- [ ] Os totais de custo, receita e saldo batem.
- [ ] Tocar em uma atividade carrega o formulário.
- [ ] Editar e salvar atualiza a lista e os totais.
- [ ] Excluir remove e recalcula.
- [ ] Voltar para a área, salvar e reabrir as atividades mantém tudo.
- [ ] Fechar e reabrir o app mantém tudo.

### Etapa 5 — GPS e nuvem

- [ ] O GPS adiciona um ponto.
- [ ] O app avisa quando a precisão está baixa.
- [ ] Coordenadas manuais inválidas são rejeitadas.
- [ ] Sem internet, o app continua funcionando localmente.
- [ ] Com internet, o CloudDB envia e recebe.
- [ ] O `userID` é criado uma vez.

---

## 2. Problemas comuns no App Inventor

### 2.1 “Screen not found”

**Causa:** o nome passado em `open another screen` não é igual ao nome da tela.

**Solução:** confira na lista de telas do Designer. Se a tela se chama `Screen2`, use exatamente `Screen2` (maiúsculas e sem espaços).

### 2.2 “Attempt to get item number X of a list of length Y”

**Causa:** índice fora da lista. Acontece ao carregar uma área que foi excluída, ao acessar `item 4` de uma lista vazia ou ao usar `SelectionIndex` 0.

**Solução:**

- valide `areaIndex >= 1 e areaIndex <= length(global areas)`;
- valide `length(global pontos) > 0` antes de acessar o último ponto;
- use `ListView.SelectionIndex` só depois do `AfterPicking`.

### 2.3 “Expected a number but got text”

**Causa:** `TextBox.Text` é texto, mesmo quando contém “10”.

**Solução:** use `NumbersOnly` nos campos numéricos e converta com `+ 0` (por exemplo, `TextBoxCusto.Text + 0`). Para campos opcionais, use `numeroOuZero`.

### 2.4 O polígono não aparece

Verifique, nesta ordem:

1. `length(global pontos) >= 3`;
2. `Polygon1.Visible` está `true`;
3. `Polygon1.Points` recebeu uma **lista de listas** `[[lat, lon], ...]`, não uma lista de textos;
4. os pontos não estão todos no mesmo lugar;
5. o mapa está com zoom na região dos pontos.

### 2.5 A área dá 0

- Todos os pontos são iguais?
- Todos estão alinhados (colineares)?
- A lista tem menos de 3 pontos?
- O polígono se auto-intersecta? (a fórmula pode dar 0 ou um valor estranho)

### 2.6 A área dá um valor muito grande ou muito pequeno

- Confira se a latitude está em graus decimais (ex.: `-15.79`), não em graus/minutos/segundos.
- Confira se longitude e latitude não foram trocadas.
- Confira se a fórmula está fechando o polígono (`j = 1` quando `i = n`).
- Lembre: 1 ha = 10.000 m².

### 2.7 O mapa fica cinza

O `Map` nativo baixa tiles da internet. Sem conexão, o fundo pode ficar cinza, mesmo que o polígono e os dados continuem funcionando. Para mapa offline de verdade, seria necessário usar uma extensão (OSMDroid) ou tiles pré-baixados.

### 2.8 TinyDB não persiste

- A tag está igual nos dois lugares?
- Você está gravando uma lista com `make a list`, não um texto?
- O app foi realmente reiniciado? No Companion, “Connect” de novo não reinicia o processo.
- Você limpou os dados do app no Android? Isso apaga o TinyDB.

> **Atenção no Companion:** todos os apps em desenvolvimento com o mesmo Companion compartilham o mesmo TinyDB. Se você abrir outro projeto, pode ver dados “fantasma”. Limpe os dados do Companion ao trocar de projeto.

### 2.9 A lista não atualiza

O `ListView` não se atualiza sozinho quando a lista global muda. Você precisa **atribuir novamente** `ListView.Elements` (ou chamar `atualizarLista`) depois de qualquer alteração.

### 2.10 CloudDB não sincroniza

- O componente está com `RedisServer = DEFAULT` e `UseSSL = true`?
- O `ProjectID` está preenchido?
- O celular está com internet?
- Você está usando a mesma tag nos dois dispositivos?
- O `CloudDB` não é instantâneo: use o evento `GotValue` para saber quando o valor chegou.

### 2.11 GPS retorna 0, 0

- A permissão de localização foi concedida?
- O GPS do celular está ligado?
- Você está em local aberto?
- O `LocationSensor.Enabled` está `true`?
- Aguarde o evento `LocationChanged` ou alguns segundos após abrir a tela.

### 2.12 O app fecha sozinho

- Algum bloco está acessando item de lista vazia.
- Algum campo de texto vazio foi usado em cálculo.
- Algum componente foi renomeado e o bloco antigo ficou “órfão”.
- Use o evento `Screen.ErrorOccurred` para mostrar a mensagem:

```text
quando Screen1.ErrorOccurred (component, functionName, errorNumber, message)
    chamar Notifier1.ShowAlert(join(functionName, ": ", message))
```

---

## 3. Dicas de depuração

1. **Mostre os valores na tela.** Crie um `LabelDebug` e faça:

   ```text
   set LabelDebug.Text to join("pontos=", length(global pontos), " área=", calcularAreaM2(global pontos))
   ```

2. **Use `Notifier.ShowAlert`** para confirmar se um evento disparou.

3. **Teste a fórmula isoladamente.** Antes de ligar o mapa, crie uma lista fixa com 4 pontos e chame `calcularAreaM2` no `Screen.Initialize`. Se der 0,96 ha, a fórmula está certa.

4. **Use o “Do It”** do App Inventor (clique com o botão direito em um bloco, com o Companion conectado) para executar um bloco e ver o resultado.

5. **Teste no celular.** Emulador não tem GPS real e pode não renderizar o mapa como o aparelho.

6. **Gere o APK** em **Build → Android App (.apk)** para testar a instalação e o comportamento offline de verdade.

---

## 4. Limitações do MVP (para o relatório)

| Limitação | Impacto | Trabalho futuro |
|---|---|---|
| Mapa depende de tiles online | Fundo cinza offline | Extensão OSMDroid ou cache de tiles |
| Área por projeção local | Erro pequeno para áreas rurais; cresce em áreas enormes ou em latitudes altas | Fórmula esférica ou biblioteca geodésica |
| Sem validação de auto-interseção | Polígono “borboleta” dá área errada | Algoritmo de interseção de segmentos |
| Edição de vértice limitada | Usuário remove e adiciona de novo | Markers arrastáveis |
| Um usuário por dispositivo | Sem login | Autenticação |
| TinyDB local | Não compartilha entre dispositivos | Backend/Firestore |
| CloudDB last-write-wins | Conflitos em edição concorrente | Versionamento e merge |
| Sem fila de sync completa | Pode perder sincronização se fechar offline | Fila persistente de operações |
| Sem paginação | Listas grandes ficam lentas | Paginação/índices |

---

## 5. Definição de pronto do MVP

Considere o MVP pronto quando:

- [ ] é possível criar, listar, editar e excluir áreas;
- [ ] a área é calculada e exibida corretamente;
- [ ] o polígono é visualizado no mapa;
- [ ] é possível adicionar, editar e excluir atividades;
- [ ] custo, receita e saldo são calculados;
- [ ] os dados sobrevivem ao fechamento do app;
- [ ] o app funciona sem internet para os dados locais;
- [ ] o fluxo principal foi testado em um celular real.
