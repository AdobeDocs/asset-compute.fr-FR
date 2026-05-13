---
title: Introduction au  [!DNL Asset Compute Service]
description: '[!DNL Asset Compute Service] est un service de traitement des ressources natif dans le cloud destiné à réduire la complexité et à améliorer l’évolutivité.'
exl-id: f8c89f65-5a94-44f3-aaac-4612ae291101
TQID: https://experienceleague.adobe.com/7B0ghzyLWIZFe5sthLaorFf8w4Az7FKW4ces27eo8I0
product_v2:
  - id: d09181b5-a36a-43de-ba01-36641440bc43
  - id: fd1f54a9-f50c-467d-8956-cebbaf4f3eb8
feature_v2:
  - id: a01bfd36-4ab8-4bf8-9dc0-5b45b890552e
  - id: da0dfbce-df02-4f8b-b32d-a4e3b1d05085
role_v2:
  - id: ff6a42d2-313e-452e-93a6-792e4fad9ff8
topic_v2:
  - id: a004cc84-67b9-4a33-a3a7-8ec7273ef4dc
source-git-commit: 2510f77fed8d0f0708e09f32d0b13a437d2ede4f
workflow-type: tm+mt
source-wordcount: 355
ht-degree: 96%

---

# Présentation d’[!DNL Asset Compute Service] {#overview}

[!DNL Asset Compute Service] est un service évolutif et extensible d’[!DNL Adobe Experience Cloud] destiné à traiter des ressources numériques. Il permet d’obtenir différents rendus à partir des images, des vidéos, des documents et d’autres formats de fichiers, notamment sous forme de miniatures, de texte extrait, de métadonnées et d’archives.

L’équipe de développement a la possibilité d’ajouter des applications de ressources personnalisées (également appelées programmes de travail personnalisés) pour traiter les cas d’utilisation spécifiques. Le service fonctionne sur Adobe [!DNL I/O Runtime]. Il peut être étendu à l’aide d’applications découplées [!DNL Adobe Developer App Builder] écrites dans Node.js. Elles peuvent procéder à des opérations personnalisées telles que des appels à des API externes pour effectuer des opérations sur des images ou pour bénéficier de la prise en charge d’[!DNL Adobe Sensei].

[!DNL Adobe Developer App Builder] est un framework destiné à créer et déployer des applications web personnalisées sur Adobe [!DNL I/O Runtime] pour étendre les solutions Adobe Experience Cloud. Pour créer des applications personnalisées, l’équipe de développement peut tirer parti de [!DNL React Spectrum] (boîte à outils d’interface d’utilisation d’Adobe), créer des microservices et des événements personnalisés, et orchestrer les API. Consultez la [documentation d’Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/intro_and_overview/#).

>[!NOTE]
>
>Actuellement, [!DNL Asset Compute Service] ne peut être utilisé que par [!DNL Experience Manager] as a [!DNL Cloud Service]. Les administrateurs et administratrices créent des profils de traitement qui peuvent appeler [!DNL Asset Compute Service] pour transmettre des ressources à des fins de traitement. Voir [Utiliser des microservices de ressources et des profils de traitement](https://experienceleague.adobe.com/fr/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

## Cas d’utilisation pris en charge par [!DNL Asset Compute Service] {#possible-use-cases-benefits}

[!DNL Asset Compute Service] prend en charge quelques cas d’utilisation métier courants, tels que le traitement de base d’images, les conversions spécifiques aux applications d’Adobe et la création d’applications personnalisées qui orchestrent des besoins métier complexes.

Vous pouvez utiliser le service web [!DNL Asset Compute] afin de générer des miniatures pour différents types de fichiers, ainsi que des rendus d’image de haute qualité pour les [formats de fichiers pris en charge](https://experienceleague.adobe.com/fr/docs/experience-manager-cloud-service/content/assets/file-format-support). Voir [Cas d’utilisation pris en charge par le biais d’une configuration personnalisée](https://experienceleague.adobe.com/fr/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

>[!NOTE]
>
>Le service n’assure pas le stockage des ressources. Les utilisateurs le fournissent et donnent des références aux emplacements de fichiers source et de rendu dans l’espace de stockage dans le cloud.

<!-- 
TBD: Should this be mentioned in the docs?

|Asset Compute Service does not do this|Expectations from implementing client|
|---|---|
| Binary uploads or API-based asset ingestion. | Use other methods to ingest assets. |
| Store binaries or any persisted data across processing requests.| Each request is independent so treat it as a standalone request by sharing binary and processing instructions. |
| Store any configurations such as processing rules or settings for a user or an organization's account. | Add processing request to each request/instruction. |
| Direct event handling of asset creation events from storage systems and processing completed notifications, and errors. | Use [!DNL Adobe I/O] Events and other methods. |

-->

>[!MORELIKETHIS]
>
>* [Présentation du traitement des ressources à l’aide de microservices de ressources dans  [!DNL Adobe Experience Manager]  as a  [!DNL Cloud Service]](https://experienceleague.adobe.com/fr/docs/experience-manager-cloud-service/content/assets/asset-microservices-overview).
>* [Documentation d’Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/intro_and_overview/#).
>* [Formats de fichiers pris en charge pour le traitement](https://experienceleague.adobe.com/fr/docs/experience-manager-cloud-service/content/assets/file-format-support).

<!-- 
**TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
