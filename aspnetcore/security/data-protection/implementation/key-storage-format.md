---
title: Format de stockage des clés dans ASP.NET Core
author: rick-anderson
description: Découvrez les détails de l’implémentation du format de stockage de la clé de protection des données ASP.NET Core.
ms.author: riande
ms.date: 04/08/2020
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: security/data-protection/implementation/key-storage-format
ms.openlocfilehash: d284927e8ff4315b813fe36b9c335d8bd75ece11
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/04/2020
ms.locfileid: "82776862"
---
# <a name="key-storage-format-in-aspnet-core"></a>Format de stockage des clés dans ASP.NET Core

<a name="data-protection-implementation-key-storage-format"></a>

Les objets sont stockés au repos dans une représentation XML. Le répertoire par défaut pour le stockage des clés est le suivant :

* Windows : *%LOCALAPPDATA%\ASP.NET\DataProtection-Keys\*
* macOS/Linux : *$Home/.AspNet/dataprotection-Keys*

## <a name="the-key-element"></a>Élément \<Key>

Les clés existent en tant qu’objets de niveau supérieur dans le dépôt de clé. Les clés de Convention ont la clé de nom de fichier **-{GUID}. xml**, où {GUID} est l’ID de la clé. Chaque fichier de ce type contient une clé unique. Le format du fichier est le suivant.

```xml
<?xml version="1.0" encoding="utf-8"?>
<key id="80732141-ec8f-4b80-af9c-c4d2d1ff8901" version="1">
  <creationDate>2015-03-19T23:32:02.3949887Z</creationDate>
  <activationDate>2015-03-19T23:32:02.3839429Z</activationDate>
  <expirationDate>2015-06-17T23:32:02.3839429Z</expirationDate>
  <descriptor deserializerType="{deserializerType}">
    <descriptor>
      <encryption algorithm="AES_256_CBC" />
      <validation algorithm="HMACSHA256" />
      <enc:encryptedSecret decryptorType="{decryptorType}" xmlns:enc="...">
        <encryptedKey>
          <!-- This key is encrypted with Windows DPAPI. -->
          <value>AQAAANCM...8/zeP8lcwAg==</value>
        </encryptedKey>
      </enc:encryptedSecret>
    </descriptor>
  </descriptor>
</key>
```

L' \<élément Key> contient les attributs et les éléments enfants suivants :

* ID de la clé. Cette valeur est traitée comme faisant autorité ; le nom de fichier est simplement un nicety pour la lisibilité humaine.

* Version de la \<clé> élément, actuellement fixée à 1.

* Dates de création, d’activation et d’expiration de la clé.

* Un \<élément descripteur>, qui contient des informations sur l’implémentation de chiffrement authentifié contenue dans cette clé.

Dans l’exemple ci-dessus, l’ID de la clé est {80732141-ec8f-4B80-af9c-c4d2d1ff8901}, il a été créé et activé le 19 mars 2015, et sa durée de vie est de 90 jours. (Parfois, la date d’activation peut être légèrement antérieure à la date de création, comme dans cet exemple. Cela est dû à un Nil dans la manière dont les API fonctionnent et est sans conséquence dans la pratique.)

## <a name="the-descriptor-element"></a>Élément \<descripteur>

L’élément \<de descripteur externe> contient un attribut deserializerType, qui est le nom qualifié d’assembly d’un type qui implémente IAuthenticatedEncryptorDescriptorDeserializer. Ce type est chargé de lire le descripteur interne \<> élément et d’analyser les informations contenues dans.

Le format particulier du \<descripteur> élément dépend de l’implémentation de chiffreur authentifiée encapsulée par la clé, et chaque type de désérialiseur attend un format légèrement différent pour ce. En général, cependant, cet élément contient des informations algorithmiques (noms, types, OID ou similaires) et un matériel de clé secrète. Dans l’exemple ci-dessus, le descripteur spécifie que cette clé encapsule AES-256-CBC Encryption + HMACSHA256 validation.

## <a name="the-encryptedsecret-element"></a>Élément \<encryptedSecret>

Un ** &lt;élément&gt; encryptedSecret** qui contient la forme chiffrée du matériel de clé secrète peut être présent si le [chiffrement des secrets au repos est activé](xref:security/data-protection/implementation/key-encryption-at-rest). L’attribut `decryptorType` est le nom qualifié d’assembly d’un type qui implémente [IXmlDecryptor](/dotnet/api/microsoft.aspnetcore.dataprotection.xmlencryption.ixmldecryptor). Ce type est chargé de lire l’élément ** &lt;EncryptedKey&gt; ** interne et de le déchiffrer pour récupérer le texte en clair d’origine.

Comme avec `<descriptor>`, le format particulier de l' `<encryptedSecret>` élément dépend du mécanisme de chiffrement au repos en cours d’utilisation. Dans l’exemple ci-dessus, la clé principale est chiffrée à l’aide de Windows DPAPI pour le commentaire.

## <a name="the-revocation-element"></a>Élément \<de> de révocation

Les révocations existent en tant qu’objets de niveau supérieur dans le dépôt de clé. Par Convention, les révocations de nom de fichier **-{timestamp}. xml** (pour révoquer toutes les clés avant une date spécifique) ou la **révocation-{GUID}. xml** (pour révoquer une clé spécifique). Chaque fichier contient un seul \<> élément de révocation.

Pour les révocations de clés individuelles, le contenu du fichier se présente comme indiqué ci-dessous.

```xml
<?xml version="1.0" encoding="utf-8"?>
<revocation version="1">
  <revocationDate>2015-03-20T22:45:30.2616742Z</revocationDate>
  <key id="eb4fc299-8808-409d-8a34-23fc83d026c9" />
  <reason>human-readable reason</reason>
</revocation>
```

Dans ce cas, seule la clé spécifiée est révoquée. Toutefois, si l’ID de clé est « * », comme dans l’exemple ci-dessous, toutes les clés dont la date de création est antérieure à la date de révocation spécifiée sont révoquées.

```xml
<?xml version="1.0" encoding="utf-8"?>
<revocation version="1">
  <revocationDate>2015-03-20T15:45:45.7366491-07:00</revocationDate>
  <!-- All keys created before the revocation date are revoked. -->
  <key id="*" />
  <reason>human-readable reason</reason>
</revocation>
```

\<Raison pour laquelle> élément n’est jamais lu par le système. Il s’agit simplement d’un emplacement pratique pour stocker une raison explicite de révocation.
