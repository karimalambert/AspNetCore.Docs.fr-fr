---
title: Utiliser des cookies SameSite dans ASP.NET Core
author: rick-anderson
description: Découvrez comment utiliser pour SameSite des cookies dans ASP.NET Core
ms.author: riande
ms.custom: mvc
ms.date: 12/03/2019
no-loc:
- Electron
uid: security/samesite
ms.openlocfilehash: eeba2c4403d33312692ed187021a125c22df5d08
ms.sourcegitcommit: 9a129f5f3e31cc449742b164d5004894bfca90aa
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/06/2020
ms.locfileid: "78667740"
---
# <a name="work-with-samesite-cookies-in-aspnet-core"></a>Utiliser des cookies SameSite dans ASP.NET Core

De [Rick Anderson](https://twitter.com/RickAndMSFT)

SameSite est un projet standard de l' [IETF](https://ietf.org/about/) conçu pour offrir une protection contre les attaques de falsification de requête intersites (CSRF). Initialement rédigée dans [2016](https://tools.ietf.org/html/draft-west-first-party-cookies-07), le brouillon standard a été mis à jour dans [2019](https://tools.ietf.org/html/draft-west-cookie-incrementalism-00). La norme mise à jour n’est pas à compatibilité descendante avec la norme précédente, avec comme suit les différences les plus perceptibles :

* Les cookies sans en-tête SameSite sont traités comme `SameSite=Lax` par défaut.
* `SameSite=None` doit être utilisé pour autoriser l’utilisation de cookies entre sites.
* Les cookies qui déclarent `SameSite=None` doivent également être marqués comme `Secure`.
* Les applications qui utilisent [`<iframe>`](https://developer.mozilla.org/docs/Web/HTML/Element/iframe) peuvent rencontrer des problèmes avec les cookies `sameSite=Lax` ou `sameSite=Strict`, car `<iframe>` est traité comme des scénarios intersites.
* La valeur `SameSite=None` n’est pas autorisée par la [norme 2016](https://tools.ietf.org/html/draft-west-first-party-cookies-07) et oblige certaines implémentations à traiter ces cookies comme des `SameSite=Strict`. Consultez [prise en charge des navigateurs plus anciens](#sob) dans ce document.

Le paramètre `SameSite=Lax` fonctionne pour la plupart des cookies d’application. Certaines formes d’authentification comme [OpenID Connect](https://openid.net/connect/) (OIDC) et [WS-Federation](https://auth0.com/docs/protocols/ws-fed) default pour la publication de redirections basées sur. Les redirections basées sur la publication déclenchent les protections du navigateur SameSite, donc SameSite est désactivé pour ces composants. La plupart des connexions [OAuth](https://oauth.net/) ne sont pas affectées en raison de différences de flux de demande.

Chaque composant ASP.NET Core qui émet des cookies doit décider si SameSite est approprié.

## <a name="samesite-test-sample-code"></a>Exemple de code de test SameSite

 ::: moniker range=">= aspnetcore-2.1 < aspnetcore-3.0"

Les exemples suivants peuvent être téléchargés et testés :

| Exemple               | Document |
| ----------------- | ------------ |
| [MVC .NET Core](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore21MVC)  | <xref:security/samesite/mvc21> |
| [Razor Pages .NET Core](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore21RazorPages)  | <xref:security/samesite/rp21> |

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

L’exemple suivant peut être téléchargé et testé :


| Exemple               | Document |
| ----------------- | ------------ |
| [Razor Pages .NET Core](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore31RazorPages)  | <xref:security/samesite/rp31> |

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

## <a name="net-core-support-for-the-samesite-attribute"></a>Prise en charge de .NET Core pour l’attribut sameSite

.NET Core 2,2 prend en charge 2019 Draft Standard pour SameSite depuis la publication des mises à jour en décembre 2019. Les développeurs sont en mesure de contrôler par programmation la valeur de l’attribut sameSite à l’aide de la propriété `HttpCookie.SameSite`. Si vous affectez à la propriété `SameSite` la valeur strict, LAX ou None, ces valeurs sont écrites sur le réseau avec le cookie. Si la valeur est égale à (SameSiteMode) (-1), aucun attribut sameSite ne doit être inclus sur le réseau avec le cookie

[!code-csharp[](samesite/snippets/Privacy.cshtml.cs?name=snippet)]

.NET Core 3,0 prend en charge les valeurs SameSite mises à jour et ajoute une valeur enum supplémentaire, `SameSiteMode.Unspecified` à l’énumération `SameSiteMode`.
Cette nouvelle valeur indique qu’aucun sameSite ne doit être envoyé avec le cookie.

::: moniker-end

::: moniker range="= aspnetcore-2.1"

## <a name="december-patch-behavior-changes"></a>Modifications du comportement du correctif décembre

Le changement de comportement spécifique pour .NET Framework et .NET Core 2,1 est la manière dont la propriété `SameSite` interprète la valeur `None`. Avant que le correctif ait une valeur de `None` signifiait « ne pas émettre l’attribut », après le correctif, cela signifie « émettre l’attribut avec une valeur de `None`». Une fois que le correctif a `SameSite` valeur de `(SameSiteMode)(-1)`, l’attribut n’est pas émis.

La valeur SameSite par défaut pour les cookies d’authentification par formulaire et d’état de session a été modifiée de `None` à `Lax`.

::: moniker-end

## <a name="api-usage-with-samesite"></a>Utilisation des API avec SameSite

[HttpContext. Response. cookies. Append](xref:Microsoft.AspNetCore.Http.IResponseCookies.Append*) est défini par défaut sur `Unspecified`, ce qui signifie qu’aucun attribut SameSite n’a été ajouté au cookie et que le client utilise son comportement par défaut (LAX pour les nouveaux navigateurs, None pour les anciens navigateurs). Le code suivant montre comment modifier la valeur du cookie SameSite pour `SameSiteMode.Lax`:

[!code-csharp[](samesite/sample/Pages/Index.cshtml.cs?name=snippet)]

Tous les composants ASP.NET Core qui émettent des cookies remplacent les valeurs par défaut précédentes par les paramètres appropriés pour leurs scénarios. Les valeurs par défaut substituées ne sont pas modifiées.

| Composant | cookie | Default |
| ------------- | ------------- |
| <xref:Microsoft.AspNetCore.Http.CookieBuilder> | <xref:Microsoft.AspNetCore.Http.CookieBuilder.SameSite> | `Unspecified` |
| <xref:Microsoft.AspNetCore.Http.HttpContext.Session>  | [SessionOptions. cookie](xref:Microsoft.AspNetCore.Builder.SessionOptions.Cookie) |`Lax` |
| <xref:Microsoft.AspNetCore.Mvc.ViewFeatures.CookieTempDataProvider>  | [CookieTempDataProviderOptions. cookie](xref:Microsoft.AspNetCore.Mvc.CookieTempDataProviderOptions.Cookie) | `Lax` |
| <xref:Microsoft.AspNetCore.Antiforgery.IAntiforgery> | [AntiforgeryOptions. cookie](xref:Microsoft.AspNetCore.Antiforgery.AntiforgeryOptions.Cookie)| `Strict` |
| [Authentification par cookie](xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*) | [CookieAuthenticationOptions. cookie](xref:Microsoft.AspNetCore.Builder.CookieAuthenticationOptions.CookieName) | `Lax` |
| <xref:Microsoft.Extensions.DependencyInjection.TwitterExtensions.AddTwitter*> | [TwitterOptions.StateCookie](xref:Microsoft.AspNetCore.Authentication.Twitter.TwitterOptions.StateCookie) | `Lax`  |
| <xref:Microsoft.AspNetCore.Authentication.RemoteAuthenticationHandler`1> | [RemoteAuthenticationOptions. CorrelationCookie](xref:Microsoft.AspNetCore.Authentication.RemoteAuthenticationOptions.CorrelationCookie)  | `None` |
| <xref:Microsoft.Extensions.DependencyInjection.OpenIdConnectExtensions.AddOpenIdConnect*> | [OpenIdConnectOptions.NonceCookie](xref:Microsoft.AspNetCore.Authentication.OpenIdConnect.OpenIdConnectOptions.NonceCookie)| `None` |
| [HttpContext. Response. cookies. Append](xref:Microsoft.AspNetCore.Http.IResponseCookies.Append*) | <xref:Microsoft.AspNetCore.Http.CookieOptions> | `Unspecified` |

::: moniker range=">= aspnetcore-3.1"

ASP.NET Core 3,1 et versions ultérieures fournissent la prise en charge SameSite suivante :

* Redéfinit le comportement de `SameSiteMode.None` pour émettre `SameSite=None`
* Ajoute une nouvelle valeur `SameSiteMode.Unspecified` pour omettre l’attribut SameSite.
* Par défaut, toutes les API de cookies `Unspecified`. Certains composants qui utilisent des cookies définissent des valeurs plus spécifiques à leurs scénarios. Consultez le tableau ci-dessus pour obtenir des exemples.

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

Dans ASP.NET Core 3,0 et versions ultérieures, les valeurs par défaut SameSite ont été modifiées pour éviter les conflits avec les paramètres par défaut incohérents du client. Les API suivantes ont modifié la valeur par défaut de `SameSiteMode.Lax ` à `-1` afin d’éviter l’émission d’un attribut SameSite pour ces cookies :

* <xref:Microsoft.AspNetCore.Http.CookieOptions> utilisé avec [HttpContext. Response. cookies. Append](xref:Microsoft.AspNetCore.Http.IResponseCookies.Append*)
* <xref:Microsoft.AspNetCore.Http.CookieBuilder> utilisé comme fabrique pour `CookieOptions`
* [CookiePolicyOptions.MinimumSameSitePolicy](xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions.MinimumSameSitePolicy)

::: moniker-end

## <a name="history-and-changes"></a>Historique et modifications

La prise en charge de SameSite a été implémentée pour la première fois dans ASP.NET Core dans 2,0 à l’aide de la [norme 2016](https://tools.ietf.org/html/draft-west-first-party-cookies-07#section-4.1). La norme 2016 était opt-in. ASP.NET Core choisi en définissant plusieurs cookies sur `Lax` par défaut. Après avoir rencontré plusieurs [problèmes](https://github.com/aspnet/Announcements/issues/318) avec l’authentification, la plupart des utilisations de SameSite ont été [désactivées](https://github.com/aspnet/Announcements/issues/348).

Les [correctifs](https://devblogs.microsoft.com/dotnet/net-core-November-2019/) ont été émis en novembre 2019 pour être mis à jour de la norme 2016 vers la norme 2019. Le [brouillon 2019 de la spécification SameSite](https://github.com/aspnet/Announcements/issues/390):

* N’est **pas** à compatibilité descendante avec le brouillon 2016. Pour plus d’informations, consultez [prise en charge des navigateurs plus anciens](#sob) dans ce document.
* Spécifie que les cookies sont traités comme `SameSite=Lax` par défaut.
* Spécifie les cookies qui déclarent explicitement `SameSite=None` afin d’activer la remise entre sites qui doit être marquée comme `Secure`. `None` est une nouvelle entrée à refuser.
* Est pris en charge par les correctifs émis pour ASP.NET Core 2,1, 2,2 et 3,0. ASP.NET Core 3,1 dispose d’une prise en charge supplémentaire de SameSite.
* Est planifié pour être activé par [chrome](https://chromestatus.com/feature/5088147346030592) par défaut au [2020 février](https://blog.chromium.org/2019/10/developers-get-ready-for-new.html). Les navigateurs ont commencé à passer à cette norme dans 2019.

## <a name="apis-impacted-by-the-change-from-the-2016-samesite-draft-standard-to-the-2019-draft-standard"></a>Les API affectées par la modification de la norme préliminaire 2016 SameSite à la norme de projet 2019

* [Http. SameSiteMode](xref:Microsoft.AspNetCore.Http.SameSiteMode)
* [CookieOptions. SameSite](xref:Microsoft.AspNetCore.Http.CookieOptions.SameSite)
* [CookieBuilder. SameSite](xref:Microsoft.AspNetCore.Http.CookieBuilder.SameSite)
* [CookiePolicyOptions.MinimumSameSitePolicy](xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions.MinimumSameSitePolicy)
* <xref:Microsoft.Net.Http.Headers.SameSiteMode?displayProperty=fullName>
* <xref:Microsoft.Net.Http.Headers.SetCookieHeaderValue.SameSite?displayProperty=fullName>

<a name="sob"></a>

## <a name="supporting-older-browsers"></a>Prise en charge des navigateurs plus anciens

La norme 2016 SameSite stipule que les valeurs inconnues doivent être traitées comme des valeurs `SameSite=Strict`. Les applications accessibles depuis des navigateurs plus anciens qui prennent en charge la norme 2016 SameSite peuvent s’arrêter lorsqu’ils obtiennent une propriété SameSite avec la valeur `None`. Les applications Web doivent implémenter la détection du navigateur si elles envisagent de prendre en charge des navigateurs plus anciens. ASP.NET Core n’implémente pas la détection du navigateur, car les valeurs des agents utilisateur sont très volatiles et changent fréquemment. Un point d’extension dans <xref:Microsoft.AspNetCore.CookiePolicy> permet de brancher une logique spécifique à l’agent utilisateur.

Dans `Startup.Configure`, ajoutez du code qui appelle <xref:Microsoft.AspNetCore.Builder.CookiePolicyAppBuilderExtensions.UseCookiePolicy*> avant d’appeler <xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication*> ou une méthode qui écrit *des* cookies :

[!code-csharp[](samesite/sample/Startup.cs?name=snippet5&highlight=18-19)]

Dans `Startup.ConfigureServices`, ajoutez du code similaire à ce qui suit :

::: moniker range="= aspnetcore-3.1"

[!code-csharp[](samesite/sample/Startup31.cs?name=snippet)]

::: moniker-end

::: moniker range="< aspnetcore-3.1"

[!code-csharp[](samesite/sample/Startup.cs?name=snippet)]

::: moniker-end

Dans l’exemple précédent, `MyUserAgentDetectionLib.DisallowsSameSiteNone` est une bibliothèque fournie par l’utilisateur, qui détecte si l’agent utilisateur ne prend pas en charge SameSite `None`:

[!code-csharp[](samesite/sample/Startup31.cs?name=snippet2)]

Le code suivant illustre un exemple de méthode `DisallowsSameSiteNone` :

> [!WARNING]
> Le code suivant est uniquement à des fins de démonstration :
> * Elle ne doit pas être considérée comme terminée.
> * Elle n’est pas gérée ni prise en charge.

[!code-csharp[](samesite/sample/Startup31.cs?name=snippetX)]

## <a name="test-apps-for-samesite-problems"></a>Tester des applications pour les problèmes SameSite

Les applications qui interagissent avec des sites distants, par exemple via une connexion tierce, doivent :

* Testez l’interaction sur plusieurs navigateurs.
* Appliquez la [détection et l’atténuation du navigateur CookiePolicy](#sob) présentées dans ce document.

Testez les applications Web à l’aide d’une version du client qui peut s’abonner au nouveau comportement SameSite. Chrome, Firefox et chrome Edge ont tous des indicateurs de fonctionnalités d’abonnement qui peuvent être utilisés à des fins de test. Une fois que votre application applique les correctifs SameSite, testez-la avec des versions client plus anciennes, en particulier Safari. Pour plus d’informations, consultez [prise en charge des navigateurs plus anciens](#sob) dans ce document.

### <a name="test-with-chrome"></a>Tester avec chrome

Chrome 78 + donne des résultats trompeurs, car une atténuation temporaire est en place. Le chrome 78 + atténuation temporaire autorise les cookies datant de moins de deux minutes. Le chrome 76 ou 77 avec les indicateurs de test appropriés activés fournit des résultats plus précis. Pour tester le nouveau comportement SameSite, basculez `chrome://flags/#same-site-by-default-cookies` sur **activé**. Les anciennes versions de chrome (75 et versions antérieures) sont signalées pour échouer avec le nouveau paramètre de `None`. Consultez [prise en charge des navigateurs plus anciens](#sob) dans ce document.

Google ne rend pas les versions de chrome plus anciennes disponibles. Suivez les instructions de [Télécharger chrome](https://www.chromium.org/getting-involved/download-chromium) pour tester les anciennes versions de chrome. Ne téléchargez **pas** chrome à partir des liens fournis en recherchant les anciennes versions de chrome.

* [Chrome 76 Win64](https://commondatastorage.googleapis.com/chromium-browser-snapshots/index.html?prefix=Win_x64/664998/)
* [Chrome 74 Win64](https://commondatastorage.googleapis.com/chromium-browser-snapshots/index.html?prefix=Win_x64/638880/)

À partir de la version `80.0.3975.0`de la version de la balise de service, les tests d’atténuation temporaire de type Lax + postal peuvent être désactivés à des fins de test à l’aide du nouvel indicateur `--enable-features=SameSiteDefaultChecksMethodRigorously` pour permettre le test des sites et des services dans l’état final éventuel de la fonctionnalité dans laquelle l’atténuation a été supprimée. Pour plus d’informations, consultez [mises à jour](https://www.chromium.org/updates/same-site) des projets de chrome SameSite

### <a name="test-with-safari"></a>Tester avec Safari

Safari 12 implémentait strictement le brouillon précédent et échoue lorsque la nouvelle valeur de `None` se trouve dans un cookie. `None` est évité par le biais du code de détection du navigateur qui [prend en charge les navigateurs plus anciens](#sob) dans ce document. Testez les connexions de style du système d’exploitation Safari 12, Safari 13 et WebKit à l’aide de MSAL, ADAL ou toute bibliothèque que vous utilisez. Le problème dépend de la version du système d’exploitation sous-jacent. OSX Mojave (10,14) et iOS 12 sont connus pour avoir des problèmes de compatibilité avec le nouveau comportement de SameSite. La mise à niveau du système d’exploitation vers OSX Catalina (10,15) ou iOS 13 résout le problème. Safari ne dispose pas actuellement d’un indicateur d’abonnement pour tester le nouveau comportement des spécifications.

### <a name="test-with-firefox"></a>Test avec Firefox

La prise en charge de Firefox pour la nouvelle norme peut être testée sur la version 68 + en choisissant dans la page `about:config` avec l’indicateur de fonctionnalité `network.cookie.sameSite.laxByDefault`. Il n’y a eu aucun rapport sur les problèmes de compatibilité avec les versions antérieures de Firefox.

### <a name="test-with-edge-browser"></a>Tester avec le navigateur Edge

Edge prend en charge l’ancien standard SameSite. Edge version 44 ne présente aucun problème de compatibilité connu avec la nouvelle norme.

### <a name="test-with-edge-chromium"></a>Test avec Edge (chrome)

Les indicateurs SameSite sont définis sur la page `edge://flags/#same-site-by-default-cookies`. Aucun problème de compatibilité n’a été découvert avec le chrome Edge.

### <a name="test-with-electron"></a>Test avec électron

Les versions d’électrons incluent des versions plus anciennes de chrome. Par exemple, la version de l’électron utilisée par les équipes est le chrome 66, qui présente le comportement plus ancien. Vous devez effectuer vos propres tests de compatibilité avec la version d’électron que votre produit utilise. Consultez [prise en charge des navigateurs plus anciens](#sob) dans la section suivante.

## <a name="additional-resources"></a>Ressources supplémentaires

* [Blog du chrome : développeurs : Préparez-vous à la nouvelle SameSite = None ; Sécuriser les paramètres de cookie](https://blog.chromium.org/2019/10/developers-get-ready-for-new.html)
* [Explication des cookies SameSite](https://web.dev/samesite-cookies-explained/)
* [Correctifs de novembre 2019](https://devblogs.microsoft.com/dotnet/net-core-November-2019/)

 ::: moniker range=">= aspnetcore-2.1 < aspnetcore-3.0"

| Exemple               | Document |
| ----------------- | ------------ |
| [MVC .NET Core](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore21MVC)  | <xref:security/samesite/mvc21> |
| [Razor Pages .NET Core](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore21RazorPages)  | <xref:security/samesite/rp21> |

::: moniker-end

 ::: moniker range=">= aspnetcore-3.0"

| Exemple               | Document |
| ----------------- | ------------ |
| [Razor Pages .NET Core](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore31RazorPages)  | <xref:security/samesite/rp31> |

::: moniker-end
