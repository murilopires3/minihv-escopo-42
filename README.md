## MiniHV — Hypervisor Simulator

MiniHV é um projeto de simulação de conceitos fundamentais de sistemas operacionais e virtualização. Seu objetivo é representar, em um ambiente controlado, o gerenciamento de máquinas virtuais e recursos computacionais, permitindo observar o comportamento de mecanismos de alocação, escalonamento e sincronização.

Status: Em desenvolvimento.

Objetivos

* Simular a criação, o gerenciamento e a execução de máquinas virtuais (VMs).
* Representar o gerenciamento de recursos compartilhados.
* Explorar algoritmos de alocação de memória e escalonamento.
* Demonstrar conceitos de concorrência, sincronização e gerenciamento de processos.
* Permitir a observação do estado interno do simulador durante sua execução.

Conceitos abordados

Os mecanismos abaixo devem ser documentados conforme sua implementação no projeto.

Conceito	Descrição
Máquinas virtuais	Representação de ambientes de execução isolados.
Gerenciamento de memória	Simulação de alocação e liberação de memória.
Escalonamento	Seleção e organização de tarefas para execução.
Sincronização	Coordenação do acesso a recursos compartilhados.
Gerenciamento de processos	Controle do estado e da execução de tarefas simuladas.

Tecnologias

As linguagens, bibliotecas e ferramentas utilizadas serão documentadas conforme a implementação do projeto.

Requisitos

Consulte as dependências e os requisitos de ambiente definidos no projeto antes de executar o simulador.

Como executar

1. Clone o repositório:

git clone https://github.com/murilopires3/minihv-escopo-42.git

2. Acesse a pasta do projeto:

cd minihv-escopo-42

3. Entre no diretório que contém a implementação e siga as instruções de instalação e execução específicas da tecnologia utilizada.

Arquitetura

O MiniHV busca organizar a simulação em componentes responsáveis pela representação das máquinas virtuais, pelo gerenciamento de recursos e pela execução das operações definidas no escopo do projeto.

A arquitetura e as responsabilidades de cada módulo serão descritas conforme os componentes efetivamente implementados.

Limitações

O MiniHV é um simulador conceitual. A representação de virtualização neste projeto não implica a criação de máquinas virtuais reais, a execução de sistemas operacionais convidados ou o isolamento de hardware fornecido por um hipervisor de produção.

Objetivo acadêmico

Projeto desenvolvido para estudar e aplicar conceitos de sistemas operacionais, virtualização, gerenciamento de recursos computacionais e concorrência.