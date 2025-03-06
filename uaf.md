# uaf

###### Solved by @beatriz-a-cardozo & @CupNudous

## About the Challenge

O desafio 'uaf' gira em torno da vulnerabilidade 'use after free', indicado tanto pelo nome do exerício quanto pelo enunciado. 

## Solution

Usando o enunciado para acessar o shell, ao garantir a conexão com o user e a senha dados, é aberto um diretório simples e analisando seu conteúdo, o que mais chama atenção é um arquivo 'flag', porém não há como acessa-lo sem ser o root.

[![Captura-de-tela-2025-02-25-153151.png](https://i.postimg.cc/fLm7zzYC/Captura-de-tela-2025-02-25-153151.png)](https://postimg.cc/R6CJGzsJ)

Antes de tentar qualquer coisa, é bom verificar o código '*uaf.c*' no diretório, usando o comando `cat uaf.cpp`, igual no exemplo abaixo. 

[![Captura-de-tela-2025-02-24-184135.png](https://i.postimg.cc/VLd6n7VQ/Captura-de-tela-2025-02-24-184135.png)](https://postimg.cc/6yxtNhKY)

[![Captura-de-tela-2025-02-24-184152.png](https://i.postimg.cc/152zHpkg/Captura-de-tela-2025-02-24-184152.png)](https://postimg.cc/56519FN1)

Logo de cara, é possível ver o que poderá dar acesso ao usuário root para conseguir a flag. Na classe 'Human' um dos métodos envolve dar acesso ao shell, então esse deve ser o jeito de chegar ao objetivo do desafio.

Esse código utiliza orientação a objetos para demonstrar o uso de ponteiros, e é ai que entra a vulnerabilidade citada no nome do desafio, 'use after free' ocorre quando um ponteiro ainda aponta para um local de memória mesmo após ela ter sido liberada. Então, é possível traçar uma rota do que se deve fazer: pegar algum objeto do código que é alocado, limpar/deletar o endereço,  descobrir endereço de memória para o método *give_shell()* e fazer com que o ponteiro aponte para ele.

Os dois primeiros passos são os mais simples, pois o código te dá *opção 1* de usar um ponteiro que aponta para a classe 'Man'/'Woman' e a *opção 3* para liberar os ponteiros, também é possível usar a *opção 2* para alocar um dado, nesse caso, um novo endereço de memória. Com isso, resta apenas descobir o endereço de memória do *give_shell()* usando o *gdb* para verficiar o código em assembly da main com o comando em `disassemble main`.

[![Captura-de-tela-2025-02-25-171450.png](https://i.postimg.cc/Z0tpX1Wg/Captura-de-tela-2025-02-25-171450.png)](https://postimg.cc/5QgXCKqq)

Abaixo, usando o guia [x64 cheat sheet](https://cs.brown.edu/courses/cs033/docs/guides/x64_cheatsheet.pdf) é possível identificar que o endereço de give_shell() poderia ser reutilizado através da manipulação do ponteiro após a liberação da memória.

[![Captura-de-tela-2025-02-25-172153.png](https://i.postimg.cc/fbpcxtwD/Captura-de-tela-2025-02-25-172153.png)](https://postimg.cc/zLwHNfbc)

A análise do conteúdo do print indica que:

>O ponteiro, após ser liberado, poderia ser redirecionado para give_shell().

Com isso, uma chamada subsequente ao ponteiro resultaria na execução do shell como root.

[![Captura-de-tela-2025-02-25-172441.png](https://i.postimg.cc/nVv4DhNf/Captura-de-tela-2025-02-25-172441.png)](https://postimg.cc/fkRS4Dw8)

[![Captura-de-tela-2025-02-25-173129.png](https://i.postimg.cc/5Ng8skxP/Captura-de-tela-2025-02-25-173129.png)](https://postimg.cc/zVLLB7HW)

Para realizar essa manipulação, os seguintes passos foram executados:

>Criar um objeto (opção 1).

>Liberar a memória do objeto (opção 3).

>Alocar um novo objeto (opção 2), forçando a reutilização do espaço de memória previamente ocupado.

>Manipular a estrutura interna para substituir o ponteiro da VTable pelo endereço da função give_shell().

>Executar o objeto modificado para obter acesso ao shell.

Obtendo acesso ao shell dentro do servidor e acessando a flag.
>flag:'yay_f1ag_aft3r_pwning'