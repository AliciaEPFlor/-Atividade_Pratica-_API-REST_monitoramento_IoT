<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
</head>
<body>

<h1>🌡️ API de Monitoramento de Temperatura e Umidade com ESP32 (Documentação da API REST)</h1>

<p>API RESTful desenvolvida em Node.js + Express para receber e armazenar leituras de temperatura e umidade de um sensor ESP32, servindo como backend para um sistema de monitoramento ambiental.</p>

<hr>

<h2>📚 Sumário</h2>
<ol>
    <li><a href="#pesquisa-conceitual">Pesquisa Conceitual</a></li>
    <li><a href="#documentacao-dos-endpoints">Documentação dos Endpoints</a></li>
    <li><a href="#diagrama-da-arquitetura">Diagrama da Arquitetura</a></li>
    <li><a href="#como-rodar">Como Rodar</a></li>
</ol>

<hr>

<h2 id="pesquisa-conceitual">📖 PARTE 1 — PESQUISA CONCEITUAL</h2>

<h3>1.1) O que é uma API?</h3>

<p><strong>API</strong> significa <strong>Application Programming Interface</strong> (Interface de Programação de Aplicações). Em termos simples, é um conjunto de regras e definições que permite que diferentes softwares se comuniquem entre si. Ela funciona como um "contrato" que especifica como um programa pode solicitar serviços ou dados de outro, sem precisar saber como esse outro programa funciona internamente.</p>

<p>Para entender melhor, pense na API como um <strong>cardápio de um restaurante</strong>. O cliente (aplicação que consome a API) não precisa entrar na cozinha e saber como os pratos são preparados (os detalhes internos do servidor). Ele simplesmente olha o cardápio (a documentação da API), escolhe um item (um endpoint) e faz o pedido ao garçom (a API). O garçom leva o pedido para a cozinha (o servidor) e, quando o prato está pronto, traz a resposta de volta para o cliente. Essa abstração é fundamental porque desacopla o cliente do servidor, permitindo que cada um evolua independentemente, desde que mantenham o contrato da API.</p>

<p>No contexto da web, as APIs são a espinha dorsal da integração entre serviços. Quando você usa um aplicativo de clima no seu celular, ele está consumindo uma API de um serviço meteorológico. Quando você faz login em um site usando sua conta do Google, o site está utilizando a API do Google para autenticação. APIs permitem que empresas exponham seus dados e funcionalidades de forma controlada e segura, fomentando um ecossistema de inovação onde desenvolvedores podem construir novas aplicações sobre plataformas existentes.</p>

<h3>1.2) O que é REST?</h3>

<p><strong>REST</strong> significa <strong>Representational State Transfer</strong> (Transferência de Estado Representacional). É um estilo arquitetural, não um protocolo ou padrão rígido, criado por Roy Fielding em sua tese de doutorado. Uma API que segue os princípios REST é chamada de <strong>API RESTful</strong>. Esses princípios definem um conjunto de restrições que, quando aplicadas, resultam em um sistema mais simples, escalável e confiável.</p>

<p>As principais características de uma API REST incluem:</p>
<ul>
    <li><strong>Arquitetura Cliente-Servidor</strong>: Separação clara entre a interface do usuário (cliente) e o gerenciamento de dados (servidor).</li>
    <li><strong>Stateless (Sem Estado)</strong>: Cada requisição do cliente ao servidor deve conter todas as informações necessárias para ser compreendida. O servidor não mantém nenhum estado sobre as requisições anteriores. Isso facilita a escalabilidade, pois qualquer servidor pode responder a qualquer requisição.</li>
    <li><strong>Cacheable (Armazenável em Cache)</strong>: As respostas do servidor devem indicar se podem ser armazenadas em cache pelo cliente, melhorando a eficiência.</li>
    <li><strong>Interface Uniforme</strong>: Um conjunto padronizado de operações, como os métodos HTTP (GET, POST, PUT, DELETE), que são aplicados a recursos identificados por URLs.</li>
</ul>

<p>Pense no REST como um <strong>sistema de bibliotecas</strong>. Cada livro (recurso) tem um identificador único (URL). Para pegar um livro, você usa a operação "pegar" (GET). Para adicionar um novo livro, você usa a operação "adicionar" (POST). O bibliotecário (servidor) não precisa lembrar quem você é entre uma visita e outra (stateless); cada vez que você chega, você informa o que deseja.</p>

<h3>1.3) O que é CRUD?</h3>

<p><strong>CRUD</strong> é um acrônimo para <strong>Create (Criar), Read (Ler), Update (Atualizar) e Delete (Deletar)</strong>. Ele representa as quatro operações básicas que podem ser realizadas em dados persistentes. É um conceito fundamental em sistemas que gerenciam informações e está diretamente relacionado aos métodos HTTP em APIs REST:</p>

<table>
    <thead>
        <tr>
            <th>Operação CRUD</th>
            <th>Método HTTP</th>
            <th>Descrição</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><strong>Create (Criar)</strong></td>
            <td><code>POST</code></td>
            <td>Cria um novo recurso. No nosso projeto, usamos <code>POST /api/dados</code> para criar um novo registro de leitura.</td>
        </tr>
        <tr>
            <td><strong>Read (Ler)</strong></td>
            <td><code>GET</code></td>
            <td>Recupera um ou mais recursos. No nosso projeto, usamos <code>GET /api/dados</code> para listar todas as leituras.</td>
        </tr>
        <tr>
            <td><strong>Update (Atualizar)</strong></td>
            <td><code>PUT</code> ou <code>PATCH</code></td>
            <td>Atualiza um recurso existente. <code>PUT</code> geralmente substitui o recurso inteiro, enquanto <code>PATCH</code> aplica uma atualização parcial.</td>
        </tr>
        <tr>
            <td><strong>Delete (Deletar)</strong></td>
            <td><code>DELETE</code></td>
            <td>Remove um recurso.</td>
        </tr>
    </tbody>
</table>

<p>Embora nosso projeto atual implemente apenas o <strong>C</strong> e o <strong>R</strong>, o <strong>U</strong> e o <strong>D</strong> seriam implementados em uma versão mais completa. Por exemplo, teríamos um endpoint <code>PUT /api/dados/:id</code> para corrigir uma leitura erroneamente registrada e um <code>DELETE /api/dados/:id</code> para remover uma leitura antiga.</p>

<h3>1.4) O que é HTTP e o que são status codes?</h3>

<p><strong>HTTP</strong> significa <strong>Hypertext Transfer Protocol</strong> (Protocolo de Transferência de Hipertexto). É o protocolo fundamental da World Wide Web, responsável pela comunicação entre clientes (como navegadores ou o Postman) e servidores. Ele funciona em um modelo de <strong>requisição e resposta</strong>: o cliente envia uma requisição ao servidor, e o servidor responde com uma mensagem contendo um <strong>código de status</strong>.</p>

<p>O <strong>status code</strong> é um número de três dígitos que indica o resultado da tentativa de processar a requisição. É a primeira coisa que o cliente olha para entender se sua ação foi bem-sucedida ou se algo deu errado. Eles são agrupados em cinco classes:</p>
<ul>
    <li><strong>1xx (Informativo)</strong>: A requisição foi recebida e o processo continua.</li>
    <li><strong>2xx (Sucesso)</strong>: A requisição foi recebida, entendida e aceita com sucesso.</li>
    <li><strong>3xx (Redirecionamento)</strong>: Ações adicionais precisam ser tomadas para completar a requisição.</li>
    <li><strong>4xx (Erro do Cliente)</strong>: A requisição contém sintaxe incorreta ou não pode ser atendida.</li>
    <li><strong>5xx (Erro do Servidor)</strong>: O servidor falhou ao processar uma requisição aparentemente válida.</li>
</ul>

<h4>Tabela de Status Codes</h4>

<table>
    <thead>
        <tr>
            <th>Código</th>
            <th>Nome</th>
            <th>Significado</th>
            <th>Quando aparece no nosso projeto</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><strong>200</strong></td>
            <td>OK</td>
            <td>A requisição foi bem-sucedida. O corpo da resposta contém os dados solicitados.</td>
            <td>Ao listar todas as leituras com <code>GET /api/dados</code> com sucesso.</td>
        </tr>
        <tr>
            <td><strong>201</strong></td>
            <td>Created</td>
            <td>A requisição foi bem-sucedida e um novo recurso foi criado.</td>
            <td>Ao enviar uma nova leitura com <code>POST /api/dados</code> e ela é salva com sucesso.</td>
        </tr>
        <tr>
            <td><strong>400</strong></td>
            <td>Bad Request</td>
            <td>O servidor não pôde processar a requisição devido a um erro do cliente, como dados inválidos.</td>
            <td>Se o ESP32 enviar dados em um formato JSON incorreto ou faltando campos obrigatórios (por exemplo, se faltar o campo <code>temperatura</code>).</td>
        </tr>
        <tr>
            <td><strong>404</strong></td>
            <td>Not Found</td>
            <td>O recurso solicitado não foi encontrado no servidor.</td>
            <td>Se o cliente tentar acessar uma rota que não existe, como <code>GET /api/leituras</code>.</td>
        </tr>
        <tr>
            <td><strong>500</strong></td>
            <td>Internal Server Error</td>
            <td>O servidor encontrou uma condição inesperada que o impediu de atender a requisição.</td>
            <td>Se houver um erro inesperado no código do servidor, como uma falha ao acessar o banco de dados em memória.</td>
        </tr>
    </tbody>
</table>

<h3>1.5) O que é JSON e por que usamos?</h3>

<p><strong>JSON</strong> significa <strong>JavaScript Object Notation</strong> (Notação de Objetos JavaScript). É um formato de texto leve e fácil de ler para humanos e fácil de interpretar e gerar para máquinas. Embora tenha se originado do JavaScript, é um formato independente de linguagem, sendo suportado pela maioria das linguagens de programação modernas.</p>

<p>Sua estrutura é baseada em dois elementos principais:</p>
<ul>
    <li><strong>Pares de chave/valor</strong>: Como um dicionário ou objeto.</li>
    <li><strong>Listas ordenadas de valores</strong>: Como um array.</li>
</ul>

<p>No nosso projeto, cada leitura de dados é representada em JSON. Um exemplo típico de uma leitura enviada pelo ESP32 via <code>POST /api/dados</code> seria:</p>

<pre><code>{
  "temperatura": 25.5,
  "umidade": 60.0
}</code></pre>

<p>E a resposta do servidor, com o ID gerado, seria:</p>

<pre><code>{
  "id": 1,
  "temperatura": 25.5,
  "umidade": 60.0,
}</code></pre>

<p>Usamos JSON por diversos motivos: é <strong>simples</strong> e legível, <strong>compacto</strong> (ocupando menos espaço que XML), é um formato nativo do JavaScript (tornando-o muito eficiente em aplicações web), e é suportado por praticamente todas as linguagens de programação, o que o torna a escolha padrão para a maioria das APIs modernas.</p>

<hr>

<h2 id="documentacao-dos-endpoints">🛠️ PARTE 2 — DOCUMENTAÇÃO DOS ENDPOINTS/ROTAS</h2>

<h3>Endpoint 1: GET /api/dados</h3>

<div class="endpoint">
    <h4>📌 Método e Rota</h4>
    <p><code><span class="badge badge-info">GET</span> /api/dados</code></p>

    📝 Descrição
    Recupera a lista de todas as leituras de temperatura e umidade armazenadas.</p>

    📋 Parâmetros
    Nenhum.

    📤 Exemplo de Requisição
    GET http://localhost:3000/api/dados

    ✅ Exemplo de Resposta (Sucesso)
    Status Code:</strong> <code><span class="badge">200 OK
    {
    "id": 1,
    "temperatura": 25.5,
    "umidade": 60.0,
    },
    
    {
    "id": 2,
    "temperatura": 26.0,
    "umidade": 58.5,
    }
</code></pre>
</div>

<h3>Endpoint 2: POST /api/dados</h3>

<div class="endpoint">
    <h4>📌 Método e Rota</h4>
    <p><code><span class="badge badge-success">POST</span> /api/dados</code></p>

    📝 Descrição
    Cria e armazena uma nova leitura de temperatura e umidade.

    📋 Parâmetros
    Body (JSON):
    temperatura = (number, obrigatório): Valor da temperatura medida.
    umidade = (number, obrigatório): Valor da umidade medida.
    
    <h4>📤 Exemplo de Requisição</h4>
    URL: POST http://localhost:3000/api/dados</code></p>
     Body:
    {
    "temperatura": 25.5,
    "umidade": 60.0
    }

    <h4>✅ Exemplo de Resposta (Sucesso)</h4>
    Status Code:
    "mensagem": "Leitura salva com sucesso!",
    "dado": 
    {
    "id": 3,
    "temperatura": 25.5,
    "umidade": 60.0,
    }

    <h4>❌ Exemplo de Resposta (Erro)</h4>
    Status Code:
    "mensagem": "Dados inválidos. Certifique-se de enviar 'temperatura' e 'umidade' como números."
    }
</code></pre>
</div>

<hr>

<h2 id="diagrama-da-arquitetura">🏗️ PARTE 3 — DIAGRAMA DA ARQUITETURA</h2>

<p>O diagrama a seguir ilustra o fluxo completo de dados do nosso sistema, desde a captura pelos sensores no ESP32 até a visualização no Postman.</p>
<figure>
  <figcaption>
  <img width="746" height="236" alt="image" src="https://github.com/user-attachments/assets/1506c629-edc0-4a47-814c-5a08b847b9ba" />

</div>

<h3>Fluxo de Dados:</h3>

<ol>
    <li><strong>Coleta:</strong> O ESP32 lê os dados dos sensores de temperatura e umidade.</li>
    <li><strong>Envio (Criação):</strong> O ESP32 envia uma requisição <code>POST</code> para a rota <code>/api/dados</code> da API, contendo os valores lidos no corpo da requisição em formato JSON.</li>
    <li><strong>Persistência:</strong> A API recebe a requisição, valida os dados e os armazena no banco de dados em memória.</li>
    <li><strong>Resposta de Criação:</strong> A API responde ao ESP32 com o status code <code>201 Created</code> e os dados da leitura, incluindo um ID único e um timestamp.</li>
    <li><strong>Consulta (Leitura):</strong> Um cliente, como o Postman, envia uma requisição <code>GET</code> para a mesma rota <code>/api/dados</code>.</li>
    <li><strong>Recuperação:</strong> A API consulta o banco de dados em memória para obter a lista de todas as leituras.</li>
    <li><strong>Resposta de Leitura:</strong> A API retorna uma resposta com status code <code>200 OK</code> e um array JSON contendo todas as leituras armazenadas.</li>
</ol>

<hr>

<h2 id="como-rodar">🚀 PARTE 4 — COMO RODAR E REFLEXÃO</h2>

<h3>4.1) Como Rodar</h3>

<h4>Pré-requisitos</h4>
<ul>
    <li><strong>Node.js:</strong> Versão 14.x ou superior instalada no seu computador. Você pode verificar a versão com o comando <code>node -v</code> no terminal.</li>
    <li><strong>npm:</strong> O gerenciador de pacotes do Node.js, que geralmente vem junto com o Node.js.</li>
    <li><strong>Postman (ou similar):</strong> Para testar as requisições à API.</li>
</ul>

<h4>Passos para Executar</h4>

<ol>
    <li>
        <p><strong>Clone o repositório (ou crie a pasta do projeto):</strong></p>
        <pre><code>git clone &lt;url-do-seu-repositorio&gt;
cd &lt;nome-da-pasta&gt;</code></pre>
        <p>(Ou crie uma nova pasta e inicialize-a com <code>npm init -y</code>)</p>
    </li>
    <li>
        <p><strong>Instale as dependências:</strong></p>
        <p>Execute o comando abaixo para instalar o Express e outras dependências listadas no <code>package.json</code>.</p>
        <pre><code>npm install</code></pre>
    </li>
    <li>
        <p><strong>Execute o servidor:</strong></p>
        <pre><code>node server.js</code></pre>
        <p>Você verá uma mensagem no terminal confirmando que o servidor está rodando, como:</p>
        <pre><code>Servidor rodando em http://localhost:3000</code></pre>
    </li>
    <li>
        <p><strong>Verifique se está funcionando:</strong></p>
        <p>Para testar rapidamente, abra seu navegador e acesse <code>http://localhost:3000</code>. Você deve ver uma mensagem simples, como "API de Temperatura e Umidade funcionando!".</p>
    </li>
    <li>
        <p><strong>Teste com o Postman:</strong></p>
        <ul>
            <li><strong>Para <code>GET /api/dados</code></strong>: Crie uma requisição do tipo GET com a URL <code>http://localhost:3000/api/dados</code> e clique em "Send".</li>
            <li><strong>Para <code>POST /api/dados</code></strong>: Crie uma requisição do tipo POST com a URL <code>http://localhost:3000/api/dados</code>. Vá na aba "Body", selecione "raw" e o formato "JSON", e cole o seguinte conteúdo:
                <pre><code>{
  "temperatura": 23.4,
  "umidade": 65.2
}</code></pre>
                Clique em "Send". O servidor deve retornar um status <code>201 Created</code>.
            </li>
        </ul>
    </li>
</ol>

<h3>4.2) Tecnologias Usadas</h3>

<ul>
    <li><strong>Node.js</strong>: Ambiente de execução JavaScript que permite rodar o código do servidor fora do navegador, utilizando o motor V8 do Google Chrome. É a base da nossa aplicação backend.</li>
    <li><strong>Express</strong>: Framework para Node.js que simplifica a criação de servidores e APIs, fornecendo ferramentas para definir rotas, lidar com requisições e respostas HTTP, e integrar middlewares. Usamos o Express para criar os endpoints da nossa API.</li>
    <li><strong>body-parser</strong>: Middleware do Express utilizado para interpretar o corpo das requisições HTTP e extrair dados em formato JSON, tornando-os acessíveis no objeto <code>req.body</code>. Sem ele, não conseguiríamos receber os dados enviados pelo ESP32.</li>
</ul>

<h3>4.3) Reflexão</h3>

<p>O que mais gostei nessa atividade foi ver a teoria se transformando em algo prático e funcional. Construir uma API do zero, entender cada parte do fluxo (desde a requisição até a resposta) e ver o Postman retornar os dados que acabamos de enviar é uma experiência muito gratificante. A sensação de "fazer acontecer" e conectar diferentes partes do sistema (servidor, banco de dados em memória, cliente) foi o ponto alto do aprendizado.</p>

<p>A maior dificuldade, sem dúvida, foi entender e lidar com os conceitos de forma integrada. No início, era desafiador conectar a ideia abstrata de uma "requisição HTTP" com o código que escrevemos. Coisas como configurar corretamente as rotas, garantir que o <code>body-parser</code> estivesse funcionando para receber o JSON, e tratar os diferentes status codes de erro (como <code>400</code> para dados inválidos) exigiram atenção e depuração constante. Superar esses desafios foi fundamental para consolidar o entendimento de como uma API REST realmente funciona.</p>

<hr>

<p><strong>Desenvolvido com ❤️ para o projeto de Monitoramento com ESP32</strong></p>

</body>
</html>
