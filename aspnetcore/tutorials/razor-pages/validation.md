---
title: Adicionar validação a uma Página Razor do ASP.NET Core
author: rick-anderson
description: Saiba como adicionar validação a uma Página Razor no ASP.NET Core.
manager: wpickett
ms.author: riande
ms.date: 08/07/2017
ms.prod: aspnet-core
ms.technology: aspnet
ms.topic: get-started-article
uid: tutorials/razor-pages/validation
ms.openlocfilehash: 271a5ce517ae550845d96e3969b39b1eda6ae51b
ms.sourcegitcommit: f8852267f463b62d7f975e56bea9aa3f68fbbdeb
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/06/2018
---
# <a name="add-validation-to-an-aspnet-core-razor-page"></a>Adicionar validação a uma Página Razor do ASP.NET Core

Por [Rick Anderson](https://twitter.com/RickAndMSFT)

Nesta seção, a lógica de validação é adicionada para o modelo `Movie`. As regras de validação são impostas sempre que um usuário cria ou edita um filme.

## <a name="validation"></a>Validação

Um princípio-chave do desenvolvimento de software é chamado [DRY](https://wikipedia.org/wiki/Don%27t_repeat_yourself) (“**D**on't **R**epeat **Y**ourself”). As Páginas Razor incentivam o desenvolvimento quando a funcionalidade é especificada uma vez e ela é refletida em todo o aplicativo. O DRY pode ajudar a reduzir a quantidade de código em um aplicativo. O DRY faz com que o código seja menos propenso a erros e mais fácil de testar e manter.

O suporte de validação fornecido pelas Páginas Razor e pelo Entity Framework é um bom exemplo do princípio DRY. As regras de validação são especificadas de forma declarativa em um único lugar (na classe de modelo) e as regras são impostas em qualquer lugar no aplicativo.

### <a name="adding-validation-rules-to-the-movie-model"></a>Adicionando regras de validação ao modelo de filme

Abra o arquivo *Movie.cs*. [DataAnnotations](https://docs.microsoft.com/aspnet/mvc/overview/older-versions/mvc-music-store/mvc-music-store-part-6) fornece um conjunto interno de atributos de validação que são aplicados de forma declarativa a uma classe ou propriedade. DataAnnotations também contém atributos de formatação como `DataType`, que ajudam com a formatação e não fornecem validação.

Atualize a classe `Movie` para aproveitar os atributos de validação `Required`, `StringLength`, `RegularExpression` e `Range`.

[!code-csharp[](../../tutorials/first-mvc-app/start-mvc//sample/MvcMovie/Models/MovieDateRatingDA.cs?name=snippet1)]

Os atributos de validação especificam o comportamento que é imposto nas propriedades do modelo:

* Os atributos `Required` e `MinimumLength` indicam que uma propriedade deve ter um valor. No entanto, nada impede que um usuário digite espaços em branco para atender à restrição de validação para um tipo de permite valor nulo. Os [tipos de valor](https://docs.microsoft.com/dotnet/csharp/language-reference/keywords/value-types) que não permitem valores nulos (como `decimal`, `int`, `float` e `DateTime`) são inerentemente necessários e não precisam do atributo `Required`.
* O atributo `RegularExpression` limita os caracteres que o usuário pode inserir. No código anterior, `Genre` e `Rating` devem usar apenas letras (espaço em branco, números e caracteres especiais não são permitidos).
* O atributo `Range` restringe um valor a um intervalo especificado.
* O atributo `StringLength` define o tamanho máximo de uma cadeia de caracteres e, opcionalmente, o tamanho mínimo. 

Ter as regras de validação automaticamente impostas pelo ASP.NET Core ajuda a tornar um aplicativo mais robusto. A validação automática em modelos ajuda a proteger o aplicativo porque você não precisa se lembrar de aplicá-los quando um novo código é adicionado.

### <a name="validation-error-ui-in-razor-pages"></a>Interface do usuário do erro de validação nas Páginas Razor

Execute o aplicativo e navegue para Pages/Movies.

Selecione o link **Criar Novo**. Preencha o formulário com alguns valores inválidos. Quando a validação do lado do cliente do jQuery detecta o erro, ela exibe uma mensagem de erro.

![Formulário da exibição de filmes com vários erros de validação do lado do cliente do jQuery](validation/_static/val.png)

> [!NOTE]
> Talvez você não consiga inserir pontos decimais ou vírgulas no campo `Price`. Para dar suporte à [validação do jQuery](https://jqueryvalidation.org/) para localidades de idiomas diferentes do inglês que usam uma vírgula (“,”) para um ponto decimal e formatos de data diferentes do inglês dos EUA, você deve tomar medidas para globalizar o aplicativo. Consulte [Recursos adicionais](#additional-resources) para obter mais informações. Por enquanto, insira apenas números inteiros como 10.

Observe como o formulário renderizou automaticamente uma mensagem de erro de validação em cada campo que contém um valor inválido. Os erros são impostos no lado do cliente (usando o JavaScript e o jQuery) e no lado do servidor (quando um usuário tem o JavaScript desabilitado).

Uma vantagem significativa é que **nenhuma** alteração de código foi necessária nas páginas Criar ou Editar. Depois que DataAnnotations foi aplicado ao modelo, a interface do usuário de validação foi habilitada. As Páginas Razor criadas neste tutorial selecionaram automaticamente as regras de validação (usando os atributos de validação nas propriedades da classe do modelo `Movie`). Validação do teste usando a página Editar: a mesma validação é aplicada.

Os dados de formulário não serão postados no servidor enquanto houver erros de validação do lado do cliente. Verifique se os dados de formulário não são postados por uma ou mais das seguintes abordagens:

* Coloque um ponto de interrupção no método `OnPostAsync`. Envie o formulário (selecione **Criar** ou **Salvar**). O ponto de interrupção nunca é atingido.
* Use a [ferramenta Fiddler](http://www.telerik.com/fiddler).
* Use as ferramentas do desenvolvedor do navegador para monitorar o tráfego de rede.

### <a name="server-side-validation"></a>Validação do servidor

Quando o JavaScript está desabilitado no navegador, o envio do formulário com erros será postado no servidor.

(Opcional) Teste a validação do servidor:

* Desabilite o JavaScript no navegador. Se você não conseguir desabilitar o JavaScript no navegador, tente outro navegador.
* Defina um ponto de interrupção no método `OnPostAsync` da página Criar ou Editar.
* Envie um formulário com erros de validação.
* Verifique se o estado do modelo é inválido:

  ```csharp
   if (!ModelState.IsValid)
   {
      return Page();
   }
  ```

O código a seguir mostra uma parte da página *Create.cshtml* gerada por scaffolding anteriormente no tutorial. Ele é usado pelas páginas Criar e Editar para exibir o formulário inicial e exibir o formulário novamente, em caso de erro.

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie/Pages/Movies/Create.cshtml?range=14-20)]

O [Auxiliar de Marcação de Entrada](xref:mvc/views/working-with-forms) usa os atributos de [DataAnnotations](https://docs.microsoft.com/aspnet/mvc/overview/older-versions/mvc-music-store/mvc-music-store-part-6) e produz os atributos HTML necessários para a Validação do jQuery no lado do cliente. O [Auxiliar de Marcação de Validação](xref:mvc/views/working-with-forms#the-validation-tag-helpers) exibe erros de validação. Consulte [Validação](xref:mvc/models/validation) para obter mais informações.

As páginas Criar e Editar não têm nenhuma regra de validação. As regras de validação e as cadeias de caracteres de erro são especificadas somente na classe `Movie`. Essas regras de validação são aplicadas automaticamente às Páginas Razor que editam o modelo `Movie`.

Quando a lógica de validação precisa ser alterada, ela é feita apenas no modelo. A validação é aplicada de forma consistente em todo o aplicativo (a lógica de validação é definida em um único lugar). A validação em um único lugar ajuda a manter o código limpo e facilita sua manutenção e atualização.

## <a name="using-datatype-attributes"></a>Usando atributos DataType

Examine a classe `Movie`. O namespace `System.ComponentModel.DataAnnotations` fornece atributos de formatação, além do conjunto interno de atributos de validação. O atributo `DataType` é aplicado às propriedades `ReleaseDate` e `Price`.

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie/Models/MovieDateRatingDA.cs?highlight=2,6&name=snippet2)]

Os atributos `DataType` fornecem apenas dicas para que o mecanismo de exibição formate os dados (e fornece atributos como `<a>` para as URLs e `<a href="mailto:EmailAddress.com">` para o email). Use o atributo `RegularExpression` para validar o formato dos dados. O atributo `DataType` é usado para especificar um tipo de dados mais específico do que o tipo intrínseco de banco de dados. Os atributos `DataType` não são atributos de validação. No aplicativo de exemplo, apenas a data é exibida, sem a hora.

A Enumeração `DataType` fornece muitos tipos de dados, como Date, Time, PhoneNumber, Currency, EmailAddress e muito mais. O atributo `DataType` também pode permitir que o aplicativo forneça automaticamente recursos específicos a um tipo. Por exemplo, um link `mailto:` pode ser criado para `DataType.EmailAddress`. Um seletor de data pode ser fornecido para `DataType.Date` em navegadores que dão suporte a HTML5. Os atributos `DataType` emitem atributos `data-` HTML 5 (pronunciados “data dash”) que são consumidos pelos navegadores HTML 5. Os atributos `DataType` **não** fornecem nenhuma validação.

`DataType.Date` não especifica o formato da data exibida. Por padrão, o campo de dados é exibido de acordo com os formatos padrão com base nas `CultureInfo` do servidor.

O atributo `DisplayFormat` é usado para especificar explicitamente o formato de data:

```csharp
[DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
public DateTime ReleaseDate { get; set; }
```

A configuração `ApplyFormatInEditMode` especifica que a formatação deve ser aplicada quando o valor é exibido para edição. Não é recomendável ter esse comportamento em alguns campos. Por exemplo, em valores de moeda, você provavelmente não deseja que o símbolo de moeda seja exibido na interface do usuário de edição.

O atributo `DisplayFormat` pode ser usado por si só, mas geralmente é uma boa ideia usar o atributo `DataType`. O atributo `DataType` transmite a semântica dos dados, ao invés de apresentar como renderizá-lo em uma tela e oferece os seguintes benefícios que você não obtém com DisplayFormat:

* O navegador pode habilitar os recursos do HTML5 (por exemplo, mostrar um controle de calendário, o símbolo de moeda apropriado à localidade, links de email, etc.)
* Por padrão, o navegador renderizará os dados usando o formato correto de acordo com a localidade.
* O atributo `DataType` pode permitir que a estrutura ASP.NET Core escolha o modelo de campo correto para renderizar os dados. O `DisplayFormat`, se usado por si só, usa o modelo de cadeia de caracteres.

Observação: a validação do jQuery não funciona com os atributos `Range` e `DateTime`. Por exemplo, o seguinte código sempre exibirá um erro de validação do lado do cliente, mesmo quando a data estiver no intervalo especificado:

```csharp
[Range(typeof(DateTime), "1/1/1966", "1/1/2020")]
   ```

Geralmente, não é uma boa prática compilar datas rígidas nos modelos e, portanto, o uso do atributo `Range` e de `DateTime` não é recomendado.

O seguinte código mostra como combinar atributos em uma linha:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie/Models/MovieDateRatingDAmult.cs?name=snippet1)]

A [Introdução às Páginas Razor e ao EF Core](xref:data/ef-rp/intro) mostra operações mais avançadas do EF Core com Páginas Razor.

### <a name="publish-to-azure"></a>Publicar no Azure

Confira as instruções sobre como publicar este aplicativo no Azure em [Publicar um aplicativo Web ASP.NET Core no Serviço de Aplicativo do Azure usando o Visual Studio](xref:tutorials/publish-to-azure-webapp-using-vs).

## <a name="additional-resources"></a>Recursos adicionais

* [Trabalhando com formulários](xref:mvc/views/working-with-forms)
* [Globalização e localização](xref:fundamentals/localization)
* [Introdução aos auxiliares de marcação](xref:mvc/views/tag-helpers/intro)
* [Auxiliares de marca de autor](xref:mvc/views/tag-helpers/authoring)

> [!div class="step-by-step"]
> [Antes: adicionando um novo campo](xref:tutorials/razor-pages/new-field)
> [Depois: carregando arquivos](xref:tutorials/razor-pages/uploading-files)
