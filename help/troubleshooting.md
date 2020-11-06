---
title: Résolution des problèmes d’ [!DNL Asset Compute Service].
description: Dépanner et déboguer les applications personnalisées à l’aide d’ [!DNL Asset Compute Service].
translation-type: ht
source-git-commit: 68d910cd092fccb599c361f24daff80460129e1c
workflow-type: ht
source-wordcount: '306'
ht-degree: 100%

---


# Résolution des problèmes d’{#troubleshoot}

Voici quelques conseils de dépannage génériques qui peuvent vous aider à résoudre les problèmes liés à Asset Compute Service :

* Assurez-vous que l’application JavaScript ne se bloque pas au démarrage. Ces blocages sont généralement liés à une bibliothèque ou à une dépendance manquante.
* Assurez-vous que toutes les dépendances à installer sont référencées dans le fichier `package.json` de l’application.
* Assurez-vous que les erreurs pouvant provenir du nettoyage en cas d’échec ne génèrent pas leurs propres erreurs en masquant le problème d’origine.

* Lors du démarrage initial de l’outil de développement avec une nouvelle intégration [!DNL Asset Compute Service], il se peut que la première demande de traitement échoue, car le journal des événements Asset Compute n’est pas entièrement configuré. Patientez un certain temps pour que le journal soit configuré avant d’envoyer une autre demande.
* Si vous rencontrez des erreurs lors de l’envoi de requêtes `/register` ou `/process` Asset Compute, assurez-vous que toutes les API nécessaires sont ajoutées au projet et à l’espace de travail Adobe I/O, c’est-à-dire Asset Compute, IO Events, IO Events Management et Runtime.

## Problèmes de connexion à l’aide de l’Adobe I/O CLI {#login-via-aio-cli}

Si vous rencontrez des problèmes lors de la connexion à [!DNL Adobe Developer Console] [par le biais de l’Adobe I/O CLI](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#3-signing-in-from-cli), ajoutez manuellement les informations d’identification requises pour le développement, le test et le déploiement de votre application personnalisée :

1. Accédez à votre projet et à votre espace de travail Firefly sur [Adobe Developer Console](https://console.adobe.io/), puis appuyez sur **[!UICONTROL Download]** dans le coin supérieur droit. Ouvrez ce fichier JSON et enregistrez-le à un emplacement sécurisé sur votre ordinateur.

1. Accédez au fichier ENV de votre application Firefly.

1. Ajoutez les informations d’identification Adobe I/O Runtime. Récupérez les informations d’identification Adobe I/O Runtime à partir du fichier JSON téléchargé. Les informations d’identification se trouvent dans `project.workspace.services.runtime`. Ajoutez les informations d’identification I/O Runtime dans les variables `AIO_runtime_XXX` :

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