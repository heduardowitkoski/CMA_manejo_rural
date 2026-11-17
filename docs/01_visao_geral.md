# Visão geral e escopo do MVP

## 1. O que o projeto pede

O sistema deve permitir que um produtor rural:

- defina uma **unidade de manejo** a partir de 3 ou mais coordenadas;
- informe as coordenadas de três formas: manualmente, por GPS ou tocando no mapa;
- visualize o polígono da área em um mapa;
- calcule a área delimitada;
- registre atividades na área com descrição, tempo estimado, custo, receita e avaliação;
- edite os dados depois;
- funcione offline e sincronize quando houver internet;
- seja eficiente em bateria, armazenamento, comunicação e processamento.

## 2. É complexo?

**Resposta curta: complexidade média.** A parte que assusta — desenhar o polígono e calcular a área — é a mais fácil no App Inventor, porque o componente `Map` já tem `Polygon` e o evento `TapAtPoint`. O cálculo da área é uma fórmula de ~15 blocos.

| Recurso | Dificuldade | Como resolver no App Inventor |
|---|---|---|
| Exibir mapa | Baixa | Componente `Map` |
| Desenhar polígono | Baixa | `Polygon.Points` |
| Capturar toque no mapa | Baixa | evento `Map.TapAtPoint` |
| Obter GPS | Baixa | `LocationSensor` |
| Calcular área | Média | Fórmula de Gauss (shoelace) |
| Salvar offline | Baixa | `TinyDB` |
| CRUD de atividades | Média | Listas dentro de listas |
| Sincronizar na nuvem | Média | `CloudDB` ou `FirebaseDB` |
| Mapa realmente offline | Alta | O `Map` nativo depende de tiles online; precisaria de extensão/OSMDroid |
| Escala global multi-usuário | Alta | Exige backend real (Postgres/Firebase), autenticação e resolução de conflitos |

O que torna o projeto “grande” não é o mapa: é a soma de requisitos (offline, sincronização, escalabilidade, relatório, diagramas). Por isso o MVP precisa cortar escopo.

## 3. O que o App Inventor resolve bem

- Prototipagem rápida no Android.
- `Map` com `Marker`, `LineString`, `Polygon`, `Circle` e `Rectangle`.
- `Map.TapAtPoint(latitude, longitude)` para marcar vértices.
- `LocationSensor` para GPS.
- `TinyDB` para persistência local (funciona offline).
- `CloudDB`/`FirebaseDB` para uma primeira versão remota.
- `ListView`, `Spinner`, `Notifier` para uma interface simples e utilizável.

## 4. Limitações importantes (para o relatório)

1. **Mapa offline não é nativo.** O componente `Map` baixa tiles da internet (OpenStreetMap/USGS). Sem internet, o polígono e os dados continuam funcionando, mas o fundo do mapa pode ficar cinza. Para mapa offline de verdade seria necessário usar uma extensão (por exemplo, OSMDroid) ou tiles pré-baixados.
2. **Não existe cálculo de área pronto.** É preciso calcular com blocos.
3. **`CloudDB` é simples demais para escala global.** Ele guarda valores por tag; não tem consultas, índices nem resolução de conflito. Para o protótipo serve; para a entrega “avançada”, o relatório deve reconhecer essa limitação.
4. **Listas no App Inventor usam índice começando em 1.** Isso vale para pontos, áreas e atividades.
5. **A edição de vértices por arrasto no mapa é limitada.** No MVP, a edição será feita por “desfazer”, “limpar” e adicionar de novo. Edição vértice a vértice pode ficar para depois.

## 5. Escopo do MVP

### Dentro do MVP

- 3 telas:
  1. **Lista de áreas**;
  2. **Editor da área** (mapa, polígono, nome, salvar);
  3. **Atividades da área** (lista + formulário).
- Um usuário local, sem login.
- Área definida por toque no mapa, GPS ou coordenadas manuais.
- Cálculo da área em m² e exibição em hectares.
- Persistência local com `TinyDB`.
- Atividades com:
  - descrição;
  - tempo estimado (horas);
  - custo;
  - receita;
  - avaliação (positiva/neutra/negativa);
  - observações;
  - data.
- Edição e exclusão de áreas e atividades.
- Totais de custo, receita e saldo por área.

### Fora do MVP (por enquanto)

- Login e multiusuário.
- Sincronização em nuvem.
- Mapa offline real.
- Relatórios em PDF.
- Gráficos.
- Edição de vértices por arrasto.
- Escala global.
- Validação geográfica avançada (sobreposição de áreas, auto-interseção etc.).

## 6. Arquitetura do MVP

```text
┌─────────────────────┐       areaIndex        ┌─────────────────────┐
│  Screen1            │ ─────────────────────> │  Screen2            │
│  Lista de áreas     │                        │  Editor + mapa      │
│  ListView + botão   │ <───────────────────── │  Polygon + pontos   │
└─────────┬───────────┘        voltar          └──────────┬──────────┘
          │                                               │
          │                                               │ areaIndex
          │                                               v
          │                                    ┌─────────────────────┐
          │                                    │  Screen3            │
          │                                    │  Atividades         │
          │                                    │  Lista + formulário │
          │                                    └──────────┬──────────┘
          │                                               │
          └────────────── TinyDB "areas" <───────────────┘
```

O `TinyDB` é compartilhado por todas as telas do mesmo aplicativo. Para não passar listas grandes entre telas, o MVP guarda o índice da área selecionada em uma tag separada (`areaIndex`) e cada tela lê os dados diretamente do `TinyDB`.

## 7. Modelo de dados do MVP

Uma lista de áreas. Cada área é uma lista de 4 posições:

```text
areas = [
  [ nome, pontos, areaM2, atividades ],
  ...
]

pontos = [
  [latitude, longitude],
  ...
]

atividades = [
  [descricao, horas, custo, receita, avaliacao, observacoes, data],
  ...
]
```

Posições (índice começa em 1):

| Posição | Campo | Tipo |
|---|---|---|
| 1 | `nome` | texto |
| 2 | `pontos` | lista de listas `[lat, lon]` |
| 3 | `areaM2` | número |
| 4 | `atividades` | lista de atividades |

Atividade:

| Posição | Campo | Tipo |
|---|---|---|
| 1 | `descricao` | texto |
| 2 | `horas` | número |
| 3 | `custo` | número |
| 4 | `receita` | número |
| 5 | `avaliacao` | número: `1` positiva, `0` neutra, `-1` negativa |
| 6 | `observacoes` | texto |
| 7 | `data` | texto `dd/MM/yyyy HH:mm` |

Tags do `TinyDB`:

| Tag | Conteúdo |
|---|---|
| `areas` | lista de todas as áreas |
| `areaIndex` | índice da área selecionada (0 = nova) |
| `atividadeIndex` | índice da atividade em edição (0 = nova) |

## 8. Roadmap

| Etapa | Entrega | Tempo estimado |
|---|---|---|
| 0 | Criar projeto e testar o Companion no celular | 30 min |
| 1 | Mapa + toque + polígono + área calculada | 1–2 h |
| 2 | Salvar/carregar uma área no TinyDB | 30–60 min |
| 3 | Lista de áreas + edição | 2–3 h |
| 4 | Atividades + totais + avaliação | 2–3 h |
| 5 | GPS, coordenadas manuais e sincronização | 2–4 h |
| 6 | Diagramas, relatório e apresentação | em paralelo |

As estimativas são para quem já mexeu com App Inventor. Na primeira vez, some 50–100% em cada etapa.
