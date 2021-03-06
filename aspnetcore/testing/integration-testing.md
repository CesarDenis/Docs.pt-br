---
title: Testes de integração no núcleo do ASP.NET
author: ardalis
description: Como usar a integração do ASP.NET Core testes para garantir que os componentes de um aplicativo funcionem corretamente.
manager: wpickett
ms.author: riande
ms.date: 09/25/2017
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: testing/integration-testing
ms.openlocfilehash: ac3a9e00edfd4c736ee1e7d5c0c724c3e52d0b6b
ms.sourcegitcommit: 74be78285ea88772e7dad112f80146b6ed00e53e
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 05/10/2018
---
# <a name="integration-tests-in-aspnet-core"></a>Testes de integração no núcleo do ASP.NET

Por [Steve Smith](https://ardalis.com/)

Testes de integração garantem que componentes de um aplicativo funcionam corretamente quando montado juntos. O ASP.NET Core é compatível com testes de integração por meio de estruturas de teste de unidade e um host Web de testes interno que pode ser utilizado para lidar com solicitações sem sobrecarga de rede.

[Exibir ou baixar código de exemplo](https://github.com/aspnet/Docs/tree/master/aspnetcore/testing/integration-testing/sample) ([como baixar](xref:tutorials/index#how-to-download-a-sample))

## <a name="introduction-to-integration-testing"></a>Introdução ao teste de integração

Testes de integração verificar que diferentes partes de um aplicativo funcionem corretamente. Ao contrário de [testes de unidade](https://docs.microsoft.com/dotnet/articles/core/testing/unit-testing-with-dotnet-test), testes de integração com frequência envolvem preocupações de infraestrutura de aplicativo, como um banco de dados, sistema de arquivos, recursos de rede, ou web solicitações e respostas. Testes de unidade usam falsificações ou objetos fictícios no lugar essas preocupações, mas a finalidade de testes de integração é confirmar se o sistema está funcionando conforme o esperado com esses sistemas.

Testes de integração, porque praticados maior de segmentos de código e como eles se basear em elementos de infraestrutura, tendem a ser ordens de magnitude menor do que testes de unidade. Portanto, é uma boa ideia limitar quantos testes de integração que você escreve, especialmente se você pode testar o mesmo comportamento com um teste de unidade.

> [!NOTE]
> Se algum comportamento pode ser testado usando um teste de unidade ou um teste de integração, prefira o teste de unidade, já que ele é quase sempre ser mais rápido. Você pode ter dúzias ou centenas de testes de unidade com muitas entradas diferentes, mas apenas uma série de testes de integração que cobrem os cenários mais importantes.

A lógica dentro de seus próprios métodos de teste é geralmente o domínio de testes de unidade. Testar o funcionamento do seu aplicativo em sua estrutura, por exemplo, com ASP.NET Core, ou com um banco de dados é onde os testes de integração entram em ação. Ele não tem muitos testes de integração para confirmar que você é capaz de gravar uma linha para o banco de dados e lê-lo novamente. Não é necessário testar cada permutação possíveis do seu código de acesso de dados - você precisa testar suficiente para que você a certeza de que seu aplicativo está funcionando corretamente.

## <a name="integration-testing-aspnet-core"></a>Integração de teste ASP.NET Core

Para obter configurou a testes de integração de execução, você precisará criar um projeto de teste, adicione uma referência ao projeto web ASP.NET Core e instalar um executor de teste. Esse processo é descrito no [testes de unidade](https://docs.microsoft.com/dotnet/articles/core/testing/unit-testing-with-dotnet-test) documentação, junto com instruções mais detalhadas sobre como executar testes e recomendações para nomear suas classes de teste e testes.

> [!NOTE]
> Separe os testes de unidade e os testes de integração usando projetos diferentes. Isso ajuda a garantir que você não acidentalmente introduzir problemas de infraestrutura em seus testes de unidade e permite que você escolher facilmente qual conjunto de testes para executar.

### <a name="the-test-host"></a>O Host de teste

ASP.NET Core inclui um host de teste que pode ser adicionado a projetos de teste de integração e usado para hospedar o ASP.NET Core aplicativos, solicitações de serviço de teste sem a necessidade de um host da web real. O exemplo fornecido inclui um projeto de teste de integração que foi configurado para usar [xUnit](https://xunit.github.io) e o Host de teste. Ele usa o [Microsoft.AspNetCore.TestHost](https://www.nuget.org/packages/Microsoft.AspNetCore.TestHost/) pacote NuGet.

Uma vez o `Microsoft.AspNetCore.TestHost` pacote está incluído no projeto, você poderá criar e configurar um `TestServer` em seus testes. O teste a seguir mostra como verificar uma solicitação feita para a raiz de um site retorna "Hello World!" e deve ser executado com êxito em relação ao padrão modelo de Web do ASP.NET Core vazio criado pelo Visual Studio.

[!code-csharp[](../testing/integration-testing/sample/test/PrimeWeb.IntegrationTests/PrimeWebDefaultRequestShould.cs?name=snippet_WebDefault&highlight=7,16,22)]

Este teste é usando o padrão de organizar Act declaração. A etapa de organizar é feita no construtor, que cria uma instância de `TestServer`. Configurado `WebHostBuilder` será usado para criar um `TestHost`; neste exemplo, o `Configure` método do sistema em teste (SUT) `Startup` classe é passada para o `WebHostBuilder`. Esse método será usado para configurar o pipeline de solicitação do `TestServer` identicamente a como o servidor SUT deve ser configurado.

Na parte do Act do teste, é feita uma solicitação para a `TestServer` instância para o caminho "/" e a resposta é lido novamente em uma cadeia de caracteres. Essa cadeia de caracteres é comparada com a cadeia de caracteres esperada de "Hello World!". Se elas corresponderem, o teste seja aprovado; Caso contrário, ele falhará.

Agora você pode adicionar alguns testes de integração adicionais para confirmar que o primo verificando funcionalidade funciona por meio do aplicativo web:

[!code-csharp[](../testing/integration-testing/sample/test/PrimeWeb.IntegrationTests/PrimeWebCheckPrimeShould.cs?name=snippet_CheckPrime)]

Observe que não é realmente está tentando testar a exatidão do verificador de número primo com esses testes, mas em vez disso, que o aplicativo web está fazendo o que você espera. Você já tem cobertura de teste de unidade que proporciona confiança em `PrimeService`, como você pode ver aqui:

![Gerenciador de Testes](integration-testing/_static/test-explorer.png)

Você pode aprender mais sobre os testes de unidade no [testes de unidade](https://docs.microsoft.com/dotnet/articles/core/testing/unit-testing-with-dotnet-test) artigo.


### <a name="integration-testing-mvcrazor"></a>Integração de teste/Razor do Mvc

Projetos de teste que contêm modos de exibição Razor exigem `<PreserveCompilationContext>` ser definido como true no *. csproj* arquivo:


```xml
    <PreserveCompilationContext>true</PreserveCompilationContext>
```

Este elemento de projetos irá gerar um erro semelhante à seguinte:
```
Microsoft.AspNetCore.Mvc.Razor.Compilation.CompilationFailedException: 'One or more compilation failures occurred:
ooebhccx.1bd(4,62): error CS0012: The type 'Attribute' is defined in an assembly that is not referenced. You must add a reference to assembly 'netstandard, Version=2.0.0.0, Culture=neutral, PublicKeyToken=cc7b13ffcd2ddd51'.
```


## <a name="refactoring-to-use-middleware"></a>Para usar o middleware de refatoração

Refatoração é o processo de alteração de código do aplicativo para melhorar seu design sem alterar seu comportamento. Idealmente deve ser feito quando há um conjunto de passar os testes, pois esses ajuda garantir o que comportamento do sistema permanece a mesma antes e após as alterações. Examinar a maneira na qual o primo lógica de verificação é implementado no aplicativo da web `Configure` método, consulte:

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    app.Run(async (context) =>
    {
        if (context.Request.Path.Value.Contains("checkprime"))
        {
            int numberToCheck;
            try
            {
                numberToCheck = int.Parse(context.Request.QueryString.Value.Replace("?", ""));
                var primeService = new PrimeService();
                if (primeService.IsPrime(numberToCheck))
                {
                    await context.Response.WriteAsync($"{numberToCheck} is prime!");
                }
                else
                {
                    await context.Response.WriteAsync($"{numberToCheck} is NOT prime!");
                }
            }
            catch
            {
                await context.Response.WriteAsync("Pass in a number to check in the form /checkprime?5");
            }
        }
        else
        {
            await context.Response.WriteAsync("Hello World!");
        }
    });
}
```

Esse código funciona, mas está longe de como você deseja implementar esse tipo de funcionalidade em um aplicativo ASP.NET Core, até mesmo simple como este é. Imagine que o `Configure` método seria semelhante, se necessário para adicionar essa quantidade código para ele sempre que você adicionar outro ponto de extremidade de URL!

Adição de uma opção a ser considerada [MVC](xref:mvc/overview) para o aplicativo e criando um controlador para lidar com a verificação principal. No entanto, supondo que você atualmente não precisa de qualquer outra MVC funcionalidade, que é um bit exageros.

No entanto, você pode tirar proveito do ASP.NET Core [middleware](xref:fundamentals/middleware/index), que nos ajudarão a encapsular o primo verificação lógica em sua própria classe e obter melhor [separação de preocupações](http://deviq.com/separation-of-concerns/) no `Configure` método.

Para permitir que o caminho do middleware usa a ser especificado como um parâmetro para a classe de middleware espera um `RequestDelegate` e um `PrimeCheckerOptions` instância em seu construtor. Se o caminho da solicitação não corresponde ao que este middleware configurado para esperar, basta chamar o próximo middleware da cadeia e não fazer nada. O restante do código de implementação que estava no `Configure` está agora no `Invoke` método.

> [!NOTE]
> Desde que o middleware depende do `PrimeService` serviço, você também está solicitando uma instância do serviço com o construtor. A estrutura oferecer esse serviço por meio de [injeção de dependência](xref:fundamentals/dependency-injection), supondo que ele tiver sido configurado, por exemplo, em `ConfigureServices`.

[!code-csharp[](../testing/integration-testing/sample/src/PrimeWeb/Middleware/PrimeCheckerMiddleware.cs?highlight=39-63)]

Como este middleware atua como um ponto de extremidade da cadeia de delegado solicitação quando o caminho corresponde, não há nenhuma chamada para `_next.Invoke` quando este middleware manipula a solicitação.

Com este middleware em vigor e alguns útil métodos de extensão criados para facilitar a configuração, o refatorado `Configure` método tem esta aparência:

[!code-csharp[](../testing/integration-testing/sample/src/PrimeWeb/Startup.cs?highlight=9&range=19-33)]

Seguindo essa refatoração estiver certo de que o aplicativo web ainda funciona como antes, desde que os testes de integração estão passando.

> [!NOTE]
> É uma boa ideia para confirmar as alterações ao controle de origem depois de concluir uma refatoração e transmitir seus testes. Se você está praticando desenvolvimento orientado por testes, [considere a adição de confirmação para o ciclo de vermelho-verde-refatorar](https://ardalis.com/rgrc-is-the-new-red-green-refactor-for-test-first-development).

## <a name="resources"></a>Recursos

* [Teste de unidade](https://docs.microsoft.com/dotnet/articles/core/testing/unit-testing-with-dotnet-test)
* [Middleware](xref:fundamentals/middleware/index)
* [Controladores de teste](xref:mvc/controllers/testing)
