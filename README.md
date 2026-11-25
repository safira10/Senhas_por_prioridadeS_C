# Sistema de Senhas de Atendimento

Projeto em linguagem C desenvolvido para a **Tia Karla Soluções**. É um gerenciador de senhas de atendimento para pacientes: o atendente informa os dados do paciente, o sistema classifica a categoria automaticamente, emite uma senha sequencial e controla a fila de chamadas.

## Objetivo

Organizar o atendimento de pacientes por ordem de prioridade (urgente, preferencial e normal), emitindo senhas sequenciais e permitindo chamar, acompanhar e finalizar atendimentos por meio de um menu no terminal.

## Funcionalidades

- Registro de senha com validação dos dados (gênero, idade, gestante, lactante, necessidade especial, obesidade)
- Classificação automática em três categorias
- Senha sequencial por categoria: `N001`, `P001`, `U001`...
- Consultar a próxima senha a ser chamada
- Chamar paciente (registra a chamada)
- Identificar a senha de um paciente (situação e posição na fila)
- Ver a senha em atendimento
- Finalizar atendimento e atualizar a fila
- Painel de chamada com resumo
- Listagem da fila

### Estruturas de dados

- `struct Paciente`: dados informados pelo atendente  
- `struct Senha`: código, categoria e status  (Representa uma senha emitida)
- `enum Categoria`: `NORMAL`, `PREFERENCIAL`, `URGENTE` 
- `enum StatusSenha`: `AGUARDANDO`, `EM_ATENDIMENTO`, `FINALIZADA`  ( Indica em que ponto do atendimento a senha está) 
- A fila é um **vetor (array)** de `Senha` com até 100 posições.

## Compilação e execução

Requer `gcc`.

```bash
gcc -Wall -Wextra -std=c99 -o senhas main.c painel_chamada.c registro_senha.c registro_atendimento.c classificacao_senha.c utils.c
./senhas
```

No Windows (PowerShell ou CMD), execute com `senhas.exe`.

### Testes

```bash
gcc -Wall -Wextra -std=c99 -o test_sistema test_sistema.c classificacao_senha.c registro_atendimento.c
./test_sistema
```

Os testes verificam a classificação (incluindo os limites 59/60, 64/65 e 80/81 anos), a geração dos códigos e o fluxo da fila (ordem de chamada, finalização e atualização).

## Arquivo .gitignore

O `.gitignore` diz ao Git quais arquivos **não devem ser enviados** ao GitHub. Neste projeto ele ignora os arquivos gerados na compilação (`*.o`, `senhas.exe` e `test_sistema.exe`), para que o repositório guarde apenas o código-fonte, que é o que foi escrito.

## Funções implementadas

### utils 
Lê os dados do teclado com validação (número dentro da faixa, S/N, F/M), sem travar com entradas erradas.

| Função | O que faz |
|---|---|
| `void lerLinha(char*, int)` | lê uma linha com `fgets` e remove o Enter |
| `int lerInteiro(msg, min, max)` | repete até o usuário digitar um número válido na faixa |
| `bool lerSimNao(msg)` | repete até receber S ou N |
| `char lerGenero(void)` | repete até receber F ou M |

### registro_senha (módulo 1)
Faz as perguntas ao atendente (gênero, idade, gestante, lactante, necessidade especial e obesidade), 
valida as respostas e emite a senha do paciente.

| Função | O que faz |
|---|---|
| `Paciente coletarDadosPaciente(void)` | pergunta os dados ao atendente |
| `Senha emitirSenha(Paciente)` | classifica e gera o código da senha |
| `void registrarSenha(void)` | fluxo completo: coleta, emite e coloca na fila |

### classificacao_senha (módulo 2)
Decide se o paciente é Normal, Preferencial ou Urgente seguindo as regras do sistema e gera o código
sequencial da senha (N001, P001, U001).

| Função | O que faz |
|---|---|
| `bool temCondicaoPrioritaria(Paciente)` | verifica as condições de preferencial |
| `Categoria classificarPaciente(Paciente)` | decide urgente, preferencial ou normal |
| `char prefixoCategoria(Categoria)` | devolve `N`, `P` ou `U` |
| `const char* nomeCategoria(Categoria)` | nome para exibir |
| `void gerarCodigoSenha(Categoria, char*)` | gera `N001`, `P001`, `U001`... |
| `void reiniciarContadores(void)` | zera os contadores (testes) |

### registro_atendimento (módulo 3)
Controla a fila: consulta a próxima senha, registra a chamada, identifica a senha do paciente e a que
está em atendimento, finaliza o atendimento e atualiza a fila.

| Função | O que faz |
|---|---|
| `bool adicionarNaFila(Senha)` | coloca a senha no fim da fila |
| `int consultarProximaSenha(void)` | índice da próxima senha a chamar |
| `bool chamarPaciente(void)` | marca a próxima senha como em atendimento |
| `int buscarSenha(const char*)` | identifica uma senha pelo código |
| `int indiceEmAtendimento(void)` | identifica a senha em atendimento |
| `bool finalizarAtendimento(void)` | conclui o atendimento atual |
| `void atualizarFila(void)` | remove da fila as senhas finalizadas |
| `int totalAguardando / totalNaFila / totalAtendidos` | contadores |
| `int posicaoNaFila(int)` | posição na ordem de chamada |
| `Senha obterSenha(int)` | devolve uma cópia da senha na posição indicada |

### painel_chamada (módulo 4)
Mostra o menu e o painel de chamada e liga cada opção escolhida à função certa. É o módulo que
controla a execução do programa.

| Função | O que faz |
|---|---|
| `void exibirMenu(void)` | mostra as opções |
| `void exibirPainel(void)` | senha atual, próxima e resumo |
| `void listarFila(void)` | lista a fila |
| `void identificarSenhaPaciente(void)` | pede o código e mostra a situação |
| `void executarOpcao(int)` | trata a opção escolhida (`switch case`) |
| `void iniciarPainel(void)` | laço do menu (`do...while`) |

## Exemplo de utilização

```
Escolha uma opcao: 1

--- REGISTRO DE SENHA ---
Genero do paciente (F/M): F
Idade do paciente (0 a 120): 25
Paciente esta gestante? (S/N): S
Paciente e lactante? (S/N): N
Possui necessidade especial? (S/N): N
Possui condicao de obesidade? (S/N): N

>>> Senha emitida: P001 (PREFERENCIAL)
>>> Pacientes aguardando: 1
```

Após registrar `P001`, `N001`, `U001` e outra `P002`, a opção 8 (painel) mostra:

```
############# PAINEL DE CHAMADA #############
 EM ATENDIMENTO : U001 (URGENTE)
 PROXIMA SENHA  : P001 (PREFERENCIAL)
 Aguardando     : 3
 Atendidos      : 0 (0.0% do total do dia)
#############################################
```

## Boas práticas adotadas

- Nenhuma lógica de negócio na `main()`
- Um `.c` e um `.h` por módulo, com *include guards* (`proteção contra inclusão repetida`)
  *Como funciona : o #ifndef pergunta se o nome TIPOS_H ainda não foi criado. Na primeira vez, o #define cria o nome e o conteúdo é lido. Nas próximas vezes o nome já existe, então o compilador pula tudo até o #endif.*
- Funções com responsabilidade única e comentadas
- Validação de todas as entradas (sem travar com letras ou valores fora da faixa)
- Constantes nomeadas (`MAX_FILA` (Quantidade máxima de senhas na fila), `TAM_CODIGO` (Tamanho do texto da senha)) em vez de números soltos
- Compilação sem avisos com `-Wall -Wextra`

## Autor

Projeto desenvolvido por Stéffany Rodrigues, trainee na Tia Karla Soluções.
