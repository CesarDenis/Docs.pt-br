---
uid: mvc/overview/older-versions-1/controllers-and-routing/aspnet-mvc-controllers-overview-cs
title: Visão geral do controlador MVC do ASP.NET (c#) | Microsoft Docs
author: StephenWalther
description: Neste tutorial, Stephen Walther apresenta controladores do ASP.NET MVC. Você aprenderá a criar novos controladores e retornar tipos diferentes de res de ação...
ms.author: aspnetcontent
manager: wpickett
ms.date: 02/16/2008
ms.topic: article
ms.assetid: b985c49a-3668-455c-a366-f85f6bc64b12
ms.technology: dotnet-mvc
ms.prod: .net-framework
msc.legacyurl: /mvc/overview/older-versions-1/controllers-and-routing/aspnet-mvc-controllers-overview-cs
msc.type: authoredcontent
ms.openlocfilehash: 95e7c555a52c8c3b765a6fffab15276491cf5714
ms.sourcegitcommit: f8852267f463b62d7f975e56bea9aa3f68fbbdeb
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/06/2018
---
<a name="aspnet-mvc-controller-overview-c"></a>Visão geral do controlador MVC do ASP.NET (c#)
====================
por [Stephen Walther](https://github.com/StephenWalther)

> Neste tutorial, Stephen Walther apresenta controladores do ASP.NET MVC. Você aprenderá a criar novos controladores e retornar tipos diferentes de resultados da ação.


Este tutorial explora o tópico de controladores do ASP.NET MVC, ações do controlador e resultados de ação. Depois de concluir este tutorial, você entenderá como são usados para controlar a maneira como um visitante interage com um site ASP.NET MVC.

## <a name="understanding-controllers"></a>Noções básicas sobre controladores

Controladores MVC serão responsáveis por responder a solicitações feitas em relação a um site ASP.NET MVC. Cada solicitação do navegador é mapeada para um controlador específico. Por exemplo, imagine que você digite a seguinte URL na barra de endereços do navegador:

`http://localhost/Product/Index/3`

Nesse caso, um controlador chamado ProductController é invocado. O ProductController é responsável por gerar a resposta para a solicitação do navegador. Por exemplo, o controlador pode retornar uma exibição específica para o navegador ou o controlador pode redirecionar o usuário para outro controlador.

Listar 1 contém um controlador simple chamado ProductController.

**Listing1 - Controllers\ProductController.cs**

[!code-csharp[Main](aspnet-mvc-controllers-overview-cs/samples/sample1.cs)]

Como você pode ver na listagem 1, um controlador é apenas uma classe (uma classe .NET do Visual Basic ou c#). Um controlador é uma classe que deriva da classe Controller base. Como um controlador herda dessa classe base, um controlador herda diversos métodos úteis gratuitamente (vamos abordar esses métodos em breve).

## <a name="understanding-controller-actions"></a>Noções básicas sobre ações do controlador

Um controlador expõe as ações do controlador. Uma ação é um método em um controlador que é chamado quando você inserir uma URL específica na barra de endereços do navegador. Por exemplo, imagine que você faz uma solicitação para a seguinte URL:

`http://localhost/Product/Index/3`

Nesse caso, o método de Index () é chamado na classe ProductController. O método Index () é um exemplo de uma ação do controlador.

Uma ação do controlador deve ser um método público de uma classe de controlador. Métodos c#, por padrão, são métodos privados. Observe que nenhum método público que você adicionar a uma classe de controlador é exposto como uma ação do controlador automaticamente (você deve ter cuidado sobre isso como uma ação do controlador pode ser chamada por qualquer pessoa no universo simplesmente digitando a URL correta em uma barra de endereços do navegador).

Há alguns requisitos adicionais que devem ser atendidos por uma ação do controlador. Um método usado como uma ação de controlador não pode estar sobrecarregado. Além disso, uma ação do controlador não pode ser um método estático. Além disso, você pode usar qualquer método como uma ação do controlador.

## <a name="understanding-action-results"></a>Entendendo os resultados da ação

Uma ação do controlador retorna algo chamado de um *resultado da ação*. Resultado de uma ação é o que retorna uma ação do controlador em resposta a uma solicitação do navegador.

A estrutura ASP.NET MVC oferece suporte a vários tipos de resultados de ação, inclusive:

1. ViewResult - representa HTML e marcação.
2. EmptyResult - não representa nenhum resultado.
3. RedirectResult - representa um redirecionamento para uma nova URL.
4. JsonResult - representa um resultado de JavaScript Object Notation que pode ser usado em um aplicativo AJAX.
5. JavaScriptResult - representa um script de JavaScript.
6. ContentResult - representa um resultado de texto.
7. FileContentResult - representa um arquivo baixável (com o conteúdo binário).
8. FilePathResult - representa um arquivo baixável (com um caminho).
9. FileStreamResult - representa um arquivo baixável (com um fluxo de arquivos).

Todos esses resultados de ação herdam da classe ActionResult base.

Na maioria dos casos, uma ação do controlador retorna um ViewResult. Por exemplo, a ação de controlador de índice na lista 2 retorna um ViewResult.

**A listagem 2 - Controllers\BookController.cs**

[!code-csharp[Main](aspnet-mvc-controllers-overview-cs/samples/sample2.cs)]

Quando uma ação retorna um ViewResult, HTML é retornado para o navegador. O método Index () na lista 2 retorna uma exibição chamada índice para o navegador.

Observe que a ação Index () na lista 2 não retorna um ViewResult(). Em vez disso, é chamado o método View() da classe base do controlador. Normalmente, você não retornar um resultado de ação diretamente. Em vez disso, chame um dos seguintes métodos da classe base do controlador:

1. Exibição - retorna um resultado de ação ViewResult.
2. Redirecionar - retorna um resultado de ação RedirectResult.
3. RedirectToAction - retorna um resultado de ação RedirectToRouteResult.
4. RedirectToRoute - retorna um resultado de ação RedirectToRouteResult.
5. JSON - retorna um resultado de ação JsonResult.
6. JavaScriptResult - retorna um JavaScriptResult.
7. Content - retorna um resultado de ação ContentResult.
8. Arquivo - retorna um FileContentResult, FilePathResult ou FileStreamResult dependendo dos parâmetros passado para o método.

Portanto, se você quiser retornar um modo de exibição no navegador, você chamar o método de View(). Se você deseja redirecionar o usuário da ação de um controlador para outro, você chamar o método RedirectToAction(). Por exemplo, a ação de Details() na listagem 3 mostra uma exibição tanto redireciona o usuário para a ação Index (), dependendo se o parâmetro de Id tem um valor.

**A listagem 3 - CustomerController.cs**

[!code-csharp[Main](aspnet-mvc-controllers-overview-cs/samples/sample3.cs)]

O resultado da ação ContentResult é especial. Você pode usar o resultado da ação ContentResult para retornar o resultado de uma ação como texto sem formatação. Por exemplo, o método Index () na listagem 4 retorna uma mensagem como texto sem formatação e não como HTML.

**A listagem 4 - Controllers\StatusController.cs**

[!code-csharp[Main](aspnet-mvc-controllers-overview-cs/samples/sample4.cs)]

Quando a ação de StatusController.Index() é invocada, uma exibição não é retornada. Em vez disso, o texto não processado "Hello World!" é retornado para o navegador.

Se uma ação do controlador retorna um resultado não é um resultado de ação - por exemplo, uma data ou um inteiro -, em seguida, o resultado é encapsulado em um ContentResult automaticamente. Por exemplo, quando a ação Index () de WorkController na listagem 5 é chamada, a data é retornada como um ContentResult automaticamente.

**Listagem 5 - WorkController.cs**

[!code-csharp[Main](aspnet-mvc-controllers-overview-cs/samples/sample5.cs)]

A ação Index () na listagem 5 retorna um objeto DateTime. A estrutura ASP.NET MVC converte o objeto DateTime em uma cadeia de caracteres e encapsula o valor de DateTime em um ContentResult automaticamente. O navegador recebe a data e hora como texto sem formatação.

## <a name="summary"></a>Resumo

O objetivo deste tutorial foi apresentar os conceitos de controladores do ASP.NET MVC, ações do controlador e resultados de ação do controlador. A primeira seção, você aprendeu a adicionar novos controladores para um projeto ASP.NET MVC. Em seguida, você aprendeu como públicos métodos de um controlador são expostos para o universo como ações do controlador. Por fim, discutimos os diferentes tipos de resultados de ação que podem ser retornados de uma ação do controlador. Em particular, discutimos como retornar um ViewResult, RedirectToActionResult e ContentResult de uma ação do controlador.

> [!div class="step-by-step"]
> [Anterior](creating-an-action-vb.md)
> [Próximo](creating-custom-routes-cs.md)
