---
title: Définition de l’environnement de développement requis pour [!DNL Asset Compute Service]
description: Configuration de l’environnement de développement [!DNL Asset Compute Service] pour commencer à créer et tester le code personnalisé.
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: 9404ffcc66a3b6ba206155d1b1a5c16a43e22a39
workflow-type: ht
source-wordcount: '370'
ht-degree: 100%

---

# Configuration d’un environnement de développement {#create-dev-environment}

Pour créer une configuration qui permet de développer pour [!DNL Asset Compute Service], suivez ces instructions et exigences.

1. [Obtenez l’accès et les informations d’identification](https://www.adobe.io/project-firefly/docs/getting_started/#acquire-access-and-credentials) pour [!DNL Project Firefly].

1. [Configurez l’environnement local](https://www.adobe.io/project-firefly/docs/getting_started/#local-environment-set-up) et les outils nécessaires.

1. Voici d’autres outils pour commencer à développer avec aisance :

   * [Git](https://git-scm.com/).
   * [Docker Desktop](https://www.docker.com/get-started).
   * [NodeJS](https://nodejs.org) (LTS v10 à v12, les versions impaires ne sont pas recommandées) et [NPM](https://www.npmjs.com). L’utilisateur d’OSX HomeBrew peut exécuter `brew install node` pour installer l’un et l’autre. Sinon, vous pouvez effectuer le téléchargement sur la [page de téléchargement de NodeJS](https://nodejs.org/fr/).
   * Comme IDE adapté à NodeJS, nous recommandons [Visual Studio Code (VS Code)](https://code.visualstudio.com), car c’est l’IDE pris en charge pour le débogueur. Vous pouvez utiliser un autre IDE à titre d’éditeur de code, mais il n’y a pas encore de prise en charge pour une utilisation avancée (un débogueur, par exemple).
   * [[!DNL Adobe I/O] Interface en ligne de commande ](https://github.com/adobe/aio-cli) (`aio`) : installation à l’aide de la commande `npm install -g @adobe/aio-cli@7.1.0`.

1. Veillez à respecter les [conditions préalables](/help/understand-extensibility.md#prerequisites-and-provisioning).

>[!NOTE]
>
>Pour l’instant, utilisez l’interface de ligne de commande [!DNL Adobe I/O] v7.1.0 et n’utilisez pas l’interface de ligne de commande [!DNL Adobe I/O] v8.

## Configuration d’un projet Firefly {#create-firefly-project}

1. Assurez-vous du rôle d’administrateur système ou de développeur dans l’organisation [!DNL Experience Cloud]. Un administrateur système peut configurer ce rôle dans [Admin Console](https://adminconsole.adobe.com/overview).

1. Connectez-vous à [Adobe Developer Console](https://console.adobe.io/). Assurez-vous que vous faites partie de la même organisation [!DNL Experience Cloud] que l’intégration [!DNL Experience Manager] as a [!DNL Cloud Service]. Pour plus d’informations sur Adobe Developer Console, consultez la [documentation de la console](https://www.adobe.io/apis/experienceplatform/console/docs.html).

1. [Créez un projet Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md). Cliquez sur **[!UICONTROL Create new project]** > **[!UICONTROL Project from template]**. Sélectionnez Firefly. Un projet Firefly est créé avec deux espaces de travail : `Production` et `Stage`. Ajoutez des espaces de travail supplémentaires, par exemple `Development`, selon les besoins.

1. Dans le projet Firefly, sélectionnez un espace de travail et abonnez-vous aux services requis pour Asset Compute. Cliquez sur **Add to Project** > **API** et ajoutez les services `Asset Compute`, `IO Events` et `IO Events Management`. Lors de l’ajout de la première API, une invite propose de créer une clé privée. Enregistrez ces informations sur votre ordinateur, car vous en aurez besoin pour tester votre application personnalisée à l’aide de l’outil de développement.

## Étape suivante {#next-step}

Maintenant que votre environnement est configuré, vous êtes prêt à [créer une application personnalisée](develop-custom-application.md).

<!-- More ideas:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)

TBD: When aio-cli v8 bugs are resolved, update the AIO CLI install command to remove v7.x reference and instruct users to use the latest version. See CQDOC-18346.

-->
