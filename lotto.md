# lotto

###### Solved by @beatriz-a-cardozo & @CupNudous

## About the Challenge

O desafio 'lotto' é apresentado apenas com uma frase 'Mãe! Fiz um programa de loteria como dever de casa. Você quer jogar?' e o secure shell de acesso ao código.

## Solution

Ao estabelecer a conexão do shell, a primeira coisa a se fazer é vasculhar o que tem no diretório atual. Com um ``ls``, os arquivos/diretórios que estão disponíveis ficam a mostra, e logo já se encontra algo importante, o arquivo **lotto.c**, que é muito provavelmente o código ao qual o enunciado se referia. 

O ideal então, é que se analise o código para confirmar se é esse mesmo, para isso, uma boa prática é colocar no terminal ``cat lotto.c``. Assim, o conteúdo do arquivo é impresso:

[![Captura-de-tela-2025-02-24-173057.png](https://i.postimg.cc/dt7QmkVf/Captura-de-tela-2025-02-24-173057.png)](https://postimg.cc/Vd8czNM9)

[![Captura-de-tela-2025-02-24-173143.png](https://i.postimg.cc/B6Ys249s/Captura-de-tela-2025-02-24-173143.png)](https://postimg.cc/FdL24tHn)

[![Captura-de-tela-2025-02-24-173215.png](https://i.postimg.cc/yxDKPtsR/Captura-de-tela-2025-02-24-173215.png)](https://postimg.cc/dZcgQW7Q)

Acima está a captura de tela do terminal, com o código inteiro, e a função ``help`` já explica o que acontece dentro dele. Basicamente, é um jogo de loteria em que o usuário insere 6 números naturais menores que 46, com o objetivo de acertar os 6 números da loteria **para obter a flag**, mas como que são gerados esses números? Eles são gerados de maneira aleatória de acordo com o arquivo '/dev/urandom'. De acordo com o programador, isso faz com que a chance de ganhar seja 1/8145060, mas ao analisar o código a fundo, é fácil de perceber que não está funcionando dessa maneira.

O trecho abaixo é a parte em que é realizada a comparação entre a entrada do usuário e o número da loteria, caso um deles batam, a variável 'match' é incrementada, assim, **se 'match' for igual a 6, a flag será impressa no terminal**. O problema está no loop que é usado para fazer essa comparação, pois o segundo loop é repitido 6 vezes em todas as chegagens do primero, logo, se o usuário acertar apenas um número, a variável 'match' será incrementada 6 vezes, e acarretará na vitória. Logo, a chance de conseguir a flag é apenas de 1/46.

[![Captura-de-tela-2025-02-24-173143.png](https://i.postimg.cc/cC09185L/Captura-de-tela-2025-02-24-173143.png)](https://postimg.cc/472bBngD)

Logo não há necessidade de fazer qualquer alteração no arquivo, apenas roda-lo e chutar um número qualquer (em ASCII) várias vezes até obter o resultado, como feito abaixo:

[![Captura-de-tela-2025-02-24-183343.png](https://i.postimg.cc/sghp8gw6/Captura-de-tela-2025-02-24-183343.png)](https://postimg.cc/4YX7HsCV)

E assim, a flag é printada após algumas tentativas.

>flag: 'sorry mom... I FORGOT to check duplicate numbers... :('
