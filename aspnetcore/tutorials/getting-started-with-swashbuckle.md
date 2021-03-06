---
title: Introdução ao Swashbuckle e ao ASP.NET Core
author: zuckerthoben
description: Saiba como adicionar o Swashbuckle ao seu projeto do ASP.NET Core para integrar a interface do usuário do Swagger.
manager: wpickett
ms.author: scaddie
ms.custom: mvc
ms.date: 03/26/2018
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: tutorials/get-started-with-swashbuckle
ms.openlocfilehash: e90339f2884dd9b20cf135f879c9cab6110efecf
ms.sourcegitcommit: f8852267f463b62d7f975e56bea9aa3f68fbbdeb
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/06/2018
---
# <a name="get-started-with-swashbuckle-and-aspnet-core"></a>Introdução ao Swashbuckle e ao ASP.NET Core

Por [Shayne Boyer](https://twitter.com/spboyer) e [Scott Addie](https://twitter.com/Scott_Addie)

Há três componentes principais bo Swashbuckle:

* [Swashbuckle.AspNetCore.Swagger](https://www.nuget.org/packages/Swashbuckle.AspNetCore.Swagger/): um modelo de objeto e um middleware do Swagger para expor objetos `SwaggerDocument` como pontos de extremidade JSON.

* [Swashbuckle.AspNetCore.SwaggerGen](https://www.nuget.org/packages/Swashbuckle.AspNetCore.SwaggerGen/): um gerador do Swagger cria objetos `SwaggerDocument` diretamente de modelos, controladores e rotas. Normalmente, ele é combinado com o middleware de ponto de extremidade do Swagger para expor automaticamente o JSON do Swagger.

* [Swashbuckle.AspNetCore.SwaggerUI](https://www.nuget.org/packages/Swashbuckle.AspNetCore.SwaggerUI/): uma versão incorporada da ferramenta de interface do usuário do Swagger. Ele interpreta o JSON do Swagger para criar uma experiência rica e personalizável para descrever a funcionalidade da API da Web. Ela inclui o agente de teste interno para os métodos públicos.

## <a name="package-installation"></a>Instalação do pacote

O Swashbuckle pode ser adicionado com as seguintes abordagens:

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* Da janela **Console do Gerenciador de Pacotes**:

    ```powershell
    Install-Package Swashbuckle.AspNetCore
    ```

* Da caixa de diálogo **Gerenciar Pacotes NuGet**:

  * Clique com o botão direito do mouse no projeto em **Gerenciador de Soluções** > **Gerenciar Pacotes NuGet**
  * Defina a **Origem do pacote** para "nuget.org"
  * Insira "Swashbuckle.AspNetCore" na caixa de pesquisa
  * Selecione o pacote "Swashbuckle.AspNetCore" na guia **Procurar** e clique em **Instalar**

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio para Mac](#tab/visual-studio-mac)

* Clique com o botão direito do mouse na pasta *Pacotes* em **Painel de Soluções** > **Adicionar Pacotes...**
* Defina a lista suspensa **Origem** da janela **Adicionar Pacotes** para "nuget.org"
* Insira Swashbuckle.AspNetCore na caixa de pesquisa
* Selecione o pacote "Swashbuckle.AspNetCore" no painel de resultados e clique em **Adicionar Pacote**

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Execute o comando a seguir do **Terminal Integrado**:

```console
dotnet add TodoApi.csproj package Swashbuckle.AspNetCore
```

# <a name="net-core-clitabnetcore-cli"></a>[CLI do .NET Core](#tab/netcore-cli)

Execute o seguinte comando:

```console
dotnet add TodoApi.csproj package Swashbuckle.AspNetCore
```

---

## <a name="add-and-configure-swagger-middleware"></a>Adicionar e configurar o middleware do Swagger

Adicione o gerador do Swagger à coleção de serviços no método `Startup.ConfigureServices`:

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/TodoApi.Swashbuckle/Startup2.cs?name=snippet_ConfigureServices&highlight=7-10)]

Importe o namespace a seguir para usar a classe `Info`:

```csharp
using Swashbuckle.AspNetCore.Swagger;
```

No método `Startup.Configure`, habilite o middleware para atender ao documento JSON gerado e à interface do usuário do Swagger:

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/TodoApi.Swashbuckle/Startup2.cs?name=snippet_Configure&highlight=4,7-10)]

Inicie o aplicativo e navegue até `http://localhost:<random_port>/swagger/v1/swagger.json`. O documento gerado que descreve os pontos de extremidade é exibido conforme é mostrado na [Especificação do Swagger (swagger.json)](xref:tutorials/web-api-help-pages-using-swagger#swagger-specification-swaggerjson).

A interface do usuário do Swagger pode ser encontrada em `http://localhost:<random_port>/swagger`. Explore a API por meio da interface do usuário do Swagger e incorpore-a em outros programas.

> [!TIP]
> Para atender à interface do usuário do Swagger na raiz do aplicativo (`http://localhost:<random_port>/`), defina a propriedade `RoutePrefix` como uma cadeia de caracteres vazia:
> 
> [!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/TodoApi.Swashbuckle/Startup3.cs?name=snippet_UseSwaggerUI&highlight=4)]

## <a name="customize--extend"></a>Personalizar e estender

O Swagger fornece opções para documentar o modelo de objeto e personalizar a interface do usuário para corresponder ao seu tema.

### <a name="api-info-and-description"></a>Descrição e informações da API

A ação de configuração passada para o método `AddSwaggerGen` adiciona informações como o autor, a licença e a descrição:

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/TodoApi.Swashbuckle/Startup.cs?range=21-40,46)]

A interface do usuário do Swagger exibe as informações da versão:

![A interface do usuário do Swagger com informações de versão: descrição, autor e link veja mais](web-api-help-pages-using-swagger/_static/custom-info.png)

### <a name="xml-comments"></a>comentários XML

Comentários XML podem ser habilitados com as seguintes abordagens:

#### <a name="visual-studiotabvisual-studio-xml"></a>[Visual Studio](#tab/visual-studio-xml/)
* Clique com o botão direito do mouse no projeto no **Gerenciador de Soluções** e selecione **Propriedades**
* Verifique a caixa **Arquivo de documentação XML** sob a seção **Saída** da guia **Build**:

![Compile a guia das propriedades do projeto](web-api-help-pages-using-swagger/_static/swagger-xml-comments.png)

#### <a name="visual-studio-for-mactabvisual-studio-mac-xml"></a>[Visual Studio para Mac](#tab/visual-studio-mac-xml/)
* Abra a caixa de diálogo **Opções do Projeto** > **Compilar** > **Compilador**
* Verifique a caixa **Gerar documentação XML** sob a seção **Opções Gerais**:

![Seção Opções Gerais das opções do projeto](web-api-help-pages-using-swagger/_static/swagger-xml-comments-mac.png)

#### <a name="visual-studio-codetabvisual-studio-code-xml"></a>[Visual Studio Code](#tab/visual-studio-code-xml/)
Adicione manualmente o trecho a seguir ao arquivo *.csproj*:

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/TodoApi.Swashbuckle/TodoApi.csproj?name=snippet_SuppressWarnings&highlight=2)]

* * *
A habilitação de comentários XML fornece informações de depuração para os membros e os tipos públicos não documentados. Os membros e tipos não documentados são indicados por mensagem de aviso. Por exemplo, a seguinte mensagem indica uma violação do código de aviso 1591:

```text
warning CS1591: Missing XML comment for publicly visible type or member 'TodoController.GetAll()'
```

Suprima os avisos definindo uma lista separada por ponto-e-vírgula dos códigos de aviso a serem ignorados no arquivo *.csproj*:

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/TodoApi.Swashbuckle/TodoApi.csproj?name=snippet_SuppressWarnings&highlight=3)]

Configure o Swagger para usar o arquivo XML gerado. Para sistemas operacionais Linux ou que não sejam Windows, os caminhos e nomes de arquivo podem diferenciar maiúsculas de minúsculas. Por exemplo, um arquivo *TodoApi.XML* é válido no Windows, mas não no CentOS.

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/TodoApi.Swashbuckle/Startup.cs?name=snippet_ConfigureServices&highlight=29-31)]

No código anterior, a [Reflexão](/dotnet/csharp/programming-guide/concepts/reflection) é usada para criar um nome de arquivo XML correspondente ao do projeto de API da Web. Essa abordagem garante que o nome do arquivo XML gerado corresponda ao nome do projeto. A propriedade [AppContext.BaseDirectory](/dotnet/api/system.appcontext.basedirectory#System_AppContext_BaseDirectory) é usada para construir um caminho para o arquivo XML.

Adicionar comentários de barra tripla a uma ação aprimora a interface do usuário do Swagger adicionando a descrição ao cabeçalho da seção. Adicione um elemento [\<summary>](/dotnet/csharp/programming-guide/xmldoc/summary) acima da ação `Delete`:

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_Delete&highlight=1-3)]

A interface do usuário do Swagger exibe o texto interno do elemento `<summary>` do código anterior:

![A interface do usuário do Swagger, mostrando o comentário XML 'Exclui um TodoItem específico'. para o método DELETE](web-api-help-pages-using-swagger/_static/triple-slash-comments.png)

A interface do usuário é controlada pelo esquema JSON gerado:

```json
"delete": {
    "tags": [
        "Todo"
    ],
    "summary": "Deletes a specific TodoItem.",
    "operationId": "ApiTodoByIdDelete",
    "consumes": [],
    "produces": [],
    "parameters": [
        {
            "name": "id",
            "in": "path",
            "description": "",
            "required": true,
            "type": "integer",
            "format": "int64"
        }
    ],
    "responses": {
        "200": {
            "description": "Success"
        }
    }
}
```

Adicione um elemento [\<remarks>](/dotnet/csharp/programming-guide/xmldoc/remarks) na documentação do método da ação `Create`. Ele complementa as informações especificadas no elemento `<summary>` e fornece uma interface de usuário do Swagger mais robusta. O conteúdo do elemento `<remarks>` pode consistir em texto, JSON ou XML.

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_Create&highlight=4-14)]

Observe os aprimoramentos da interface do usuário com esses comentários adicionais:

![Interface do usuário do Swagger com comentários adicionais mostrados](web-api-help-pages-using-swagger/_static/xml-comments-extended.png)

### <a name="data-annotations"></a>Anotações de dados

Decore o modelo com atributos, encontrados no namespace [System.ComponentModel.DataAnnotations](/dotnet/api/system.componentmodel.dataannotations), para ajudar a gerar os componentes da interface do usuário do Swagger.

Adicione o atributo `[Required]` à propriedade `Name` da classe `TodoItem`:

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/TodoApi.Swashbuckle/Models/TodoItem.cs?highlight=10)]

A presença desse atributo altera o comportamento da interface do usuário e altera o esquema JSON subjacente:

```json
"definitions": {
    "TodoItem": {
        "required": [
            "name"
        ],
        "type": "object",
        "properties": {
            "id": {
                "format": "int64",
                "type": "integer"
            },
            "name": {
                "type": "string"
            },
            "isComplete": {
                "default": false,
                "type": "boolean"
            }
        }
    }
},
```

Adicione o atributo `[Produces("application/json")]` ao controlador da API. Sua finalidade é declarar que as ações do controlador permitem o tipo de conteúdo de resposta *application/json*:

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_TodoController&highlight=3)]

A lista suspensa **Tipo de Conteúdo de Resposta** seleciona esse tipo de conteúdo como o padrão para ações GET do controlador:

![Interface do usuário do Swagger com o tipo de conteúdo de resposta padrão](web-api-help-pages-using-swagger/_static/json-response-content-type.png)

À medida que aumenta o uso de anotações de dados na API Web, a interface do usuário e as páginas de ajuda da API se tornam mais descritivas e úteis.

### <a name="describe-response-types"></a>Descrever os tipos de resposta

Os desenvolvedores de consumo estão mais preocupados com o que é retornado, especificamente, o tipos de resposta e o códigos de erro (se eles não forem padrão). Os tipos de resposta e os códigos de erro são indicados nos comentários XML e nas anotações de dados.

A ação `Create` retorna um código de status HTTP 201 em caso de sucesso. Um código de status HTTP 400 é retornado quando o corpo da solicitação postada é nulo. Sem a documentação adequada na interface do usuário do Swagger, o consumidor não tem conhecimento desses resultados esperados. Corrija esse problema adicionando as linhas realçadas no exemplo a seguir:

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_Create&highlight=17,18,20,21)]

A interface do usuário do Swagger agora documenta claramente os códigos de resposta HTTP esperados:

![A interface do usuário do Swagger mostra a descrição da classe de resposta POST, 'Retorna o item de tarefa pendente recém-criado' e '400 – se o item for nulo' para o código de status e o motivo em Mensagens de Resposta](web-api-help-pages-using-swagger/_static/data-annotations-response-types.png)

### <a name="customize-the-ui"></a>Personalizar a interface do usuário

A interface do usuário de estoque é apresentável e funcional. No entanto, as páginas de documentação da API devem representar a sua marca ou o seu tema. Para inserir sua marca nos componentes do Swashbuckle é necessário adicionar recursos para atender aos arquivos estáticos e criar a estrutura de pasta para hospedar esses arquivos.

Se você estiver direcionando ao .NET Framework ou ao .NET Core 1.x, adicione o pacote do NuGet [Microsoft.AspNetCore.StaticFiles](https://www.nuget.org/packages/Microsoft.AspNetCore.StaticFiles) ao projeto:

```xml
<PackageReference Include="Microsoft.AspNetCore.StaticFiles" Version="2.0.0" />
```

O pacote do NuGet anterior já estará instalado se você estiver direcionando ao .NET Core 2.x e usando o [metapackage](xref:fundamentals/metapackage).

Habilite o middleware de arquivos estáticos:

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/TodoApi.Swashbuckle/Startup.cs?name=snippet_Configure&highlight=3)]

Adquira o conteúdo da pasta *dist* do [repositório GitHub da interface do usuário do Swagger](https://github.com/swagger-api/swagger-ui/tree/master/dist). Essa pasta contém os ativos necessários para a página da interface do usuário do Swagger.

Crie uma pasta *swagger/wwwroot/ui* e copie para ela o conteúdo da pasta *dist*.

Crie um arquivo *custom.css* em *swagger/wwwroot/ui*, com o seguinte CSS para personalizar o cabeçalho da página:

[!code-css[](../tutorials/web-api-help-pages-using-swagger/samples/TodoApi.Swashbuckle/wwwroot/swagger/ui/custom.css)]

Referencie *custom.css* no arquivo *index.html* depois de todos os outros arquivos CSS:

[!code-html[](../tutorials/web-api-help-pages-using-swagger/samples/TodoApi.Swashbuckle/wwwroot/swagger/ui/index.html?name=snippet_SwaggerUiCss&highlight=3)]

Navegue até a página *index.html* em `http://localhost:<random_port>/swagger/ui/index.html`. Digite `http://localhost:<random_port>/swagger/v1/swagger.json` na caixa de texto do cabeçalho e clique no botão **Explorar**. A página resultante será semelhante ao seguinte:

![Interface do usuário do Swagger com o título do cabeçalho personalizado](web-api-help-pages-using-swagger/_static/custom-header.png)

Há muito mais que você pode fazer com a página. Consulte as funcionalidades completas para os recursos de interface do usuário no [repositório GitHub da interface do usuário do Swagger](https://github.com/swagger-api/swagger-ui).
