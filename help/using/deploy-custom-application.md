---
title: Déploiement d’une application personnalisée  [!DNL Asset Compute Service]
description: Déployer une application personnalisée  [!DNL Asset Compute Service] .
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '170'
ht-degree: 94%

---

# Déploiement d’une application personnalisée {#deploy-custom-application}

Pour déployer votre application, utilisez la commande [aio app deploy](https://github.com/adobe/aio-cli#aio-appdeploy). Dans le terminal, la commande affiche une URL pour accéder à l’application personnalisée. L’URL est au format `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

Pour obtenir la même URL sans redéployer l’application, utilisez la commande [`aio app get-url`](https://github.com/adobe/aio-cli#aio-app-get-url-action).

Utilisez l’URL dans un [profil de traitement d’ [!DNL Experience Manager]  as a  [!DNL Cloud Service]](https://experienceleague.adobe.com/fr/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use) pour intégrer votre application avec [!DNL Experience Manager] as a [!DNL Cloud Service].

Assurez-vous que votre projet App Builder et votre espace de travail correspondent à l’environnement [!DNL Experience Manager] as a [!DNL Cloud Service] où vous voulez utiliser votre action. Il dispose de différents environnements de développement, d’évaluation et de production. Vous pouvez vérifier l’environnement en examinant les informations d’identification `AIO_runtime_*` définies dans votre fichier ENV dans la racine de votre application Adobe Developer App Builder. Par exemple, pour effectuer un déploiement dans un espace de travail `Stage`, le format de `AIO_runtime_namespace` est `xxxxxx_xxxxxxxxx_stage`. Pour l’intégration à l’environnement de production [!DNL Experience Manager] as a [!DNL Cloud Service], utilisez les URL d’application de votre espace de travail `Production` Adobe Developer App Builder.

>[!CAUTION]
>
>N’utilisez pas d’espace de travail personnel dans les cas critiques [!DNL Experience Manager] des environnements.

>[!MORELIKETHIS]
>
>* [Comprendre et gérer les environnements dans  [!DNL Experience Manager]  as a  [!DNL Cloud Service]](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/using-cloud-manager/manage-environments).
