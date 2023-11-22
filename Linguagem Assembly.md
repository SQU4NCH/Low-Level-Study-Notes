## Configurando o ambiente

- Sistema Operacional: Linux
- Compilador Assembly: NASM 2.11.05
- Compilador C: GCC 4.9.2
- GNU Make 4.0
- GDB como depurador
## Escrevendo "Hello, world"

Seguindo a ideologia Unix, "tudo é um arquivo". Por meio de arquivos é possível abstrair operações como:
- acesso a dados em disco rígido/SSD
- troca de dados entre programas
- interação com dispositivos externos

Esse programa simplesmente irá mostrar uma mensagem "Hello, world!" na tela. No entanto um programa como esse deve mostrar caracteres na tela, o que não pode ser feito se o programa não estiver executando direto no hardware, para isso é necessário o sistema operacional para gerenciar as atividades. O sistema operacional oferece uma série de rotinas para tratar a comunicação com dispositivos externos, outros programas, sistemas de arquivo, etc... Um programa não pode ignorar o sistema operacional e nem interagir diretamente com os recursos. O programa fica limitado às chamadas de sistema (system calls) que são rotinas oferecidas pelo sistema operacional para o usuário

O Unix identifica um arquivo pelo seu **descritor** assim que ele é aberto por um programa. Um arquivo é aberto por meio da chamada de sistema **open**. Outros 3 arquivos são abertos assim que um programa inicia, são eles: **stdin, stdout** e **stderr**. Seus descritores são 0, 1 e 2 respectivamente. **stdin** é usado para tratar a entrada, **stdout** para tratar a saída e **stderr** é usado para a saída de informações sobre o processo de execução do programa

Para executar o programa a mensagem deve ser escrita em stdout e para isso é  necessário utilizar a chamada de sistema **write**. Ela escreve uma dada quantidade de bytes da memória, começando em um dado endereço, em um arquivo com um dado descritor (nesse caso 1)

código:
```Assembly
global _start

section .data
message: db 'hello, world!', 10

section .text
_start:
	mov rax, 1         ; O número da chamada do sistema que deve ser armazenado em rax
	mov rdi, 1         ; Onde escrever (descritor) - arg1
	mov rsi, message   ; Onde começa a string - arg2
	mov rdx, 14        ; Quantos bytes devem ser escritos - arg3
	syscall            ; Faz a chamada de sistema
```
## Estrutura do programa

Há apenas uma memória tanto para o código quanto para os dados. Entretanto o programador pode querer separa-los. Um programa Assembly é dividido em **seções**. Cada seção tem uma finalidade: por exemplo, **.text** armazena instruções, **.data** é usada para variáveis globais.

Para não precisar usar os valores dos endereços, pode-se usar **rótulos (labels).** Esses são apenas nomes legíveis e endereços. Podem anteceder qualquer comando e, geralmente, estão separados por dois-pontos. Nesse programa tem um label o **\_start**

Um programa Assembly pode ser dividido em vários arquivos. Um deles deve conter o label \_start que marca a primeira instrução do programa e ele deve ser declarado como global

Os comentários começam com ";" e se estendem até o final da linha

A linguagem Assembly é feita de comandos, mas nem todas as construções da linguagem são comandos, algumas controlam o processo de tradução e normalmente são chamadas de **diretivas**. Nesse exemplo há três diretivas: global, section e db

*A linguagem Assembly, em geral, não faz distinção entre letras maiúsculas e minúsculas, mas isso não vale para os nomes de labels. mov, mOV, Mov são o mesmo comando, porém global _start e global _START não são iguais. Os nomes de seção também distinguem letras maiúsculas de minúsculas*

A diretiva **db** é usada para criar bytes de dados. Em geral os dados são definidos usando uma dessas diretivas:
- db - bytes
- dw - (words), correspondem a 2 bytes
- dd - (double words), correspondem a 4 bytes
- dq - (quad words), correspondem a 8 bytes

```Assembly
message: db 'hello, world!', 10
```

Letras, digitos e outros caracteres são codificados em ASCII. Na linha acima, começamos com o endereço do label "message" e então é armazenado cada código da tabela ASCII correspondente as letras da frase, e no final é adicionado um byte igual a 10, que representa o caractere especial de nova linha
## Instruções básicas

A instrução **mov** é usada para escrever um valor no registrador ou memória. O valor pode ser obtido de outro registrador ou memória, ou pode ser um valor imediato. Entretanto:
- mov não pode copiar dados da memória para a memória
- os operandos de origem e destino devem ter o mesmo tamanho

A instrução **syscall** é usada para fazer chamadas de sistema (system calls) em sistemas \*nix. Cada chamada de sistema tem um número único. Para executá-la:
- O registrador rax deve armazenar o número da chamada de sistema
- Os seguintes registradores devem armazenar seus argumentos: rdi, rsi, rdx, r10, r8 e r9
	Uma chamada de sistema não pode aceitar mais do que seis argumentos
- Executar a instrução syscall

No programa "hello world" foi utilizado a syscall **write** simples. Ela aceita:
- Um descritor de arquivo
- O endereço do buffer
- A quantidade de bytes para escrever

Para compilar o programa foi utilizado a seguinte sequencia de comandos:

```shell
> nasm -felf64 hello.asm -o hello.o
> ld -o hello hello.o
> chmod u+x hello
```

Executando o programa:

![[Segmentation fault.png]]

A mensagem foi exibida, porém o programa acabou causando um erro. O problema é que não foi escrita nenhuma instrução após a syscall e o programa continuou executando lendo o "lixo" existente na memória

Para resolver esse problema é necessário utilizar a chamada de sistema **exit** que encerra o programa de forma correta

```Assembly
global _start

section .data
message: db 'hello, world!', 10

section .text
_start:
	mov rax, 1         ; O número da chamada do sistema que deve ser armazenado em rax
	mov rdi, 1         ; Onde escrever (descritor) - arg1
	mov rsi, message   ; Onde começa a string - arg2
	mov rdx, 14        ; Quantos bytes devem ser escritos - arg3
	syscall            ; Faz a chamada de sistema

	mov rax, 60        ; Número da syscall exit
	xor rdi, rdi       ; Valor que será retornado pela syscall
	syscall
```

Ao executar a instrução xor rdi, rdi o valor de rdi é zerado, e esse é o valor do argumento utilizado pela syscall exit, que será retornado pelo programa

![[hello.asm.png]]

continua....