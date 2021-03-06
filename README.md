
MC723 - Laboratório de Projetos de Sistemas Computacionais
====
######Campinas, 06 de maio de 2016
####Professor: Lucas Wanner
####RA: &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; Alunos:
####&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;136242 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; João Guilherme Daros Fidélis
####&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;139546 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; João Pedro Ramos Lopes
####&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;145539 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; Bruno Takeshi Hori
####&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;146009 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; Felipe de Oliveira Emos

Projeto 2 - Desempenho do Processador
====

### Introdução

O objetivo do projeto foi avaliar como diferentes configurações de processadores podem impactar no desempenho de diferentes programas (benchmarks). Ao variarmos o tamanho do pipeline, simularmos processadores escalar e superescalar, diferentes confiurações de cache, diferentes branch predictors, conseguimos medir dados como número de miss na L1 de dados, número de miss na L1 de instruções, número de miss na L2 unificada, número de hazards, tempo de execução, entre outros dados. Com esse dados, é possível avaliar o desempenho de cada configuração de processador.


### Metodologia de Pesquisa

Os objetivos do projeto foram medir desempenho de um processador em certos pontos:
* Tamanho do pipeline: 5, 7 e 13 estágios
* Processador escalar vs superescalar
* Hazard de dados e controle
* Branch predictor (3 configurações distintas)
* Cache (4 configurações distintas)

Escolhemos certos benchmarks para avaliarmos esses parâmetros:
* Jpeg coder (small)
* Rijndael coder (small)
* GSM coder (large)
* Dijkstra (large)

As propriedades que avaliaremos serão:
* Miss L1 de dados
* Miss L1 de instruções
* Miss L2
* Número de Hazards
* Numero de branchs realizados
* Número de branchs previstos
* Ciclos 
* CPI

#### Cache

Os dados de cache miss foram obtidos através do programa Dinero IV. Para utilizá-lo, geramos os traces de nossos benchmarks no formato .din. Nesse formato, cada linha do arquivo tem a seguinte característica:
código endereço_acessado

O código é 0 se for um read, 1 se for um write e 2 se for um instruction fetch. Assim, modificamos o arquivo mips_isa.cpp para que cada função imprimisse no arquivo o código da função e o endereço calculado que será acessado.

Depois, rodamos esses arquivos gerados no Dinero IV que nos deu os dados de miss na L1 de dados, miss na L1 de instruções e miss na L2 unificada.

Eis as configurações de cache utilizadas:
* Configurações de cache
  1. Configuração encontrada no exercício 2 por um dos integrantes.
  2. Baseado no processador AMD FX-8350
  3. Baseado no processador AMD Athlon II P340
  4. Baseado no Intel i7
 
|Config | L1 isize | L1 ibsize | L1 dsize | L1 dbsize | L2 usize | L2 ubsize | L1 iassoc | L1 dassoc | L2uassoc
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 32k | 64 | 64k | 64 | 256k | 16 | 1 | 1 | 1 |
| 2 | 128k | 64 | 128k | 64 | 1024k | 64 | 1 | 1 | 1 |
| 3 | 256k | 64 | 128k | 64 | 8192k | 64 | 1 | 1 | 1 |
| 4 | 32k | 64 | 32k | 64 | 256k | 64 | 4 | 8 | 8 |


#### Hazards de dados
Após descobrir os tipos de hazards de dados existentes[2], foram feitas formas diferentes em relação à escalar e superescalar. Porém, a ideia básica foi a mesma: criar um vetor de structs com as informações da instrução em cada estágio.

__Escalar:__ Como o único hazard de dados que não pode ser resolvido por forwarding é RAW, basta conferir se a instrução anterior é do tipo _load_ e se o registrador é um dos registradores da instrução atual.

__Superescalar:__ Neste caso, foi utilizado um pipeline 2-way.

Como todos os tipos de hazards de dados (RAW, WAR e WAW) podem acontecer, foi conferido extensivamente as possibilidades de causar uma bolha.

Foi feita uma máscara (vetor de registradores) para ver se o registrador já foi atualizado na devida hora, em instruções de load. Utiliza-se a máscara para conferir se o registrador a ser utilizado ainda não foi atualizado. Caso isso seja verdade, cria-se uma bolha. 

Outro possível caso, é quando duas instruções ao estarem no IF do pipeline ao mesmo tempo, utilizam os mesmos registradores, ocasionando mais uma bolha.

#### Hazards de Controle
Foram feitos três tipos de configurações em relação ao branch prediction: sem branch prediction, branch prediction always taken e branch prediction 2-bit prediction.

__Sem Branch prediction:__ O código apenas soma as bolhas que ocorrem. Caso o branch ocorra, adiciona-se três bolhas, caso não, adiciona-se uma (pipeline de 5 estágios).

__Branch Prediction Always Taken:__ Neste caso, só é contabilizada as bolhas caso o branch não ocorrer. No código, existe uma variável que verifica se o branch foi feito. Caso não tenha sido feito, contabiliza as bolhas a mais. Caso o predictor tenha acertado, adiciona-se somente uma bolha, caso erre, adiciona-se duas bolhas (pipeline de 5 estágios).

__Branch Prediction 2-bit predictor:__ Por fim, utiliza uma verificação dependendo de quantas vezes foi errada a predição, modificando-a caso tenha ocorrido, porém toda vez que erra, é contabilizada as bolhas. Por exemplo: predictor começa como taken; caso erre a predição 2 vezes, predictor se torna not taken, e por assim vai. Caso o predicotor tenha acertado, se o branch for feito, adiciona-se uma bolha, caso o branch não seja feito, não se adiciona bolha. Caso ele erre, adiciona-se duas bolhas (pipeline de 5 estágios).

#### CPI

Calculamos CPI com a fórmula: CPI = [ número de instruções + ( número de estágios do pipeline - 1 ) + número de bolhas ] / (número de instruções).

## Análise de Resultados

Pra cada benchmark:

| Configuração | Cache | Escalar ou Super | Branch Predictor | 
| --- | --- | --- | --- |
| Config1 | 1 | Escalar | Sem branch |
| Config2 | 2 | Escalar | Estático |
| Config3 | 3 | Escalar | Dinamico |
| Config4 | 4 | Super | Sem branch |

__Jpeg coder (small)__

| Eventos | Config1 | Config2 | Config3 | Config4 |
| --- | --- | --- | --- | --- | --- | --- |
| Miss L1-instr | 0.001 | 0.0007 | 0.0001 | 0.0003 |
| Miss L1-data | 0.0271 | 0.0183 | 0.0183 | 0.0198 |
| Miss L2 | 0.434 | 0.4176 | 0.1129 | 0.3092 |
| Hazards | 8241245 | 4981313 | 3897393 | 8708259 |
| Branch Realizados | 0 | 2219688 | 2219688 | 0 |
| Branch Previstos | 0 | 3399132 | 2058392 | 0 |
| Ciclos | 38098999 | 34839067 | 33755147 | 38566013 |
| instruções | 29857750 | 29857750 | 29857750 | 29857750 |
| CPI | 1.28 | 1.17 | 1.13 | 1.29 |

__Rijndael coder (small)__

Foi usado o encode little endian

| Eventos | Config1 | Config2 | Config3 | Config4 |
| --- | --- | --- | --- | --- | --- | --- |
| Miss L1-instr | 0.0591 | 0.027 | 0.0018 | 0.0748 |
| Miss L1-data | 0.2188 | 0.144 | 0.144 | 0.1926 |
| Miss L2 | 0.3382 | 0.3382 | 0.2268 | 0.0817 | 0.0706 |
| Hazards | 2497936 | 1897371 | 1626815 | 5732511 |
| Branch Realizados | 0 | 496874 | 496874 | 0 |
| Branch Previstos | 0 | 890057 | 576390 | 0 |
| Ciclos | 46059637 | 45459072 | 45188516 | 49294208 |
| instruções | 43561697 | 43561697 | 43561697 | 43561697 |
| CPI | 1.06 | 1.04 | 1.04 | 1.13 |

__GSM coder (small)__

| Eventos | Config1 | Config2 | Config3 | Config4 | 
| --- | --- | --- | --- | --- | --- | --- |
| Miss L1-instr | 0.0066 | 0.0017 | 0.0007 | 0.0041 |
| Miss L1-data | 0.0021 | 0.0015 | 0.0015 | 0.0001 |
| Miss L2 | 0.1146 | 0.0351 | 0.0564 | 0.0166 |
| Hazards | 3185272 | 2491191 | 1712797 | 3381979 |
| Branch Realizados | 0 | 729028 | 729028 | 0 |
| Branch Previstos | 0 | 1493003 | 484186 | 0 |
| Ciclos | 30646228 | 29952147 | 29173753 | 30843226 |
| instruções | 27460952 | 27460952 | 27460952 | 27461243 |
| CPI | 1.12 | 1.09 | 1.06 | 1.12 |

__Dijkstra (large)__

| Eventos | Config1 | Config2 | Config3 | Config4 |
| --- | --- | --- | --- | --- |
| Miss L1-instr | 0.0022 | 0.0004 | 0.0002 | 0.0004 |
| Miss L1-data | 0.0309 | 0.0157 | 0.0157 | 0.0231 |
| Miss L2 | 0.1767 | 0.0476 | 0.003 | 0.0621 |
| Hazards | 88107656 | 68515147 | 38220239 | 112874483 |
| Branch Realizados | 0 | 17956084 | 17956084 | 0 |
| Branch Previstos | 0 | 41909436 | 10499229 | 0 |
| Ciclos | 311799497 | 292206988 | 261912080 | 336566324 |
| instruções | 223691837 | 223691837 | 223691837 | 223691837 |
| Tempo | 1.39 | 1.31 | 1.17 | 1.50 |


| Benchmark | Pipeline 7 Bolhas | Pipeline 13 Bolhas |
| --- | --- | --- |
| Jpeg coder (small) | 12043114 | 20144258 |
| Dijkstra (large) | 140303144 | 197594002 |
| Gsm coder (small) | 4912488 | 7290724 |
| Rijndael coder (small) | 4002124 | 6151725 |

## Conclusão

Vemos que o Branch Predictor teve os menores valores para CPI, o que mostra que ocorre uma melhoria na eficiência da resposta aos branchs. Isso mostra também que ao usar Branch Predctor Dinâmico ao invés de Estático o número de hazard de controle diminuirá.

Outra observação, é que ao usar um pipeline superescalar, o número de hazards aumenta. Isso ocorre pois existem tipos diferentes de hazards (WAR e WAW), o que é demonstrado na tabela.

É possível dizer que os resultados foram coerentes, já que a princípio utilizamos o código do hello.c inicialmente para testar as configurações. Como a mudança feita é no código do mips_isa.cpp, os benchmarks influenciam apenas nas instruções a serem avaliadas, por isso é só observar a reação dessas instruções no hello.c para saber se a saída é coerente ou não.

## Referências
1. http://www.ic.unicamp.br/~lucas/teaching/mc723/2016-1/p2.html
2. [**Tratamento de Hazards**](HAZARD_TREATMENT.md)
3. http://i.stack.imgur.com/9dQt4.png
4. https://www.cs.umd.edu/class/spring2012/cmsc411/lectures/lec05.pdf
