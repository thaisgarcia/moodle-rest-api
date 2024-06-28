# API REST - Moodle

## Obter Detalhes de Cada Questão para Todas Tentativas de um Questionário

Para obter os detalhes de cada questão para todas tentativas, primeiro temos que encontrar as tentativas desse questionário. Para evitar a inserção manual do ID das tentativas, usaremos duas funções ao mesmo tempo. Isso envolve duas etapas principais: obter as tentativas do questionário e, em seguida, obter os detalhes de cada questão para essas tentativas.

Vamos utilizar a linguagem M, que é a linguagem de script usada no Power BI para manipulação de dados.

### Passo a Passo

1. No Power BI, clique em "Obter dados".
2. Crie uma "Consulta em branco".
3. Clique em "Editor Avançado".
4. Copie o código a seguir e cole no campo de texto do "Editor Avançado".

```m
let
    // Parâmetros (substituir)
    token = "your_token",
    quizid = id_quiz,
    courseid = id_course,

    // Função para fazer a chamada à API do Moodle
    call_moodle_api = (wsfunction as text, optional params as record) =>
    let
        url = "https://your-moodle-site.com/webservice/rest/server.php", // Substituir
        defaultParams = [
            wstoken = token,
            moodlewsrestformat = "json"
        ],
        allParams = if (params <> null) then Record.Combine({defaultParams, params}) else defaultParams,
        fullUrl = url & "?" & Uri.BuildQueryString(allParams),
        response = Web.Contents(fullUrl),
        jsonResponse = Json.Document(response)
    in
        jsonResponse,

    // Função para obter usuários inscritos no curso
    get_enrolled_users = () =>
    let
        params = [
            wsfunction = "core_enrol_get_enrolled_users",
            courseid = Text.From(courseid)
        ],
        response = call_moodle_api("core_enrol_get_enrolled_users", params)
    in
        response,

    // Função para obter tentativas do quiz de um usuário
    get_user_attempts = (userid as text) =>
    let
        params = [
            wsfunction = "mod_quiz_get_user_attempts",
            quizid = Text.From(quizid),
            userid = userid
        ],
        response = call_moodle_api("mod_quiz_get_user_attempts", params)
    in
        response,

    // Função para obter detalhes de uma tentativa
    get_attempt_review = (attemptid as text) =>
    let
        params = [
            wsfunction = "mod_quiz_get_attempt_review",
            attemptid = attemptid
        ],
        response = call_moodle_api("mod_quiz_get_attempt_review", params)
    in
        response,

    // Obter lista de usuários inscritos no curso
    users = get_enrolled_users(),

    // Listar todas as tentativas
    all_attempts = List.Accumulate(users, {}, (state, current) =>
        let
            userid = Text.From(current[id]),
            user_attempts = get_user_attempts(userid),
            attempts_with_review = List.Transform(user_attempts[attempts], each Record.AddField(_, "details", get_attempt_review(Text.From(_[id])))),
            new_state = state & attempts_with_review
        in
            new_state
    )
in
    all_attempts
```

5. Substitua `your_token` pelo seu token de acesso à API do Moodle, `id_quiz` pelo ID do questionário que deseja consultar e `id_course` pelo ID do curso correspondente.
6. Clique em concluido, o Power BI irá executar o script e carregar os dados resultantes. 

### Conclusão

Seguindo esses passos, você conseguirá obter os detalhes de cada questão para todas as tentativas de um questionário no Moodle usando o Power BI.

### Próximos Passos

**a.** Adicionar visualizações no Power BI para analisar os detalhes das questões.  
**b.** Criar métricas personalizadas para avaliar o desempenho dos estudantes com base nas respostas das questões.
