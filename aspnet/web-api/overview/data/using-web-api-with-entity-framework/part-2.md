---
uid: web-api/overview/data/using-web-api-with-entity-framework/part-2
title: Adicionar modelos e controladores | Microsoft Docs
author: MikeWasson
description: ''
ms.author: aspnetcontent
manager: wpickett
ms.date: 06/16/2014
ms.topic: article
ms.assetid: 88908ff8-51a9-40eb-931c-a8139128b680
ms.technology: dotnet-webapi
ms.prod: .net-framework
msc.legacyurl: /web-api/overview/data/using-web-api-with-entity-framework/part-2
msc.type: authoredcontent
ms.openlocfilehash: 015bb9698d81387d03ea8f9629316fb53232e708
ms.sourcegitcommit: f8852267f463b62d7f975e56bea9aa3f68fbbdeb
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/06/2018
---
<a name="add-models-and-controllers"></a>Adicionar modelos e controladores
====================
por [Mike Wasson](https://github.com/MikeWasson)

[Baixe o projeto concluído](https://github.com/MikeWasson/BookService)

Nesta seção, você irá adicionar classes de modelo que definem as entidades de banco de dados. Em seguida, você irá adicionar controladores de API da Web que executem operações CRUD dessas entidades.

## <a name="add-model-classes"></a>Adicionar Classes de modelo

Neste tutorial, vamos criar o banco de dados usando a abordagem "Code First" para o Entity Framework (EF). Com o Code First, você escrever classes c# que correspondem às tabelas de banco de dados e EF cria o banco de dados. (Para obter mais informações, consulte [abordagens de desenvolvimento do Entity Framework](https://msdn.microsoft.com/library/ms178359%28v=vs.110%29.aspx#dbfmfcf).)

Vamos começar definindo nossos objetos de domínio como POCOs (objetos CLR antigo simples). Vamos criar POCOs a seguir:

- Autor
- Catálogo

No Gerenciador de soluções, clique com botão direito na pasta modelos. Selecione **adicionar**, em seguida, selecione **classe**. Nomeie a classe `Author`.

![](part-2/_static/image1.png)

Substitua todo o código clichê em Author.cs com o código a seguir.

[!code-csharp[Main](part-2/samples/sample1.cs)]

Adicionar outra classe chamada `Book`, com o código a seguir.

[!code-csharp[Main](part-2/samples/sample2.cs)]

Entity Framework usará esses modelos para criar tabelas de banco de dados. Para cada modelo, o `Id` propriedade tornam-se a coluna de chave primária da tabela de banco de dados.

Na classe do catálogo, o `AuthorId` define uma chave estrangeira para a `Author` tabela. (Para manter a simplicidade, suponho que cada livro tem um autor único.) A classe de catálogo também contém uma propriedade de navegação relacionado `Author`. Você pode usar a propriedade de navegação para acessar relacionado `Author` no código. Devo dizer mais sobre as propriedades de navegação na parte 4, [tratar relações de entidade](part-4.md).

## <a name="add-web-api-controllers"></a>Adicionar controladores de API da Web

Nesta seção, vamos adicionar controladores de API da Web que oferecem suporte a operações CRUD (criar, ler, atualizar e excluir). Os controladores usará o Entity Framework para se comunicar com a camada de banco de dados.

Primeiro, você pode excluir o arquivo Controllers/ValuesController.cs. Esse arquivo contém um controlador de API da Web de exemplo, mas ele não é necessário para este tutorial.

![](part-2/_static/image2.png)

Em seguida, crie o projeto. O scaffolding de API da Web usa reflexão para localizar as classes de modelo, para que ele precisa do assembly compilado.

No Gerenciador de soluções, clique na pasta de controladores. Selecione **adicionar**, em seguida, selecione **controlador**.

![](part-2/_static/image3.png)

No **adicionar Scaffold** caixa de diálogo, selecione "Web API 2 controlador com ações, usando o Entity Framework". Clique em **Adicionar**.

![](part-2/_static/image4.png)

No **Adicionar controlador** caixa de diálogo, faça o seguinte:

1. No **classe modelo** lista suspensa, selecione o `Author` classe. (Se você não vê-lo listado na lista suspensa, verifique se que você criou o projeto.)
2. Verifique se "Use ações assíncronas do controlador".
3. Deixe o nome do controlador como &quot;AuthorsController&quot;.
4. Clique em mais (+) próximo ao botão **classe de contexto de dados**.

![](part-2/_static/image5.png)

No **novo contexto de dados** caixa de diálogo, deixe o nome padrão e clique em **adicionar**.

![](part-2/_static/image6.png)

Clique em **adicionar** para concluir o **Adicionar controlador** caixa de diálogo. A caixa de diálogo adiciona duas classes ao seu projeto:

- `AuthorsController` define um controlador Web API. O controlador implementa a API REST que os clientes usam para executar operações CRUD na lista de autores.
- `BookServiceContext` gerencia objetos de entidade durante o tempo de execução, o que inclui a popular objetos com dados de um banco de dados, controle de alterações e manter dados para o banco de dados. Ele herda de `DbContext`.

![](part-2/_static/image7.png)

Neste ponto, compile o projeto novamente. Agora percorrer as mesmas etapas para adicionar um controlador de API para `Book` entidades. Neste momento, selecione `Book` para a classe de modelo e selecione existente `BookServiceContext` classe para a classe de contexto de dados. (Não criar um novo contexto de dados). Clique em **adicionar** para adicionar o controlador.

![](part-2/_static/image8.png)

> [!div class="step-by-step"]
> [Anterior](part-1.md)
> [Próximo](part-3.md)
