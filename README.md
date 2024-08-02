# Projeto MPI: Análise de Matriz com MPI

Este projeto implementa um programa em C utilizando MPI (Message Passing Interface) para distribuir e processar uma matriz de inteiros aleatórios entre múltiplos processos. O objetivo é calcular várias estatísticas da matriz, como a contagem de elementos negativos, valor mínimo, valor máximo, média, desvio padrão e a contagem de zeros.

## Índice

- [Descrição](#descrição)
- [Como Compilar](#como-compilar)
- [Como Executar](#como-executar)
- [Explicação do Código](#explicação-do-código)
- [Resultados Esperados](#resultados-esperados)
- [Referências](#referências)

## Descrição

O programa gera uma matriz aleatória de tamanho 1000x1000 no processo de rank 0. Essa matriz é então distribuída entre vários processos usando MPI, e cada processo calcula estatísticas locais sobre a parte da matriz que recebeu. Essas estatísticas locais são então reduzidas para estatísticas globais que são exibidas pelo processo de rank 0.

## Como Compilar

Para compilar o código, execute o seguinte comando:

```sh
mpicc -o trabalho_MPI_Franklin trabalho_MPI_Franklin.c -lm
```

### Explicação da Flag `-lm`

A flag `-lm` é utilizada para linkar a biblioteca matemática `math.h`, que não faz parte da biblioteca padrão do C. Ela é necessária para usar funções matemáticas como `sqrt()`.

## Como Executar

Para executar o programa, use o comando `mpirun` especificando o número de processos desejado. Por exemplo, para executar com 4 processos:

```sh
mpirun -np 4 ./trabalho_MPI_Franklin
```

## Explicação do Código

### Inicialização

```c
MPI_Init(&argc, &argv);
MPI_Comm_rank(MPI_COMM_WORLD, &rank);
MPI_Comm_size(MPI_COMM_WORLD, &size);
```

Essas funções inicializam o ambiente MPI, obtêm o rank do processo atual e o número total de processos.

### Geração e Distribuição da Matriz

```c
if (rank == 0) {
    generate_matrix(matrix);
    printf("Matrix generated by rank 0\n");
}

MPI_Barrier(MPI_COMM_WORLD);
MPI_Scatter(matrix, (N * N) / size, MPI_INT, local_matrix, (N * N) / size, MPI_INT, 0, MPI_COMM_WORLD);
```

O processo de rank 0 gera a matriz e usa `MPI_Scatter` para distribuir partes iguais da matriz para todos os processos.

### Cálculo Local

Cada processo calcula a contagem de elementos negativos, valor mínimo, valor máximo, soma, e contagem de zeros para sua parte da matriz.

```c
for (int i = 0; i < N / size; i++) {
    for (int j = 0; j < N; j++) {
        int val = local_matrix[i][j];
        // Cálculos locais
    }
}
```

### Redução dos Resultados

Os valores locais são reduzidos para valores globais usando `MPI_Reduce`.

```c
MPI_Reduce(&local_neg_count, &global_neg_count, 1, MPI_INT, MPI_SUM, 0, MPI_COMM_WORLD);
MPI_Reduce(&local_min, &global_min, 1, MPI_INT, MPI_MIN, 0, MPI_COMM_WORLD);
MPI_Reduce(&local_max, &global_max, 1, MPI_INT, MPI_MAX, 0, MPI_COMM_WORLD);
MPI_Reduce(&local_sum, &global_sum, 1, MPI_LONG_LONG, MPI_SUM, 0, MPI_COMM_WORLD);
MPI_Reduce(&local_zero_count, &global_zero_count, 1, MPI_INT, MPI_SUM, 0, MPI_COMM_WORLD);
```

### Cálculo da Média e Desvio Padrão

O processo de rank 0 calcula a média e o desvio padrão a partir dos resultados reduzidos.

```c
mean = (double)global_sum / (N * N);
local_variance_sum += pow(local_matrix[i][j] - mean, 2);

MPI_Reduce(&local_variance_sum, &global_variance_sum, 1, MPI_DOUBLE, MPI_SUM, 0, MPI_COMM_WORLD);

if (rank == 0) {
    double stddev = sqrt(global_variance_sum / (N * N));
    // Exibe os resultados
}
```

## Resultados Esperados

Ao executar o programa, os seguintes resultados são exibidos pelo processo de rank 0:

- Contagem de elementos negativos
- Menor valor
- Maior valor
- Média
- Desvio padrão
- Número de ocorrências do valor zero
- Informação sobre se a matriz é esparsa ou não

Exemplo de saída:

```
Contagem de elementos negativos: 500694
Menor valor: -1000
Maior valor: 1000
Média: -1.01
Desvio padrão: 577.36
Número de ocorrências do valor zero: 512
A matriz é não esparsa.
```

## Referências

- [Documentação do MPI](https://www.mpich.org/documentation/)
- [Explicação sobre a flag `-lm`](https://stackoverflow.com/questions/44175151/what-is-the-meaning-of-lm-in-gcc)