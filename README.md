
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
* Cache (3 configurações distintas)

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
* Tempo

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
 
|Config | L1 isize | L1 ibsize | L1 dsize | L1 dbsize | L2 usize | L2 ubsize |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | 32k | 64 | 64k | 64 | 256k | 16 |
| 2 | 128k | 64 | 128k | 64 | 1024k | 64 |
| 3 | 256k | 64 | 128k | 64 | 8192k | 64 |


#### Hazards de dados
Após descobrir os tipos de hazards de dados existentes[2], foram feitas formas diferentes em relação à escalar e superescalar. Porém, a ideia básica foi a mesma: criar um vetor de structs com as informações da instrução em cada estágio.

__Escalar:__ Como o único hazard de dados que não pode ser resolvido por forwarding é RAW, basta conferir se a instrução anterior é do tipo _load_ e se o registrador é um dos registradores da instrução atual.

__Superescalar:__ Neste caso, foi utilizado um pipeline 2-way.

Como todos os tipos de hazards de dados (RAW, WAR e WAW) podem acontecer, foi conferido extensivamente as possibilidades de causar uma bolha.

Foi feita uma máscara (vetor de registradores) para ver se o registrador já foi atualizado na devida hora. Utiliza-se a máscara para conferir se o registrador a ser utilizado ainda não foi atualizado. Caso isso seja verdade, cria-se uma bolha. 



#### Hazards de Controle
Foram feitos três tipos de configurações em relação ao branch prediction: sem branch prediction, branch prediction always taken e branch prediction 2-bit prediction.

__Sem Branch prediction:__ O código apenas soma as bolhas que ocorrerão.

__Branch Prediction Always Taken:__ Neste caso, só é contabilizada as bolhas caso o branch não ocorrer. No código, existe uma variável que verifica se o branch foi feito. Caso não tenha sido feito, contabiliza as bolhas a mais.

__Branch Prediction 2-bit predictor:__ Por fim, utiliza uma verificação dependendo de quantas vezes foi errada a predição, modificando-a caso tenha ocorrido, porém toda vez que erra, é contabilizada as bolhas. Por exemplo: predictor começa como taken; caso erre a predição 2 vezes, predictor se torna not taken, e por assim vai.

## Análise de Resultados

| Eventos | Config1 | Config2 | Config3 |  |  |  |
| --- | --- | --- | --- | --- | --- | --- |
| Miss L1 |  |  |  |  |  |  |
| Miss L2 |  |  |  |  |  |  |
| Hazards |  |  |  |  |  |  |
| Branch Realizados |  |  |  |  |  |  |
| Branch Previstos |  |  |  |  |  |  |
| Ciclos |  |  |  |  |  |  |
| Tempo |  |  |  |  |  |  |

## Conclusão

## Referências
1. http://www.ic.unicamp.br/~lucas/teaching/mc723/2016-1/p2.html
2. [**Tratamento de Hazards**](HAZARD_TREATMENT.md)
