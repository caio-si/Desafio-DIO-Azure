# Desafio-DIO-Azure


Projeto: API RESTful de Gerenciamento de Tarefas
Descrição do Projeto
O objetivo deste projeto é criar uma API que permita aos usuários criar, ler, atualizar e excluir tarefas. A API será desenvolvida usando Azure Functions e os dados serão armazenados no Azure Cosmos DB.

Tecnologias Utilizadas
Azure Functions: Para criar a API sem servidor.
Azure Cosmos DB: Para armazenar as tarefas.
C# ou JavaScript: Para escrever as funções.
Passo a Passo
1. Criar um Banco de Dados no Azure Cosmos DB
Acesse o portal do Azure e crie um novo recurso "Azure Cosmos DB".
Escolha a API (por exemplo, SQL API) e crie um banco de dados chamado TaskDB.
Dentro desse banco de dados, crie uma coleção chamada Tasks.
2. Criar um Projeto de Azure Functions
Use o Visual Studio ou o Azure Functions Core Tools para criar um novo projeto de Azure Functions.
Se você estiver usando C#, crie um novo projeto do tipo "Function App".
Se estiver usando JavaScript, crie um novo projeto com func init.
3. Implementar Funções da API
a. Criar Tarefa (POST)

csharp
Copiar código
[FunctionName("CreateTask")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "post", Route = "tasks")] HttpRequest req,
    [CosmosDB(databaseName: "TaskDB", collectionName: "Tasks", ConnectionStringSetting = "CosmosDBConnectionString")] IAsyncCollector<TaskItem> tasksOut,
    ILogger log)
{
    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    TaskItem task = JsonConvert.DeserializeObject<TaskItem>(requestBody);
    await tasksOut.AddAsync(task);
    return new CreatedResult($"/tasks/{task.Id}", task);
}
b. Ler Tarefas (GET)

csharp
Copiar código
[FunctionName("GetTasks")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", Route = "tasks")] HttpRequest req,
    [CosmosDB(databaseName: "TaskDB", collectionName: "Tasks", ConnectionStringSetting = "CosmosDBConnectionString")] DocumentClient client,
    ILogger log)
{
    var tasks = await client.CreateDocumentQuery<TaskItem>(UriFactory.CreateDocumentCollectionUri("TaskDB", "Tasks"))
        .ToListAsync();
    return new OkObjectResult(tasks);
}
c. Atualizar Tarefa (PUT)

csharp
Copiar código
[FunctionName("UpdateTask")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "put", Route = "tasks/{id}")] HttpRequest req,
    string id,
    [CosmosDB(databaseName: "TaskDB", collectionName: "Tasks", ConnectionStringSetting = "CosmosDBConnectionString")] DocumentClient client,
    ILogger log)
{
    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    TaskItem updatedTask = JsonConvert.DeserializeObject<TaskItem>(requestBody);
    await client.ReplaceDocumentAsync(UriFactory.CreateDocumentUri("TaskDB", "Tasks", id), updatedTask);
    return new OkObjectResult(updatedTask);
}
d. Excluir Tarefa (DELETE)

csharp
Copiar código
[FunctionName("DeleteTask")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "delete", Route = "tasks/{id}")] HttpRequest req,
    string id,
    [CosmosDB(databaseName: "TaskDB", collectionName: "Tasks", ConnectionStringSetting = "CosmosDBConnectionString")] DocumentClient client,
    ILogger log)
{
    await client.DeleteDocumentAsync(UriFactory.CreateDocumentUri("TaskDB", "Tasks", id));
    return new NoContentResult();
}
4. Configuração do Azure Functions
No arquivo local.settings.json, adicione a string de conexão do Cosmos DB:
json
Copiar código
{
  "IsEncrypted": false,
  "Values": {
    "CosmosDBConnectionString": "<sua-connection-string>"
  }
}
5. Implantar no Azure
Publique a Function App diretamente do Visual Studio ou use a CLI do Azure para implantar seu código no Azure.
6. Testar a API
Use ferramentas como Postman ou Insomnia para testar os endpoints da API (criar, ler, atualizar e excluir tarefas).
7. Documentação
Crie um arquivo README.md no repositório do GitHub explicando como configurar e usar a API.
Conclusão
Este projeto oferece uma base sólida para um sistema de gerenciamento de tarefas utilizando Azure Functions e Cosmos DB. Você pode expandir este projeto com autenticação, interface de usuário, ou integrar outras funcionalidades, como notificações. Sinta-se à vontade para personalizar e melhorar conforme necessário! Se precisar de mais detalhes ou ajuda em alguma parte, é só avisar!
