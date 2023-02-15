# Logbook 8

### Lab Environment Setup

Alteramos com sucesso o ficheiro etc/hosts e executamos o servidor através dos comandos docker descritos no guião.

## Task 1

O objetivo desta tarefa é utilizar os comandos SQL, para ganhar à vontade com a tecnologia e ganhar o mínimo de experiência com queries. Os dados da aplicação web estão armazenados numa base de dados MYSQL, que por sua vez está contida no container MYSQL fornecido na virtual machine SEED LABS.

Numa shell no container MYSQL, executamos o comando `# mysql -u root -pdees`, para fazer login.

Iremos usar a base de dados criada pelo SEED LABS, que possui o nome sqllab users. Para acedermos à mesma executamos `use sqllab_users;`

Com `show tables;` visualizamos as relações que estão criadas e com `select * from credential where name = 'Alice'` conseguimos visualizar todas as informações da funcionária Alice, como pretendido.

![](https://i.imgur.com/BonrouH.png)


## Task 2

O objetivo desta segunda tarefa é injetar código SQL (denominado por payload), para conseguir atacar a base de dados da aplicação web e recolher informação privada. O ataque pretendido consiste em autenticar na plataforma, sem saber nenhuma das credenciais presentes na base de dados, sendo que a plataforma apenas aceita logins de funcionários que sabem a sua password.

O seguinte código demonstra como os funcionários são autenticados na aplicação:
```
$input_uname = $_GET[’username’];
$input_pwd = $_GET[’Password’];
$hashed_pwd = sha1($input_pwd);
...
$sql = "SELECT id, name, eid, salary, birth, ssn, address, email,
               nickname, Password
        FROM credential
        WHERE name= ’$input_uname’ and Password=’$hashed_pwd’";
$result = $conn -> query($sql);

// The following is Pseudo Code
if(id != NULL) {
  if(name==’admin’) {
     return All employees information;
  } else if (name !=NULL){
    return employee information;
  }
} else {
  Authentication Fails;
}
```

### Task 2.1

A tarefa é fazer login na aplicação web, como administrador, para poder ver as informações de todos os funcionários. O nome da conta a que queremos aceder é admin. Temos que descobrir o que colocar nos campos username e Password, para fazermos login com sucesso.

Para conseguirmos completar esta tarefa colocamos no campo username a seguinte *string* "admin' #". Desta maneira, conseguimos fazer com que a operação SELECT utilizada na base de dados tenha apenas a condição `WHERE username = 'admin'` em vez de por exemplo `WHERE username = 'admin' AND password = 'password'`. Isto acontece porque a (') termina a string do username e o (#) comenta a seguinte condição da operação.

![](https://i.imgur.com/vmw7v8V.png)

![](https://i.imgur.com/J2AZOAh.png)


### Task 2.2

Nesta tarefa, pretendemos o mesmo objetivo atingido na tarefa anterior, mas desta vez sem utilizar a página web. Podemos enviar um HTTP GET REQUEST para a aplicação web (com os dois parâmetros, username e Password) através da execução do comando `$ curl ’www.seed-server.com/unsafe_home.php?username=alice&Password=11’` na shell, por exemplo. 

Tem que se ter cuidado com os caracteres na execução dos comandos, porque pode fazer com que a shell interprete de uma forma diferente da pretendida. Por exemplo, se quisermos colocar plicas (') nos campos username e Password, teremos que utilizar antes %27. Se quisermos colocar um espaço, usamos %20.

Com a execução do comando `curl www.seed-server.com/unsafe_home.php?username=admin%27%23&Password=` tivemos acesso à página html do admin, tal como pretendido. Conseguimos observar as informações dos funcionários no código apresentado no resultado do comando.

![](https://i.imgur.com/ceHch5S.png)


### Task 2.3

O objetivo desta tarefa é conseguir modificar a base de dados da aplicação web, através da exploração da mesma vulnerabilidade encontrada (na página de login). Desta vez queremos correr duas operações SQL, sendo que a segunda será um update ou um delete.

Não conseguimos executar duas instruções SQL neste ataque porque existe o uso de uma função que previne a injeção de duas queries dentro de uma, que é a [mysql `query` function](https://www.php.net/manual/en/mysqli.query.php). Se fosse utilizada a função [mysql's `multi-query`](https://www.php.net/manual/en/mysqli.multi-query.php) já seria possível a injeção de múltiplas queries.

![](https://i.imgur.com/v1VipR8.png)


## Task 3

Existe uma página para a edição das informações pessoais de cada funcionário. É possível aceder a esta página após fazer login com sucesso.
Quando é submetida uma edição de perfil, é executado o seguinte código:
```
$hashed_pwd = sha1($input_pwd);
$sql = "UPDATE credential SET
   nickname=’$input_nickname’,
   email=’$input_email’,
   address=’$input_address’,
   Password=’$hashed_pwd’,
   PhoneNumber=’$input_phonenumber’
WHERE ID=$id;";
$conn->query($sql);
```

### Task 3.1

Os funcionários apenas conseguem alterar os campos Nickname, Email, Address, Phone Number e Password. Nesta tarefa  o valor armazenado no campo do salário da Alice, através desta página de edição. O campo do salário possui o nome 'salary'.

Ao colocar a string `Alice', Salary='100000000' WHERE EID='10000' #` no campo do nickname, conseguimos fazer com que a instrução UPDATE também altere o campo do salário, como pretendido.

![](https://i.imgur.com/LyBaB0g.png)

### Task 3.2

Agora pretendemos reduzir o salário do patrão Boby para 1$, por causa do seu mau comportamento com a Alice. Colocamos `Boby', Salary='1' WHERE EID='20000' #` no campo do nickname, à semelhança da tarefa anterior, para completar o objetivo.

![](https://i.imgur.com/TBwBLBg.png)

![](https://i.imgur.com/gMrBCrF.png)

### Task 3.3

Nesta tarefa pretendemos alterar a password do Boby, para depois poderem ser feitos mais danos. Temos que conseguir fazer login com a nova password da conta do Boby. A base de dados armazena o valor hash das passwords em vez do texto exato da password.

Utizamos um SHA 1 decoder para obtermos o valor hash da password que queremos introduzir.

![](https://i.imgur.com/8mIDGvi.png)

Colocamos `Boby', Password='7110eda4d09e062aa5e4a390b0a572ac0d2c0220' where Name = 'Boby' #` no campo do nickname para assim alterarmos a password para '1234'. O valor presente na string que colocamos, é o valor hash da nova password.

![](https://i.imgur.com/x5gI06k.png)

![](https://i.imgur.com/sENqWZQ.png)


### CTFs

## Desafio 1

A nossa tarefa é entrar no servidor com a conta admin.
Explorando a source da página de entrada, encontramos a seguinte linha:

![](https://i.imgur.com/B1clQOr.png)

Se o nosso username for:

![](https://i.imgur.com/dr19oU8.png)

Estaremos a terminar a string mais cedo, e tudo o que vem a seguir ao "--" passará a ser apenas um comment.

Usamos uma qualquer password já que não podemos deixar em branco.

![](https://i.imgur.com/16df3BN.png)

"I'm in."

![](https://i.imgur.com/kRUhnZF.png)

## Desafio 2

Começamos por explorar a source, e verificar as proteções do executável.

![](https://i.imgur.com/F6YtrTz.png)

Certamente o nosso professor diria "A stack é executável, e não há canário. Ya?"

A cereja no topo do bolo é mesmo o programa dar-nos o endereço exato do buffer, em runtime.

![](https://i.imgur.com/wZftkU3.png)

Algo que podemos extrair no nosso script.

![](https://i.imgur.com/gLIPRTd.png)

A receita da payload, é:
- Shellcode,
- (tamanho do buffer + 8 - tamanho do shellcode) Caractéres de lixo,
- Endereço do buffer.

O return address fica 8 bytes acima do fim do buffer, daí adicionarmos 8 aos caractéres de lixo.
Também podiamos ter usado um padrão cíclico juntamente com gdb, para encontrar a posição do endereço de retorno, sem ter de a calcular.

![](https://i.imgur.com/TmKFp3r.png)

Chamamos também à atenção que não precisamos de NOP-sled, já que sabemos o endereço exato do buffer em runtime.

![](https://i.imgur.com/rWzrtpw.png)

## Echo

![](https://i.imgur.com/IiUVY27.png)

Boss battle.

Vamos partir do princípio que ASLR está ligado também no servidor.

Comecemos por explorar o programa:

![](https://i.imgur.com/zOYSdJd.png)

Se passarmos pelo menos 20 caractéres, o programa deteta stack smashing (o que significa que o canário foi alterado!).

Isto é estranho, já que nos foi dito que no máximo poderiamos mandar 20 caractéres.
Mas para já há algo mais importante a verificar:

![](https://i.imgur.com/NGMDgZE.png)

O mesmo buffer tem uma vulnerabilidade de format string.

E igualmente interessante:

![](https://i.imgur.com/gxq9zGj.png)

Se passarmos mais do que 99 caractéres ao buffer, os caractéres adicionais não são lidos nessa operação.
Sabemos o máximo de caractéres que podemos passar ao buffer.
Isto também é mais um indicador do que referimos acima, na source poderemos ter:

![](https://i.imgur.com/tMsNaP9.png)

O que explicaria os 99 caractéres, e o overflow caso o buffer tenha menos de 100 bytes.

![](https://i.imgur.com/W1oAIt0.png)

![](https://i.imgur.com/trXH29d.png)

Algumas funções de input de C colocam '\0' além do que é lido para prevenir overruns, fgets é uma delas. (Spoiler: isto vai ser útil mais tarde)

Já que a stack não é executável, podemos usar um ataque chamado Return to Libc, ou ret2libc, que consiste em dar overwrite ao valor de retorno, mas em vez de saltarmos para o shellcode ou NOP-sled, saltamos para um endereço da libc.
Normalmente usa-se a função system, com um único argumento, um pointer para a string "/bin/sh".

Para fazermos o exploit, precisamos de encontrar alguns valores na libc primeiro.

Offset da função system a partir do início da libc:

![](https://i.imgur.com/x8jNNsn.png)

Offset da string "/bin/sh" a partir do início da libc:

![](https://i.imgur.com/MUgEO8Z.png)

Precisamos também do endereço base da libc na nossa máquina SEM ASLR.

Desativamos ASLR:

![](https://i.imgur.com/xahoQ5w.png)

IMPORTANTE: não esquecer de usar a library que nos é dada:

![](https://i.imgur.com/OtKy1PI.png)

E com gdb encontramos onde está mapeada a libc:

![](https://i.imgur.com/SkEgJiL.png)

Qualquer endereço entre 0xf7d89000 e 0xf7f2b000 é um valor da libc, ou seja, se encontrarmos um desses valores, podemos usá-lo como referência e encontrar a distância entre um endereço sem ASLR, e com ASLR.

Usamos um script para descobrir valores na stack.

![](https://i.imgur.com/rKKvPTu.png)

![](https://i.imgur.com/yNdT9mc.png)

Index 8 é o canário, se corrermos várias vezes reparamos que acaba sempre em 0x00, teremos de usar fgets para inserir um '\0' nessa posição, index 20 do buffer.

Index 11 é um valor da libc, podemos usar este valor 0xf7daa519 para calcular a diferença devido a ASLR.

Já que o programa corre numa loop, enviamos payloads diferentes em várias iterações, para fins diferentes:
 - primeiro extrair canário, e valor de referência para calcular o offset ASLR,
 - segundo dar overwrite ao endereço de retorno,
 - terceiro escrever '\0' no index 20 do buffer.

Criámos uma função para facilitar a passagem pela loop e o envio de payloads para o buffer vulnerável:

![](https://i.imgur.com/ygDpZ42.png)

Tudo a postos:

![](https://i.imgur.com/hijaVsU.png)

Enviamos o primeiro payload, extraímos o canário e o valor de referência, e calculamos os endereços reais da função system e da string "/bin/sh":

![](https://i.imgur.com/gB2opT7.png)

Enviamos a segunda payload:

![](https://i.imgur.com/07j0gFk.png)

(Adicionamos 1 ao canário porque não podemos enviar nenhum byte a 0, e sabemos que o LSB do canário é 0)

E finalmente enviamos 0 de volta ao LSB do canário:

![](https://i.imgur.com/K59lr6I.png)
![](https://i.imgur.com/FRcSbjX.png)

![](https://i.imgur.com/Kqg9ndz.png)

![](https://i.imgur.com/OuyDEK2.png)

