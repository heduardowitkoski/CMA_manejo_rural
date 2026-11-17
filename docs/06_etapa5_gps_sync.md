# Etapa 5 — GPS, coordenadas manuais, nuvem e otimizações

Esta etapa não é obrigatória para validar o MVP. Ela cobre os requisitos que faltam para chegar mais perto do enunciado: GPS, entrada manual, sincronização e eficiência.

**Ordem recomendada:**

1. GPS;
2. coordenadas manuais;
3. sincronização simples com CloudDB;
4. otimizações de bateria/rede;
5. anotações de escalabilidade para o relatório.

---

## 1. GPS

### 1.1 Componente

Na `Screen2`, adicione:

| Componente | Propriedades |
|---|---|
| `LocationSensor1` | `Enabled`: true; `TimeInterval`: 60000; `DistanceInterval`: 10 |

> Se quiser mostrar o ponto azul do usuário no mapa, ative também `Map1.ShowUser`. Isso pode pedir permissão de localização ao abrir a tela.

`TimeInterval` de 60 s e `DistanceInterval` de 10 m evitam consumo excessivo de bateria. Para marcar um ponto, o app usa a última posição conhecida; não é necessário ficar atualizando a cada segundo.

### 1.2 Botão

Adicione `ButtonGPS` ao `HorizontalArrangement1` da `Screen2`:

| Componente | Propriedades |
|---|---|
| `ButtonGPS` | `Text`: `Usar meu GPS` |

### 1.3 Blocos

```text
quando ButtonGPS.Click
    se LocationSensor1.HasLongitudeLatitude então
        se LocationSensor1.Accuracy > 50 então
            chamar Notifier1.ShowAlert(
                join("Precisão baixa (±", round(LocationSensor1.Accuracy), " m). Aguarde alguns segundos e tente de novo.")
            )
        senão
            adicionar itens à lista (global pontos)
                item: make a list (LocationSensor1.Latitude, LocationSensor1.Longitude)
            chamar atualizarMapa
            chamar Map1.PanTo(LocationSensor1.Latitude, LocationSensor1.Longitude, 18)
    senão
        chamar Notifier1.ShowAlert("Ainda não há posição. Vá para um local aberto e aguarde o GPS.")
```

### 1.4 Permissão

Na primeira vez que o `LocationSensor` for usado, o Android pede permissão de localização. Se o usuário negar, o sensor não funciona. Para tratar isso:

```text
quando Screen2.PermissionDenied (component, functionName, permissionName)
    se permissionName = "android.permission.ACCESS_FINE_LOCATION" então
        chamar Notifier1.ShowAlert("Sem permissão de localização, use o mapa ou as coordenadas manuais.")
```

> O App Inventor já solicita a permissão automaticamente quando o componente é usado. O evento acima serve para explicar ao usuário.

### 1.5 Limitação

O GPS do celular pode ter erro de 5–20 m em campo aberto e muito mais sob árvores ou nuvens. Para uma área pequena, isso distorce o polígono. No relatório, registre que o MVP usa a precisão disponível e que uma versão futura poderia fazer média de várias leituras (filtro de Kalman simples ou média de N amostras).

---

## 2. Coordenadas manuais

Útil para testar no emulador e para o produtor que tem as coordenadas anotadas.

### 2.1 Componentes

Na `Screen2`, adicione ao `VerticalArrangement1`:

| Componente | Propriedades |
|---|---|
| `TextBoxLat` | `Hint`: `Latitude (ex.: -15.793889)`; `NumbersOnly`: true |
| `TextBoxLon` | `Hint`: `Longitude (ex.: -47.882778)`; `NumbersOnly`: true |
| `ButtonAdicionarManual` | `Text`: `Adicionar coordenada` |

> `NumbersOnly` aceita ponto decimal e sinal negativo, então serve para latitudes/longitudes.

### 2.2 Blocos

```text
quando ButtonAdicionarManual.Click
    se is empty(TextBoxLat.Text) ou is empty(TextBoxLon.Text) então
        chamar Notifier1.ShowAlert("Preencha latitude e longitude.")
    senão
        local lat = TextBoxLat.Text + 0
        local lon = TextBoxLon.Text + 0

        se lat < -90 ou lat > 90 então
            chamar Notifier1.ShowAlert("Latitude deve estar entre -90 e 90.")
        senão se lon < -180 ou lon > 180 então
            chamar Notifier1.ShowAlert("Longitude deve estar entre -180 e 180.")
        senão
            adicionar itens à lista (global pontos)
                item: make a list (lat, lon)
            chamar atualizarMapa
            chamar Map1.PanTo(lat, lon, 18)
            set TextBoxLat.Text to ""
            set TextBoxLon.Text to ""
```

> O `+ 0` converte o texto em número. Como o campo é `NumbersOnly`, não há risco de letras.

---

## 3. Edição das coordenadas

O MVP já permite:

- **Desfazer último** — remove o último ponto;
- **Limpar tudo** — recomeça;
- **Adicionar** — toque, GPS ou manual.

Para editar um ponto específico, a solução mais simples é remover e adicionar de novo. Edição vértice a vértice (arrastar o ponto no mapa) exigiria:

- criar um `Marker` por ponto;
- tratar o evento `Drag` de cada marker;
- atualizar a lista `pontos` na posição correspondente;
- redesenhar o polígono.

Isso é possível no App Inventor, mas aumenta bastante a complexidade. Deixe para a versão “avançada” e registre como trabalho futuro.

---

## 4. Sincronização com CloudDB

### 4.1 O que o CloudDB faz

O `CloudDB` é um armazenamento chave-valor remoto (Redis). Ele permite compartilhar dados entre dispositivos. É a opção mais rápida no App Inventor, mas **não é um banco relacional** e não tem consultas, índices nem controle de concorrência sofisticado.

### 4.2 Estratégia mínima para o protótipo

1. Cada instalação gera um identificador local (`userID`) e guarda no `TinyDB`.
2. Ao salvar uma área, o app grava primeiro no `TinyDB` e depois no `CloudDB` com a tag `areas_<userID>`.
3. Ao abrir a lista, o app lê o `TinyDB` e pede o valor remoto. Quando o `GotValue` chegar, substitui ou mescla.

Componentes:

| Componente | Onde |
|---|---|
| `CloudDB1` | `Screen1` |

> O próprio `CloudDB` tem o método `CloudConnected()`, que retorna `true` quando há rede e o servidor parece acessível. Use esse método em vez de procurar um componente separado de conectividade.

### 4.3 Gerar o `userID`

Na `Screen1.Initialize`:

```text
se is empty(TinyDB1.GetValue("userID", "")) então
    chamar TinyDB1.StoreValue("userID", join("produtor_", random integer(100000, 999999)))
```

> Em um app real, o `userID` viria do login. Para o protótipo, um código aleatório evita que dois dispositivos sobrescrevam a mesma tag.

### 4.4 Enviar

Depois de `TinyDB1.StoreValue("areas", global areas)` no `salvarArea` da `Screen2`, acrescente:

```text
se CloudDB1.CloudConnected() então
    chamar CloudDB1.StoreValue(join("areas_", TinyDB1.GetValue("userID", "demo")), global areas)
```

> A `Screen2` também precisa de um `CloudDB1`, ou você pode centralizar a sincronização na `Screen1`. Cada tela tem seus próprios componentes não visuais.

### 4.5 Receber

Na `Screen1.Initialize`:

```text
chamar CloudDB1.GetValue(join("areas_", TinyDB1.GetValue("userID", "demo")), create empty list)
```

```text
quando CloudDB1.GotValue (tag, value)
    se not is empty(value) então
        set global areas to value
        chamar TinyDB1.StoreValue("areas", global areas)
        chamar atualizarLista
```

### 4.6 Limitações que você deve registrar

- **Última escrita vence.** Se dois dispositivos editarem a mesma área, um sobrescreve o outro.
- **Lista inteira por tag.** O app envia todas as áreas de uma vez; para muitas áreas, isso fica pesado.
- **Sem autenticação real.** O `userID` aleatório não é segurança.
- **Sem sincronização offline de verdade.** O código acima só envia quando há conexão; não guarda uma fila de operações.
- **CloudDB não é adequado para escala global.** Para o relatório, proponha Firebase Firestore ou um backend próprio com Postgres e API REST.

### 4.7 Fila de sincronização (opcional)

Para uma versão um pouco melhor:

1. Ao salvar, grave `sync_pendente = true` no `TinyDB`.
2. Quando `CloudDB1.CloudConnected()` for verdadeiro, envie e marque `false`.
3. Se o envio falhar (`CloudDBError`), mantenha `true` e tente de novo na próxima abertura.

Isso já demonstra o conceito de “offline first” sem construir um sistema de fila completo.

---

## 5. Eficiência de bateria, armazenamento e rede

Coloque estes pontos no relatório; alguns já estão implementados, outros são recomendações.

| Recurso | Decisão no MVP | Por quê |
|---|---|---|
| GPS | `TimeInterval` 60 s, `DistanceInterval` 10 m, só na tela do editor | Evita GPS contínuo |
| Mapa | Zoom 16–18, tiles online | Menos tiles baixados |
| TinyDB | Grava só ao salvar | Evita I/O a cada toque |
| CloudDB | Sincroniza ao salvar/abrir, não a cada tecla | Menos rede e bateria |
| Dados | Listas pequenas por área | Simples e suficiente para o MVP |
| Processamento | Cálculo de área só quando os pontos mudam | O(n) com n pequeno |

Recomendações para a versão final:

- ativar o GPS só quando o usuário tocar em “Usar meu GPS”;
- agrupar sincronizações (batch) a cada N minutos;
- comprimir ou enviar apenas deltas;
- usar Wi-Fi para sincronizar volumes maiores;
- evitar manter a tela ligada desnecessariamente;
- usar `CloudDB.CloudConnected()` para não tentar sincronizar offline.

---

## 6. Escalabilidade (para o relatório)

O MVP é local e de usuário único. Para atender “grande quantidade de usuários dispersos geograficamente”, a arquitetura precisaria de:

1. **Autenticação** (login) para identificar o produtor.
2. **Backend com API REST** ou Firebase Firestore.
3. **Banco relacional/geográfico** (PostgreSQL + PostGIS) para consultas por região.
4. **Versionamento por registro** (`atualizadoEm`, `versao`) para resolver conflitos.
5. **Sincronização incremental** (enviar só o que mudou).
6. **Paginação** e índices para listas grandes.
7. **Validação no servidor** (não confiar só no app).
8. **CDN/cache de tiles** para o mapa.

No App Inventor, o componente `Web` consegue chamar APIs REST e ler JSON, mas a lógica de sincronização fica trabalhosa. Para a entrega, o mais honesto é apresentar o MVP com TinyDB/CloudDB e justificar que a arquitetura de produção usaria um backend dedicado.

---

## 7. Checklist desta etapa

- [ ] GPS adiciona um ponto com um toque.
- [ ] O app avisa quando a precisão está baixa.
- [ ] Coordenadas manuais validam os limites.
- [ ] O `userID` é criado uma vez e reutilizado.
- [ ] Salvar com internet envia para o CloudDB.
- [ ] Abrir o app com internet recebe os dados remotos.
- [ ] Sem internet, o app continua funcionando com o TinyDB.
- [ ] O relatório explica as limitações de sincronização e escala.
