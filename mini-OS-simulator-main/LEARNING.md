# Por que C++? ? Benef?cios, recursos e o que este projeto ensina

Este documento explica por que C++ foi escolhido para o Mini OS Simulator, quais recursos da linguagem se mostraram mais ?teis, como o uso de threads e concorr?ncia funciona e o que voc? aprende ao desenvolver ou estudar este projeto, tanto sobre conceitos de sistemas operacionais quanto sobre programa??o.

---

## 1. Por que C++ para este projeto e outros projetos de sistemas operacionais

### 1.1 Correspond?ncia direta com os conceitos de sistemas operacionais

Nos cursos de sistemas operacionais, estudamos **processos**, **threads**, **mutexes**, **sem?foros** e **escalonadores**. Em C++, existem recursos correspondentes diretamente na linguagem e em sua biblioteca padr?o:

| Conceito de SO | Biblioteca padr?o do C++ | Como ? utilizado no simulador |
|---|---|---|
| Thread | `std::thread` | M?dulo 3: threads concorrentes reais para demonstra??es de sincroniza??o |
| Mutex | `std::mutex` | Prote??o do buffer compartilhado, do sistema de registro de eventos e do estado do RAG |
| Vari?vel de condi??o | `std::condition_variable` | Sem?ntica de sem?foros e de ?aguardar at? que um recurso esteja dispon?vel? |
| Opera??es at?micas | `std::atomic<T>` | Contadores de itens do produtor-consumidor sem necessidade de mutexes |

O uso das mesmas primitivas disponibilizadas ou encapsuladas pelos sistemas operacionais reais mant?m o modelo mental consistente: aquilo que voc? simula ? por exemplo, uma ?thread bloqueada em um sem?foro? ? tamb?m acontece de fato no programa.

Uma linguagem sem threads ou mutexes nativos exigiria um modelo mais abstrato, tornando o aprendizado de concorr?ncia menos direto.

### 1.2 Aus?ncia de uma camada de execu??o oculta ou de coletor de lixo

O simulador ? um **execut?vel independente**, sem interpretador ou m?quina virtual. N?o h? coletor de lixo, portanto:

- **Determinismo:** ao executar um algoritmo de escalonamento ou substitui??o de p?ginas, o comportamento pode ser reproduzido, sem pausas causadas por coleta de lixo.
- **Controle de recursos:** ? poss?vel analisar quando a mem?ria ? alocada e liberada, por exemplo, nos blocos do disco virtual e nas listas de processos. Isso se relaciona diretamente aos temas de gerenciamento de mem?ria e pol?ticas de aloca??o estudados em sistemas operacionais.
- **Portabilidade:** o mesmo c?digo pode ser compilado em plataformas que disponham de uma ferramenta de compila??o compat?vel com C++17, sem exigir o envio de um ambiente de execu??o adicional.

Em projetos que simulam kernels, escalonadores ou alocadores, essa previsibilidade e esse controle s?o vantagens importantes.

### 1.3 Desempenho onde ele ? relevante

N?o precisamos de velocidade m?xima em um simulador educacional, mas C++ ainda oferece benef?cios:

- **Concorr?ncia (M?dulo 3):** threads reais de `std::thread` podem ser executadas em paralelo em m?quinas com m?ltiplos n?cleos. Assim, ? poss?vel observar o entrela?amento da execu??o e o comportamento temporal sem precisar falsific?-los.
- **Baixo custo por opera??o:** la?os intensivos, como os do algoritmo do banqueiro, da substitui??o de p?ginas e do escalonamento de disco, executam com pouca sobrecarga, mantendo o programa responsivo mesmo com entradas grandes.
- **Organiza??o dos dados:** estruturas como `ProcessControlBlock` e `DirEntry` possuem uma organiza??o de dados expl?cita e previs?vel. Isso ? ?til para compreender posteriormente os efeitos de cache ou implementar algo mais pr?ximo de um kernel real.

### 1.4 Biblioteca padr?o abrangente, sem depend?ncias externas

O projeto utiliza **somente a biblioteca padr?o do C++**:

- **Cont?ineres:** `std::vector`, `std::queue`, `std::map`, `std::set` e `std::list` para processos, filas de prontos, quadros de p?gina, FAT, entradas de diret?rio e estruturas de grafos.
- **Algoritmos:** `std::sort`, `std::min` e `std::max` para pol?ticas de escalonamento e substitui??o.
- **Entrada e sa?da:** `std::fstream` para leitura e escrita bin?ria no disco virtual.
- **Threads e sincroniza??o:** `std::thread`, `std::mutex`, `std::condition_variable`, `std::lock_guard` e `std::atomic`.
- **Tempo:** `std::chrono` para registros de data e hora e atrasos opcionais nas demonstra??es.

Dessa forma, temos um conjunto completo de ferramentas de programa??o de sistemas sem depender de bibliotecas de terceiros, mantendo o projeto port?til e f?cil de compilar em diferentes ambientes.

### 1.5 Adequa??o a projetos semelhantes

Os mesmos motivos se aplicam a outros projetos educacionais ou de sistemas leves, por exemplo:

- **Simuladores de escalonamento de CPU ou disco:** exigem estruturas de dados claras, filas e algoritmos.
- **Demonstra??es de concorr?ncia:** threads e mutexes reais s?o essenciais.
- **Sistemas de arquivos simples ou alocadores:** entrada e sa?da bin?ria, organiza??o em bytes e aus?ncia de coletor de lixo combinam bem com C++.
- **M?quinas de estados**, como as utilizadas em protocolos ou no ciclo de vida de processos: enums e structs representam diretamente os estados e suas transi??es.

Para projetos pr?ximos do hardware ou voltados ao ensino de conceitos de sistemas operacionais e arquitetura, C++ ? uma escolha natural.

---

## 2. Recursos do C++ que realmente ajudam neste projeto

A seguir est?o os recursos da linguagem e da biblioteca que contribuem diretamente para a arquitetura e a clareza do simulador.

### 2.1 `enum class` para representar estados

Os estados de processos e threads s?o representados por uma **enumera??o fortemente tipada**:

```cpp
enum class ProcessState {
    NEW,
    READY,
    RUNNING,
    WAITING,
    TERMINATED
};
```

- **Seguran?a de tipos:** n?o ? poss?vel misturar acidentalmente esses valores com n?meros inteiros ou outras enumera??es. As transi??es s?o expl?citas, como `ProcessState::READY`.
- **Nomes delimitados:** evita a polui??o do espa?o de nomes global. O conjunto de estados fica claro e documentado em um ?nico lugar.
- **Verifica??o de casos:** o compilador pode emitir avisos quando um caso ? esquecido em fun??es como `stateToString()` ou em um escalonador.

Esse recurso ? utilizado em todos os pontos que lidam com estados de processos e threads, especialmente nos m?dulos 1 e 2 e nos registros de eventos.

### 2.2 Structs e sem?ntica de valores

Os dados centrais s?o representados por **structs simples**, com uma organiza??o clara:

```cpp
struct ProcessControlBlock {
    int pid = 0;
    ProcessState state = ProcessState::NEW;
    int burst_time = 0;
    // ...
    std::vector<ThreadControlBlock> threads;
};
```

- **Inicializadores padr?o de membros:** valores como `= 0` e `= ProcessState::NEW` ajudam a evitar erros causados por vari?veis n?o inicializadas ao criar processos.
- **Sem?ntica de valores:** os escalonadores recebem um `std::vector<ProcessControlBlock>` e podem copiar o vetor para executar a mesma carga de trabalho com algoritmos diferentes, sem compartilhar estado mut?vel. Isso simplifica e torna mais segura a compara??o entre algoritmos.
- **Composi??o:** um PCB cont?m um `std::vector<ThreadControlBlock>`. A mesma ideia ? combinar dados e cont?ineres ? ? usada em tabelas de p?ginas, FAT e entradas de diret?rio.

### 2.3 Cont?ineres e algoritmos da STL

- **`std::vector`:** utilizado para processos, threads, quadros de p?gina, filas de requisi??es e entradas de diret?rio. Redimensionar e acessar elementos por ?ndice ? simples.
- **`std::queue`:** utilizada na fila de prontos do Round Robin e do MLFQ, assim como na fila de requisi??es do escalonamento de disco.
- **`std::map` / `std::set`:** utilizados no Grafo de Aloca??o de Recursos (RAG), que representa processos, recursos e arestas de atribui??o ou solicita??o, al?m de estruturas usadas na substitui??o de p?ginas LRU.
- **`std::sort`:** ordena processos pelo tempo de chegada, no FCFS e no Round Robin, ou pelo tempo de execu??o e pela prioridade, no SJF e no escalonamento por prioridade.
- **`std::pair`:** representa pares de dados, como entradas do gr?fico de Gantt `(pid, duration)` e outras combina??es simples.

O uso da biblioteca padr?o mant?m o c?digo conciso e leg?vel, al?m de ensinar a utilizar componentes bem testados em vez de implementar manualmente todas as listas e filas.

### 2.4 RAII e `std::lock_guard`

O sistema de registro de eventos ? compartilhado por v?rias threads, portanto cada chamada de registro precisa ser sincronizada:

```cpp
void log(const std::string& tag, const std::string& msg) {
    std::lock_guard<std::mutex> lk(mtx_);
    // ... formatar e imprimir ...
}
```

- **RAII:** o mutex ? liberado automaticamente quando `lk` sai do escopo, inclusive em casos de retorno antecipado ou exce??o. Isso evita o risco de esquecer de desbloque?-lo.
- **`std::lock_guard`:** ? a forma padr?o de ?manter o bloqueio durante a execu??o deste bloco?. O mesmo padr?o ? utilizado nos wrappers de mutexes e sem?foros simulados, garantindo que cada aquisi??o tenha uma libera??o correspondente.

Esse recurso d? suporte direto ao tema de primitivas de sincroniza??o do projeto e representa como programas de sistemas reais utilizam se??es cr?ticas.

### 2.5 Lambdas e captura de vari?veis

No M?dulo 3, cada produtor e consumidor ? implementado como uma lambda passada para `std::thread`:

```cpp
auto producer = [&](int id) {
    std::string name = "Producer-" + std::to_string(id);

    while (true) {
        int item = produced.fetch_add(1);
        // ...
        empty.wait(name);
        bufMtx.lock(name);
        buffer.push_back(item);
        // ...
    }
};

std::thread t(producer, i + 1);
```

- **Captura por refer?ncia com `[&]`:** a lambda acessa o `buffer`, `bufMtx`, `empty`, `full` e os contadores `produced` e `consumed` compartilhados. Isso mant?m o c?digo da demonstra??o compacto.
- **Argumentos por valor, como `(int id)`:** cada thread recebe seu pr?prio identificador e cria sua pr?pria string `name`, evitando o compartilhamento de estado mut?vel para esse dado.
- **Mesmo padr?o em outros cen?rios:** a mesma abordagem ? usada para leitores, escritores, fil?sofos e a thread do barbeiro. Assim, existe um modelo claro para ?executar esta fun??o em uma thread com este identificador?.

### 2.6 `std::atomic` para contadores sem bloqueio por mutex

O problema do produtor-consumidor utiliza opera??es at?micas para o ?ndice do pr?ximo item e a quantidade de itens consumidos:

```cpp
std::atomic<int> produced{0};
std::atomic<int> consumed{0};

// ...
int item = produced.fetch_add(1);
int c = consumed.fetch_add(1);
```

- **Sem mutex para os contadores:** v?rias threads podem atualizar os contadores sem precisar bloquear umas ?s outras por meio de um mutex.
- **Sem?ntica clara:** `fetch_add` realiza uma opera??o at?mica de leitura, modifica??o e escrita, correspondendo ao conceito de contador compartilhado na teoria de concorr?ncia.
- **Introdu??o a padr?es sem bloqueio:** ensina uma t?cnica simples de programa??o *lock-free* antes de abordar estruturas mais complexas dessa categoria.

### 2.7 Entrada e sa?da bin?ria e controle da organiza??o dos dados

O disco virtual, no M?dulo 7, utiliza `std::fstream`, com tamanho fixo de bloco e deslocamentos expl?citos:

```cpp
void readBlock(int blockNum, void* buffer) {
    std::ifstream f(path_, std::ios::binary);
    f.seekg(blockNum * BLOCK_SIZE);
    f.read((char*)buffer, BLOCK_SIZE);
}
```

- **Sem codifica??o oculta:** o modo bin?rio e as opera??es `read` e `write` oferecem controle preciso sobre os bytes. O superbloco, a FAT e as entradas de diret?rio s?o gravados com `memcpy` para corresponder ao formato armazenado em disco.
- **Portabilidade:** o projeto utiliza tipos de tamanho fixo e uma organiza??o expl?cita dos dados, sem pressupor uma determinada ordem de bytes (*endianness*) da m?quina al?m daquilo que estiver documentado.

Assim, o C++ ensina como controlar cada byte sem sair da biblioteca padr?o.

### 2.8 Singleton para o sistema global de registros

O sistema de registro de eventos ? um **singleton seguro para uso concorrente**:

```cpp
static Logger& instance() {
    static Logger inst;
    return inst;
}
```

- **Inst?ncia ?nica:** qualquer m?dulo pode chamar `Logger::instance().log(...)` sem precisar receber uma refer?ncia ao logger. A inicializa??o da vari?vel est?tica local ocorre uma ?nica vez, com garantia da linguagem a partir do C++11.
- **Encapsulamento:** o construtor e o mutex interno s?o privados. O restante do c?digo utiliza somente a interface p?blica. Esse ? um padr?o comum em C++ para servi?os globais.

### 2.9 Cabe?alhos (*headers*) ou unidades de tradu??o pequenas

Grande parte da l?gica est? nos **arquivos de cabe?alho**, como `scheduler.h`, `page_replacement.h` e `disk_scheduler.h`, por meio de c?digo *inline* ou com uso intensivo de templates.

Benef?cios:

- **Compila??o simples:** existem poucos arquivos `.cpp`, o que pode acelerar a compila??o e facilitar a identifica??o das depend?ncias.
- **Inlining:** pequenas fun??es auxiliares, como `stateToString` e as de c?lculo de m?tricas, podem ser expandidas durante a compila??o. Isso ? adequado para um simulador no qual a clareza importa mais do que o tempo de compila??o.
- **Refatora??o mais f?cil:** mover uma fun??o entre arquivos ? relativamente simples, sem precisar lidar com um n?mero muito grande de unidades de tradu??o.

Em um projeto educacional de porte m?dio, essa abordagem ajuda a manter a estrutura compreens?vel.

---

## 3. Uso de threads e concorr?ncia

### 3.1 Onde as threads s?o utilizadas

O **M?dulo 3 ? Ambiente de Experimenta??o de Sincroniza??o** ? a ?nica parte que cria m?ltiplas **threads reais do sistema operacional**:

- **Produtor-consumidor:** v?rias threads produtoras e consumidoras compartilham um buffer, um mutex e dois sem?foros (`empty` e `full`).
- **Leitores-escritores:** threads leitoras e escritoras compartilham uma vari?vel que representa os dados e utilizam um mutex compartilhado, outro mutex para a contagem de leitores e um sem?foro de leitura e escrita (`rw`).
- **Jantar dos fil?sofos:** cada fil?sofo possui uma thread que pensa repetidamente, adquire os sem?foros dos dois garfos em uma ordem fixa para evitar deadlock, come e depois libera os recursos.
- **Barbeiro dorminhoco:** uma thread representa o barbeiro e v?rias outras representam clientes. Os clientes aguardam utilizando um sem?foro e um mutex que protegem a sala de espera.

Em todos os casos, s?o utilizados:

- `std::thread` para executar a l?gica em threads.
- `std::mutex`, ?s vezes encapsulado em `SimMutex` para permitir registros de eventos, para proteger dados compartilhados.
- `std::condition_variable`, dentro de `SimSemaphore`, para bloquear a execu??o at? que um recurso esteja dispon?vel.
- `std::atomic` quando ? necess?rio compartilhar apenas um contador.

### 3.2 O que isso ensina sobre concorr?ncia

- **Se??es cr?ticas:** somente uma thread por vez pode atualizar o buffer ou os dados compartilhados no problema dos leitores-escritores. O mutex define a se??o cr?tica, e os registros mostram quando os bloqueios s?o adquiridos e liberados, permitindo observar a serializa??o das opera??es.
- **Bloqueio versus espera ativa:** os sem?foros utilizam `condition_variable::wait()`, fazendo com que a thread aguarde em vez de executar continuamente um la?o de espera. Os registros mostram mensagens como ?aguardando Empty? e ?passou por Empty?, correspondentes aos estados de bloqueio e execu??o.
- **Preven??o de deadlock:** no jantar dos fil?sofos, a ordem de aquisi??o dos bloqueios ? por exemplo, adquirir primeiro o garfo de menor identificador ou fazer o ?ltimo fil?sofo inverter a ordem ? ? expl?cita. Isso impede a forma??o de uma espera circular. ? poss?vel comparar essa estrat?gia com uma implementa??o incorreta para compreender como o deadlock pode surgir.
- **Inani??o e justi?a:** com muitos leitores e escritores ou muitos fil?sofos, ? poss?vel observar que algumas threads podem executar com menor frequ?ncia. Os registros com data e hora ajudam a discutir justi?a (*fairness*) e escalonamento.
- **Paralelismo real:** em m?quinas com m?ltiplos n?cleos, as threads produtoras e consumidoras podem executar realmente em paralelo. O entrela?amento das opera??es n?o ? determin?stico e varia entre execu??es, exatamente como se espera ao estudar concorr?ncia.

### 3.3 Sistema de registros seguro para uso concorrente

O logger ? utilizado simultaneamente por v?rias threads, portanto cada chamada `log(...)` ? protegida por um mutex. Isso garante que:

- As linhas n?o sejam misturadas no meio de uma mensagem.
- Seja poss?vel observar uma linha do tempo ?nica e consistente dos eventos, com registros de data e hora, ao analisar uma execu??o.

Dessa forma, o projeto tamb?m demonstra **como compartilhar com seguran?a um ?nico fluxo de sa?da entre v?rias threads**, uma necessidade recorrente em programas concorrentes.

---

## 4. O que mais o projeto ensina

Al?m de explicar por que C++ foi escolhido e como as threads s?o utilizadas, o simulador serve como ferramenta de aprendizado para os seguintes t?picos.

### 4.1 Conceitos de sistemas operacionais

- **Processos e threads:** PCB/TCB, estados (`NEW`, `READY`, `RUNNING`, `WAITING`, `TERMINATED`) e transi??es; diferen?as entre processos e threads, como o compartilhamento do espa?o de endere?amento e a exist?ncia de pilhas separadas para cada thread.
- **Escalonamento:** FCFS, SJF, Round Robin, Prioridade e MLFQ; escalonamento preemptivo e n?o preemptivo; m?tricas de espera, tempo de retorno (*turnaround*), resposta, utiliza??o e vaz?o; gr?ficos de Gantt.
- **Sincroniza??o:** mutexes, sem?foros e vari?veis de condi??o; problemas cl?ssicos do produtor-consumidor, leitores-escritores, jantar dos fil?sofos e barbeiro dorminhoco; condi??es de corrida e se??es cr?ticas.
- **Deadlock:** Grafo de Aloca??o de Recursos; detec??o de ciclos; algoritmo do banqueiro; estados seguros e inseguros; sequ?ncia segura.
- **Gerenciamento de mem?ria:** particionamento fixo e vari?vel; First Fit, Best Fit e Worst Fit; fragmenta??o interna e externa; compacta??o.
- **Mem?ria virtual:** endere?os l?gicos e f?sicos; tabela de p?ginas; falha de p?gina; pol?ticas de substitui??o FIFO, LRU, ?timo e Clock; aloca??o de quadros.
- **Sistema de arquivos:** abstra??o de dispositivo de blocos; superbloco; FAT; entradas de diret?rio; cria??o, leitura, escrita, exclus?o e opera??es de diret?rio.
- **Entrada e sa?da (E/S):** fila de requisi??es; escalonamento de disco com FCFS, SSTF, SCAN e C-SCAN; tempo de busca e deslocamento total da cabe?a de leitura e escrita.

### 4.2 Algoritmos e estruturas de dados

- **Grafos:** o RAG como grafo direcionado; busca em profundidade (DFS) para detec??o de ciclos; representa??o por adjac?ncia, como um mapa de processos para conjuntos de recursos solicitados e de recursos para seus processos detentores.
- **Filas:** fila de prontos (FCFS, RR e MLFQ), fila de requisi??es de disco e buffer do produtor-consumidor.
- **Tabelas e matrizes:** tabela de p?ginas (p?gina ? quadro); matrizes Allocation, Max, Need e Available no algoritmo do banqueiro; FAT (bloco ? pr?ximo bloco ou fim de arquivo).
- **Ordena??o e sele??o:** ordena??o por chegada, tempo de execu??o ou prioridade; sele??o do menor tempo de execu??o ou da maior prioridade nos escalonadores; escolha da p?gina cujo pr?ximo uso est? mais distante na substitui??o ?tima.
- **La?os de simula??o:** la?os baseados em unidades de tempo ou eventos, como ?em cada unidade de tempo, selecionar um processo, execut?-lo e atualizar seu estado?, que reproduzem a l?gica de simuladores de eventos discretos.

### 4.3 Projeto de software e pr?ticas de C++

- **Estrutura modular:** uma ?rea para cada responsabilidade ? processos, escalonamento, sincroniza??o, deadlock, mem?ria, mem?ria virtual, sistema de arquivos e E/S ?, com tipos compartilhados em `common/` e limites claros entre os arquivos inclu?dos.
- **Interfaces por conven??o:** todos os escalonadores recebem um vetor de PCBs e produzem um resultado, como gr?fico de Gantt e m?tricas. As fun??es de substitui??o de p?ginas recebem uma sequ?ncia de refer?ncias e a quantidade de quadros, retornando uma estrutura de resultado. N?o ? necess?rio criar uma classe-base formal para deixar esse padr?o claro.
- **Reutiliza??o:** o mesmo `ProcessControlBlock` ? usado nos m?dulos 1 e 2; a exibi??o e o logger s?o compartilhados entre os m?dulos; a ideia de comparar todos os algoritmos ? aplicada ao escalonamento, ? substitui??o de p?ginas e ao escalonamento de disco.
- **Corre??o de `const`:** m?todos `const` e refer?ncias constantes s?o utilizados quando os dados n?o precisam ser modificados, por exemplo, na exibi??o do RAG e na detec??o de ciclos.
- **Gerenciamento de recursos:** RAII para mutexes; fluxos de arquivos abertos e fechados em escopos bem definidos; aus?ncia de uso de `new` e `delete` diretamente na l?gica principal, pois cont?ineres e tipos por valor gerenciam a mem?ria.

---

## 5. Tabela-resumo

| ?rea | O que C++ oferece | O que o projeto ensina |
|---|---|---|
| **Concorr?ncia** | `std::thread`, `std::mutex`, `std::condition_variable`, `std::atomic` | Paralelismo real, se??es cr?ticas, bloqueio e preven??o de deadlock |
| **Modelagem de dados** | Structs, `enum class` e cont?ineres da STL | PCB/TCB, m?quinas de estados, filas, grafos e tabelas |
| **Algoritmos** | `std::sort`, la?os e condicionais | Escalonamento, substitui??o de p?ginas, escalonamento de disco, algoritmo do banqueiro e detec??o de ciclos |
| **Entrada e sa?da** | `std::fstream`, modo bin?rio e `memcpy` | Organiza??o do disco virtual, blocos, FAT e entradas de diret?rio |
| **Seguran?a** | RAII, `lock_guard` e enumera??es tipadas | Bloqueio seguro, preven??o de vazamentos e transi??es claras de estado |
| **Projeto de software** | Cabe?alhos, unidades de tradu??o pequenas e singleton | Simulador modular de SO, componentes reutiliz?veis e execut?vel ?nico |

---

Este documento complementa o arquivo principal [README.md](README.md). Juntos, eles explicam tanto **o que** o Mini OS Simulator faz quanto **por que** ele foi implementado em C++ e o que o projeto ensina, tanto do ponto de vista conceitual quanto pr?tico.