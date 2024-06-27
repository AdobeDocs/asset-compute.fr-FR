---
title: Développer pour [!DNL Asset Compute Service]
description: Créer des applications personnalisées à l’aide d’ [!DNL Asset Compute Service].
exl-id: a0c59752-564b-4bb6-9833-ab7c58a7f38e
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '1507'
ht-degree: 56%

---

# Développement d’une application personnalisée {#develop}

Avant de commencer à développer une application personnalisée :

* Assurez-vous que toutes les [conditions préalables](/help/using/understand-extensibility.md#prerequisites-and-provisioning) sont remplies.
* Installez les [outils logiciels requis](/help/using/setup-environment.md#create-dev-environment).
* Voir [Configuration de votre environnement](setup-environment.md) pour vous assurer que vous êtes prêt à créer une application personnalisée.

## Création d’une application personnalisée {#create-custom-application}

Assurez-vous que la variable [Adobe aio-cli](https://github.com/adobe/aio-cli) installé localement.

1. Pour créer une application personnalisée, [créez un projet App Builder](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#4-bootstrapping-new-app-using-the-cli). Pour ce faire, exécutez `aio app init <app-name>` dans votre terminal.

   Si vous n’êtes pas encore connecté, cette commande appelle un navigateur qui vous invite à vous connecter à [Adobe Developer Console](https://developer.adobe.com/console/user/servicesandapis) avec votre Adobe ID. Voir [ici](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli) pour plus d’informations sur la connexion à partir de l’interface de ligne de commande.

   Adobe vous recommande de vous connecter en premier. Si vous rencontrez des problèmes, suivez les instructions [création d’une application sans connexion](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user).

1. Après vous être connecté, suivez les invites de l’interface en ligne de commande et sélectionnez les éléments `Organization`, `Project` et `Workspace` à utiliser pour l’application. Sélectionnez le projet et l’espace de travail que vous avez créés lorsque vous [configurer votre environnement ;](setup-environment.md). À l’invite `Which extension point(s) do you wish to implement ?`, veillez à sélectionner `DX Asset Compute Worker` :

   ```sh
   $ aio app init <app-name>
   Retrieving information from [!DNL Adobe I/O] Console.
   ? Select Org My Adobe Org
   ? Select Project MyAdobe Developer App BuilderProject
   ? Which extension point(s) do you wish to implement ? (Press <space> to select, <a>
   to toggle all, <i> to invert selection)
   ❯◯ DX Experience Cloud SPA
   ◯ DX Asset Compute Worker
   ```

1. Lorsque l’invite `Which Adobe I/O App features do you want to enable for this project?` s’affiche, sélectionnez `Actions`. Veillez à désélectionner l’option `Web Assets` car les ressources Web utilisent différentes vérifications d’authentification et d’autorisation.

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. À l’invite `Which type of sample actions do you want to create?`, veillez à sélectionner `Adobe Asset Compute Worker` :

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. Suivez les autres invites et ouvrez la nouvelle application dans Visual Studio Code (ou votre éditeur de code préféré). Il contient la structure et l’exemple de code pour une application personnalisée.

   Lisez ici des informations sur les [principaux composants d’une application App Builder](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#5-anatomy-of-an-app-builder-application).

   L’application de modèle tire parti du Adobe [ASSET COMPUTE SDK](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk) pour le chargement, le téléchargement et l’orchestration des rendus d’application. Les développeurs n’ont donc qu’à implémenter la logique d’application personnalisée. Dans le dossier `actions/<worker-name>`, le fichier `index.js` indique où ajouter le code d’application personnalisé.

Voir [Exemples d’applications personnalisées](#try-sample) pour consulter des exemples et des idées d’applications personnalisées.

### Ajout des informations d’identification {#add-credentials}

Lorsque vous vous connectez tandis que l’application est en train de se créer, la plupart des informations d’identification d’App Builder sont collectées dans votre fichier ENV. Toutefois, l’utilisation de l’outil de développement nécessite des informations d’identification supplémentaires.

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### Informations d’identification pour le stockage de l’outil de développement {#developer-tool-credentials}

Outil permettant aux développeurs d’évaluer des applications personnalisées à l’aide de la variable [!DNL Asset Compute service] nécessite l’utilisation d’un conteneur de stockage dans le cloud. Ce conteneur est essentiel pour stocker les fichiers de test ainsi que pour la réception et la présentation des rendus produits par les applications.

>[!NOTE]
>
>Ce conteneur est distinct de l’espace de stockage dans le cloud de [!DNL Adobe Experience Manager] as a [!DNL Cloud Service]. Il s’applique uniquement au développement et au test avec l’outil de développement Asset Compute.

Assurez-vous d’avoir accès à un [conteneur de stockage dans le cloud pris en charge](https://github.com/adobe/asset-compute-devtool#prerequisites). Ce conteneur est utilisé collectivement par différents développeurs pour différents projets, le cas échéant.

#### Ajout des informations d’identification au fichier ENV {#add-credentials-env-file}

Insérez les informations d’identification suivantes de l’outil de développement dans la variable `.env` fichier . Le fichier se trouve à la racine de votre projet App Builder :

1. Ajoutez le chemin d’accès absolu au fichier de clé privée créé lors de l’ajout de services à votre projet App Builder :

   ```conf
   ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
   ```

1. Téléchargez le fichier à partir d’Adobe Developer Console. Accédez à la racine du projet et cliquez sur « Tout télécharger » dans l’angle supérieur droit. Le fichier est téléchargé avec `<namespace>-<workspace>.json` comme nom de fichier. Utilisez l’une des méthodes suivantes :

   * Renommez le fichier `console.json` et déplacez-le dans la racine de votre projet.
   * Vous pouvez éventuellement ajouter le chemin d’accès absolu au fichier JSON d’intégration d’Adobe Developer Console. Ce fichier est le même [`console.json`](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user) fichier téléchargé dans l’espace de travail du projet.

     ```conf
     ASSET_COMPUTE_INTEGRATION_FILE_PATH=
     ```

1. Ajoutez les informations d’identification S3 ou de la solution de stockage Azure. Vous n’avez besoin d’accéder qu’à une seule solution d’espace de stockage dans le cloud.

   ```conf
   # S3 credentials
   S3_BUCKET=
   AWS_ACCESS_KEY_ID=
   AWS_SECRET_ACCESS_KEY=
   AWS_REGION=
   
   # Azure Storage credentials
   AZURE_STORAGE_ACCOUNT=
   AZURE_STORAGE_KEY=
   AZURE_STORAGE_CONTAINER_NAME=
   ```

>[!TIP]
>
>Le fichier `config.json` contient des informations d’identification. Dans votre projet, ajoutez le fichier JSON à votre fichier `.gitignore` pour empêcher son partage. Il en va de même pour `.env` et `.aio` fichiers .

## Exécution de l’application {#run-custom-application}

Avant d’exécuter l’application avec l’outil de développement Asset Compute, configurez correctement la variable [informations](#developer-tool-credentials).

Pour exécuter l’application dans l’outil de développement, utilisez la commande `aio app run`. Elle déploie l’action sur Adobe [!DNL I/O Runtime]et lance l’outil de développement sur votre ordinateur local. Cet outil est utilisé pour tester les demandes des applications au cours du développement. Voici un exemple de demande de rendu :

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg"
    }
]
```

>[!NOTE]
>
>N’utilisez pas l’indicateur `--local` avec la commande `run`. Il ne fonctionne pas avec [!DNL Asset Compute] des applications personnalisées et l’outil de développement d’Asset compute. Les applications personnalisées sont appelées par la fonction [!DNL Asset Compute] service ne pouvant pas accéder aux actions exécutées sur les ordinateurs locaux du développeur.

Voir [ici](test-custom-application.md) comment tester et déboguer votre application. Lorsque vous avez terminé de développer votre application personnalisée, [déployez-la](deploy-custom-application.md).

## Essai de l’exemple d’application fourni par Adobe {#try-sample}

Voici des exemples d’applications personnalisées :

* [worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [worker-animal-pictures](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### Modèle d’application personnalisée {#template-custom-application}

[worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic) est un modèle d’application. Il génère un rendu en copiant simplement le fichier source. Le contenu de cette application est le modèle reçu lors du choix d’`Adobe Asset Compute` lors de la création de l’application aio.

Le fichier de l’application [`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) utilise le [`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview) pour télécharger le fichier source, orchestrer le traitement de chaque rendu et charger les rendus résultants dans l’espace de stockage dans le cloud.

La variable [`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) défini dans le code de l’application permet d’exécuter toute la logique de traitement de l’application. Le rappel de rendu de `worker-basic` copie simplement le contenu du fichier source dans le fichier de rendu.

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

exports.main = worker(async (source, rendition) => {
    // copy source to rendition to transfer 1:1
    await fs.copyFile(source.path, rendition.path);
});
```

## Appel d’une API externe {#call-external-api}

Dans le code de l’application, vous pouvez effectuer des appels d’API externe pour faciliter le traitement de l’application. Vous trouverez ci-dessous un exemple de fichier d’application qui appelle une API externe.

```javascript
exports.main = worker(async function (source, rendition) {

    const response = await fetch('https://adobe.com', {
        method: 'GET',
        Authorization: params.AUTH_KEY
    })
});
```

Par exemple, [`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46) effectue une requête de récupération d’une URL statique à partir de Wikimedia à l’aide de la bibliothèque [`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer).

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### Transmission de paramètres personnalisés {#pass-custom-parameters}

Vous pouvez transmettre des paramètres personnalisés définis à l’aide des objets de rendu. Ils peuvent être référencés dans l’application au moyen de [`rendition` instructions](https://github.com/adobe/asset-compute-sdk#rendition). Voici un exemple d’objet de rendu :

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg",
        "my-custom-parameter": "my-custom-parameter-value"
    }
]
```

Voici un exemple de fichier d’application accédant à un paramètre personnalisé :

```javascript
exports.main = worker(async function (source, rendition) {

    const customParam = rendition.instructions['my-custom-parameter'];
    console.log('Custom paramter:', customParam);
    // should print out `Custom parameter: "my-custom-parameter-value"`
});
```

`example-worker-animal-pictures` transmet un paramètre personnalisé [`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39) pour déterminer le fichier à récupérer auprès de Wikimedia.

## Prise en charge de l’authentification et de l’autorisation {#authentication-authorization-support}

Par défaut, les applications personnalisées d’Asset compute sont fournies avec des contrôles d’autorisation et d’authentification pour le projet App Builder. Activé en définissant la variable `require-adobe-auth` annotation à `true` dans le `manifest.yml`.

### Accès à d’autres API d’Adobe {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

Ajoutez les services d’API à l’espace de travail de la console [!DNL Asset Compute] créé lors de la configuration. Ces services font partie du jeton d’accès JWT généré par [!DNL Asset Compute Service]. Le jeton et les autres informations d’identification sont accessibles dans l’objet `params` d’action de l’application.

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### Transmission des informations d’identification des systèmes tiers {#pass-credentials-for-tp}

Pour gérer les informations d’identification d’autres services externes, transmettez-les en tant que paramètres par défaut sur les actions. Ils sont automatiquement cryptés en transit. Pour plus d’informations, voir [création d’actions dans le guide de développement de Adobe I/O Runtime](https://developer.adobe.com/runtime/docs/guides/using/creating_actions/). Définissez-les ensuite à l’aide de variables d’environnement lors du déploiement. Ces paramètres sont accessibles dans l’objet `params` à l’intérieur de l’action.

Définissez les paramètres par défaut dans `inputs` dans le fichier `manifest.yml` :

```yaml
packages:
  __APP_PACKAGE__:
    actions:
      worker:
        function: 'index.js'
        runtime: 'nodejs:10'
        web: true
        inputs:
           secretKey: $SECRET_KEY
        annotations:
          require-adobe-auth: true
```

L’expression `$VAR` lit la valeur d’une variable d’environnement nommée `VAR`.

Lors du développement, vous pouvez attribuer la valeur dans le `.env` fichier . La raison en est que `aio` importe automatiquement les variables d’environnement depuis `.env` fichiers, ainsi que les variables définies par le shell d’initialisation. Dans cet exemple, la variable `.env` se présente comme suit :

```CONF
#...
SECRET_KEY=secret-value
```

Pour le déploiement de la production, il est possible de définir les variables d’environnement dans le système d’intégration continue (CI), par exemple en utilisant des secrets dans les actions GitHub. Enfin, accédez aux paramètres par défaut de l’application en tant que tels :

```javascript
const key = params.secretKey;
```

## Dimensionnement des applications {#sizing-workers}

Une application s’exécute dans un conteneur dans Adobe [!DNL I/O Runtime] avec [limites](https://developer.adobe.com/runtime/docs/guides/using/system_settings/) qui peuvent être configurés via la variable `manifest.yml`:

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

En raison du traitement étendu effectué par les applications Asset Compute, vous devez ajuster ces limites afin d’optimiser les performances (suffisamment pour gérer les ressources binaires) et l’efficacité (sans gaspiller les ressources en raison de la mémoire de conteneur inutilisée).

Le délai d’expiration par défaut pour les actions du Runtime est d’une minute, mais il peut être augmenté en définissant la limite `timeout` (en millisecondes). Si vous prévoyez de traiter des fichiers plus volumineux, augmentez cette durée. Tenez compte du temps total nécessaire pour télécharger la source, traiter le fichier et charger le rendu. Si une action expire, c’est-à-dire qu’elle ne renvoie pas l’activation avant la limite de délai spécifiée, Runtime ignore le conteneur et ne le réutilise pas.

Les applications d’Asset compute tendent par nature à être liées à l’entrée ou à la sortie réseau et disque. Le fichier source doit d’abord être téléchargé. Le traitement consomme souvent beaucoup de ressources, puis les rendus résultants sont chargés à nouveau.

Vous pouvez spécifier la mémoire allouée à un conteneur d’actions en mégaoctets à l’aide de la propriété `memorySize` . Actuellement, ce paramètre définit également le niveau d’accès de l’unité centrale au conteneur et, plus important encore, il s’agit d’un élément clé du coût d’utilisation du Runtime (les conteneurs plus volumineux coûtent plus cher). Utilisez ici une valeur plus élevée lorsque votre traitement nécessite plus de mémoire ou d’unité centrale, mais veillez à ne pas gaspiller les ressources, car plus les conteneurs sont volumineux, plus le débit global est faible.

En outre, il est possible de contrôler la simultanéité des actions dans un conteneur à l’aide du paramètre `concurrency`. Ce paramètre correspond au nombre d’activations simultanées qu’un seul conteneur (de la même action) obtient. Dans ce modèle, le conteneur d’action est semblable à un serveur Node.js recevant plusieurs requêtes simultanées, jusqu’à cette limite. Par défaut `memorySize` dans l’environnement d’exécution est défini sur 200 Mo, idéal pour les actions App Builder plus petites. Pour les applications Asset Compute, cette valeur par défaut peut être excessive en raison de leur utilisation plus importante du disque et du traitement local. Selon leur mise en œuvre, il est possible que certaines applications ne fonctionnent pas correctement avec les activités simultanées. Le SDK Asset Compute garantit que les activations sont séparées en écrivant des fichiers dans différents dossiers uniques.

Testez les applications pour trouver les valeurs optimales de `concurrency` et `memorySize`. Des conteneurs plus volumineux reviennent à une limite de mémoire plus haute, ce qui permet une plus grande simultanéité, mais peut aussi entraîner du gaspillage si le trafic est plus faible.
