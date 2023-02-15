# Logbook 4

## Task 1
- Usamos os comandos printenv e env para imprimir as variáveis de ambiente. Para imprimir uma variável em concreto, usamos o comando printenv seguido do nome da variável de ambiente ou "env | grep" também seguido pelo nome da variável.
- Executamos os comandos export e unset para ativar e desativar as variáveis de ambiente. Isto tudo feito na bash.

Printenv:![](https://i.imgur.com/W5N1TKh.png)
Env:![](https://i.imgur.com/4e9lhh2.png)
Printenv PWD:
![](https://i.imgur.com/mH41xV4.png)  
Export:![](https://i.imgur.com/6fkGIjO.png)

## Task 2
O objetivo desta tarefa é saber se um processo filho herda as variáveis de ambiente do seu pai.
1) Compilamos e executamos o seguinte código:
```
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
extern char **environ;
void printenv(){
    int i = 0;
    while (environ[i] != NULL) { 
        printf("%s\n", environ[i]);
        i++;
    } 
}
void main() {
  pid_t childPid;
  switch(childPid = fork()) {
    case 0: /* child process */ 
      printenv(); ➀
      exit(0);
    default:  /* parent process */
      //printenv(); ➁ 
      exit(0); 
  }
}
```

2) Posto isto comentamos o primeiro printenv() e descomentamos o segundo. 

3) Ambos os resultados foram guardados em ficheiros e com o comando diff comparamos.

##### Conclusões
Não houve diferenças no *output* (presente no ficheiro file2) de ambos os passos, portanto as variáveis de ambientes do processo pai são herdadas pelo processo filho.


## Task 3
O objetivo de estudo desta tarefa é ficar a saber como as variáveis de ambiente são afetadas quando um novo programa é executado via execve(). Esta função chama uma *system call* para carregar um comando e executá-lo, mas nunca retorna. Nenhum processo é criado, e em vez disso os dados do processo que fez a chamada são sobrescritos. A função execve() corre o programa novo dentro do processo de chamada.
 
1) Compilamos e corremos o seguinte código:
```
#include <unistd.h>
extern char **environ;
int main(){
  char *argv[2];
  argv[0] = "/usr/bin/env";
  argv[1] = NULL;
  execve("/usr/bin/env", argv, NULL); ➀
  return 0 ; 
}
```
Neste ponto não houve nenhum *output*.

2) Mudamos a linha com o apontamento "1" para:
```
execve("/usr/bin/env", argv, environ);
```
O *output*, com esta alteração no programa, está no ficheiro file3.

#### Conclusões
As variáveis de ambiente são automaticamente herdadas pelo novo programa. No primeiro ponto passamos NULL como argumento, em vez da matriz environ como no segundo ponto. Com a correção e adição de environ no parâmetro da função execve, conseguimos guardar as variáveis de ambiente (do processo pai) e imprimi-las.

## Task 4

Nesta tarefa o estudo é sobre como as variáveis de ambiente são afetadas quando um programa novo é executado pela função system. Esta função é utilizada para executar um comando e executa "/bin/sh -c comando", ou seja, executa /bin/sh e pede à shell para executar o comando, ao contrário da função execve que executa diretamente o comando.
A função system usa a função execl na sua implementação para executar /bin/sh. A função execl chama execve() e passa como argumento o *array* das variáveis de ambiente. Portanto, ao usar system(), as variáveis de ambiente, do processo que faz a chamada, são passadas para o novo programa /bin/sh.

1) Compilamos e corremos o seguinte código:
```
#include <stdio.h>
#include <stdlib.h>
int main(){
    system("/usr/bin/env");
    return 0 ;
}
```
O resultado está no file4. Como esperado, o *output* contém as variáveis de ambiente do processo pai.

## Task 5

Set-UID é um mecanismo de segurança importante em sistemas operativos UNIX. Quando se corre um programa Set-UID ele assume privilégios de *owner*. Dá para fazer imensas coisas interessantes utilizando Set-UID, mas é bastante arriscado, porque aumenta os privilégios do utilizador. Apesar dos comportamentos de programas Set-UID serem decididos pela lógica do programa em si, os utilizadores podem afetá-los através da manipulação das variáveis de ambiente. 

1) Compilamos o seguinte código, que imprime todas as variáveis de ambiente do processo atual:
```
#include <stdio.h>
#include <stdlib.h>
extern char **environ;
int main(){
    int i = 0;
    while (environ[i] != NULL) {
        printf("%s\n", environ[i]);
        i++;
    }
}
```
2) Mudamos a propriedade do programa para *root*, tornámos-lo um programa Set-UID e executamos os seguintes comandos:
```
$ sudo chown root a.out
$ sudo chmod 4755 a.out
```

3) Utilizamos o comando export para alterar as variáveis de ambiente PATH, LD_LIBRARY_PATH e criamos a variável ANY_NAME.

Os nossos resultados e execuções estão nos seguintes *screenshots*:

![](https://i.imgur.com/hAJ66SS.png)

![](https://i.imgur.com/Fy39AFp.png)
![](https://i.imgur.com/iLKtF0H.png)

#### Conclusões
Após executarmos todos os passos descritos anteriormente, conseguimos visualizar as três variáveis de ambiente mencionadas acima com os valores que inserimos.

## Task 6
Chamar a função system num programa Set-UID é bastante perigoso, por causa do programa da shell ser invocado. O comportamento do programa da shell pode ser afetado pelas variáveis de ambiente, como por exemplo, a PATH. As variáveis de ambiente são fornecidas pelo utilizador, e este pode ser malicioso. 
Ao alterar estas variáveis os utilizadores conseguem controlar o comportamento do programa Set-UID.

1) O seguinte código cria um programa "myls".c. No código apenas é utilizado o caminho do comando ls relativo ao diretório, em vez do caminho absoluto, daí poderem ser gerados problemas. Compilamos, alteramos a propriedade para *owner* e tornámos-lo um programa SET-UID.
```
#include <stdio.h>
#include <stdlib.h>

int main(){
    system("ls");
    return 0;
}
```
2) Compilamos o seguinte código, mudamos a sua propriedade para *owner* e tornámo-lo um programa Set-UID:
```
#include <stdio.h>
#include <stdlib.h>

int main() {
    printf("Programa malicioso\n");
    system("chmod 777 ~/Downloads/FSI/some_file");
    return 0;
}
```

3) Executamos o seguinte comando, para mudar a variável PATH na Bash:
```
$ export PATH=~/Downloads/FSI/:$PATH
```
Com este comando adicionamos o diretório ~/Downloads/FSI/ ao início PATH.

4) Ao executar o nosso programa do ponto 2 e executar o comando ls -l, para ver as permissões dos ficheiros, conseguimos ver que o programa alterou as propriedades do ficheiro.

5) O único obstáculo é que o sistema acabará por levar a uma chamada a /bin/dash, que possui contramedidas para esta situação, diminuindo as permissões do processo para as permissões de quem chama. Para isso mudamos o programa da shell para /bin/zsh, com o seguinte comando:
```
$ sudo ln -sf /bin/zsh /bin/sh
```

Os resultados estão nos *screenshots* seguintes:

![](https://i.imgur.com/kuEKgWw.png)

![](https://i.imgur.com/h914Iec.png)

---

# CTF

### Vulnerabilidade encontrada:
Booster for Woocommerce, plugin do WordPress, é vulnerável a bypass de autenticação por e-mail até à versão 5.4.3, CVE-2021-34646.

### Utilizadores identificados:
- Orval Sanford (ID?)
- admin (ID: 1)

### Como executamos o ataques:
Encontrámos um script no site exploit-db.com que automaticamente gerava tokens e verificava se "passavam" como login para um certo user, user esse sendo o de ID 1 no nosso caso.

### Versões de Software:
- WordPress: 5.8.1
- WooCommerce: 5.7.1
- Booster for WooCommerce: 5.4.3
