---
title: « API HTTP [!DNL Asset Compute Service] »
description: « API HTTP [!DNL Asset Compute Service] pour créer des applications personnalisées. »
exl-id: 4b63fdf9-9c0d-4af7-839d-a95e07509750
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '2862'
ht-degree: 100%

---

# API HTTP [!DNL Asset Compute Service] {#asset-compute-http-api}

L’utilisation de l’API est limitée à des fins de développement. L’API est fournie à titre de contexte pour le développement d’applications personnalisées. [!DNL Adobe Experience Manager] as a [!DNL Cloud Service] utilise l’API pour transmettre les informations de traitement à une application personnalisée. Pour plus d’informations, voir [Utilisation des microservices de ressources et des profils de traitement](https://experienceleague.adobe.com/fr/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

>[!NOTE]
>
>[!DNL Asset Compute Service] est disponible uniquement pour une utilisation avec [!DNL Experience Manager] as a [!DNL Cloud Service].

Toute personne cliente de l’API HTTP [!DNL Asset Compute Service] doit appliquer ce flux de haut niveau :

1. Une personne cliente est configurée en tant que projet [!DNL Adobe Developer Console] dans une organisation IMS. Chaque personne cliente (système ou environnement) distincte a besoin de son propre projet pour séparer le flux de données d’événement.

1. Un client génère un jeton d’accès pour le compte technique à l’aide de l’[authentification JWT (compte de service)](https://developer.adobe.com/developer-console/docs/guides/).

1. Un client n’appelle [`/register`](#register) qu’une seule fois pour récupérer l’URL du journal.

1. Un client appelle [`/process`](#process-request) pour chaque ressource pour laquelle il souhaite générer des rendus. L’appel est asynchrone.

1. Un client interroge régulièrement le journal pour [recevoir des événements](#asynchronous-events). Il reçoit des événements pour chaque rendu demandé et traité (type d’événement `rendition_created`) ou en cas d’erreur (type d’événement `rendition_failed`).

Le module [adobe-asset-compute-client](https://github.com/adobe/asset-compute-client) facilite l’utilisation de l’API dans le code Node.js.

## Authentification et autorisation {#authentication-and-authorization}

Toutes les API nécessitent une authentification par jeton d’accès. Les requêtes doivent définir les en-têtes suivants :

1. En-tête `Authorization` avec jeton support, qui est le jeton de compte technique, reçu par le biais d’un [échange JWT](https://developer.adobe.com/developer-console/docs/guides/) issu du projet Adobe Developer Console. Les [portées](#scopes) sont décrites ci-dessous.

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in Adobe I/O's GitHub repo to get a new URL.
-->

1. En-tête `x-gw-ims-org-id` avec l’ID d’organisation IMS.

1. `x-api-key` avec l’ID client du projet [!DNL Adobe Developers Console].

### Portées {#scopes}

Vérifiez les portées suivantes pour le jeton d’accès :

* `openid`
* `AdobeID`
* `asset_compute`
* `read_organizations`
* `event_receiver`
* `event_receiver_api`
* `adobeio_api`
* `additional_info.roles`
* `additional_info.projectedProductContext`

Ces portées nécessitent que le projet [!DNL Adobe Developer Console] souscrive aux services `Asset Compute`, `I/O Events` et `I/O Management API`. La répartition des portées individuelles est la suivante :

* De base
   * Portées : `openid,AdobeID`

* Asset Compute
   * Metascope : `asset_compute_meta`
   * Portées : `asset_compute,read_organizations`

* Adobe [!DNL `I/O Events`]
   * Metascope : `event_receiver_api`
   * Portées : `event_receiver,event_receiver_api`

* Adobe [!DNL `I/O Management API`]
   * Metascope : `ent_adobeio_sdk`
   * Portées : `adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## Enregistrement {#register}

Chaque personne cliente d’[!DNL Asset Compute service] (un projet [!DNL Adobe Developer Console] unique, abonné au service) doit s’[enregistrer](#register-request) avant de procéder à des demandes de traitement. L’étape d’enregistrement renvoie le journal unique des événements, nécessaire pour récupérer les événements asynchrones issus du traitement du rendu.

À la fin de son cycle de vie, une personne cliente peut [annuler son enregistrement](#unregister-request).

### Demande d’enregistrement {#register-request}

Cet appel d’API configure une personne cliente [!DNL Asset Compute] et fournit l’URL du journal des événements. Ce processus est une opération idempotente (c’est-à-dire ayant le même effet si elle est appliquée une ou plusieurs fois), qui ne doit être appelée qu’une seule fois pour chaque personne cliente. Elle peut être de nouveau appelée pour récupérer l’URL du journal.

| Paramètre | Valeur |
|--------------------------|------------------------------------------------------|
| Méthode | `POST` |
| Chemin | `/register` |
| En-tête `Authorization` | Tous les [en-têtes relatifs aux autorisations](#authentication-and-authorization). |
| En-tête `x-request-id` | Facultatif. Il peut être défini par les clientes et clients pour un identifiant unique de bout en bout des demandes de traitement sur l’ensemble des systèmes. |
| Corps de la requête | Doit être vide. |

### Réponse à l’enregistrement {#register-response}

| Paramètre | Valeur |
|-----------------------|------------------------------------------------------|
| Type MIME | `application/json` |
| En-tête `X-Request-Id` | Identique à l’en-tête de la requête `X-Request-Id` ou à celui généré de manière unique. Sert à identifier les requêtes entre systèmes et/ou les demandes d’assistance. |
| Corps de la réponse | Objet JSON contenant les champs `journal`, `ok` ou `requestId`. |

Les codes d’état HTTP sont les suivants :

* **200 Success** : lorsque la requête est acceptée. L’URL `journal` reçoit des notifications sur les résultats du traitement asynchrone initié par le biais de `/process`. Elle signale les événements `rendition_created` une fois l’opération terminée, ou les événements `rendition_failed` en cas d’échec du processus.

  ```json
  {
      "ok": true,
      "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "requestId": "1234567890"
  }
  ```

* **401 Unauthorized** : survient lorsque la requête ne dispose pas d’une [authentification](#authentication-and-authorization) valide. Il peut s’agir, par exemple, d’un jeton d’accès ou d’une clé d’API non valide.

* **403 Forbidden** : survient lorsque la demande ne dispose pas d’une [autorisation](#authentication-and-authorization) valide. Par exemple, si le jeton d’accès est valide, mais que le projet Adobe Developer Console (compte technique) n’est pas abonné à tous les services requis.

* **429 Trop de requêtes** : survient lorsque ce client ou cette cliente, ou une autre personne, surcharge le système. Les clients doivent effectuer un nouvel essai avec un [backoff exponentiel](https://fr.wikipedia.org/wiki/Binary_exponential_backoff) (pour diminuer la fréquence du processus). Le corps est vide.
* **Error 4xx** : survient en cas d’erreur client, quelle qu’elle soit, et d’échec de l’enregistrement. Généralement, une réponse JSON de ce type est renvoyée, même si ce n’est pas garanti pour toutes les erreurs :

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **Error 5xx** : survient en cas d’erreur côté serveur, quelle qu’elle soit, et d’échec de l’enregistrement. Généralement, une réponse JSON de ce type est renvoyée, même si ce n’est pas garanti pour toutes les erreurs :

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

### Requête d’annulation d’enregistrement {#unregister-request}

Cet appel d’API annule l’enregistrement d’une personne cliente [!DNL Asset Compute]. Après l’annulation de cet enregistrement, il n’est plus possible d’appeler `/process`. L’utilisation de l’appel d’API, pour un client non enregistré ou un client à enregistrer, renvoie une erreur `404`.

| Paramètre | Valeur |
|--------------------------|------------------------------------------------------|
| Méthode | `POST` |
| Chemin | `/unregister` |
| En-tête `Authorization` | Tous les [en-têtes relatifs aux autorisations](#authentication-and-authorization). |
| En-tête `x-request-id` | Facultatif. Il peut être défini par les clientes et les clients pour un identifiant unique de bout en bout des demandes de traitement sur l’ensemble des systèmes. |
| Corps de la requête | Vide. |

### Réponse à l’annulation de l’enregistrement {#unregister-response}

| Paramètre | Valeur |
|-----------------------|------------------------------------------------------|
| Type MIME | `application/json` |
| En-tête `X-Request-Id` | Identique à l’en-tête de la requête `X-Request-Id` ou à celui généré de manière unique. Sert à identifier les requêtes entre systèmes ou les demandes d’assistance. |
| Corps de la réponse | Objet JSON contenant les champs `ok` et `requestId`. |

Les codes d’état sont les suivants :

* **200 Success** : survient lorsque l’enregistrement et le journal sont trouvés et supprimés.

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **401 Unauthorized** : survient lorsque la requête ne dispose pas d’une [authentification](#authentication-and-authorization) valide. Il peut s’agir, par exemple, d’un jeton d’accès ou d’une clé d’API non valide.

* **403 Forbidden** : survient lorsque la demande ne dispose pas d’une [autorisation](#authentication-and-authorization) valide. Par exemple, si le jeton d’accès est valide, mais que le projet Adobe Developer Console (compte technique) n’est pas abonné à tous les services requis.

* **404 Introuvable** : ce statut s’affiche lorsque les informations d’identification fournies ne sont pas enregistrées ou ne sont pas valides.

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **429 Too many requests** : survient lorsque le système est surchargé. Les clients doivent effectuer un nouvel essai avec un [backoff exponentiel](https://fr.wikipedia.org/wiki/Binary_exponential_backoff) (pour diminuer la fréquence du processus). Le corps est vide.

* **Error 4xx** : survient lorsqu’une autre erreur client s’est produite et que l’annulation de l’enregistrement a échoué. Généralement, une réponse JSON de ce type est renvoyée, même si ce n’est pas garanti pour toutes les erreurs :

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **Error 5xx** : survient en cas d’erreur côté serveur, quelle qu’elle soit, et d’échec de l’enregistrement. Généralement, une réponse JSON de ce type est renvoyée, même si ce n’est pas garanti pour toutes les erreurs :

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

## Requête de traitement {#process-request}

L’opération `process` lance une tâche chargée de transformer une ressource source en plusieurs rendus, en fonction des instructions contenues dans la requête. Des notifications de succès de l’exécution (type d’événement `rendition_created`) ou d’erreur (type d’événement `rendition_failed`) sont envoyées à un journal des événements. Celui-ci doit être récupéré en utilisant une fois [`/register`](#register) avant de lancer un nombre quelconque de requêtes `/process`. Les requêtes incorrectement formulées échouent immédiatement avec un code d’erreur 400.

Les fichiers binaires sont référencés à l’aide d’URL, telles que les URL présignées Amazon AWS S3 ou les URL de stockage Blob Azure SAS. Utilisé pour lire la ressource `source` (URL `GET`) et écrire les rendus (URL `PUT`). Il appartient au client de générer ces URL présignées.

| Paramètre | Valeur |
|--------------------------|------------------------------------------------------|
| Méthode | `POST` |
| Chemin | `/process` |
| Type MIME | `application/json` |
| En-tête `Authorization` | Tous les [en-têtes relatifs aux autorisations](#authentication-and-authorization). |
| En-tête `x-request-id` | Facultatif. Les clientes et clients peuvent définir un identifiant unique de bout en bout pour effectuer le suivi des requêtes de traitement sur l’ensemble des systèmes. |
| Corps de la requête | Doit être au format JSON de requête de traitement comme décrit ci-dessous. Il fournit des instructions sur la ressource à traiter et les rendus à générer. |

### Requête de traitement JSON {#process-request-json}

Le corps de la requête de `/process` est un objet JSON dont le schéma est le suivant :

```json
{
    "source": "",
    "renditions" : []
}
```

Les champs disponibles sont les suivants :

| Nom | Type | Description | Exemple |
|--------------|----------|-------------|---------|
| `source` | `string` | URL de la ressource source qui est traitée. Facultatif, en fonction du format de rendu demandé (par exemple, `fmt=zip`). | `"http://example.com/image.jpg"` |
| `source` | `object` | Description de la ressource source qui est traitée. Voir la description des [champs d’objet source](#source-object-fields) ci-dessous. Facultatif, en fonction du format de rendu demandé (par exemple, `fmt=zip`). | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | Rendus à générer à partir du fichier source. Chaque objet de rendu prend en charge une [instruction de rendu](#rendition-instructions). Requis. | `[{ "target": "https://....", "fmt": "png" }]` |

La `source` peut être soit une chaîne `<string>`, considérée comme une URL, ou un objet (`<object>`) avec un champ supplémentaire. Les variantes suivantes sont similaires :

```json
"source": "http://example.com/image.jpg"
```

```json
"source": {
    "url": "http://example.com/image.jpg"
}
```

### Champs d’objet source {#source-object-fields}

| Nom | Type | Description | Exemple |
|-----------|----------|-------------|---------|
| `url` | `string` | URL de la ressource source à traiter. Requis. | `"http://example.com/image.jpg"` |
| `name` | `string` | Nom du fichier de ressource source. L’extension de fichier du nom peut être utilisée si aucun type MIME ne peut être détecté. Elle a la priorité sur le nom de fichier spécifié dans le chemin d’accès à l’URL. Et elle a la priorité sur le nom de fichier dans l’en-tête `content-disposition` de la ressource binaire. La valeur par défaut est « file ». | `"image.jpg"` |
| `size` | `number` | Taille du fichier source en octets. Prévaut sur l’en-tête `content-length` de la ressource binaire. | `10234` |
| `mimetype` | `string` | Type MIME du fichier de ressource source. Prévaut sur l’en-tête `content-type` de la ressource binaire. | `"image/jpeg"` |

### Exemple de requête `process` complète {#complete-process-request-example}

```json
{
    "source": "https://www.adobe.com/content/dam/acom/en/lobby/lobby-bg-bts2017-logged-out-1440x860.jpg",
    "renditions" : [{
            "name": "image.48x48.png",
            "target": "https://some-presigned-put-url-for-image.48x48.png",
            "fmt": "png",
            "width": 48,
            "height": 48
        },{
            "name": "image.200x200.jpg",
            "target": "https://some-presigned-put-url-for-image.200x200.jpg",
            "fmt": "jpg",
            "width": 200,
            "height": 200
        },{
            "name": "cqdam.xmp.xml",
            "target": "https://some-presigned-put-url-for-cqdam.xmp.xml",
            "fmt": "xmp"
        },{
            "name": "cqdam.text.txt",
            "target": "https://some-presigned-put-url-for-cqdam.text.txt",
            "fmt": "text"
    }]
}
```

## Réponse du traitement {#process-response}

La requête `/process` renvoie immédiatement un succès ou un échec en fonction de la validation de la requête de base. Le traitement des ressources se produit de manière asynchrone.

| Paramètre | Valeur |
|-----------------------|------------------------------------------------------|
| Type MIME | `application/json` |
| En-tête `X-Request-Id` | Identique à l’en-tête de la requête `X-Request-Id` ou à celui généré de manière unique. Sert à identifier les requêtes entre systèmes ou les demandes d’assistance. |
| Corps de la réponse | Objet JSON contenant les champs `ok` et `requestId`. |

Codes d’état :

* **200 Success** : si la demande a été envoyée avec succès. La réponse JSON comprend `"ok": true` :

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **400 Requête non valide** : si la requête n’est pas correctement structurée, par exemple s’il manque les champs requis dans le payload JSON. La réponse JSON comprend `"ok": false` :

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **401 Unauthorized** : lorsque la requête ne dispose pas d’une [authentification](#authentication-and-authorization) valide. Il peut s’agir, par exemple, d’un jeton d’accès ou d’une clé d’API non valide.
* **403 Forbidden** : lorsque la demande ne dispose pas d’une [autorisation](#authentication-and-authorization) valide. Par exemple, si le jeton d’accès est valide, mais que le projet Adobe Developer Console (compte technique) n’est pas abonné à tous les services requis.
* **429 Trop de requêtes** : survient lorsque le système est dépassé, soit en raison de ce client ou de cette cliente spécifique, soit en raison de la demande globale. Les clients peuvent effectuer un nouvel essai avec un [backoff exponentiel](https://fr.wikipedia.org/wiki/Binary_exponential_backoff) (pour diminuer la fréquence du processus). Le corps est vide.
* **Error 4xx** : en cas d’erreur client, quelle qu’elle soit. Généralement, une réponse JSON de ce type est renvoyée, même si ce n’est pas garanti pour toutes les erreurs :

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **Error 5xx** : survient en cas d’erreur côté serveur, quelle qu’elle soit. Généralement, une réponse JSON de ce type est renvoyée, même si ce n’est pas garanti pour toutes les erreurs :

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

La plupart des clientes et des clients sont susceptibles de réessayer la même requête avec un [backoff exponentiel](https://fr.wikipedia.org/wiki/Binary_exponential_backoff) suite à une erreur, *à l’exception* des problèmes de configuration, comme 401 ou 403, ou des requêtes non valides comme 400. Outre la limitation du débit normal par le biais de réponses 429, une interruption ou une limitation de service temporaire peut entraîner des erreurs 5xx. Il est dans ce cas conseillé de réessayer après un certain temps.

Toutes les réponses JSON (le cas échéant) incluent la valeur `requestId`, identique à celle de l’en-tête `X-Request-Id`. Adobe recommande la lecture à partir de l’en-tête, car il est toujours présent. La valeur `requestId` est également renvoyée dans tous les événements relatifs aux requêtes de traitement sous la forme `requestId`. Les clientes et clients ne doivent pas présumer du format de cette chaîne. Il s’agit d’un identifiant de chaîne opaque.

## Souscrire au post-traitement {#opt-in-to-post-processing}

Le [SDK Asset Compute](https://github.com/adobe/asset-compute-sdk) prend en charge un ensemble d’options de base pour le post-traitement d’images. Les programmes de travail personnalisés peuvent explicitement souscrire au post-traitement en définissant le champ `postProcess` de l’objet de rendu sur `true`.

Les cas d’utilisation pris en charge sont les suivants :

* Le recadrage est un rendu d’un rectangle dont les limites sont définies par crop.w, crop.h, crop.x et crop.y. Les détails du recadrage sont spécifiés dans le champ `instructions.crop` de l’objet du rendu.
* Redimensionner les images en utilisant la largeur, la hauteur ou les deux. `instructions.width` et `instructions.height` les définissent dans l’objet de rendu. Pour redimensionner une image en utilisant uniquement la largeur ou la hauteur, définissez une seule valeur. Compute Service conserve les proportions.
* Définir la qualité d’une image JPEG. `instructions.quality` la définit dans l’objet de rendu. Un niveau de qualité de 100 représente la qualité la plus élevée, tandis qu’un nombre plus faible représente une baisse de qualité.
* Créer des images entrelacées. `instructions.interlace` les définit dans l’objet de rendu.
* Définissez la résolution en PPP pour ajuster la taille des rendus à des fins de publication sur ordinateur en ajustant l’échelle appliquée aux pixels. `instructions.dpi` les définit dans l’objet de rendu pour modifier la résolution en PPP. Cependant, pour redimensionner l’image afin qu’elle ait la même taille, mais avec une résolution différente, utilisez les instructions `convertToDpi`.
* Redimensionner l’image pour que sa largeur ou sa hauteur de rendu reste identique à celle de l’image d’origine avec la résolution cible (ppp) spécifiée. `instructions.convertToDpi` les définit dans l’objet de rendu.

## Mise en filigrane de ressources {#add-watermark}

Le [SDK Asset Compute](https://github.com/adobe/asset-compute-sdk) prend en charge l’ajout d’un filigrane aux fichiers d’image PNG, JPEG, TIFF et GIF. Le filigrane est ajouté à la suite des instructions de rendu dans l’objet `watermark`.

Le filigrane est appliqué pendant le post-traitement du rendu. Pour les ressources de filigrane, le programme de travail personnalisé [opte pour le post-traitement](#opt-in-to-post-processing) en définissant le champ `postProcess` de l’objet de rendu sur `true`. Si l’utilisateur ou l’utilisatrice ne souscrit pas, le filigrane n’est pas appliqué, même si, dans la requête, l’objet de filigrane est défini sur l’objet de rendu.

## Instructions de rendu {#rendition-instructions}

Les options suivantes sont disponibles pour le tableau `renditions` dans [`/process`](#process-request).

### Champs communs {#common-fields}

| Nom | Type | Description | Exemple |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | Le format de cible des rendus peut également être `text` pour l’extraction de texte et `xmp` pour l’extraction de métadonnées XMP au format XML. Voir [Formats pris en charge](https://experienceleague.adobe.com/fr/docs/experience-manager-cloud-service/content/assets/file-format-support) | `png` |
| `worker` | `string` | URL d’une [application personnalisée](develop-custom-application.md). Doit être une URL `https://`. Si ce champ est présent, une application personnalisée crée le rendu. Tout autre champ de rendu défini est ensuite utilisé dans l’application personnalisée. | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | URL vers laquelle le rendu généré doit être chargé à l’aide de la méthode HTTP PUT. | `http://w.com/img.jpg` |
| `target` | `object` | Informations de chargement d’URL présignée en plusieurs parties pour le rendu généré. Ces informations sont pour un [chargement binaire direct AEM/Oak](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) avec ce [comportement de chargement en plusieurs parties](https://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html).<br>Champs :<ul><li>`urls` : tableau de chaînes, une pour chaque URL de partie pré-signée</li><li>`minPartSize` : taille minimale à utiliser pour une partie = url</li><li>`maxPartSize` : taille maximale à utiliser pour une partie = url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | Facultatif. Le client ou la cliente contrôle l’espace réservé et le transmet tel quel aux événements de rendu. Permet aux clientes et clients d’ajouter des informations personnalisées pour identifier les événements de rendu. Cela ne doit pas être modifié ni utilisé dans les applications personnalisées, car les clientes et les clients sont libres de modifier à tout moment ce paramètre. | `{ ... }` |

### Champs spécifiques au rendu {#rendition-specific-fields}

Pour obtenir la liste des formats de fichiers actuellement pris en charge, voir [Formats de fichiers pris en charge](https://experienceleague.adobe.com/fr/docs/experience-manager-cloud-service/content/assets/file-format-support).

| Nom | Type | Description | Exemple |
|-------------------|----------|-------------|---------|
| `*` | `*` | Il est possible d’ajouter des champs personnalisés avancés qu’une [application personnalisée](develop-custom-application.md) peut comprendre. | |
| `embedBinaryLimit` | `number` en octets | Lorsque la taille de fichier du rendu est inférieure à la valeur spécifiée, elle est incluse dans l’événement envoyé une fois sa création terminée. La taille maximale autorisée pour l’incorporation est de 32 Ko (32 x 1 024 octets). Si un rendu a une taille supérieure à la limite `embedBinaryLimit`, il est enregistré dans un espace de stockage dans le cloud et n’est pas incorporé à l’événement. | `3276` |
| `width` | `number` | Largeur en pixels. uniquement pour les rendus d’image. | `200` |
| `height` | `number` | Hauteur en pixels. uniquement pour les rendus d’image. | `200` |
|                   |          | Le format est toujours conservé dans les cas suivants : <ul> <li> Les valeurs `width` et `height` sont spécifiées. L’image est alors adaptée à la taille tout en conservant les proportions. </li><li> Si seule la valeur `width` ou `height` est spécifiée, l’image obtenue utilise la dimension correspondante tout en conservant le format.</li><li> Si la valeur `width` ou `height` n’est spécifiée, la taille en pixels de l’image d’origine est utilisée. Cela dépend du type de source. Pour certains formats, comme les fichiers PDF, une taille par défaut est utilisée. Il peut y avoir une limite de taille maximale.</li></ul> | |
| `quality` | `number` | Spécifier la qualité jpeg dans une plage comprise entre `1` et `100`. Applicable uniquement aux rendus d’image. | `90` |
| `xmp` | `string` | Utilisé uniquement par l’écriture différée des métadonnées XMP, il applique le format XMP avec codage base64 pour écrire dans le rendu spécifié. | |
| `interlace` | `bool` | Créer un fichier PNG entrelacé, GIF ou JPEG progressif en fixant sa valeur à `true`. N’a aucun effet sur les autres formats de fichiers. | |
| `jpegSize` | `number` | Taille approximative du fichier JPEG en octets. Il remplace tout paramètre `quality`. Cela n’a aucun effet sur les autres formats. | |
| `dpi` | `number` ou `object` | Définir la résolution en ppp pour x et y. Pour plus de simplicité, il est également possible de définir un nombre unique appliqué à la fois à x et y. Ce paramétrage n’a aucun effet sur l’image elle-même. | `96` ou `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` ou `object` | Les résolutions en ppp pour x et y rééchantillonnent les valeurs tout en conservant la taille physique. Pour plus de simplicité, il est également possible de définir un nombre unique appliqué à la fois à x et y. | `96` ou `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | Liste des fichiers à inclure dans l’archive ZIP (`fmt=zip`). Chaque entrée peut être soit une chaîne URL, soit un objet avec les champs :<ul><li>`url` : URL de téléchargement du fichier</li><li>`path` : stockage du fichier selon ce chemin d’accès dans le fichier ZIP</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | Gestion des doublons pour les archives ZIP (`fmt=zip`). Par défaut, une erreur est générée si plusieurs fichiers de l’archive ZIP sont stockés sous le même chemin d’accès. Si vous définissez la valeur de `duplicate` sur `ignore`, seule la première ressource sera stockée et le reste sera ignoré. | `ignore` |
| `watermark` | `object` | Contient des instructions à propos du [filigrane](#watermark-specific-fields). |  |

### Champs spécifiques aux filigranes {#watermark-specific-fields}

Le format PNG est utilisé comme filigrane.

| Nom | Type | Description | Exemple |
|-------------------|----------|-------------|---------|
| `scale` | `number` | Échelle du filigrane, entre `0.0` et `1.0`. `1.0` signifie que le filigrane est à son échelle d’origine (1:1) et que les valeurs inférieures réduisent sa taille. | Une valeur de `0.5` correspond à la moitié de la taille d’origine. |
| `image` | `url` | URL d’accès au fichier PNG à utiliser comme filigrane. | |

## Événements asynchrones {#asynchronous-events}

Une fois le traitement d’un rendu terminé ou si une erreur se produit, un événement est envoyé à un [!DNL `I/O Events Journal`] Adobe. Les clientes et clients doivent écouter l’URL du journal fournie par le biais de [`/register`](#register). La réponse du journal comprend un tableau `event` constitué d’un objet pour chaque événement, dont le champ `event` contient le payload réel de l’événement.

Le type [!DNL `I/O Events`] Adobe pour tous les événements d’[!DNL Asset Compute Service] est `asset_compute`. Le journal est automatiquement abonné exclusivement à ce type d’événement et il n’est pas nécessaire d’effectuer un filtrage en fonction du type Événement [!DNL Adobe Developer]. Les types d’événements spécifiques au service sont disponibles dans la propriété `type` de l’événement.

### Types d’événements {#event-types}

| Événement | Description |
|---------------------|-------------|
| `rendition_created` | Envoyé pour chaque rendu traité et chargé. |
| `rendition_failed` | Envoyé pour chaque rendu dont le traitement ou le chargement a échoué. |

### Attributs d’événement {#event-attributes}

| Attribut | Type | Événement | Description |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | Date et heure du moment où l’événement a été envoyé au format étendu simplifié [ISO-8601](https://fr.wikipedia.org/wiki/ISO_8601), tel que défini par JavaScript [Date.toISOString()](https://developer.mozilla.org/fr-FR/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString). |
| `requestId` | `string` | `*` | Identifiant de la requête d’origine à `/process`, identique à l’en-tête `X-Request-Id`. |
| `source` | `object` | `*` | `source` de la requête `/process`. |
| `userData` | `object` | `*` | L’objet `userData` du rendu issu de la requête `/process`, si une définition existe. |
| `rendition` | `object` | `rendition_*` | Objet de rendu correspondant transmis dans `/process`. |
| `metadata` | `object` | `rendition_created` | Les propriétés des [métadonnées](#metadata) du rendu. |
| `errorReason` | `string` | `rendition_failed` | La [raison](#error-reasons) de l’échec du rendu, le cas échéant. |
| `errorMessage` | `string` | `rendition_failed` | Texte donnant, le cas échéant, davantage de détails sur l’échec du rendu. |

### Métadonnées {#metadata}

| Propriété | Description |
|--------|-------------|
| `repo:size` | Taille du rendu en octets. |
| `repo:sha1` | Résumé sha1 du rendu. |
| `dc:format` | Type MIME du rendu. |
| `repo:encoding` | Codage du jeu de caractères du rendu dans le cas où il s’agit d’un format texte. |
| `tiff:ImageWidth` | Largeur du rendu en pixels. Uniquement présente pour les rendus d’image. |
| `tiff:ImageLength` | Longueur du rendu en pixels. Uniquement présente pour les rendus d’image. |

### Raisons de l’erreur {#error-reasons}

| Raison | Description |
|---------|-------------|
| `RenditionFormatUnsupported` | Le format de rendu demandé n’est pas pris en charge pour la source donnée. |
| `SourceUnsupported` | La source spécifique n’est pas prise en charge même si le type est pris en charge. |
| `SourceCorrupt` | Les données source sont endommagées. Cela inclut des fichiers vides. |
| `RenditionTooLarge` | Impossible de charger le rendu à l’aide des URL présignées indiquées dans `target`. La taille réelle du rendu est disponible sous forme de métadonnées dans `repo:size` et est utilisée par le client ou la cliente pour traiter à nouveau ce rendu avec le nombre approprié d’URL présignées. |
| `GenericError` | Toute autre erreur inattendue. |
