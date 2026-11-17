# Status de Atendimento dos Requisitos do Sistema

Este documento descreve a aderência do sistema desenvolvido aos requisitos estipulados na especificação do projeto, detalhando os módulos implementados, os mecanismos técnicos adotados, as pendências existentes e o plano para sua execução.

---

## 1. Requisitos Atendidos e Mecanismos de Implementação

### 1.1 Definição da Unidade de Manejo e Delimitação Geográfica
* **Três ou mais coordenadas:**
  * *Implementação:* O sistema valida se a lista de vértices possui cardinalidade igual ou superior a 3 antes de habilitar cálculos e fechamento de polígono, alertando o operador caso a quantidade seja insuficiente.
* **Formas de entrada das coordenadas:**
  1. **Toque no mapa:** Captura via evento `Map1.TapAtPoint`, convertendo o toque diretamente em pares [latitude, longitude].
  2. **Geolocalização do dispositivo (GPS):** Leitura de coordenadas sob demanda via `LocationSensor1` ao acionar o comando de marcação por GPS.
  3. **Inserção manual:** Formulário dedicado com validação de campos para latitude e longitude, plotando o vértice e centralizando o mapa na posição informada.
* **Delimitação e cálculo de área por triangulação:**
  * *Implementação:* Renderização do polígono via componente `Polygon1`. A área é calculada analiticamente por meio da **Fórmula de Gauss (Shoelace)** com projeção métrica local (fator de correção de latitude e longitude em metros). Matematicamente, o algoritmo realiza a soma algébrica das áreas dos triângulos orientados formados pelos vértices consecutivos em relação à origem, atendendo com precisão ao princípio da triangulação.

### 1.2 Registro e Gestão de Atividades
* **Associação à área:** As atividades são armazenadas em estrutura aninhada, vinculadas à respectiva unidade de manejo.
* **Atributos contemplados:**
  * *Descrição:* Campo de texto livre para tipificação da operação (adubação, plantio, monitoramento, etc.).
  * *Tempo estimado:* Campo numérico para registro de horas trabalhadas.
  * *Custos operacionais:* Campo numérico para registro de gastos com insumos, maquinário e mão de obra.
  * *Receitas esperadas:* Campo numérico para projeção ou apuração de retornos financeiros.
  * *Avaliação da atividade:* Seleção categórica (*Positiva*, *Neutra*, *Negativa*).
* **Consolidação financeira:** Painel em tempo real que calcula o somatório de custos, somatório de receitas e o saldo operacional decorrente da área selecionada.

### 1.3 Navegação, Usabilidade e Edição
* **Estrutura de telas (Virtual Screens):** A interface foi organizada em painéis independentes gerenciados por visibilidade dinâmica (*PainelLista*, *PainelEditor*, *PainelAtividades*), evitando lentidão de transição e mantendo a persistência de estados.
* **Edição e exclusão contínuas:** O operador pode, a qualquer momento, selecionar áreas cadastradas ou atividades específicas para modificar dados, excluir registros ou adicionar novos vértices.
* **Padrão visual:** Adoção de hierarquia visual com cartões de dados (Cards), botões arredondados e diferenciação semântica por cores para ações de confirmação, alerta e remoção.

### 1.4 Operação Offline e Eficiência de Recursos
* **Persistência local (Offline-First):** Utilização do `TinyDB`. O aplicativo executa todas as operações de leitura, escrita, cálculo e consulta diretamente no armazenamento local do dispositivo móvel, operando sem dependência de conexão com a internet.
* **Eficiência energética (Bateria):** O sensor de localização opera com intervalo de amostragem de 60 segundos e deslocamento mínimo de 10 metros, sem escuta contínua de alta frequência (*no polling*). A leitura fina do GPS ocorre sob demanda.
* **Eficiência de processamento:** O algoritmo de cálculo geométrico possui complexidade assintótica $O(N)$, sendo executado instantaneamente sem sobrecarga de CPU ou consumo elevado de memória RAM.

---

## 2. Requisitos Pendentes

Embora a totalidade dos fluxos funcionais locais esteja operacional, dois requisitos arquiteturais necessitam de finalização:

1. **Sincronização automática em nuvem:** Envio e recebimento bidirecional dos dados quando houver restabelecimento da conectividade de rede.
2. **Escalabilidade concorrente:** Estruturação para suporte a múltiplos operadores simultâneos geograficamente distribuídos sem colisões de dados.

---

## 3. Plano de Execução dos Itens Pendentes

Para suprir os pontos pendentes sem comprometer o funcionamento offline já estabelecido, o seguinte plano deve ser executado:

### Etapa A: Sincronização em Nuvem (CloudDB / Firebase)
* **Objetivo:** Persistir cópia remota das áreas e atividades assim que a conexão de dados for detectada.
* **Ações:**
  1. Integrar o componente `CloudDB` (ou `FirebaseDB`) ao projeto.
  2. Implementar rotina de sincronização disparada nos eventos de salvamento local (`btnSalvar.Click` e `btnSalvarAtividade.Click`).
  3. Estruturar a gravação remota com chaves particionadas por identificador de dispositivo ou usuário: `usuario_id/areas`.
  4. Adicionar rotina de verificação no início da execução: ao abrir o sistema com conexão ativa, efetuar leitura na nuvem; caso contrário, manter os dados carregados do `TinyDB`.

### Etapa B: Tratamento de Concorrência e Escalabilidade
* **Objetivo:** Evitar sobreposição de dados entre múltiplos usuários operando no mesmo sistema.
* **Ações:**
  1. **Identificadores Únicos Universais (UUID):** Substituir a identificação posicional por índices simples pela geração de UUIDs para cada área e atividade, garantindo que registros criados offline por diferentes aparelhos não entrem em conflito.
  2. **Timestamp de modificação:** Incluir carimbo de data/hora (*timestamp*) em cada operação para resolução de concorrência baseada em *Last-Write-Wins* (a alteração mais recente prevalece).

