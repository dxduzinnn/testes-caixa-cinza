# Testes Caixa Cinza em API de Autenticação — Supabase + Postman

> **Aluno:** PREENCHER_NOME — **RA:** PREENCHER_RA — **Turma:** PREENCHER_TURMA
> **Data de execução dos testes:** PREENCHER_DATA

## Sumário

1. [Introdução](#introdução)
2. [Objetivo da Atividade](#objetivo-da-atividade)
3. [Configuração do Supabase](#configuração-do-supabase)
4. [Configuração do Postman](#configuração-do-postman)
5. [Configuração das Requisições](#configuração-das-requisições)
6. [Execução dos Testes](#execução-dos-testes)
7. [Registro dos Testes](#registro-dos-testes)
8. [Resultados Obtidos](#resultados-obtidos)
9. [Conclusão](#conclusão)

### Estrutura do repositório

```
/README.md      → documentação completa da atividade
/postman/       → coleção e environment exportados do Postman
/prints/        → evidências (capturas de tela) produzidas durante a atividade
/planilha/      → planilha de registro dos testes
```

---

# Introdução

Testes de software verificam se um sistema se comporta como esperado em situações normais e em situações de erro. Em APIs de autenticação, isso é ainda mais importante: uma falha pode permitir que alguém entre sem credenciais válidas ou impedir que um usuário legítimo acesse o sistema.

Nesta atividade foi testado o serviço de autenticação do **Supabase** (Supabase Auth), uma plataforma que oferece banco de dados PostgreSQL, autenticação e APIs prontas. Os testes foram feitos com o **Postman Desktop**, ferramenta que permite montar, enviar e validar requisições HTTP.

A abordagem utilizada foi a de **teste caixa cinza**. No teste caixa preta, o testador conhece apenas entradas e saídas; no caixa branca, conhece o código-fonte. No caixa cinza, o testador tem **conhecimento parcial** da estrutura: neste caso, eu conhecia o endpoint, o método HTTP, os headers obrigatórios, o formato do body JSON e os possíveis códigos de resposta, mas não o código interno do serviço de autenticação. Esse conhecimento permitiu planejar cenários mais precisos, como o envio de requisições sem a chave de API ou com campos vazios.

# Objetivo da Atividade

Compreender e aplicar testes caixa cinza em uma API de autenticação, sendo capaz de:

- configurar um ambiente de autenticação por e-mail e senha no Supabase;
- criar um usuário para testes;
- entender a estrutura de uma requisição HTTP (método, URL, headers e body);
- configurar workspace, environment, variáveis e requisições no Postman;
- executar cenários de sucesso e de erro e validar status HTTP, respostas JSON, mensagens de erro e o access token;
- registrar e documentar tecnicamente todo o processo.

# Configuração do Supabase

### Criação do projeto

1. Acessei [supabase.com](https://supabase.com) e criei uma conta (PREENCHER: com GitHub ou com e-mail).
2. No painel, cliquei em **New project**, escolhi a organização, defini o nome **PREENCHER_NOME_DO_PROJETO**, uma senha para o banco de dados e a região **PREENCHER_REGIÃO** (ex.: South America – São Paulo).
3. Aguardei o provisionamento do projeto até o painel ficar disponível.

### Tipo de autenticação

Foi utilizada a autenticação por **e-mail e senha** (provider *Email*), em **Authentication → Sign In / Providers → Email**, que já vem habilitada por padrão. Esse é o tipo de autenticação usado pelo endpoint `/auth/v1/token?grant_type=password`, em que o cliente envia e-mail e senha e recebe um token de acesso.

PREENCHER: informar se a opção *Confirm email* ficou ligada ou desligada.

### Criação do usuário de teste

Em **Authentication → Users → Add user → Create new user**, criei o usuário **PREENCHER_EMAIL_DO_USUARIO** com uma senha definida por mim, marcando a opção **Auto Confirm User**, para que o usuário já ficasse confirmado e pudesse fazer login sem precisar confirmar o e-mail.

### Credenciais da API

Em **Project Settings → API Keys** (e **Data API** para a URL), obtive:

| Informação | Onde encontrar | Uso nos testes |
|---|---|---|
| API URL (Project URL) | Project Settings → Data API | Base da URL das requisições (`base_url`) |
| API KEY (anon / publishable) | Project Settings → API Keys | Headers `apikey` e `Authorization` (`api_key`) |

> **Segurança:** foi usada apenas a chave **pública** (anon/publishable), que é feita para uso em clientes. A chave **service_role/secret** nunca foi usada nem exposta, pois ignora as regras de segurança do banco. A senha do usuário de teste não está publicada neste repositório.

### Objetivo da configuração

Criar um ambiente de autenticação real e controlado, com um usuário conhecido, para que fosse possível comparar o comportamento da API com credenciais corretas e incorretas.

### Evidências

| Evidência | Print |
|---|---|
| Projeto criado | ![Projeto criado](prints/01-supabase-projeto-criado.png) |
| Painel do Supabase | ![Painel](prints/02-supabase-painel.png) |
| Autenticação por e-mail habilitada | ![Email provider](prints/03-supabase-auth-email-habilitado.png) |
| Usuário criado | ![Usuário](prints/04-supabase-usuario-criado.png) |
| Credenciais da API | ![API](prints/05-supabase-credenciais-api.png) |
| Identificação do aluno no ambiente | ![Identificação](prints/06-supabase-identificacao-aluno.png) |

# Configuração do Postman

### Workspace

No Postman Desktop, em **Workspaces → Create Workspace**, criei o workspace **PREENCHER_NOME_WORKSPACE** para concentrar todos os artefatos da atividade.

### Environment

Em **Environments → +**, criei o environment **Supabase - Testes Auth** e o selecionei no canto superior direito. Um environment guarda valores que mudam de um ambiente para outro (URL, chaves, usuários), evitando digitá-los em cada requisição.

| Variável | Descrição | Tipo |
|---|---|---|
| `base_url` | URL da API do Supabase (ex.: `https://xxxx.supabase.co`) | default |
| `api_key` | API KEY pública do projeto | secret |
| `email` | E-mail do usuário criado | default |
| `password` | Senha do usuário criado | secret |
| `access_token` | Preenchida automaticamente pelo teste CT01 | secret |

### Finalidade das variáveis

As variáveis são usadas com a sintaxe `{{nome}}`. Assim, as requisições não têm dados sensíveis escritos diretamente nelas, ficam reutilizáveis (basta trocar o environment para testar outro projeto) e um mesmo valor é alterado em um único lugar. O tipo *secret* oculta o valor na tela.

### Como o Postman foi utilizado

Criei a coleção **Testes Caixa Cinza - Supabase Auth**, com uma requisição por cenário. Cada requisição tem, na aba **Scripts → Post-response**, testes automáticos (`pm.test`) que validam o status HTTP, a presença ou ausência do `access_token` e a mensagem de erro. A coleção foi executada individualmente e também pelo **Collection Runner**. A coleção e o environment exportados estão em [`/postman`](postman/) (o environment foi exportado **sem** a chave e a senha).

### Evidências

| Evidência | Print |
|---|---|
| Workspace criado | ![Workspace](prints/07-postman-workspace.png) |
| Environment criado | ![Environment](prints/08-postman-environment.png) |
| Variáveis configuradas | ![Variáveis](prints/09-postman-variaveis.png) |
| Organização do Postman (coleção) | ![Coleção](prints/10-postman-organizacao.png) |

# Configuração das Requisições

| Item | Valor |
|---|---|
| Endpoint | `{{base_url}}/auth/v1/token?grant_type=password` |
| Método HTTP | `POST` |
| Parâmetro de query | `grant_type=password` (login com e-mail e senha) |
| Finalidade | Autenticar o usuário e obter um `access_token` (JWT) |

O método **POST** é usado porque as credenciais são enviadas no corpo da requisição, e não na URL, onde poderiam ficar registradas em históricos e logs.

### Exemplo de Header

| Header | Valor | Função |
|---|---|---|
| `apikey` | `{{api_key}}` (XXXXX) | Identifica o projeto Supabase; sem ela a requisição é rejeitada |
| `Authorization` | `Bearer {{api_key}}` | Envia a chave no formato Bearer |
| `Content-Type` | `application/json` | Informa que o body está em JSON |

### Exemplo do Body JSON

```json
{
  "email": "usuario@email.com",
  "password": "123456"
}
```

Na coleção, o body do login válido usa as variáveis:

```json
{
  "email": "{{email}}",
  "password": "{{password}}"
}
```

### Exemplo de resposta de sucesso (estrutura)

```json
{
  "access_token": "eyJhbGciOi...",
  "token_type": "bearer",
  "expires_in": 3600,
  "expires_at": 1700000000,
  "refresh_token": "...",
  "user": { "id": "...", "email": "...", "role": "authenticated" }
}
```

### Evidências

| Evidência | Print |
|---|---|
| Endpoint configurado | ![Endpoint](prints/11-requisicao-endpoint.png) |
| Headers | ![Headers](prints/12-requisicao-headers.png) |
| Body JSON | ![Body](prints/13-requisicao-body.png) |
| Requisição criada no Postman | ![Requisição](prints/14-requisicao-criada.png) |

# Execução dos Testes

### Cenários executados

Foram executados os cinco cenários obrigatórios e um cenário extra (CT06), planejado a partir do conhecimento parcial da API (característica do teste caixa cinza): o que acontece se a requisição estiver correta, mas sem a chave de API.

Em cada execução foram validados: **status HTTP**, **resposta JSON**, **mensagem de erro**, **presença/ausência do access token** e **comportamento da autenticação**.

### Tabela de Cenários

| Cenário | Entrada Utilizada | Resultado Esperado | Resultado Obtido | Status |
|---|---|---|---|---|
| CT01 – Login válido | Usuário correto + senha correta | Status 200 + access token | PREENCHER (ex.: 200, access_token retornado) | PREENCHER |
| CT02 – Senha incorreta | Usuário correto + `SenhaErrada!999` | Erro de autenticação, sem token | PREENCHER (status + msg) | PREENCHER |
| CT03 – Usuário inexistente | `usuario.inexistente.teste@exemplo.com` | Acesso negado, sem token | PREENCHER | PREENCHER |
| CT04 – Campos vazios | `email: ""`, `password: ""` | Erro de validação | PREENCHER | PREENCHER |
| CT05 – Credenciais inválidas | `email-sem-arroba` + `123` | Mensagem de erro | PREENCHER | PREENCHER |
| CT06 – Sem apikey (extra) | Credenciais corretas, sem headers de chave | Status 401 | PREENCHER | PREENCHER |

### Respostas observadas

PREENCHER — cole aqui o JSON real retornado em cada cenário de erro. Exemplo de como documentar:

```json
// CT02 – Senha incorreta – Status PREENCHER
COLE_AQUI_O_JSON_RETORNADO
```

### Erros encontrados

PREENCHER — descreva as diferenças entre o esperado e o obtido. Pontos a observar:

- CT02 e CT03 retornaram a **mesma** mensagem? (isso é esperado e desejável: a API não revela se o e-mail existe, evitando enumeração de usuários);
- CT04 retornou mensagem de validação clara ou genérica?
- CT05: a API validou o formato do e-mail ou apenas recusou as credenciais?

### Evidências

| Evidência | Print |
|---|---|
| CT01 – login válido (status 200 + access token) | ![CT01](prints/15-ct01-login-valido.png) |
| CT02 – senha incorreta | ![CT02](prints/16-ct02-senha-incorreta.png) |
| CT03 – usuário inexistente | ![CT03](prints/17-ct03-usuario-inexistente.png) |
| CT04 – campos vazios | ![CT04](prints/18-ct04-campos-vazios.png) |
| CT05 – credenciais inválidas | ![CT05](prints/19-ct05-credenciais-invalidas.png) |
| CT06 – sem apikey | ![CT06](prints/20-ct06-sem-apikey.png) |
| Aba Test Results (testes automáticos) | ![Test results](prints/21-test-results.png) |
| Collection Runner | ![Runner](prints/22-collection-runner.png) |

# Registro dos Testes

### Como os testes foram registrados

Cada execução foi registrada na planilha [`planilha/Planilha_Testes_Caixa_Cinza.xlsx`](planilha/), com os campos:

| Campo | Descrição |
|---|---|
| ID do teste | Identificação (CT01 a CT06) |
| Cenário | Descrição do caso |
| Entrada | Dados utilizados |
| Resultado esperado | Resultado correto previsto antes da execução |
| Resultado obtido | Resultado real (status HTTP + trecho da resposta) |
| Status | Sucesso / Falha |
| Observações | Comentários |

O resultado esperado foi preenchido **antes** da execução e o resultado obtido **depois**, com base no que o Postman mostrou, para que a comparação fosse honesta.

### Importância da documentação dos testes

A documentação permite repetir os testes (reprodutibilidade), comprovar o que foi testado (evidência), comparar resultados entre versões do sistema (regressão) e comunicar falhas à equipe de desenvolvimento com dados objetivos: entrada, resultado esperado e resultado obtido.

### Falhas identificadas

PREENCHER — liste os testes com status *Falha* (se houver) e o motivo. Se todos passaram, explique que o comportamento obtido foi igual ao esperado e cite as observações relevantes (por exemplo, mensagens genéricas de erro).

### Evidências

| Evidência | Print |
|---|---|
| Planilha preenchida | ![Planilha](prints/23-planilha-preenchida.png) |

Exemplo de registro realizado:

| ID | Cenário | Entrada | Resultado esperado | Resultado obtido | Status | Observações |
|---|---|---|---|---|---|---|
| CT01 | Login válido | e-mail e senha corretos | 200 + access_token | PREENCHER | PREENCHER | PREENCHER |
| CT02 | Senha incorreta | senha `SenhaErrada!999` | Erro, sem token | PREENCHER | PREENCHER | PREENCHER |

# Resultados Obtidos

PREENCHER após a execução:

| Total de testes | Sucesso | Falha |
|---|---|---|
| 6 | PREENCHER | PREENCHER |

PREENCHER — resumo em 1 ou 2 parágrafos: o login válido retornou status e token? Os cenários inválidos foram bloqueados sem devolver token? Quais códigos e mensagens apareceram? Qual o tempo médio de resposta mostrado pelo Postman?

# Conclusão

PREENCHER com suas palavras (o professor exige análise própria). Responda obrigatoriamente:

- **Os testes foram executados corretamente?**
- **A autenticação funcionou?** (login válido gerou token? os inválidos foram bloqueados?)
- **Quais dificuldades encontrei?** (ex.: localizar a API Key, confirmação de e-mail, configurar variáveis)
- **Quais falhas foram identificadas?**
- **Quais melhorias poderiam ser implementadas?** Ideias: mensagens de validação mais específicas para campos vazios e e-mail malformado; limite de tentativas (rate limit) e bloqueio após várias senhas erradas; exigência de senha forte; autenticação em dois fatores; automação dos testes com Newman em pipeline de CI.
- **Qual a importância dos testes caixa cinza em APIs?** Conhecer parte da estrutura (endpoint, headers, formato do body e códigos de resposta) permite criar cenários mais direcionados do que no caixa preta — como remover a `apikey` ou enviar campos vazios — sem depender do código-fonte, aproximando o teste do que um cliente real ou um atacante faria.
