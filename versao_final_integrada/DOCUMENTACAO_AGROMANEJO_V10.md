# Documentação do Projeto de Software e Relatório Técnico

**Sistema:** AgroManejo Integrado (v10)  
**Ambiente:** Mobile (Android / MIT App Inventor)  
**Backend Remoto:** Supabase (PostgreSQL / PostGIS / REST RPC)  
**Armazenamento Local:** TinyDB (Arquitetura *Offline-First*)  
**Versão da Documentação:** 2.0  

---

# PARTE 1 — PROJETO DO SOFTWARE

## 1. Descrição do Software

O **AgroManejo Integrado (v10)** é uma solução móvel completa voltada à gestão territorial e operacional de propriedades agrícolas. O aplicativo estrutura o gerenciamento em três níveis hierárquicos: **Propriedade Rural**, **Unidades de Manejo (Talhões)** e **Atividades Agrícolas**.

Diferenciais da versão v10:
* **Autenticação Real de Usuários:** Módulo nativo de login e criação de conta com controle de sessão (`arrLogin`, `arrInicio`).
* **Hierarquia Territorial Georreferenciada:**
  1. *Propriedade Mãe (Screen1):* Delimitação do contorno geral da fazenda com cálculo de área em hectares.
  2. *Unidades de Manejo (Screen2):* Parcelas internas (talhões) que devem obrigatoriamente estar contidas dentro dos limites da propriedade mãe, com validação topológica que impede polígonos cruzados ou pontos projetados para fora do limite da fazenda.
* **Mapeamento Interativo e Edição de Vértices:** Além de adicionar pontos por toque no mapa, GPS ou coordenadas manuais, o aplicativo possui suporte a seleção de vértices, ajuste fino de posição arrastando marcadores (`Marker`), alteração de coordenadas individuais, desfazer e exclusão pontual.
* **Detecção de Auto-interseção e Triangulação:** Procedimentos matemáticos que verificam se o polígono possui lados cruzados antes de salvar e calculam a área em hectares via fórmula de Gauss (Shoelace) projetada.
* **Módulo de Atividades Agrícolas e Controle de Custos (Screen3):** Registro detalhado contendo tipo da operação, situação, data prevista com `DatePicker`, tempo estimado (horas), decomposição tripla de custos (mão de obra, insumos e maquinário), receita projetada, avaliação qualitativa e observações, com balanço financeiro automático (saldo operacional).
* **Persistência Híbrida e Sincronização REST:** Operação local offline em `TinyDB` e integração na nuvem com **Supabase (PostgreSQL)** via chamadas HTTP REST (`Web1`), suportando versionamento, controle de alterações e isolamento multi-inquilino.

---

## 2. Requisitos do Sistema

### 2.1 Requisitos Funcionais (RF)

| ID | Requisito | Como é atendido na v10 |
|---|---|---|
| **RF01** | Autenticação e Controle de Sessão | Tela de login e cadastro (`txtUsuario`, `txtSenha`, `btnEntrar`, `btnCriarConta`) com persistência em `TinyDBSessao`. |
| **RF02** | Delimitação da Propriedade Rural | Demarcação da fronteira da fazenda na `Screen1` com no mínimo 3 pontos. |
| **RF03** | Delimitação de Unidades de Manejo | Cadastro de talhões internos na `Screen2` associados à propriedade. |
| **RF04** | Três Modos de Entrada de Coordenadas | Toque interativo no mapa (`Map.TapAtPoint`), coordenadas por GPS sob demanda (`LocationSensor`) e digitação manual de latitude/longitude com validação. |
| **RF05** | Edição e Ajuste Fino de Vértices | Seleção de pontos na lista, edição individual de coordenadas, arrasto de marcador no mapa, desfazer e remoção seletiva com confirmação. |
| **RF06** | Validação Topológica de Polígonos | Verificação de pontos repetidos, validação de auto-interseção (lados cruzados) e validação de contenção (unidade contida na propriedade). |
| **RF07** | Cálculo de Área por Triangulação | Fórmula de Gauss (Shoelace) com projeção trigonométrica métrica local, calculando a área em hectares ($ha$). |
| **RF08** | Visualização Cartográfica em Camadas | Exibição simultânea do contorno da propriedade e das unidades de manejo no `Map`. |
| **RF09** | Gestão de Atividades Agrícolas | Cadastro completo na `Screen3` contendo descrição, tipo de operação, data prevista via calendário, situação, tempo (horas) e observações. |
| **RF10** | Decomposição de Custos e Receita | Separação minuciosa dos custos: mão de obra, insumos e maquinário, confrontados com a receita esperada. |
| **RF11** | Avaliação da Atividade | Seleção de desempenho (*Positiva, Neutra, Negativa*). |
| **RF12** | Consolidação Financeira Dinâmica | Card financeiro com atualização instantânea de Custo Total, Receita e Saldo Líquido da unidade. |
| **RF13** | Operação Offline (*Offline-First*) | Todos os cadastros e cálculos funcionam localmente no `TinyDB1`. |
| **RF14** | Sincronização em Nuvem (PostgreSQL) | Chamadas REST para API do **Supabase** via componente `Web1`, sincronizando alterações e versionamento. |

### 2.2 Requisitos Não Funcionais (RNF)

| ID | Requisito | Atendimento Técnico |
|---|---|---|
| **RNF01** | Eficiência Energética (Bateria) | Sensores de GPS ativados sob demanda nos botões de captura (`btnUsarGPS`, `btnGPSUnidade`), sem drenagem por polling contínuo. |
| **RNF02** | Integridade Espacial | Algoritmos de verificação geométrica que impedem o salvamento de áreas inconsistentes ou talhões sobrepostos para fora da fazenda. |
| **RNF03** | Usabilidade e Navegação | Divisão estruturada em 3 telas especializadas (`Screen1` Propriedade, `Screen2` Unidades, `Screen3` Atividades), diálogos de confirmação de exclusão e formulários limpos com seletores de data. |
| **RNF04** | Escalabilidade e Concorrência Global | Backend baseado em PostgreSQL na nuvem (Supabase), permitindo milhares de produtores simultâneos com autenticação segura e particionamento de dados por chave de usuário. |

---

## 3. Casos de Uso

### 3.1 Diagrama de Casos de Uso (PlantUML)

```plantuml
@startuml
left to right direction
skinparam packageStyle rectangle

actor "Produtor Rural" as Produtor
actor "Backend Supabase (PostgreSQL)" as Backend

rectangle "AgroManejo Integrado v10" {
    usecase "UC01 - Autenticar / Criar Conta" as UC01
    usecase "UC02 - Delimitar Propriedade Rural" as UC02
    usecase "UC03 - Demarcar por Toque / GPS / Manual" as UC03
    usecase "UC04 - Ajustar e Mover Vértices" as UC04
    usecase "UC05 - Delimitar Unidade de Manejo (Talhão)" as UC05
    usecase "UC06 - Validar Contenção e Cruzamento" as UC06
    usecase "UC07 - Lançar Atividade Agrícola" as UC07
    usecase "UC08 - Calcular Balanço Financeiro da Área" as UC08
    usecase "UC09 - Sincronizar Dados com a Nuvem" as UC09

    UC02 ..> UC03 : <<include>>
    UC02 ..> UC04 : <<extend>>
    UC05 ..> UC03 : <<include>>
    UC05 ..> UC06 : <<include>>
    UC07 ..> UC08 : <<include>>
}

Produtor --> UC01
Produtor --> UC02
Produtor --> UC05
Produtor --> UC07
Produtor --> UC09

UC01 --> Backend
UC09 --> Backend
@enduml
```

### 3.2 Descrição dos Casos de Uso Principais
* **UC02 — Delimitar Propriedade Rural:** O produtor demarca os limites externos da fazenda inserindo coordenadas por toque no mapa, GPS ou digitação manual. O sistema valida se não há lados cruzados e calcula os hectares totais.
* **UC05 — Delimitar Unidade de Manejo:** O produtor cria um talhão específico. O sistema verifica se o talhão está contido geograficamente dentro da propriedade mãe antes de permitir a gravação.
* **UC07 — Lançar Atividade Agrícola:** O produtor aponta custos divididos em mão de obra, insumos e maquinário, define a data prevista pelo calendário e a receita esperada. O sistema consolida o saldo operacional da unidade.

---

## 4. Diagrama de Classes (PlantUML)

```plantuml
@startuml
skinparam classAttributeIconSize 0

class Usuario {
    - idUsuario: String
    - login: String
    - tokenSessao: String
    + autenticar(): Boolean
    + registrar(): Boolean
}

class Propriedade {
    - idPropriedade: String
    - nome: String
    - areaTotalHa: Double
    + calcularArea(): Double
    + validarPoligono(): Boolean
}

class UnidadeManejo {
    - idUnidade: String
    - nome: String
    - areaHa: Double
    + verificarContencao(prop: Propriedade): Boolean
    + calcularSaldo(): Double
}

class PontoCoordenada {
    - latitude: Double
    - longitude: Double
    - ordem: Integer
}

class AtividadeAgricola {
    - idAtividade: String
    - descricao: String
    - tipo: String
    - dataPrevista: Date
    - situacao: String
    - horas: Double
    - custoMaoObra: Double
    - custoInsumos: Double
    - custoMaquinario: Double
    - receitaEsperada: Double
    - avaliacao: String
    - observacoes: String
    + getCustoTotal(): Double
    + getSaldo(): Double
}

class GerenciadorSincronizacao {
    - statusRede: Boolean
    + sincronizarSupabase(): void
}

Usuario "1" -- "1" Propriedade : possui
Propriedade "1" *-- "1..*" UnidadeManejo : contém
Propriedade "1" *-- "3..*" PontoCoordenada : delimitada por
UnidadeManejo "1" *-- "3..*" PontoCoordenada : delimitada por
UnidadeManejo "1" *-- "0..*" AtividadeAgricola : registra
Usuario ..> GerenciadorSincronizacao : aciona
@enduml
```

---

## 5. Diagramas de Sequência (PlantUML)

### 5.1 Sequência: Cadastro da Unidade com Validação Topológica
```plantuml
@startuml
autonumber
actor Produtor
boundary "Screen2 (Unidades)" as UI
control "Motor Geometrico" as Geo
entity "TinyDB (Local)" as DB
database "Supabase (Nuvem)" as Cloud

Produtor -> UI : Informa pontos do talhão (Toque/GPS)
Produtor -> UI : Clica em "Salvar Unidade"
UI -> Geo : poligonoCruzado(pontos)
Geo --> UI : Retorna falso (polígono válido)
UI -> Geo : verificarContencao(unidade, propriedade)
Geo --> UI : Retorna verdadeiro (100% contido)
UI -> Geo : areaHa(pontos)
Geo --> UI : Retorna área calculada
UI -> DB : StoreValue("unidades", listaUnidades)
UI -> Cloud : POST /rest/v1/rpc/sincronizar
Cloud --> UI : 200 OK (Sincronizado)
UI --> Produtor : Notificação "Unidade salva com sucesso"
@enduml
```

### 5.2 Sequência: Registro de Atividade com Balanço de Custos
```plantuml
@startuml
autonumber
actor Produtor
boundary "Screen3 (Atividades)" as UI
control "Calculadora Financeira" as Calc
entity "TinyDB" as DB
database "Supabase (Nuvem)" as Cloud

Produtor -> UI : Preenche mão de obra, insumos, maquinário e receita
Produtor -> UI : Clica em "Salvar Atividade"
UI -> Calc : Somar custos triplos e abater da receita
Calc --> UI : Atualiza Card de Saldo
UI -> DB : StoreValue("atividades", listaAtividades)
alt Internet Disponível
    UI -> Cloud : POST /rest/v1/rpc/sincronizar
    Cloud --> UI : Sucesso
    UI --> Produtor : "Atividade salva e sincronizada na nuvem"
else Sem Conexão
    UI --> Produtor : "Salvo localmente no aparelho"
end
@enduml
```

---

## 6. Modelo Entidade-Relacionamento (ER - PlantUML)

```plantuml
@startuml
entity "USUARIO" as user {
  * id_usuario [PK]
  --
  login
  senha_hash
  data_criacao
}

entity "PROPRIEDADE" as prop {
  * id_propriedade [PK]
  --
  * id_usuario [FK]
  nome_propriedade
  area_ha
  pontos_geojson
}

entity "UNIDADE_MANEJO" as unidade {
  * id_unidade [PK]
  --
  * id_propriedade [FK]
  nome_unidade
  area_ha
  pontos_geojson
}

entity "ATIVIDADE_AGRICOLA" as ativ {
  * id_atividade [PK]
  --
  * id_unidade [FK]
  descricao
  tipo
  data_prevista
  situacao
  horas_estimadas
  custo_mao_obra
  custo_insumos
  custo_maquinario
  receita_esperada
  avaliacao
  observacoes
}

user ||--|| prop : possui
prop ||--|{ unidade : subdividida em
unidade ||--o{ ativ : registra
@enduml
```

---

# PARTE 2 — RELATÓRIO TÉCNICO

## 1. Tecnologias Utilizadas
1. **Front-End e Lógica:** MIT App Inventor 2 (Android).
2. **Camada de Mapas:** OpenStreetMap (`Map`, `Polygon`, `LineString`, `Marker`).
3. **Sensoriamento:** Android Location API (`LocationSensor`).
4. **Armazenamento Local:** `TinyDB` (armazenamento não-volátil embutido).
5. **Backend em Nuvem:** **Supabase** (PostgreSQL com suporte geoespacial e endpoints REST via componente `Web1`).

---

## 2. Justificativa das Escolhas Técnicas

### Por que a divisão em 3 telas (Screen1, Screen2, Screen3)?
Na versão v10, a separação em telas independentes foi adotada para acomodar o fluxo de trabalho natural da fazenda:
* **Screen1:** Foco no proprietário e na área global da fazenda.
* **Screen2:** Foco no zoneamento agrícola (talhões de manejo e limites internos).
* **Screen3:** Foco na rotina de campo e controle operacional/financeiro.

### Por que Supabase / PostgreSQL na nuvem?
O enunciado solicita escalabilidade global e concorrência para múltiplos usuários. A utilização do Supabase (PostgreSQL) viabiliza:
* Autenticação segura por tokens.
* Chamadas atômicas via RPC (`/rest/v1/rpc/sincronizar`), garantindo que alterações enviadas por múltiplos celulares não se sobreponham.
* Suporte nativo a tipos espaciais geográficos.

---

## 3. Explicação de como a Solução Atende à Demanda Proposta

### 3.1 Triangulação e Geometria da Área
* A área em hectares é apurada por meio da **Fórmula de Gauss (Shoelace)** com projeção métrica local baseada na latitude média do terreno. A fórmula decompõe analiticamente o polígono em triângulos a partir do centroide e totaliza suas áreas, suprindo o requisito de triangulação.
* O sistema conta com validação que rejeita polígonos que não fecham área (pontos alinhados ou insuficientes) e auto-interseções (arestas cruzadas).

### 3.2 Relação de Contenção Espacial (Unidade dentro da Propriedade)
O aplicativo impede que uma unidade de manejo seja cadastrada fora da fazenda. Ele verifica os vértices e os segmentos de reta do talhão em relação ao contorno da propriedade cadastrada na `Screen1`.

### 3.3 Decomposição Financeira e Apontamento Completo
Atendendo a todas as solicitações de gestão, o custo operacional é decomposto em três frentes essenciais da lavoura: mão de obra, insumos e maquinário. O confronto direto com as receitas esperadas fornece o saldo líquido da área em tempo real.

### 3.4 Eficiência Energética e Operação Offline
* **GPS sob Demanda:** O sensor de geolocalização não realiza polling ininterrupto, poupando a bateria do smartphone durante o uso prolongado sob o sol.
* **Offline-First:** O aplicativo realiza todas as operações e consultas prioritariamente no `TinyDB1`. A conexão à nuvem é acionada de forma assíncrona, não travando a interface do usuário quando não houver sinal de telefonia no campo.
