Sala de Metodologias Ativas: Tarefa sobre planejamento 

Aluna: Rosineide Santana Santos 

O código parece ser um planejador para resolver uma espécie de Sudoku 4x4, onde zeros representam células vazias. A ideia geral do algoritmo está boa, mas ele pode não funcionar corretamente por alguns motivos que detalho abaixo: 

1. Comentários corrompidos 

O código está com vários símbolos estranhos como %%%%%%%%%%%%%%%%%%%%%%%%%%%¢%, que provavelmente vieram de um erro de codificação ou colagem. Isso pode impedir a execução. 

 

2. Fim de linha corrompido 

Há vários pontos em que as quebras de linha estão mal formatadas, por exemplo: 

 

subgrid(State, Row, Col, Subgrid) :-% Determine top-left corner of the subgrid 

SubRow is ((Row 

- 1) // 2) * 2 + 1, 

SubCol is ((Col 

- 1) // 2) * 2 + 1, 

 

Aqui o código foi quebrado incorretamente. Isso impede a leitura correta por parte do interpretador Prolog. O correto seria: 

 

SubRow is ((Row - 1) // 2) * 2 + 1, 

SubCol is ((Col - 1) // 2) * 2 + 1, 

 

3. Falta de controle de repetição / retrocesso eficiente 

 

Mesmo corrigindo a sintaxe, o código pode não terminar em casos complexos ou retornar false, porque: 

 

A ordem de tentativa dos números e das células pode levar a caminhos errados que não são podados rapidamente. 

 

Não há nenhuma heurística (como tentar células com menos opções primeiro). 

 

4. Possível uso incorreto de copy_term 

 

Essa linha: 

 

copy_term(InitialState, CurrentState), 

 

É seguida de: 

 

plan_step(CurrentState, FinalState, [], Plan). 

 

Mas copy_term/2 apenas faz uma cópia sem variáveis ligadas. Se InitialState e FinalState forem parcialmente preenchidos com variáveis (ou forem iguais), isso pode gerar inconsistência. Dependendo da sua intenção, copy_term pode ser desnecessário aqui. 

 

5. Falta de caso para quando não existe solução 

 

Se não houver nenhuma solução possível, o código entra em false silenciosamente. Poderia ser interessante incluir um rastreamento ou impressão de tentativa para facilitar o debug. 

 

CORREÇÃO DO CÓDIGO  

1. Corrigi a formatação quebrada e ilegível 

Havia várias quebras de linha no meio de expressões que deixavam o código ilegível para o interpretador Prolog. Exemplo: 

 

Antes (quebrado): 

 

SubRow is ((Row 

- 1) // 2) * 2 + 1, 

 

Depois (corrigido): 

 

SubRow is ((Row - 1) // 2) * 2 + 1, 

 

Esse tipo de erro acontecia em várias expressões is/2, como em SubRow, SubCol, NextPos, etc. 

 

2. Removi símbolos corrompidos 

O início do código tinha caracteres inválidos como %%%%%%%%%%%%%%%%%%%%%%%%%%%¢%, o que certamente causaria erro de sintaxe. Removi isso e deixei o início limpo com: 

 

% Define numbers 1-4 

num(1). num(2). num(3). num(4). 

 

3. Corrigi uso direto de expressões aritméticas em nth1 

Havia chamadas como: 

 

nth1(SubRow + 1, State, Row2) 

 

Isso está errado em Prolog, porque nth1/3 exige que o índice esteja avaliado, e não como uma expressão. 

Corrigido assim: 

 

SubRow1 is SubRow + 1, 

nth1(SubRow1, State, Row2), 

 

Fiz o mesmo para SubCol + 1. 

 

4. Adicionei vírgulas que estavam faltando 

Algumas linhas de código terminavam direto com outra instrução, sem vírgula, o que gera erro de parsing. Exemplo: 

 

nth1(Row, State, RowList)\+ member(Num, RowList), 

 

Foi corrigido para: 

 

nth1(Row, State, RowList), 

\+ member(Num, RowList), 

 

5. Corrigi identação para leitura e execução 

Embora Prolog não dependa da indentação, ela ajuda muito a evitar erros de estrutura visual. O código agora está mais claro e organizado, o que facilita testes e manutenção. 

CÓDIGO FINAL 

% Define numbers 1-4 

num(1). num(2). num(3). num(4). 

 

% Main planning predicate 

plan(InitialState, FinalState, Plan) :- 

    % Copy InitialState to work on it 

    copy_term(InitialState, CurrentState), 

    % Generate the plan by filling empty cells 

    plan_step(CurrentState, FinalState, [], Plan). 

 

% Base case: If CurrentState matches FinalState, stop. 

plan_step(FinalState, FinalState, Plan, Plan). 

 

% Recursive case: Find an empty cell, fill it, and continue. 

plan_step(CurrentState, FinalState, PartialPlan, Plan) :- 

    % Find an empty cell (0) at (Row, Col) 

    nth1(Row, CurrentState, RowList), 

    nth1(Col, RowList, 0), 

    % Try filling it with Num (1-4) 

    num(Num), 

    % Check if Num is valid (no conflicts) 

    is_valid(CurrentState, Row, Col, Num), 

    % Fill the cell 

    fill_cell(CurrentState, Row, Col, Num, NewState), 

    % Record the action 

    Action = fill(Row, Col, Num), 

    % Continue planning 

    plan_step(NewState, FinalState, [Action | PartialPlan], Plan). 

 

% Check if Num can be placed at (Row, Col) without conflicts 

is_valid(State, Row, Col, Num) :- 

    % Check Row 

    nth1(Row, State, RowList), 

    \+ member(Num, RowList), 

    % Check Column 

    column(State, Col, ColumnList), 

    \+ member(Num, ColumnList), 

    % Check 2x2 sub-square 

    subgrid(State, Row, Col, Subgrid), 

    \+ member(Num, Subgrid). 

 

% Extract a column from the grid 

column([], _, []). 

column([Row|Rest], Col, [Value|Values]) :- 

    nth1(Col, Row, Value), 

    column(Rest, Col, Values). 

 

% Extract the 2x2 subgrid containing (Row, Col) 

subgrid(State, Row, Col, Subgrid) :- 

    % Determine top-left corner of the subgrid 

    SubRow is ((Row - 1) // 2) * 2 + 1, 

    SubCol is ((Col - 1) // 2) * 2 + 1, 

    % Extract 4 cells 

    nth1(SubRow, State, Row1), 

    SubRow1 is SubRow + 1, 

    nth1(SubRow1, State, Row2), 

    nth1(SubCol, Row1, A), 

    SubCol1 is SubCol + 1, 

    nth1(SubCol1, Row1, B), 

    nth1(SubCol, Row2, C), 

    nth1(SubCol1, Row2, D), 

    Subgrid = [A, B, C, D]. 

 

% Fill cell (Row, Col) with Num 

fill_cell(State, Row, Col, Num, NewState) :- 

    nth1(Row, State, RowList), 

    replace(RowList, Col, Num, NewRowList), 

    replace(State, Row, NewRowList, NewState). 

 

% Helper: Replace element at position in a list 

replace([_|T], 1, X, [X|T]). 

replace([H|T], Pos, X, [H|NewT]) :- 

    Pos > 1, 

    NextPos is Pos - 1, 

    replace(T, NextPos, X, NewT). 

 

Como usar 

 

Com uma consulta como: 

 

InitialState = [[1,0,0,0], 

                [0,0,0,0], 

                [0,0,0,0], 

                [0,0,0,0]], 

FinalState = [[1,3,4,2], 

              [4,2,1,3], 

              [2,4,3,1], 

              [3,1,2,4]], 

plan(InitialState, FinalState, Plan). 

 

O Prolog deve gerar corretamente o Plan que transforma InitialState em FinalState. 

 

GOAL REGRESSION E MEANS-END 

Goal Regression é uma técnica de planejamento reverso. Em vez de começar do estado inicial e tentar chegar ao estado final (como no planejamento progressivo), ela faz o caminho inverso: parte do objetivo e tenta descobrir quais ações levariam a esse objetivo a partir de um estado anterior. 

Como funciona: 

Começa com o estado final desejado. 

Escolhe um subobjetivo (ex: uma célula com valor definido). 

Determina qual ação poderia ter produzido esse subobjetivo. 

Regride o estado: aplica a ação ao contrário e obtém um novo estado anterior. 

Continua o processo até que o estado inicial seja alcançado. 

Essa abordagem é útil porque: 

Reduz o espaço de busca: só considera ações relevantes para os objetivos. 

Evita ações “aleatórias” que não ajudam a atingir o objetivo. 

 

O que é Means-End Analysis (Análise de Meios e Fins)? 

Means-End Analysis (MEA) é uma estratégia de resolução de problemas baseada na diferença entre o estado atual e o objetivo. A ideia central é: 

 

“O que me impede de alcançar meu objetivo? Que ação pode reduzir essa diferença?” 

Como funciona: 

Compara o estado atual com o objetivo. 

Identifica uma diferença entre eles. 

Seleciona uma ação que elimine ou reduza essa diferença. 

Aplica a ação e atualiza o estado. 

Repete o processo até que o estado atual seja igual ao objetivo. 

É como um loop: diferença → ação → novo estado → nova diferença… 

 

Diferença entre Goal Regression e MEA 

Direção de Planejamento 

Goal Regression: 

Parte do objetivo final e tenta descobrir quais ações teriam levado até ele. 

→ Começa pelo fim e vai voltando até o início. 

Means-End Analysis (MEA): 

Parte do estado inicial e tenta encontrar ações que reduzam a diferença em relação ao objetivo. 

→ Começa do começo e vai tentando se aproximar do fim. 

 

 

Estratégia 

Goal Regression: 

Analisa o que precisa ser verdadeiro no passo anterior para que o objetivo atual seja alcançado. 

Means-End Analysis: 

Analisa o que está faltando para atingir o objetivo e qual ação pode diminuir essa diferença. 

Exemplo Simples (Sudoku) 

Objetivo: A célula (1,2) deve ser 3. 

 

Goal Regression pergunta: 

“Qual ação poderia ter colocado o número 3 na posição (1,2)? E o que teria que ser verdade antes dessa ação?” 

→ Vai montando o plano de trás pra frente. 

Means-End Analysis pergunta: 

“Hoje a posição (1,2) está vazia, mas quero que tenha 3. Qual ação posso fazer agora para colocar o 3 lá, sem violar regras?” 

→ Vai passo a passo preenchendo até alcançar o estado final. 

Quando usar cada um? 

Goal Regression é bom quando: 

O objetivo está bem definido. 

Quer restringir a busca a ações diretamente relacionadas ao objetivo. 

Means-End Analysis é bom quando: 

Você tem vários caminhos possíveis. 

Precisa de uma abordagem adaptativa, ajustando o plano conforme o progresso. 


 

5. CÓDIGO COM GOAL REGRESSION 

% Define numbers 1-4 num(1). num(2). num(3). num(4). 

% Main planning predicate using goal regression plan(InitialState, FinalState, Plan) :- goal_regression(InitialState, FinalState, [], Plan). 

% Goal regression: Base case goal_regression(State, State, Plan, Plan). 

% Goal regression: Regress a goal (cell with known value in FinalState) goal_regression(CurrentState, FinalState, PartialPlan, Plan) :- nth1(Row, FinalState, FinalRow), nth1(Col, FinalRow, Num), Num = 0, nth1(Row, CurrentState, CurrentRow), nth1(Col, CurrentRow, CurrentVal), CurrentVal =:= 0, is_valid(CurrentState, Row, Col, Num), fill_cell(CurrentState, Row, Col, Num, NewState), Action = fill(Row, Col, Num), goal_regression(NewState, FinalState, [Action | PartialPlan], Plan). 

% Check if Num can be placed at (Row, Col) without conflicts is_valid(State, Row, Col, Num) :- nth1(Row, State, RowList), + member(Num, RowList), column(State, Col, ColumnList), + member(Num, ColumnList), subgrid(State, Row, Col, Subgrid), + member(Num, Subgrid). 

% Extract a column from the grid column([], _, []). column([Row|Rest], Col, [Value|Values]) :- nth1(Col, Row, Value), column(Rest, Col, Values). 

% Extract the 2x2 subgrid containing (Row, Col) subgrid(State, Row, Col, Subgrid) :- SubRow is ((Row - 1) // 2) * 2 + 1, SubCol is ((Col - 1) // 2) * 2 + 1, nth1(SubRow, State, Row1), SubRow1 is SubRow + 1, nth1(SubRow1, State, Row2), nth1(SubCol, Row1, A), SubCol1 is SubCol + 1, nth1(SubCol1, Row1, B), nth1(SubCol, Row2, C), nth1(SubCol1, Row2, D), Subgrid = [A, B, C, D]. 

% Fill cell (Row, Col) with Num fill_cell(State, Row, Col, Num, NewState) :- nth1(Row, State, RowList), replace(RowList, Col, Num, NewRowList), replace(State, Row, NewRowList, NewState). 

% Helper: Replace element at position in a list replace([_|T], 1, X, [X|T]). replace([H|T], Pos, X, [H|NewT]) :- Pos > 1, NextPos is Pos - 1, replace(T, NextPos, X, NewT). 

 