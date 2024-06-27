---
title: Comprendre l’extension d’ [!DNL Asset Compute Service]
description: Quand et comment étendre les fonctionnalités d’ [!DNL Asset Compute Service] pour effectuer un traitement personnalisé des ressources.
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '229'
ht-degree: 76%

---

# Introduction à l’extensibilité {#introduction-to-extensibilty}

De nombreuses exigences de rendu, comme la conversion des formats et le redimensionnement des images, sont prises en charge par les [Profils de traitement dans  [!DNL Experience Manager]  as a  [!DNL Cloud Service]](https://experienceleague.adobe.com/fr/docs/experience-manager-cloud-service/content/assets/asset-microservices-overview). Les exigences plus complexes peuvent nécessiter une solution personnalisée adaptée aux besoins d’une entreprise. Il est ainsi possible d’étendre [!DNL Asset Compute Service] en créant des applications personnalisées appelées à partir de profils de traitement dans [!DNL Experience Manager]. Ces applications personnalisées s’adaptent aux [cas d’utilisation pris en charge](https://experienceleague.adobe.com/fr/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

>[!NOTE]
>
>[!DNL Asset Compute Service] est disponible uniquement pour une utilisation avec [!DNL Experience Manager] as a [!DNL Cloud Service].

Les applications personnalisées sont des applications [Adobe Developer App Builder](https://github.com/AdobeDocs/app-builder) découplées. Il est plus simple d’étendre [!DNL Asset Compute Service] à l’aide d’applications personnalisées grâce au [SDK Asset Compute](https://github.com/adobe/asset-compute-sdk) et à l’outil de développement Adobe Developer App Builder. Ces outils permettent aux développeurs de se concentrer sur la logique commerciale. La création d’applications personnalisées est aussi simple que la création d’un Adobe sans serveur ordinaire [!DNL I/O Runtime] action. Il s’agit d’une fonction JavaScript Node.js unique. L’[exemple d’application personnalisée de base](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) en est une illustration.

## Conditions préalables et exigences de configuration {#prerequisites-and-provisioning}

Veillez à respecter les conditions préalables suivantes :

* Les outils Adobe Developer App Builder sont installés sur votre ordinateur.
* Une organisation [!DNL Experience Cloud]. Pour plus d’informations, voir [Démarrage de votre Parcours App Builder](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials).
* [!DNL Experience Manager] as a [!DNL Cloud Service] doit être activé pour l’organisation Experience.
* [!DNL Adobe Experience Cloud] l’organisation fait partie de la [!DNL Adobe Developer App Builder] programme sneak peek du développeur. Accédez à [Comment demander l’accès](https://developer.adobe.com/app-builder/docs/overview/getting_access).
* Veillez à ce que le développeur ait un rôle de développeur ou des autorisations d’administrateur au sein de l’organisation.
* Assurez-vous que l’Adobe [[!DNL aio-cli]](https://github.com/adobe/aio-cli) est installé localement.

<!-- TBD for later:

* What all accesses and licenses are required?
* What all permissions are required to create, debug, and deploy custom applications?
* How do developers get access and provision the required apps?
* What is repository management?
* Anything on security and data transfer?
* What about handling personal or sensitive information?
* Custom application SLA is dependent on SLAs of various services it depends on.
* Document how the devs can get to know the KPIs of their custom applications. The KPIs are dependent on the performance at Adobe's side, amongst other things.
-->
