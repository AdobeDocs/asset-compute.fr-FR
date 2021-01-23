---
title: Comprendre le fonctionnement d’une application personnalisée
description: Fonctionnement interne d’une application personnalisée  [!DNL Asset Compute Service]  pour faciliter votre compréhension.
translation-type: tm+mt
source-git-commit: 95e384d2a298b3237d4f93673161272744e7f44a
workflow-type: tm+mt
source-wordcount: '751'
ht-degree: 94%

---


# Fonctionnement interne d’une application personnalisée {#how-custom-application-works}

Utilisez l’illustration suivante pour comprendre le workflow de bout en bout lorsqu’une ressource numérique est traitée par un client à l’aide d’une application personnalisée.

![Workflow des applications personnalisées](assets/customworker.png)

*Figure : Procédure de traitement d’une ressource à l’aide d’[!DNL Asset Compute Service].*

## Enregistrement {#registration}

Le client doit appeler une fois [`/register`](api.md#register) avant la première requête à [`/process`](api.md#process-request) pour configurer et récupérer l’URL du journal afin de recevoir les événements pour Adobe Asset Compute.[!DNL Adobe I/O]

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

Il est possible d’utiliser la bibliothèque JavaScript [`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) dans les applications NodeJS pour gérer toutes les étapes nécessaires, depuis l’enregistrement jusqu’au traitement en passant par la gestion asynchrone des événements. Pour plus d’informations sur les en-têtes requis, voir [Authentification et autorisation](api.md).

## Traitement {#processing}

Le client envoie une requête de [traitement](api.md#process-request).

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

Le client est chargé de mettre correctement en forme les rendus à l’aide d’URL présignées. Il est possible d’utiliser la bibliothèque JavaScript [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) dans les applications NodeJS pour présigner les URL. Actuellement, la bibliothèque ne prend en charge que les conteneurs Azure Blob Storage et AWS S3.

La demande de traitement renvoie un `requestId` utilisable pour interroger les événements [!DNL Adobe I/O]

Vous trouverez ci-dessous un exemple de requête de traitement d’application personnalisée.

```json
{
    "source": "https://www.adobe.com/some-source-file.jpg",
    "renditions" : [
        {
            "worker": "https://my-project-namespace.adobeioruntime.net/api/v1/web/my-namespace-version/my-worker",
            "name": "rendition1.jpg",
            "target": "https://some-presigned-put-url-for-rendition1.jpg",
        }
    ],
    "userData": {
        "my-asset-id": "1234567890"
    }
}
```

L’[!DNL Asset Compute Service] envoie les requêtes de rendu d’application personnalisée à l’application personnalisée. Il utilise une requête HTTP POST sur l’URL de l’application fournie. Il s’agit de l’URL d’action web sécurisée de Project Firefly. Toutes les requêtes utilisent le protocole HTTPS pour maximiser la sécurité des données.

Le [SDK Asset Compute](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) utilisé par une application personnalisée traite la requête HTTP POST. Il gère également le téléchargement de la source, le transfert de rendus, l&#39;envoi de [!DNL Adobe I/O] événements et la gestion des erreurs.

<!-- TBD: Add the application diagram. -->

### Code de l’application {#application-code}

Le code personnalisé doit uniquement fournir un rappel qui prend le fichier source disponible localement (`source.path`). Le paramètre `rendition.path` correspond à l’emplacement où placer le résultat final d’une requête de traitement de ressource. L’application personnalisée utilise le rappel pour transformer les fichiers source disponibles localement en fichier de rendu à l’aide du nom transmis dans (`rendition.path`). Une application personnalisée doit écrire vers `rendition.path` pour créer un rendu :

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

// worker() is the entry point in the SDK "framework".
// The asynchronous function defined is the rendition callback.
exports.main = worker(async (source, rendition) => {

    // Tip: custom worker parameters are available in rendition.instructions.
    console.log(rendition.instructions.name); // should print out `rendition.jpg`.

    // Simplest example: copy the source file to the rendition file destination so as to transfer the asset as is without processing.
    await fs.copyFile(source.path, rendition.path);
});
```

### Téléchargement des fichiers source {#download-source}

Une application personnalisée traite uniquement des fichiers locaux. Le téléchargement du fichier source est géré par le [SDK Asset Compute](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk).

### Création de rendu {#rendition-creation}

Le SDK appelle une [fonction de rappel de rendu](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) asynchrone pour chaque rendu.

La fonction de rappel a accès aux objets [source](https://github.com/adobe/asset-compute-sdk#source) et de [rendu](https://github.com/adobe/asset-compute-sdk#rendition). `source.path` existe déjà et il s’agit du chemin d’accès à la copie locale du fichier source. `rendition.path` est le chemin d’accès où le rendu traité doit être stocké. Sauf si [l’indicateur disableSourceDownload](https://github.com/adobe/asset-compute-sdk#worker-options-optional) est défini, l’application doit utiliser exactement le chemin `rendition.path`. Sinon, le SDK ne peut ni localiser ni identifier le fichier de rendu et échoue.

La simplification extrême de l’exemple sert à illustrer et à se concentrer sur l’anatomie d’une application personnalisée. L’application se contente de copier le fichier source vers la destination du rendu.

Pour plus d’informations sur les paramètres de rappel de rendu, voir [API du SDK Asset Compute](https://github.com/adobe/asset-compute-sdk#api-details).

### Chargement des rendus {#upload-rendition}

Une fois chaque rendu créé et stocké dans un fichier avec le chemin d’accès fourni par `rendition.path`, le [SDK Asset Compute](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) les charge vers un espace de stockage dans le cloud (AWS ou Azure). Une application personnalisée obtient plusieurs rendus simultanés si, et seulement si, la requête entrante comporte plusieurs rendus pointant vers la même URL d’application. Le chargement vers l’espace de stockage dans le cloud est effectué après chaque rendu et avant l’exécution du rappel pour le rendu suivant.

`batchWorker()` a un comportement différent, car il traite tous les rendus, et ne les charge qu’après les avoir tous traités.

## [!DNL Adobe I/O] Événements {#aio-events}

Le SDK envoie [!DNL Adobe I/O] Événements pour chaque rendu. Ces événements sont de type `rendition_created` ou `rendition_failed`, selon le résultat. Pour plus d’informations sur les événements, voir [Événements asynchrones Asset Compute](api.md#asynchronous-events).

## Recevoir [!DNL Adobe I/O] Événements {#receive-aio-events}

Le client sonde le [[!DNL Adobe I/O] Journal de Événements](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#/Journaling) en fonction de sa logique de consommation. L’URL de journal initiale est celle fournie dans la réponse de l’API `/register`. Il est possible d’identifier les événements à l’aide du paramètre `requestId`, présent dans les événements. C’est le même que celui renvoyé dans `/process`. Chaque rendu comporte un événement distinct, envoyé dès que le rendu a été chargé (ou a échoué). Une fois qu’il reçoit un événement correspondant, le client peut afficher ou gérer les rendus résultants.

La bibliothèque JavaScript [`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) facilite l’interrogation du journal en utilisant la méthode `waitActivation()` pour obtenir tous les événements.

```javascript
const events = await assetCompute.waitActivation(requestId);
await Promise.all(events.map(event => {
    if (event.type === "rendition_created") {
        // get rendition from cloud storage location
    }
    else if (event.type === "rendition_failed") {
        // failed to process
    }
    else {
        // other event types
        // (could be added in the future)
    }
}));
```

Pour plus d’informations sur la façon d’obtenir des événements de journal, voir [[!DNL Adobe I/O] API de Événements](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#!adobedocs/adobeio-events/master/events-api-reference.yaml).

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
