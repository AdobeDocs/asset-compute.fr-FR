---
title: Résolution des problèmes liés à [!DNL Asset Compute Service]
description: Dépanner et déboguer les applications personnalisées à l’aide d’ [!DNL Asset Compute Service].
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: ht
source-wordcount: '273'
ht-degree: 100%

---

# Résolution des problèmes {#troubleshoot}

Voici quelques conseils de dépannage génériques qui peuvent vous aider à résoudre les problèmes liés à Asset Compute Service :

* Assurez-vous que l’application JavaScript ne se bloque pas au démarrage. Ces blocages sont généralement liés à une bibliothèque ou à une dépendance manquante.
* Assurez-vous que toutes les dépendances à installer sont référencées dans le fichier `package.json` de l’application.
* Assurez-vous que les erreurs pouvant provenir du nettoyage en cas d’échec ne génèrent pas leurs propres erreurs en masquant le problème d’origine.

* Lors du démarrage initial de l’outil de développement avec une nouvelle intégration [!DNL Asset Compute Service], il se peut que la première demande de traitement échoue, car le journal des événements Asset Compute n’est pas entièrement configuré. Patientez un certain temps pour que le journal soit configuré avant d’envoyer une autre demande.
* Assurez-vous que toutes les API requises, Asset Compute, Adobe [!DNL I/O Events], Events Management et Runtime, sont incluses dans votre Adobe [!DNL `I/O Project`] et Workspace pour éviter les erreurs de requêtes `/register` ou `/process`.

## Problèmes de connexion par l’intermédiaire d’Adobe [!DNL aio-cli] {#login-via-aio-cli}

Si vous rencontrez des problèmes lors de la connexion à l’[!DNL Adobe Developer Console] [par le biais d’Adobe  [!DNL aio-cli]](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli), ajoutez manuellement les informations d’identification requises pour le développement, le test et le déploiement de votre application personnalisée :

1. Accédez à votre projet et à votre espace de travail et à votre projet Adobe Developer App Builder sur [Adobe Developer Console](https://developer.adobe.com/console/user/servicesandapis), puis appuyez sur **[!UICONTROL Télécharger]** dans le coin supérieur droit. Ouvrez ce fichier JSON et enregistrez-le à un emplacement sécurisé sur votre ordinateur.

1. Accédez au fichier ENV dans votre application Adobe Developer App Builder.

1. Ajoutez les informations d’identification Adobe [!DNL I/O Runtime]. Récupérez les informations d’identification Adobe [!DNL I/O Runtime] à partir du fichier JSON téléchargé. Les informations d’identification se trouvent dans `project.workspace.services.runtime`. Ajoutez les informations d’identification Runtime [!DNL Adobe I/O] dans les variables `AIO_runtime_XXX` :

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. Ajoutez le chemin d’accès absolu au fichier JSON téléchargé à l’étape 1 :

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. Configurez le reste des [informations d’identification requises](develop-custom-application.md) pour l’outil de développement.

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
