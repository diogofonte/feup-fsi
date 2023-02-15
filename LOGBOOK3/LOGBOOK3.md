# LOGBOOK 3

## Caracterização do CVE 2017-5754 (Meltdown)
<img src="https://meltdownattack.com/meltdown.png" width = 200 height = 300>

### Introdução ao Meltdown e Spectre

Meltdown e Spectre exploram vulnerabilidades críticas em processadores modernos, permitindo que se aceda a dados que deveriam estar inacessíveis.

Atacantes podem aproveitar-se destas vulnerabilidades para obter qualquer tipo de dados armazenados em memória protegida, ou até mesmo memória de outros processos ou kernel no caso de Meltdown.

Meltdown e Spectre afetam computadores pessoais, dispositivos móveis, e na cloud. Dependendo da infraestrutura do provedor da cloud, pode ser possível roubar dados de outros clientes.

Foram divulgados em simultâneo e são muito semelhantes na forma como se aproveitam de execução especulativa aliada à falta de proteção quanto a acessos da cache.

Certos analistas de segurança chamaram de "catastróficas" a estas vulnerabilidades, sendo elas tão graves que certos investigadores inicialmente acreditavam que os relatórios eram falsos.

### Identificação 

CVE-2017-5754

Meltdown é uma vulnerabilidade a nível do hardware que não só afeta praticamente todos os microprocessadores da Intel feitos desde 1995, como também alguns CPUs ARM e IBM.

Esta vulnerabilidade permite que um processo leia totalmente a memória do sistema, incluindo outros processos ou kernel, mesmo que não tenha permissão de acesso à mesma.

Funcionamento de Meltdown:
 - Despoletar um acesso ilegal, mas certificando que o CPU irá executá-lo fora de ordem (através de um acesso a memória por exemplo).
 - Recuperar de qualquer segfault que possa acontecer.
 - Examinar a cache e extrair a informação, que se confirma estar presente facilmente já que acessos a cache são 20 vezes mais rápidos que cache-misses.

Antes da sua divulgação, afetava todos os sistemas operativos.

Estima-se que houve uma perda de performance de 5-30% nos CPUs afetados, devido aos workarounds a nível de software que tiveram de ser implementados.

### Catalogação

O Meltdown foi descoberto de forma independente e relatado, em janeiro de 2018, por três equipas:
> - Jann Horn (Google Project Zero);
> - Werner Haas, Thomas Prescher (Tecnologia Cyberus);
> - Daniel Gruss, Moritz Lipp, Stefan Mangard, Michael Schwarz (Graz University of Technology).

Foi mantido em segredo até haver workaround pronto a ser utilizado.

Está catalogado como uma vulnerabilidade de obtenção de informação, com uma Pontuação CVSS de 4.7.
Destacam-se:
 - Impacto de Confidencialidade "Completo", implicando total divulgação de informação, potencialmente de qualquer ficheiro.
 - Autenticação "Não requerida".
 - Complexidade de acesso "Médio", sugerindo que a vulnerabilidade requer certas precondições para que possa ser satisfeita.


### Exploit
Não se conhece nenhum exploit com grande impacto de Meltdown.

No entanto, há 3 ataques proof of concept que tentam demonstrar possíveis aplicações maliciosas de Spectre.

>NetSpectre, em que investigadores conseguiram demonstrar exploração remota a uma velocidade de extração de dados de 15 a 60 bits por hora.

>leaky.page, revelado em Março de 2022 pela Google, consegue extrair dados da RAM.

>Spook.js, Setembro de 2022, Google, conseguia roubar dados reais mesmo em versões do Google Chrome com proteção anti-spectre (isolamento de páginas em processos diferentes).

### Ataques 
Não foram encontrados ataques causados pelo Meltdown.

[![IMAGE ALT TEXT HERE](http://img.youtube.com/vi/fq3abPnEEGE/0.jpg)](https://www.youtube.com/watch?v=fq3abPnEEGE)
