---
title: Hospedagem no ASP.NET Core
author: guardrex
description: Saiba mais sobre o host da Web no ASP.NET Core, que é responsável pelo gerenciamento de tempo de vida e pela inicialização do aplicativo.
manager: wpickett
ms.author: riande
ms.date: 09/21/2017
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: fundamentals/hosting
ms.openlocfilehash: 344bf5f0917f4c33d67eeb14176ff2aae3ae75da
ms.sourcegitcommit: 5130b3034165f5cf49d829fe7475a84aa33d2693
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 05/03/2018
---
# <a name="hosting-in-aspnet-core"></a>Hospedagem no ASP.NET Core

Por [Luke Latham](https://github.com/guardrex)

Aplicativos ASP.NET Core configuram e inicializam um *host*. O host é responsável pelo gerenciamento de tempo de vida e pela inicialização do aplicativo. No mínimo, o host configura um servidor e um pipeline de processamento de solicitações.

## <a name="setting-up-a-host"></a>Configurando um host

#### <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x/)
Crie um host usando uma instância de [WebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder). Normalmente, isso é feito no ponto de entrada do aplicativo, o método `Main`. Em modelos de projeto, `Main` está localizado em *Program.cs*. Um *Program.cs* típico chama [CreateDefaultBuilder](/dotnet/api/microsoft.aspnetcore.webhost.createdefaultbuilder) para começar a configurar um host:

[!code-csharp[](../common/samples/WebApplication1DotNetCore2.0App/Program.cs?name=snippet_Main)]

`CreateDefaultBuilder` executa as seguintes tarefas:

* Configura o [Kestrel](servers/kestrel.md) como o servidor Web. Para conhecer as opções padrão do Kestrel, consulte [Implementação do servidor Web Kestrel no ASP.NET Core](xref:fundamentals/servers/kestrel#kestrel-options).
* Define a raiz do conteúdo como o caminho retornado por [Directory.GetCurrentDirectory](/dotnet/api/system.io.directory.getcurrentdirectory).
* Carrega configurações opcionais de:
  * *appsettings.json*.
  * *appsettings.{Environment}.json*.
  * [Segredos do usuário](xref:security/app-secrets) quando o aplicativo é executado no ambiente `Development`.
  * Variáveis de ambiente.
  * Argumentos de linha de comando.
* Configura o [registro em log](xref:fundamentals/logging/index) para a saída do console e de depuração. O registro em log inclui regras de [filtragem de log](xref:fundamentals/logging/index#log-filtering) especificadas em uma seção de configuração de registro em log de um arquivo *appsettings.json* ou *appsettings.{Environment}.json*.
* Quando executado por trás do IIS, permite a [integração de IIS](xref:host-and-deploy/iis/index). Configura o caminho base e a porta que o servidor escuta ao usar o [Módulo do ASP.NET Core](xref:fundamentals/servers/aspnet-core-module). O módulo cria um proxy reverso entre o IIS e o Kestrel. Também configura o aplicativo para [capturar erros de inicialização](#capture-startup-errors). Para conhecer as opções padrão do IIS, consulte [a seção sobre as opções do IIS para Hospedar o ASP.NET Core no Windows com o IIS](xref:host-and-deploy/iis/index#iis-options).
* Definirá [ServiceProviderOptions.ValidateScopes](/dotnet/api/microsoft.extensions.dependencyinjection.serviceprovideroptions.validatescopes) como `true` se o ambiente do aplicativo for de desenvolvimento. Para obter mais informações, confira [Validação de escopo](#scope-validation).

A *raiz do conteúdo* determina onde o host procura por arquivos de conteúdo, como arquivos de exibição do MVC. Quando o aplicativo é iniciado na pasta raiz do projeto, essa pasta é usada como a raiz do conteúdo. Esse é o padrão usado no [Visual Studio](https://www.visualstudio.com/) e nos [novos modelos dotnet](/dotnet/core/tools/dotnet-new).

Para obter mais informações sobre a configuração de aplicativos, consulte [Configuração no ASP.NET Core](xref:fundamentals/configuration/index).

> [!NOTE]
> Como uma alternativa ao uso do método `CreateDefaultBuilder` estático, criar um host de [WebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder) é uma abordagem compatível com o ASP.NET Core 2. x. Para obter mais informações, consulte a guia do ASP.NET Core 1.x.

#### <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x/)
Crie um host usando uma instância de [WebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder). A criação de um host normalmente é feita no ponto de entrada do aplicativo, o método `Main`. Em modelos de projeto, `Main` está localizado em *Program.cs*:

[!code-csharp[](../common/samples/WebApplication1/Program.cs)]

`WebHostBuilder` requer um [servidor que implementa IServer](servers/index.md). Os servidores internos são [Kestrel](servers/kestrel.md) e [HTTP.sys](servers/httpsys.md) (antes do lançamento do ASP.NET Core 2.0, HTTP.sys era chamado de [WebListener](xref:fundamentals/servers/weblistener)). Neste exemplo, o [método de extensão UseKestrel](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderkestrelextensions.usekestrel?view=aspnetcore-1.1) especifica o servidor Kestrel.

A *raiz do conteúdo* determina onde o host procura por arquivos de conteúdo, como arquivos de exibição do MVC. A raiz de conteúdo padrão é obtida para `UseContentRoot` por [Directory.GetCurrentDirectory](/dotnet/api/system.io.directory.getcurrentdirectory?view=netcore-1.1). Quando o aplicativo é iniciado na pasta raiz do projeto, essa pasta é usada como a raiz do conteúdo. Esse é o padrão usado no [Visual Studio](https://www.visualstudio.com/) e nos [novos modelos dotnet](/dotnet/core/tools/dotnet-new).

Para usar o IIS como um proxy reverso, chame [UseIISIntegration](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderiisextensions) como parte da compilação do host. `UseIISIntegration` não configura um *servidor* como [UseKestrel](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderkestrelextensions.usekestrel?view=aspnetcore-1.1) o faz. `UseIISIntegration` configura o caminho base e a porta que o servidor escuta ao usar o [Módulo do ASP.NET Core](xref:fundamentals/servers/aspnet-core-module) para criar um proxy reverso entre o Kestrel e o IIS. Para usar o IIS com o ASP.NET Core, tanto `UseKestrel` quanto `UseIISIntegration` precisam ser especificados. `UseIISIntegration` é ativado somente quando é executado por trás do IIS ou do IIS Express. Para obter mais informações, consulte [Referência de configuração do módulo do ASP.NET Core](xref:fundamentals/servers/aspnet-core-module) e [Introdução ao Módulo do ASP.NET Core](xref:host-and-deploy/aspnet-core-module).

Uma implementação mínima que configura um host (e um aplicativo ASP.NET Core) inclui a especificação de um servidor e a configuração do pipeline de solicitações do aplicativo:

```csharp
var host = new WebHostBuilder()
    .UseKestrel()
    .Configure(app =>
    {
        app.Run(context => context.Response.WriteAsync("Hello World!"));
    })
    .Build();

host.Run();
```

* * *
Ao configurar um host, os métodos [Configure](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configure?view=aspnetcore-1.1) e [ConfigureServices](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder.configureservices?view=aspnetcore-1.1) podem ser fornecidos. Se uma classe `Startup` for especificada, ela deverá definir um método `Configure`. Para obter mais informações, consulte [Inicialização do aplicativo no ASP.NET Core](startup.md). Diversas chamadas para `ConfigureServices` são acrescentadas umas às outras. Diversas chamadas para `Configure` ou `UseStartup` no `WebHostBuilder` substituem configurações anteriores.

## <a name="host-configuration-values"></a>Valores de configuração do host

[WebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder) conta com as seguintes abordagens para definir os valores de configuração do host:

* Configuração do construtor do host, que inclui variáveis de ambiente com o formato `ASPNETCORE_{configurationKey}`. Por exemplo, `ASPNETCORE_URLS`.
* Métodos explícitos, como `CaptureStartupErrors`.
* [UseSetting](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder.usesetting) e a chave associada. Ao definir um valor com `UseSetting`, o valor é definido como uma cadeia de caracteres, independentemente do tipo.

O host usa a opção que define um valor por último. Para obter mais informações, consulte [Substituindo a configuração](#overriding-configuration) na próxima seção.

### <a name="capture-startup-errors"></a>Capturar erros de inicialização

Esta configuração controla a captura de erros de inicialização.

**Chave**: captureStartupErrors  
**Tipo**: *bool* (`true` ou `1`)  
**Padrão**: o padrão é `false`, a menos que o aplicativo seja executado com o Kestrel por trás do IIS, em que o padrão é `true`.  
**Definido usando**: `CaptureStartupErrors`  
**Variável de ambiente**: `ASPNETCORE_CAPTURESTARTUPERRORS`

Quando `false`, erros durante a inicialização resultam no encerramento do host. Quando `true`, o host captura exceções durante a inicialização e tenta iniciar o servidor.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

```csharp
WebHost.CreateDefaultBuilder(args)
    .CaptureStartupErrors(true)
    ...
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

```csharp
var host = new WebHostBuilder()
    .CaptureStartupErrors(true)
    ...
```

---

### <a name="content-root"></a>Raiz do conteúdo

Essa configuração determina onde o ASP.NET Core começa a procurar por arquivos de conteúdo, como exibições do MVC. 

**Chave**: contentRoot  
**Tipo**: *string*  
**Padrão**: o padrão é a pasta em que o assembly do aplicativo reside.  
**Definido usando**: `UseContentRoot`  
**Variável de ambiente**: `ASPNETCORE_CONTENTROOT`

A raiz do conteúdo também é usada como o caminho base para a [Configuração da raiz da Web](#web-root). Se o caminho não existir, o host não será iniciado.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseContentRoot("c:\\mywebsite")
    ...
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

```csharp
var host = new WebHostBuilder()
    .UseContentRoot("c:\\mywebsite")
    ...
```

---

### <a name="detailed-errors"></a>Erros detalhados

Determina se erros detalhados devem ser capturados.

**Chave**: detailedErrors  
**Tipo**: *bool* (`true` ou `1`)  
**Padrão**: falso  
**Definido usando**: `UseSetting`  
**Variável de ambiente**: `ASPNETCORE_DETAILEDERRORS`

Quando habilitado (ou quando o <a href="#environment">Ambiente</a> é definido como `Development`), o aplicativo captura exceções detalhadas.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting(WebHostDefaults.DetailedErrorsKey, "true")
    ...
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

```csharp
var host = new WebHostBuilder()
    .UseSetting(WebHostDefaults.DetailedErrorsKey, "true")
    ...
```

---

### <a name="environment"></a>Ambiente

Define o ambiente do aplicativo.

**Chave**: ambiente  
**Tipo**: *string*  
**Padrão**: Production  
**Definido usando**: `UseEnvironment`  
**Variável de ambiente**: `ASPNETCORE_ENVIRONMENT`

O ambiente pode ser definido como qualquer valor. Os valores definidos pela estrutura incluem `Development`, `Staging` e `Production`. Os valores não diferenciam maiúsculas de minúsculas. Por padrão, o *Ambiente* é lido da variável de ambiente `ASPNETCORE_ENVIRONMENT`. Ao usar o [Visual Studio](https://www.visualstudio.com/), variáveis de ambiente podem ser definidas no arquivo *launchSettings.json*. Para obter mais informações, veja [Trabalhar com vários ambientes](xref:fundamentals/environments).

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseEnvironment("Development")
    ...
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

```csharp
var host = new WebHostBuilder()
    .UseEnvironment("Development")
    ...
```

---

### <a name="hosting-startup-assemblies"></a>Hospedando assemblies de inicialização

Define os assemblies de inicialização de hospedagem do aplicativo.

**Chave**: hostingStartupAssemblies  
**Tipo**: *string*  
**Padrão**: cadeia de caracteres vazia  
**Definido usando**: `UseSetting`  
**Variável de ambiente**: `ASPNETCORE_HOSTINGSTARTUPASSEMBLIES`

Uma cadeia de caracteres delimitada por ponto e vírgula de assemblies de inicialização de hospedagem para carregamento na inicialização. Este recurso é novo no ASP.NET Core 2.0.

Embora o valor padrão da configuração seja uma cadeia de caracteres vazia, os assemblies de inicialização de hospedagem sempre incluem o assembly do aplicativo. Quando assemblies de inicialização de hospedagem são fornecidos, eles são adicionados ao assembly do aplicativo para carregamento quando o aplicativo compilar seus serviços comuns durante a inicialização.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting(WebHostDefaults.HostingStartupAssembliesKey, "assembly1;assembly2")
    ...
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

Este recurso não está disponível no ASP.NET Core 1. x.

---

### <a name="prefer-hosting-urls"></a>Preferir URLs de hospedagem

Indica se o host deve escutar as URLs configuradas com o `WebHostBuilder` em vez daquelas configuradas com a implementação `IServer`.

**Chave**: preferHostingUrls  
**Tipo**: *bool* (`true` ou `1`)  
**Padrão**: true  
**Definido usando**: `PreferHostingUrls`  
**Variável de ambiente**: `ASPNETCORE_PREFERHOSTINGURLS`

Este recurso é novo no ASP.NET Core 2.0.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

```csharp
WebHost.CreateDefaultBuilder(args)
    .PreferHostingUrls(false)
    ...
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

Este recurso não está disponível no ASP.NET Core 1. x.

---

### <a name="prevent-hosting-startup"></a>Impedir inicialização de hospedagem

Impede o carregamento automático de assemblies de inicialização de hospedagem, incluindo assemblies de inicialização de hospedagem configurados pelo assembly do aplicativo. Confira [Adicionar recursos do aplicativo usando uma configuração específica da plataforma](xref:host-and-deploy/platform-specific-configuration) para obter mais informações.

**Chave**: preventHostingStartup  
**Tipo**: *bool* (`true` ou `1`)  
**Padrão**: falso  
**Definido usando**: `UseSetting`  
**Variável de ambiente**: `ASPNETCORE_PREVENTHOSTINGSTARTUP`

Este recurso é novo no ASP.NET Core 2.0.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting(WebHostDefaults.PreventHostingStartupKey, "true")
    ...
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

Este recurso não está disponível no ASP.NET Core 1. x.

---

### <a name="server-urls"></a>URLs de servidor

Indica os endereços IP ou endereços de host com portas e protocolos que o servidor deve escutar para solicitações.

**Chave**: urls  
**Tipo**: *string*  
**Padrão**: http://localhost:5000  
**Definido usando**: `UseUrls`  
**Variável de ambiente**: `ASPNETCORE_URLS`

Defina como uma lista separada por ponto e vírgula (;) de prefixos de URL aos quais o servidor deve responder. Por exemplo, `http://localhost:123`. Use "\*" para indicar que o servidor deve escutar solicitações em qualquer endereço IP ou nome do host usando a porta e o protocolo especificados (por exemplo, `http://*:5000`). O protocolo (`http://` ou `https://`) deve ser incluído com cada URL. Os formatos compatíveis variam dependendo dos servidores.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseUrls("http://*:5000;http://localhost:5001;https://hostname:5002")
    ...
```

O Kestrel tem sua própria API de configuração de ponto de extremidade. Para obter mais informações, consulte [Implementação do servidor Web Kestrel no ASP.NET Core](xref:fundamentals/servers/kestrel?tabs=aspnetcore2x#endpoint-configuration).

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

```csharp
var host = new WebHostBuilder()
    .UseUrls("http://*:5000;http://localhost:5001;https://hostname:5002")
    ...
```

---

### <a name="shutdown-timeout"></a>Tempo limite de desligamento

Especifica o tempo de espera para o desligamento do host da Web.

**Chave**: shutdownTimeoutSeconds  
**Tipo**: *int*  
**Padrão**: 5  
**Definido usando**: `UseShutdownTimeout`  
**Variável de ambiente**: `ASPNETCORE_SHUTDOWNTIMEOUTSECONDS`

Embora a chave aceite um *int* com `UseSetting` (por exemplo, `.UseSetting(WebHostDefaults.ShutdownTimeoutKey, "10")`), o método de extensão [UseShutdownTimeout](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.useshutdowntimeout) usa um [TimeSpan](/dotnet/api/system.timespan). Este recurso é novo no ASP.NET Core 2.0.

Durante o período de tempo limite, a hospedagem:

* Dispara [IApplicationLifetime.ApplicationStopping](/dotnet/api/microsoft.aspnetcore.hosting.iapplicationlifetime.applicationstopping).
* Tenta parar os serviços hospedados, registrando em log os erros dos serviços que falham ao parar.

Se o período de tempo limite expirar antes que todos os serviços hospedados parem, os serviços ativos restantes serão parados quando o aplicativo for desligado. Os serviços serão parados mesmo se ainda não tiverem concluído o processamento. Se os serviços exigirem mais tempo para parar, aumente o tempo limite.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseShutdownTimeout(TimeSpan.FromSeconds(10))
    ...
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

Este recurso não está disponível no ASP.NET Core 1. x.

---

### <a name="startup-assembly"></a>Assembly de inicialização

Determina o assembly para pesquisar pela classe `Startup`.

**Chave**: startupAssembly  
**Tipo**: *string*  
**Padrão**: o assembly do aplicativo  
**Definido usando**: `UseStartup`  
**Variável de ambiente**: `ASPNETCORE_STARTUPASSEMBLY`

O assembly por nome (`string`) ou por tipo (`TStartup`) pode ser referenciado. Se vários métodos `UseStartup` forem chamados, o último terá precedência.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseStartup("StartupAssemblyName")
    ...
```

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseStartup<TStartup>()
    ...
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

```csharp
var host = new WebHostBuilder()
    .UseStartup("StartupAssemblyName")
    ...
```

```csharp
var host = new WebHostBuilder()
    .UseStartup<TStartup>()
    ...
```

---

### <a name="web-root"></a>Raiz da Web

Define o caminho relativo para os ativos estáticos do aplicativo.

**Chave**: webroot  
**Tipo**: *string*  
**Padrão**: se não for especificado, o padrão será "(Raiz do conteúdo)/wwwroot", se o caminho existir. Se o caminho não existir, um provedor de arquivo não operacional será usado.  
**Definido usando**: `UseWebRoot`  
**Variável de ambiente**: `ASPNETCORE_WEBROOT`

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseWebRoot("public")
    ...
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

```csharp
var host = new WebHostBuilder()
    .UseWebRoot("public")
    ...
```

---

## <a name="overriding-configuration"></a>Substituindo a configuração

Use [Configuração](xref:fundamentals/configuration/index) para configurar o host. No exemplo a seguir, a configuração do host é especificada, opcionalmente, em um arquivo *hosting.json*. Qualquer configuração carregada do arquivo *hosting.json* pode ser substituída por argumentos de linha de comando. A configuração interna (no `config`) é usada para configurar o host com `UseConfiguration`.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

*hosting.json*:

```json
{
    urls: "http://*:5005"
}
```

Substituição da configuração fornecida por `UseUrls` pela configuração de *hosting.json* primeiro e pela configuração de argumento da linha de comando depois:

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        BuildWebHost(args).Run();
    }

    public static IWebHost BuildWebHost(string[] args)
    {
        var config = new ConfigurationBuilder()
            .SetBasePath(Directory.GetCurrentDirectory())
            .AddJsonFile("hosting.json", optional: true)
            .AddCommandLine(args)
            .Build();

        return WebHost.CreateDefaultBuilder(args)
            .UseUrls("http://*:5000")
            .UseConfiguration(config)
            .Configure(app =>
            {
                app.Run(context => 
                    context.Response.WriteAsync("Hello, World!"));
            })
            .Build();
    }
}
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

*hosting.json*:

```json
{
    urls: "http://*:5005"
}
```

Substituição da configuração fornecida por `UseUrls` pela configuração de *hosting.json* primeiro e pela configuração de argumento da linha de comando depois:

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        var config = new ConfigurationBuilder()
            .SetBasePath(Directory.GetCurrentDirectory())
            .AddJsonFile("hosting.json", optional: true)
            .AddCommandLine(args)
            .Build();

        var host = new WebHostBuilder()
            .UseUrls("http://*:5000")
            .UseConfiguration(config)
            .UseKestrel()
            .Configure(app =>
            {
                app.Run(context => 
                    context.Response.WriteAsync("Hello, World!"));
            })
            .Build();

        host.Run();
    }
}
```

---

> [!NOTE]
> Atualmente, o método de extensão [UseConfiguration](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.useconfiguration) não é capaz de analisar uma seção de configuração retornada por `GetSection` (por exemplo, `.UseConfiguration(Configuration.GetSection("section"))`). O método `GetSection` filtra as chaves de configuração da seção solicitada, mas deixa o nome da seção nas chaves (por exemplo, `section:urls`, `section:environment`). O método `UseConfiguration` espera que as chaves correspondam às chaves `WebHostBuilder` (por exemplo, `urls`, `environment`). A presença do nome da seção nas chaves impede que os valores da seção configurem o host. Esse problema será corrigido em uma próxima versão. Para obter mais informações e soluções alternativas, consulte [Passar a seção de configuração para WebHostBuilder.UseConfiguration usa chaves completas](https://github.com/aspnet/Hosting/issues/839).

Para especificar o host executado em uma URL específica, o valor desejado pode ser passado em um prompt de comando ao executar [dotnet run](/dotnet/core/tools/dotnet-run). O argumento de linha de comando substitui o valor `urls` do arquivo *hosting.json* e o servidor escuta na porta 8080:

```console
dotnet run --urls "http://*:8080"
```

## <a name="starting-the-host"></a>Iniciando o host

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

**Executar**

O método `Run` inicia o aplicativo Web e bloqueia o thread de chamada até que o host seja desligado:

```csharp
host.Run();
```

**Iniciar**

Execute o host sem bloqueio, chamando seu método `Start`:

```csharp
using (host)
{
    host.Start();
    Console.ReadLine();
}
```

Se uma lista de URLs for passada para o método `Start`, ele escutará nas URLs especificadas:

```csharp
var urls = new List<string>()
{
    "http://*:5000",
    "http://localhost:5001"
};

var host = new WebHostBuilder()
    .UseKestrel()
    .UseStartup<Startup>()
    .Start(urls.ToArray());

using (host)
{
    Console.ReadLine();
}
```

O aplicativo pode inicializar e iniciar um novo host usando os padrões pré-configurados de `CreateDefaultBuilder`, usando um método estático conveniente. Esses métodos iniciam o servidor sem uma saída do console e com [WaitForShutdown](/dotnet/api/microsoft.aspnetcore.hosting.webhostextensions.waitforshutdown) e aguardam uma quebra (Ctrl-C/SIGINT ou SIGTERM):

**Start(RequestDelegate app)**

Inicie com um `RequestDelegate`:

```csharp
using (var host = WebHost.Start(app => app.Response.WriteAsync("Hello, World!")))
{
    Console.WriteLine("Use Ctrl-C to shutdown the host...");
    host.WaitForShutdown();
}
```

Faça uma solicitação no navegador para `http://localhost:5000` para receber a resposta "Olá, Mundo!" `WaitForShutdown` bloqueia até que uma quebra (Ctrl-C/SIGINT ou SIGTERM) seja emitida. O aplicativo exibe a mensagem `Console.WriteLine` e aguarda um pressionamento de tecla para ser encerrado.

**Start(string url, RequestDelegate app)**

Inicie com uma URL e `RequestDelegate`:

```csharp
using (var host = WebHost.Start("http://localhost:8080", app => app.Response.WriteAsync("Hello, World!")))
{
    Console.WriteLine("Use Ctrl-C to shutdown the host...");
    host.WaitForShutdown();
}
```

Produz o mesmo resultado que **Start(RequestDelegate app)**, mas o aplicativo responde em `http://localhost:8080`.

**Start(Action<IRouteBuilder> routeBuilder)**

Use uma instância de `IRouteBuilder` ([Microsoft.AspNetCore.Routing](https://www.nuget.org/packages/Microsoft.AspNetCore.Routing/)) para usar o middleware de roteamento:

```csharp
using (var host = WebHost.Start(router => router
    .MapGet("hello/{name}", (req, res, data) => 
        res.WriteAsync($"Hello, {data.Values["name"]}!"))
    .MapGet("buenosdias/{name}", (req, res, data) => 
        res.WriteAsync($"Buenos dias, {data.Values["name"]}!"))
    .MapGet("throw/{message?}", (req, res, data) => 
        throw new Exception((string)data.Values["message"] ?? "Uh oh!"))
    .MapGet("{greeting}/{name}", (req, res, data) => 
        res.WriteAsync($"{data.Values["greeting"]}, {data.Values["name"]}!"))
    .MapGet("", (req, res, data) => res.WriteAsync("Hello, World!"))))
{
    Console.WriteLine("Use Ctrl-C to shutdown the host...");
    host.WaitForShutdown();
}
```

Use as seguintes solicitações de navegador com o exemplo:

| Solicitação                                    | Resposta                                 |
| ------------------------------------------ | ---------------------------------------- |
| `http://localhost:5000/hello/Martin`       | Hello, Martin!                           |
| `http://localhost:5000/buenosdias/Catrina` | Buenos dias, Catrina!                    |
| `http://localhost:5000/throw/ooops!`       | Gera uma exceção com a cadeia de caracteres "ooops!" |
| `http://localhost:5000/throw`              | Gera uma exceção com a cadeia de caracteres "Uh oh!" |
| `http://localhost:5000/Sante/Kevin`        | Sante, Kevin!                            |
| `http://localhost:5000`                    | Olá, Mundo!                             |

`WaitForShutdown` bloqueia até que uma quebra (Ctrl-C/SIGINT ou SIGTERM) seja emitida. O aplicativo exibe a mensagem `Console.WriteLine` e aguarda um pressionamento de tecla para ser encerrado.

**Start(string url, Action<IRouteBuilder> routeBuilder)**

Use uma URL e uma instância de `IRouteBuilder`:

```csharp
using (var host = WebHost.Start("http://localhost:8080", router => router
    .MapGet("hello/{name}", (req, res, data) => 
        res.WriteAsync($"Hello, {data.Values["name"]}!"))
    .MapGet("buenosdias/{name}", (req, res, data) => 
        res.WriteAsync($"Buenos dias, {data.Values["name"]}!"))
    .MapGet("throw/{message?}", (req, res, data) => 
        throw new Exception((string)data.Values["message"] ?? "Uh oh!"))
    .MapGet("{greeting}/{name}", (req, res, data) => 
        res.WriteAsync($"{data.Values["greeting"]}, {data.Values["name"]}!"))
    .MapGet("", (req, res, data) => res.WriteAsync("Hello, World!"))))
{
    Console.WriteLine("Use Ctrl-C to shutdown the host...");
    host.WaitForShutdown();
}
```

Produz o mesmo resultado que **Start(Action<IRouteBuilder> routeBuilder)**, mas o aplicativo responde em `http://localhost:8080`.

**StartWith(Action<IApplicationBuilder> app)**

Forneça um delegado para configurar um `IApplicationBuilder`:

```csharp
using (var host = WebHost.StartWith(app => 
    app.Use(next => 
    {
        return async context => 
        {
            await context.Response.WriteAsync("Hello World!");
        };
    })))
{
    Console.WriteLine("Use Ctrl-C to shutdown the host...");
    host.WaitForShutdown();
}
```

Faça uma solicitação no navegador para `http://localhost:5000` para receber a resposta "Olá, Mundo!" `WaitForShutdown` bloqueia até que uma quebra (Ctrl-C/SIGINT ou SIGTERM) seja emitida. O aplicativo exibe a mensagem `Console.WriteLine` e aguarda um pressionamento de tecla para ser encerrado.

**StartWith(string url, Action<IApplicationBuilder> app)**

Forneça um delegado e uma URL para configurar um `IApplicationBuilder`:

```csharp
using (var host = WebHost.StartWith("http://localhost:8080", app => 
    app.Use(next => 
    {
        return async context => 
        {
            await context.Response.WriteAsync("Hello World!");
        };
    })))
{
    Console.WriteLine("Use Ctrl-C to shutdown the host...");
    host.WaitForShutdown();
}
```

Produz o mesmo resultado que **StartWith(Action<IApplicationBuilder> app)**, mas o aplicativo responde em `http://localhost:8080`.

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

**Executar**

O método `Run` inicia o aplicativo Web e bloqueia o thread de chamada até que o host seja desligado:

```csharp
host.Run();
```

**Iniciar**

Execute o host sem bloqueio, chamando seu método `Start`:

```csharp
using (host)
{
    host.Start();
    Console.ReadLine();
}
```

Se uma lista de URLs for passada para o método `Start`, ele escutará nas URLs especificadas:


```csharp
var urls = new List<string>()
{
    "http://*:5000",
    "http://localhost:5001"
};

var host = new WebHostBuilder()
    .UseKestrel()
    .UseStartup<Startup>()
    .Start(urls.ToArray());

using (host)
{
    Console.ReadLine();
}
```

---

## <a name="ihostingenvironment-interface"></a>Interface IHostingEnvironment

A [interface IHostingEnvironment](/dotnet/api/microsoft.aspnetcore.hosting.ihostingenvironment) fornece informações sobre o ambiente de hospedagem na Web do aplicativo. Use a [injeção de construtor](xref:fundamentals/dependency-injection) para obter o `IHostingEnvironment` para usar suas propriedades e métodos de extensão:

```csharp
public class CustomFileReader
{
    private readonly IHostingEnvironment _env;

    public CustomFileReader(IHostingEnvironment env)
    {
        _env = env;
    }

    public string ReadFile(string filePath)
    {
        var fileProvider = _env.WebRootFileProvider;
        // Process the file here
    }
}
```

Uma [abordagem baseada em convenção](xref:fundamentals/environments#startup-conventions) pode ser usada para configurar o aplicativo na inicialização com base no ambiente. Como alternativa, injete o `IHostingEnvironment` no construtor `Startup` para uso em `ConfigureServices`:

```csharp
public class Startup
{
    public Startup(IHostingEnvironment env)
    {
        HostingEnvironment = env;
    }

    public IHostingEnvironment HostingEnvironment { get; }

    public void ConfigureServices(IServiceCollection services)
    {
        if (HostingEnvironment.IsDevelopment())
        {
            // Development configuration
        }
        else
        {
            // Staging/Production configuration
        }

        var contentRootPath = HostingEnvironment.ContentRootPath;
    }
}
```

> [!NOTE]
> Além do método de extensão `IsDevelopment`, `IHostingEnvironment` oferece os métodos `IsStaging`, `IsProduction` e `IsEnvironment(string environmentName)`. Confira [trabalhar com vários ambientes](xref:fundamentals/environments) para obter detalhes.

O serviço `IHostingEnvironment` também pode ser injetado diretamente no método `Configure` para configurar o pipeline de processamento:

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        // In Development, use the developer exception page
        app.UseDeveloperExceptionPage();
    }
    else
    {
        // In Staging/Production, route exceptions to /error
        app.UseExceptionHandler("/error");
    }

    var contentRootPath = env.ContentRootPath;
}
```

`IHostingEnvironment` pode ser injetado no método `Invoke` ao criar um [middleware](xref:fundamentals/middleware/index#writing-middleware) personalizado:

```csharp
public async Task Invoke(HttpContext context, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        // Configure middleware for Development
    }
    else
    {
        // Configure middleware for Staging/Production
    }

    var contentRootPath = env.ContentRootPath;
}
```

## <a name="iapplicationlifetime-interface"></a>Interface IApplicationLifetime

[IApplicationLifetime](/dotnet/api/microsoft.aspnetcore.hosting.iapplicationlifetime) permite atividades pós-inicialização e desligamento. Três propriedades na interface são tokens de cancelamento usados para registrar métodos `Action` que definem eventos de inicialização e desligamento.

| Token de cancelamento    | Acionado quando&#8230; |
| --------------------- | --------------------- |
| `ApplicationStarted`  | O host foi iniciado totalmente. |
| `ApplicationStopping` | O host está executando um desligamento normal. Solicitações ainda podem estar sendo processadas. O desligamento é bloqueado até que esse evento seja concluído. |
| `ApplicationStopped`  | O host está concluindo um desligamento normal. Todas as solicitações devem ser processadas. O desligamento é bloqueado até que esse evento seja concluído. |

```csharp
public class Startup 
{
    public void Configure(IApplicationBuilder app, IApplicationLifetime appLifetime) 
    {
        appLifetime.ApplicationStarted.Register(OnStarted);
        appLifetime.ApplicationStopping.Register(OnStopping);
        appLifetime.ApplicationStopped.Register(OnStopped);

        Console.CancelKeyPress += (sender, eventArgs) =>
        {
            appLifetime.StopApplication();
            // Don't terminate the process immediately, wait for the Main thread to exit gracefully.
            eventArgs.Cancel = true;
        };
    }

    private void OnStarted()
    {
        // Perform post-startup activities here
    }

    private void OnStopping()
    {
        // Perform on-stopping activities here
    }

    private void OnStopped()
    {
        // Perform post-stopped activities here
    }
}
```

[StopApplication](/dotnet/api/microsoft.aspnetcore.hosting.iapplicationlifetime.stopapplication) solicita o término do aplicativo. A classe a seguir usa `StopApplication` para desligar normalmente um aplicativo quando o método `Shutdown` da classe é chamado:

```csharp
public class MyClass
{
    private readonly IApplicationLifetime _appLifetime;

    public MyClass(IApplicationLifetime appLifetime)
    {
        _appLifetime = appLifetime;
    }

    public void Shutdown()
    {
        _appLifetime.StopApplication();
    }
}
```

## <a name="scope-validation"></a>Validação de escopo

No ASP.NET Core 2.0 ou posterior, [CreateDefaultBuilder](/dotnet/api/microsoft.aspnetcore.webhost.createdefaultbuilder) define [ServiceProviderOptions.ValidateScopes](/dotnet/api/microsoft.extensions.dependencyinjection.serviceprovideroptions.validatescopes) como `true` quando o ambiente do aplicativo é de desenvolvimento.

Quando `ValidateScopes` está definido como `true`, o provedor de serviço padrão executa verificações para saber se:

* Os serviços com escopo não são resolvidos direta ou indiretamente pelo provedor de serviço raiz.
* Os serviços com escopo não são injetados direta ou indiretamente em singletons.

O provedor de serviços raiz é criado quando [BuildServiceProvider](/dotnet/api/microsoft.extensions.dependencyinjection.servicecollectioncontainerbuilderextensions.buildserviceprovider) é chamado. O tempo de vida do provedor de serviço raiz corresponde ao tempo de vida do aplicativo/servidor quando o provedor começa com o aplicativo e é descartado quando o aplicativo é desligado.

Os serviços com escopo são descartados pelo contêiner que os criou. Se um serviço com escopo é criado no contêiner raiz, o tempo de vida do serviço é promovido efetivamente para singleton, porque ele só é descartado pelo contêiner raiz quando o aplicativo/servidor é desligado. A validação dos escopos de serviço detecta essas situações quando `BuildServiceProvider` é chamado.

Para que os escopos sempre sejam validados, incluindo no ambiente de produção, configure [ServiceProviderOptions](/dotnet/api/microsoft.extensions.dependencyinjection.serviceprovideroptions) com [UseDefaultServiceProvider](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.usedefaultserviceprovider) no construtor do host:

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseDefaultServiceProvider((context, options) => {
        options.ValidateScopes = true;
    })
```

## <a name="troubleshooting-systemargumentexception"></a>Solução de problemas de System.ArgumentException

**Aplica-se somente ao ASP.NET Core 2.0**

Um host pode ser compilado injetando `IStartup` diretamente no contêiner de injeção de dependência em vez de chamar `UseStartup` ou `Configure`:

```csharp
services.AddSingleton<IStartup, Startup>();
```

Se o host for compilado dessa maneira, o seguinte erro poderá ocorrer:

```
Unhandled Exception: System.ArgumentException: A valid non-empty application name must be provided.
```

Isso ocorre porque o [applicationName(ApplicationKey)](/dotnet/api/microsoft.aspnetcore.hosting.webhostdefaults#Microsoft_AspNetCore_Hosting_WebHostDefaults_ApplicationKey) (do assembly atual) é necessário para verificar se há `HostingStartupAttributes`. Se o aplicativo injetar manualmente `IStartup` no contêiner de injeção de dependência, adicione a seguinte chamada para `WebHostBuilder` com o nome do assembly especificado:

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting("applicationName", "<Assembly Name>")
    ...
```

Como alternativa, adicione um `Configure` fictício ao `WebHostBuilder`, que define o `applicationName`(`ApplicationKey`) automaticamente:

```csharp
WebHost.CreateDefaultBuilder(args)
    .Configure(_ => { })
    ...
```

**Observação**: isso só é necessário com a versão do ASP.NET Core 2.0 e somente quando o aplicativo não chama `UseStartup` ou `Configure`.

Para obter mais informações, consulte [Comunicado: Microsoft.Extensions.PlatformAbstractions foi removido (comentário)](https://github.com/aspnet/Announcements/issues/237#issuecomment-323786938) e o [Exemplo de StartupInjection](https://github.com/aspnet/Hosting/blob/8377d226f1e6e1a97dabdb6769a845eeccc829ed/samples/SampleStartups/StartupInjection.cs).

## <a name="additional-resources"></a>Recursos adicionais

* [Hospedar no Windows com o IIS](xref:host-and-deploy/iis/index)
* [Hospedar em Linux com o Nginx](xref:host-and-deploy/linux-nginx)
* [Hospedar em Linux com o Apache](xref:host-and-deploy/linux-apache)
* [Hospedar em um serviço Windows](xref:host-and-deploy/windows-service)
