# LOGBOOK 11

### Lab Environment Setup

Completamos todos os pontos presentes no guião deste lab, isto é, DNS Setup e Container Setup and Commands.

## Task 1

Por padrão, OpenSSL usa o arquivo de configuração /usr/lib/ssl/openssl.cnf. Como precisamos fazer alterações neste arquivo, vamos copiá-lo no nosso diretório atual e instruir o OpenSSL a usar essa cópia.
A secção [CA default] do arquivo de configuração mostra a configuração padrão que precisamos preparar. Precisamos criar vários subdiretórios. Descomentamos a linha de "unique_subject" para permitir a criação de certificações com o mesmo assunto.

![](https://i.imgur.com/uSz61iW.png)

Para o arquivo index.txt, criamos um arquivo vazio. Para o arquivo serial, colocamos um único número em formato de string. Depois de definir o arquivo de configuração openssl.cnf, conseguimos criar e emitir certificados.

![](https://i.imgur.com/XZdlwez.png)

Executamos o comando ` openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -keyout ca.key -out ca.crt` para gerar o certificado *self-assigned* para a CA:

![](https://i.imgur.com/nQLp3mi.png)

![](https://i.imgur.com/kLwxXqq.png)

Utilizamos os seguintes comandos para examinar o conteúdo decodificado do certificado X509 e a chave RSA (-text significa decodificar o conteúdo em texto simples; -noout significa não imprimir a versão codificada):
```
openssl x509 -in ca.crt -text -noout
openssl rsa  -in ca.key -text -noout
```

[Ficheiro com certificado de chave pública - ca.crt](ca_crt.txt)
[Ficheiro com chave privada da CA - ca_key](ca_key.txt)

**Questions:**
**1) What part of the certificate indicates this is a CA’s certificate?!**

![](https://i.imgur.com/W2VrYgF.png)

"CA:TRUE" indica que é um certificado da CA.

**2) What part of the certificate indicates this is a self-signed certificate?**

![](https://i.imgur.com/mCuW61B.png)

Quando o X509v3 Subject Key Identifier é igual ao X509v3 Authority Key Identifier, estamos perante um certificado *self-assigned*.

**3) In the RSA algorithm, we have a public exponent e, a private exponent d, a modulus n, and two secret numbers p and q, such that n = pq. Please identify the values for these elements in your certificate and key files.**

public exponent - e:

![](https://i.imgur.com/5imGwQZ.png)

private exponent - d:

![](https://i.imgur.com/pF6bzBb.png)

modulus - n: 

![](https://i.imgur.com/pyFOMD5.png)

secret number 1 - p:

![](https://i.imgur.com/WTwTXP3.png)

secret number 2 - q:

![](https://i.imgur.com/CfzsAnb.png)

## Task 2

Com o seguinte comando geramos um CSR(Certificate Signing Request) para www.bank32.com (utilizamos o nome de servidor presente no guião):
```
openssl req -newkey rsa:2048 -sha256 \
            -keyout server.key   -out server.csr \
            -subj "/CN=www.bank32.com/O=Bank32 Inc./C=US" \
            -passout pass:dees
```

![](https://i.imgur.com/r4NdR4o.png)

Com os seguintes comandos visualizamos o conteúdo decodificado do CSR e dos arquivos de chave privada:
```
openssl req -in server.csr -text -noout
openssl rsa -in server.key -text -noout
```

Adicionamos dois nomes alternativos ao pedido de assinatura de certificado porque são necessários para as tarefas seguintes. Esta etapa foi incluída no *printscreen* colocado acima.

## Task 3

O seguinte comando transforma o pedido de assinatura de certificado (server.csr) num certificado X509 (server.crt), urilizando ca.crt e ca.key da CA:
```
openssl ca -config myCA_openssl.cnf -policy policy_anything \
           -md sha256 -days 3650 \
           -in server.csr -out server.crt -batch \
           -cert ca.crt -keyfile ca.key
```

![](https://i.imgur.com/KJj72w9.png)

Por motivos de segurança, a configuração padrão em openssl.cnf não permite que o comando "openssl ca" copie o campo de extensão do pedido para o certificado final. Para habilitarmos isso, descomentamos a seguinte linha na nossa cópia do arquivo de configuração:

![](https://i.imgur.com/sUGRkwo.png)

Depois de assinarmos o certificado, utilizamos o comando `openssl x509 -in server.crt -text -noout` para imprimir o conteúdo decodificado do certificado e concluímos que os nomes alternativos estão incluídos.

![](https://i.imgur.com/5rZXejp.png)

[server.crt](server_crt.txt)

## Task 4

Através do exemplo do guião, configuramos o nosso próprio site HTTPS.

Demos build, executamos o nosso container Docker e posto isto executamos os seguintes comandos:
```
# a2enmod ssl                 // Enable the SSL module
# a2ensite bank32_apache_ssl  // Enable the sites described in this file
```
O primeiro comando servem para ativar o módulo Apache ssl e o segundo para ativar o site. Estes comandos também são executados no processo de built do container Docker.

O servidor Apache não é iniciado automaticamente no container, porque é preciso introduzir a password para desbloquear a chave privada. Executamos, no container, o comando `# service apache2 start` para iniciar o servidor. Os comandos `# service apache2 stop` e `# service apache2 restart` servem para parar e para reiniciar o servidor, respetivamente.

![](https://i.imgur.com/DzeLIFX.png)

Quando o servidor Apache é iniciado, precisa carregar a chave privada para cada site HTTPS. A nossa chave privada é encriptada, logo, o servidor vai pedir a password para desencriptá-la. Dentro do container, a password utilizada é dees (para este exemplo bank32).

![](https://i.imgur.com/c7mnXXe.png)

Não conseguimos aceder ao site, porque precisamos de carregar o certificado para o firefox. Carregamos o certificado como é indicado no guião e assim conseguimos aceder com o certificado.

![](https://i.imgur.com/89RpjQk.png)

![](https://i.imgur.com/XDGcv2I.png)

## Task 5

O objetivo desta tarefa é mostrar como PKI pode impedir ataques Man-In-The-Middle.

- Selecionamos www.fnac.com como o nosso target website.

- Utilizamos o mesmo certificado do nosso próprio servidor da tarefa anterior, tal como indicado no guião.

![](https://i.imgur.com/j2w5NZz.png)

- Alteramos o ficheiro etc/hosts e o bank32_apache_ssl.conf para quando acedermos ao site www.fnac.com, sermos redirecionados para o nosso servidor malicioso.

![](https://i.imgur.com/QGpENTm.png)
![](https://i.imgur.com/KytT4DC.png)

- Este é o resultado da tentativa de aceder ao site www.fnac.com que resultou num warning por causa do certificado que geramos antes para o www.bank32.com.

![](https://i.imgur.com/t9qkMZs.png)


## Task 6

Nesta tarefa geramos um novo certificado porque quando uma CA é comprometida, nós próprios podemos assinar certificados e fazer-nos passar por outros *sites*.

![](https://i.imgur.com/PR3pvpp.png)

![](https://i.imgur.com/Zj241qU.png)

Assim, já é possível aceder ao site sem que apareça um warning:

![](https://i.imgur.com/caWMZDF.png)

# CTFs

## Desafio 1

Depois de nos conectarmos ao servidor, verificámos que este envia sempre uma flag cifrada com RSA.

![](https://i.imgur.com/nByZ7BF.png)

Para decifrar a flag, usámos o seguinte script:

```python=
from binascii import hexlify, unhexlify
from sympy import *

p = nextprime(2**512) # next prime 2**512
q = nextprime(2**513) # next prime 2**513
n = p*q
e = 0x10001 # a constant -> expoente público
d = pow(e,-1,((p-1)*(q-1))) # a number such that d*e % ((p-1)*(q-1)) = 1 -> expoente privado

enc_flag = b"00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000004c8ac15da60342876d0e4da23691144ce8ff26ee9886dea91a64c8dea863e3653499857f5ebb01d8b29a8b70c2bafb7feae0c4026235f1091695156112ddb317f0a1271e0348119f7d1a770b8d6f05fcc6030acd42ecd1c7f6974cdd28c4c566882c781ca5b53754b83599330feec670f3b22553bc735f78f594bad5f8365023"

def enc(x):
	int_x = int.from_bytes(x, "big")
	y = pow(int_x,e,n)
	return hexlify(y.to_bytes(256, 'big'))

def dec(y):
	int_y = int.from_bytes(unhexlify(y), "big")
	x = pow(int_y,d,n)
	return x.to_bytes(256, 'big')

y = dec(enc_flag)
print(y.decode())

```
Primeiro, usámos a função *nextprime* do python para encontrar os números primos *p* e *q* secretos.  Em seguida, sabendo o expoente público *e* (0x10001) e usando os valores de *p* e *q*, o expoente privado *d* foi calculado usando a fórmula ``` d = (1/e) mod ((p-1)*(q-1))) ```. A variável *enc_flag* contém a mensagem que queremos decifrar que, neste caso, é a flag enviada pelo servidor. Quando corremos o programa, a flag aparece decifrada:

![](https://i.imgur.com/JVp6Pi2.png)

## Desafio 2

Neste desafio, o servidor envia duas mensagens cifradas com RSA que contêm a mesma flag. Ambas possuem o mesmo *n* mas dois expoentes públicos diferentes.

Para recuperar a flag usámos o seguinte script:

```python=
from binascii import hexlify, unhexlify
from sympy import *

c_s = int.from_bytes(unhexlify(b"97ef898baeceb7a35a2ed1d8a4c2fcddc65637e5ada272858d5e1c2ff0abffac0937505868754eb72020ec1d4c20ecb4bb8e3aab775849b8df06c7ab2c9ffac865429619a730f80f676472cb3e68d363d893feb7b2b1fb167e7591fe202ce0d249fbb9d4c50d3215498865a5d89c6f2b04c955bde6e53c0069b5572bcb6233e38b4c1b3c9f8ee56e564bf75c6e08bf6b78e7755d7adca0303a3ebe8c0213525f24966c7629c617e6634172c338083a576f36a63300d1fde58d56c571624e3f627d9e148f545fba0c465a2d1d1be97dedf5b64531aaf3d290c0edd80a3f0281f53c5dbb83e4482bfdc6822302e2e88f3a006574e40478533bbb6b23db7c927765"),"big")
c_j = int.from_bytes(unhexlify(b"ca05d6113f1e6454293c060017f60ac8ab238d614d4b1d997e7aabc5fd3621a06bb00dc043e43f0607ae17b9ad5f9433cd2da38d673c8301451150420340094467288d5a0e9a125a10828b4f581946884dc58a791a14ba15588237efacd1622ca29fc09f895ef35f5c1a3b18a59b21150ac813cc8595c9a1456e1be64a70567d392bf65a8e411a12d19a22e12d2d95f1768f4728749dd724f3fc56c733b6a1af4b40f7b6c0892eae88fb248724d27268a4a10c80503e689e67ce3f85dbc4d19fc5fd3ba5a3e16b025f9eb860d84c892b3a0d1190668c9332e76026e804ea74f7ce76fd3d73d6c6e5f634fb29564c3968b05b5d9643678f43c11dd94b9c173dc7"),"big")
e_s = 0x10001
e_j = 0x10003
n = 29802384007335836114060790946940172263849688074203847205679161119246740969024691447256543750864846273960708438254311566283952628484424015493681621963467820718118574987867439608763479491101507201581057223558989005313208698460317488564288291082719699829753178633499407801126495589784600255076069467634364857018709745459288982060955372620312140134052685549203294828798700414465539743217609556452039466944664983781342587551185679334642222972770876019643835909446132146455764152958176465019970075319062952372134839912555603753959250424342115581031979523075376134933803222122260987279941806379954414425556495125737356327411

def extended_euclidean(a, b):
    if b == 0:
        return (a, 1, 0)
    gcd, x, y = extended_euclidean(b, a % b)
    return (gcd, y, x - (a // b) * y)

(gcd,a,b) = extended_euclidean(e_s, e_j)

i = pow(c_j, -1, n)

m = (pow(c_s,a,n)*pow(i,-b,n)) % n

m = m.to_bytes(256,'big')

print(m.decode())
```

*c_s* e *c_j* contêm as mensagens enviadas pelo sevidor e *e_s* e *e_j* as chaves públicas de *c_s* e *c_j*, respetivamente. 

Se ```mdc(e_s,e_j) = 1```, então ```∃a,b∈Z: e_s*a + e_j*b = 1```. A partir de uma extensão do algoritmo de Euclides, calculámos os coeficientes *a* e *b* (*a* = 32769 e *b* = -32768). Como *b* é negativo, a equação ```m = (c_s^a * c_j^b) mod n``` daria problemas, por isso, calculámos o inverso modular de *c_j* (```i = c_j^(-1) mod n```) e, finalmente, obtivemos a mensagem com ```m = (c_s^a * i^(-b)) mod n```.

![](https://i.imgur.com/wiDoyxg.png)
