---
uid: web-forms/overview/ajax-control-toolkit/animation/adding-animation-to-a-control-vb
title: Adicionar animação a um controle (VB) | Microsoft Docs
author: wenz
description: O controle de animação no Kit de ferramentas de controle AJAX ASP.NET não é apenas um controle, mas uma estrutura inteira para adicionar animações a um controle. Este tutorial mostra como...
ms.author: aspnetcontent
manager: wpickett
ms.date: 06/02/2008
ms.topic: article
ms.assetid: c120187e-963e-4439-bb85-32771bc7f1f4
ms.technology: dotnet-webforms
ms.prod: .net-framework
msc.legacyurl: /web-forms/overview/ajax-control-toolkit/animation/adding-animation-to-a-control-vb
msc.type: authoredcontent
ms.openlocfilehash: 3da98e478c45213875b3829e51351d03571a05b8
ms.sourcegitcommit: f8852267f463b62d7f975e56bea9aa3f68fbbdeb
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/06/2018
---
<a name="adding-animation-to-a-control-vb"></a>Adicionar animação a um controle (VB)
====================
por [Christian Wenz](https://github.com/wenz)

[Baixar o código](http://download.microsoft.com/download/f/9/a/f9a26acd-8df4-4484-8a18-199e4598f411/Animation1.vb.zip) ou [baixar PDF](http://download.microsoft.com/download/6/7/1/6718d452-ff89-4d3f-a90e-c74ec2d636a3/animation1VB.pdf)

> O controle de animação no Kit de ferramentas de controle AJAX ASP.NET não é apenas um controle, mas uma estrutura inteira para adicionar animações a um controle. Este tutorial mostra como configurar esse uma animação.


## <a name="overview"></a>Visão geral

O controle de animação no Kit de ferramentas de controle AJAX ASP.NET não é apenas um controle, mas uma estrutura inteira para adicionar animações a um controle. Este tutorial mostra como configurar esse uma animação.

## <a name="steps"></a>Etapas

Normalmente, a primeira etapa é incluir o `ScriptManager` na página de forma que a biblioteca ASP.NET AJAX foi carregada e o Kit de ferramentas de controle pode ser usado:

[!code-aspx[Main](adding-animation-to-a-control-vb/samples/sample1.aspx)]

A animação neste cenário será aplicada a um painel de texto que tem esta aparência:

[!code-aspx[Main](adding-animation-to-a-control-vb/samples/sample2.aspx)]

A classe CSS associada para o painel define uma cor de plano de fundo e uma largura:

[!code-css[Main](adding-animation-to-a-control-vb/samples/sample3.css)]

Em seguida, nós precisamos de `AnimationExtender`. Depois de fornecer um `ID` e o usual `runat="server"`, o `TargetControlID` atributo deve ser definido para o controle para animar em nosso caso, o painel:

[!code-aspx[Main](adding-animation-to-a-control-vb/samples/sample4.aspx)]

A animação inteira é aplicada declarativamente, usando uma sintaxe XML, infelizmente atualmente não têm suportada completo do Visual Studio IntelliSense. O nó raiz é `<Animations>;` dentro desse nó, vários eventos são permitidos que determinam quando as animações take(s) local:

- `OnClick` (clique do mouse)
- `OnHoverOut` (quando o mouse sai um controle)
- `OnHoverOver` (quando o mouse passa sobre um controle, interrompendo o `OnHoverOut` animação)
- `OnLoad` (quando a página foi carregada)
- `OnMouseOut` (quando o mouse sai um controle)
- `OnMouseOver` (quando o mouse passa sobre um controle, não interrompendo a `OnMouseOut` animação)

O framework vem com um conjunto de animações, cada um representado por seu próprio elemento XML. Aqui está uma seleção:

- `<Color>` (uma cor de alteração)
- `<FadeIn>` (fade in)
- `<FadeOut>` (desaparecimento)
- `<Property>` (a propriedade do controle de alteração)
- `<Pulse>` (pulsating)
- `<Resize>` (o tamanho de alteração)
- `<Scale>` (proporcionalmente alterando o tamanho)

Neste exemplo, o painel deve desaparecer. A animação terão 1,5 segundos (`Duration` atributo), exibindo 24 (etapas de animação) de quadros por segundo (`Fps` attributs). Aqui está a marcação concluída para o `AnimationExtender` controle:

[!code-aspx[Main](adding-animation-to-a-control-vb/samples/sample5.aspx)]

Quando você executar esse script, o painel é exibido e desaparece em um e meio segundos.


[![O painel é desaparecimento](adding-animation-to-a-control-vb/_static/image2.png)](adding-animation-to-a-control-vb/_static/image1.png)

O painel é desaparecimento ([clique para exibir a imagem em tamanho normal](adding-animation-to-a-control-vb/_static/image3.png))

> [!div class="step-by-step"]
> [Anterior](dynamically-controlling-updatepanel-animations-cs.md)
> [Próximo](executing-several-animations-at-the-same-time-vb.md)
