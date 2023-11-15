## Modelo de computação

O modelo de computação é um conjunto de operações básicas e seus respectivos custos
## Arquitetura de von Neumann

As principais características da arquitetura:

- A memória armazena somente bits (uma unidade de informação, isto é, um valor igual a 0 ou 1)
- A memória armazena tanto as instruções codificadas quanto os dados sobre os quais as operações serão feitas. Não há nenhuma forma de distinguir dados de código: com efeito, ambos são cadeias de bits
- A memória está organizada em células, que são rotuladas com seus respectivos índices de forma natural (por exemplo , a célula de número 43 vem depois da 42). Os índices começam em 0. O tamanho das células pode variar; os computadores modernos definem um byte (oito bits) como o tamanho de uma célula de memória. Desse modo, o byte de número 0 armazena os oito primeiros bits na memória, e assim sucessivamente
- O programa é constituído de instruções que são buscadas uma após a outra. Sua execução será sequencial, a menos que uma instrução jump especial seja executada

A linguagem Assembly para um dado processador é uma linguagem de programação constituída de mnemônicos para cada possível instrução binária codificada (código de máquina). Ela deixa a programação em código de máquina muito mais simples, pois o programador então não precisa memorizar a codificação binária das instruções, apenas seus nomes e parâmetros

*Uma arquitetura nem sempre define um conjunto exato de instruções, de modo diferente de um modelo de computação*
## Arquitetura Intel 64

Cada modelo foi desenvolvido visando preservar a compatibilidade com modelos mais antigos. Os processadores são capazes de operar em uma série de modos: modo real, protegido, virtual etc. Se não for explicitamente especificado, descrevemos o funcionamento da CPU no chamado **modo longo** (long mode), que é o mais recente
## Extençoes da arquitetura

- **Registradores**: São células de memória colocadas diretamente no chip da CPU. São muito rápidas entre os circuitos porém são muito caras e complicadas. Os acessos aos registradores não utilizam os barramentos
- **Pilha de hardware**: Uma pilha, em geral, é uma estrutura de dados. Ela aceita duas operações: *push* de um elemento no topo e *pop* do elemento superior. Uma pilha é usada não só em processamento mas também para armazenar variáveis locais e implementar sequências de chamadas de função nas linguagens de programação
- **Interrupções**: Esse recurso permite alterar a ordem de execução dos programas com base em eventos externos ao programa. Depois que um sinal é capturado, a execução de um programa é suspensa, alguns registradores são salvos e a CPU começa a executar uma rotina especial para lidar com a situação
- **Anéis de proteção**: Uma CPU está sempre em um estado que corresponde a um dos chamados anéis de proteção (protection rings). Cada anel define um conjunto de instruções permitidas. O anel zero permite executar qualquer instrução do conjunto completo de instruções da CPU e, desse modo, é o mais privilegiado. O terceiro permite somente instruções mais seguras. Uma tentativa de executar instruções privilegiadas resulta em uma interrupção. Os outros dois anéis (primeiro e segundo) são intermediários, e os sistemas operacionais modernos não utilizam eles
- **Memória virtual**: Essa é uma abstração sobre memória física, que ajuda a distribuí-la entre os programas de uma maneira mais segura e eficiente. Também isola os programas uns dos outros
## Registradores

A troca de dados entre CPU e memória é uma parte crítica dos processamentos em um computador de von Neumann. As instruções precisam ser buscadas na memória, os operandos também e algumas instruções igualmente armazenam os resultados na memória. Isso cria um gargalo e resulta em tempo de CPU desperdiçado quando ele espera os dados de resposta do chip de memória. Para evitar esperas constantes, um processador era equipado com suas próprias células de memória, chamadas de registradores
## Registradores de propósito geral

Na maior parte das vezes, um programador trabalhará com registradores de propósito geral. Eles são intercambiáveis e podem ser usados em vários comandos distintos

| **Nome** | **Alias** | **Descrição** |
| :--: | :--: | :--: |
| r0 | rax | Uma espécie de "acumulador", usado em instruções aritméticas |
| r3 | rbx | Registrador base. Era usado para endereçamento da base nos primeiros modelos do processador |
| r1 | rcx | Usado para ciclos (por exemplo loops) |
| r2 | rdx | Armazena dados durante operações de entrada/saída |
| r4 | rsp | Armazena o endereço do elemento do topo da pilha de hardware |
| r5 | rbp | Base do stack frame |
| r6 | rsi | Índice de origem em comandos de manipulação de strings (como movsd) |
| r7 | rdi | Índice de destino em comandos de manipulação de strings (como movsd) |
| r8 - r15 | Não há | Surgiram depois. Usados principalmente para armazenar variáveis temporárias |

Endereçar parte de um registrados é possível. Para cada registrador, podemos endereçar seus 32 bits menos significativos, os 16 bits menos significativos ou os 8 bits menos significativos

Quando os nomes r0,...,r15 são usados, isso é feito com a adição de um sufixo apropriado ao nome de um registrador:
- d para double word - 32 bits menos significativos
- w para word - 16 bits menos significativos
- b para byte - 8 bits menos significativos

Por exemplo:

- r7b é o byte menos significativo do registrados r7
- r3w é constituído dos dois bytes menos significativos do registrados r3
- r0d é constituído dos quatro bytes menos significativos do registrados r0

Os nomes alternativos também permitem endereçar as partes menores

![[Decomposição de rax.png]]

A convenção de nomenclatura para acessar partes de rax, rbx, rcx e rdx segue o mesmo padrão. Os outros quatro registradores não permitem acesso aos seus segundos bytes menos significativos. A nomenclatura do byte menos significativo difere um pouco para rsi, rdi, rsp e rbp 

![[Pasted image 20231115112316.png]]

![[Pasted image 20231115125600.png]]

Em geral, os programadores se atêm aos nomes alternativos para os oito primeiros registradores de propósito geral (r0-r7). Os outros oito registradores (r8-r15) só podem ser nomeados usando uma convenção com indexação

***Inconsistência em escritas** Todas as leituras de registradores menores se comportam de forma óbvia. As escritas nas partes de 32 bits, porém, preenchem os 32 bits mais significativos do registrador completo com bits de sinal. Por exemplo, zerar eax zerará todo o rax; aramzenar -1 em eax preencherá os 32 bits mais signifativos com uns. Outras escritas (por exemplo, em partes de 16 bits) se comportam conforme esperado: não afetam nenhum dos demais bits*
## Outros registradores

Os outros registradores apresentam significados especiais. Alguns registradores têm importância para todo o sistema e, desse modo, não podem ser modificados, execto pelo sistema operacional

Um programador tem acesso ao registrador rip. É um registrador de 64 bits, que sempre armazena um endereço da próxima instrução a ser executada. Assim, sempre que qualquer instrução é executada, rip armazenará o endereço da próxima instrução

Outro registrador acessível se chama rflags. Ele armazena flags, que refletem o estado atual do programa - por exemplo, qual foi o resultado da última instrução aritmética: se foi negativo, se um overflow ocorreu, etc. Suas partes menores são chamadas de eflags (32 bits) e flags (16 bits)
## Registradores de sistema

Alguns registradores foram projetados para serem usados pelo sistema operacional. Esses registradores não armazenam valores usados em processamentos. Em vez disso, armazenam informações necessárias às estruturas de dados utilizadas por todo o sistema.

Alguns desses registradores são listados a seguir:

- **cr0, cr4** - armazenam flags relacionadas a diferentes modos do processador e à memória virtual
- **cr2, cr3** - são usados para suporte à memória virtual
- **cr8 (tpr)** - é usado para efetuar um ajuste fino no mecanismo de interrupção
- **efer** - é outro registrador de flag usado para controlar os modos do processador e extensões
- **idtr** - armazena o endereço da tabela de descritores de interrupção
- **gdtr e ldtr** - armazenam os endereços das tabelas de descritores
- **cd, ds, ss, es, gs, fs** - são os chamados *registradores de segmento*. O mecanismo de segmentação que oferecem é considerado legado há muitos anos, mas parte dele continua sendo usada para implementar o modo privilegiado
## Anéis de proteção

Os anéis de proteção (protection rings) são um dos mecanismos projetados para limitar a capacidade das aplicações por razões de segurança e de robustez. Cada tipo de instrução está ligado a um ou mais níveis de privilégio, e não é executável em outros níveis. O nível atual de privilégio é armazenado de algum modo.

O Intel 64 tem quatro níveis de privilégio, dos quais somente dois são usados na prática: o anel 0 (o mais privilegiado) e o anel 3 (o menos privilegiado)

No modo londo, o número do anel de proteção atual é armazenado nos dois bits menos significativos do registrador cs (e duplicados nos bits de ss)
## Pilha de hardware

Se estivermos falando de estruturas de dados em geral, uma pilha é uma estrutura de dados com duas operações: um novo elemento pode ser colocado no topo da pilha (push) e o elemento do topo pode ser retirado da pilha (pop)

As pilhas são implementadas com duas instruções de máquina (push e pop) e um registrador (rsp). O registrador rsp armazena o endereço do elemento que está no topo da pilha

As instruções executam do seguinte modo:

- *push* argumento
1. Conforme o tamanho do argumento (2, 4, 8 bytes) o valor de rsp será decrementado de 2, 4 ou 8
2. Um argumento é armazenado na memória começando no endereço obtido do rsp modificado

- *pop* argumento
1. O elemento que está no topo da pilha é copiado para o registrador/memória
2. rsp é incrementado com o tamanho de seu argumento

Alguns fatos importantes sobre pilha:

- Não há uma situação de pilha vazia, mesmo que não tenhamos executado push nenhuma vez. Um algoritmo pop pode ser executado de qualquer modo, provavelmente retornando um elemento do "topo" contendo lixo
- A pilha cresce em direção ao endereço zero
- Quase todos os tipos de operando são considerados inteiros com sinal e, desse modo, podem ser expandidos com um bit de sinal. Por exemplo, executar um push com um argumento B9 resultará nos seguintes dados armazenados na pilha:

	 0xff b9, 0xffffffb9 ou 0xff ff ff ff ff ff ff b9

	Por padrão, push utiliza um tamanho de operando de 8 bytes. Portanto, uma instrução push -1 armazenará 0xff ff ff ff ff ff ff ff na pilha

