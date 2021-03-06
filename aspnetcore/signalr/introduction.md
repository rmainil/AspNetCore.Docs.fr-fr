---
title: Présentation de ASP.NET CoreSignalR
author: bradygaster
description: Découvrez comment la bibliothèque SignalR ASP.net Core simplifie l’ajout de fonctionnalités en temps réel aux applications.
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 11/27/2019
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: signalr/introduction
ms.openlocfilehash: 4e968659d23330c4b1403ab998dda9c9e6ebe080
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/04/2020
ms.locfileid: "82775776"
---
# <a name="introduction-to-aspnet-core-signalr"></a>Présentation de ASP.NET CoreSignalR

## <a name="what-is-signalr"></a>Qu’est SignalR-ce que c’est ?

ASP.NET Core SignalR est une bibliothèque open source qui simplifie l’ajout de fonctionnalités Web en temps réel aux applications. La fonctionnalité Web en temps réel permet au code côté serveur de transmettre instantanément du contenu aux clients.

Bons candidats pour SignalR:

* Les applications ayant besoin de mises à jour fréquentes auprès du serveur. Exemples : jeux, réseaux sociaux, scrutin, enchères, cartes et applications GPS.
* Les tableaux de bord et les applications de monitoring. Exemples : tableaux de bord des entreprises, mises à jour instantanées des ventes et alertes de voyage.
* Les applications de collaboration. Exemples : applications de tableau blanc et logiciels de réunion d’équipe.
* Les applications qui envoient des notifications. Exemples : réseaux sociaux, messagerie, conversation instantanée, jeux, alertes de voyage, etc.

SignalRfournit une API permettant de créer des [appels de procédure distante (RPC)](https://wikipedia.org/wiki/Remote_procedure_call)de serveur à client. Les RPC appellent des fonctions JavaScript sur les clients à partir du code .NET Core côté serveur.

Voici quelques-unes des SignalR fonctionnalités de pour ASP.net Core :

* Gère automatiquement la gestion des connexions.
* Envoie des messages à tous les clients connectés simultanément. Par exemple, une salle de conversation.
* Envoie des messages à des clients ou groupes de clients spécifiques.
* Met à l’échelle pour gérer le trafic qui augmente.

La source est hébergée dans un [ SignalR référentiel sur GitHub](https://github.com/dotnet/AspNetCore/tree/master/src/SignalR).

## <a name="transports"></a>Transports

SignalRprend en charge les techniques suivantes pour gérer les communications en temps réel (par ordre de secours normal) :

* [WebSockets](https://tools.ietf.org/html/rfc7118)
* Événements envoyés par le serveur
* Interrogation longue

SignalRchoisit automatiquement la meilleure méthode de transport parmi les capacités du serveur et du client.

## <a name="hubs"></a>Hubs

SignalRutilise des *concentrateurs* pour la communication entre les clients et les serveurs.

Un Hub est un pipeline de haut niveau qui permet à un client et un serveur d’appeler des méthodes les unes sur les autres. SignalRgère automatiquement la distribution à travers les limites de l’ordinateur, ce qui permet aux clients d’appeler des méthodes sur le serveur et vice versa. Vous pouvez passer des paramètres fortement typés à des méthodes, ce qui active la liaison de modèle. SignalRfournit deux protocoles Hub intégrés : un protocole texte basé sur JSON et un protocole binaire basé sur [MessagePack](https://msgpack.org/).  MessagePack crée généralement des messages plus petits par rapport à JSON. Les anciens navigateurs doivent prendre en charge le [niveau 2 de XHR](https://caniuse.com/#feat=xhr2) pour fournir la prise en charge du protocole MessagePack.

Les hubs appellent le code côté client en envoyant des messages qui contiennent le nom et les paramètres de la méthode côté client. Les objets envoyés en tant que paramètres de méthode sont désérialisés à l’aide du protocole configuré. Le client tente de faire correspondre le nom à une méthode dans le code côté client. Lorsque le client trouve une correspondance, il appelle la méthode et lui passe les données de paramètre désérialisées.

## <a name="additional-resources"></a>Ressources supplémentaires

* [Prise en SignalR main de pour ASP.net Core](xref:tutorials/signalr)
* [Plateformes prises en charge](xref:signalr/supported-platforms)
* [Hubs](xref:signalr/hubs)
* [Client JavaScript](xref:signalr/javascript-client)
