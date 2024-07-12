---
title: Architecture d’ [!DNL Asset Compute Service]
description: Comment l’API, les applications et le SDK [!DNL Asset Compute Service] fonctionnent ensemble pour fournir un service de traitement des ressources natif dans le cloud.
exl-id: 658ee4b7-5eb1-4109-b263-1b7d705e49d6
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '478'
ht-degree: 100%

---

# Architecture de [!DNL Asset Compute Service] {#overview}

[!DNL Asset Compute Service] repose sur la plateforme Adobe [!DNL `I/O Runtime`] sans serveur. Il permet la prise en charge des ressources par les services de contenu Adobe Sensei. Le client appelant (uniquement si [!DNL Experience Manager] as a [!DNL Cloud Service] est pris en charge) reçoit les informations générées par Adobe Sensei qu’il a recherchées pour la ressource. Les informations renvoyées sont au format JSON.

[!DNL Asset Compute Service] est extensible en créant des applications personnalisées basées sur [!DNL Adobe Developer App Builder]. Ces applications personnalisées sont des applications [!DNL Project Adobe Developer App Builder] sans interface utilisateur graphique. Elles effectuent des tâches comme l’ajout d’outils de conversion personnalisés ou l’appel d’API externes pour réaliser des opérations sur des images.

[!DNL Project Adobe Developer App Builder] est un framework destiné à créer et déployer des applications web personnalisées sur Adobe [!DNL `I/O Runtime`]. Pour créer des applications personnalisées, l’équipe de développement peut tirer parti de [!DNL React Spectrum] (boîte à outils d’interface d’utilisation d’Adobe), créer des microservices et des événements personnalisés, et orchestrer les API. Consultez la [documentation d’Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/overview).

Les fondements de l’architecture sont les suivants :

* La modularité des applications (qui ne contiennent que le nécessaire pour une tâche donnée) permet de les découpler les unes des autres et d’en maintenir la légèreté.

* Le concept sans serveur de Runtime [!DNL Adobe I/O] présente de nombreux avantages : le traitement asynchrone, hautement évolutif, isolé et basé sur les tâches est idéal pour le traitement des ressources.

* Le stockage des contenus binaires dans le cloud apporte les fonctionnalités nécessaires pour stocker et accéder individuellement aux fichiers et aux rendus de ressources, sans exiger d’autorisations d’accès complètes au stockage grâce à des références à des URL présignées. L’accélération du transfert, la mise en cache du réseau de diffusion de contenu et la co-localisation des applications informatiques avec l’espace de stockage dans le cloud permettent un accès aux contenus optimal à faible latence. Les clouds AWS et Azure sont pris en charge.

![Architecture d’Asset Compute Service](assets/architecture-diagram.png)

*Figure : Architecture d’[!DNL Asset Compute Service] et intégration avec [!DNL Experience Manager], le stockage et l’application de traitement.*

L’architecture se compose des parties suivantes :

* **Une couche d’API et d’orchestration** reçoit les requêtes (au format JSON) qui demandent au service de transformer une ressource source en différents rendus. Ces requêtes sont asynchrones et renvoyées avec un ID d’activation équivalent à l’ID du traitement. Les instructions sont purement déclaratives et, pour tous les travaux de traitement standard (par exemple, génération de miniatures, extraction de texte), les consommateurs et consommatrices ne précisent que le résultat souhaité, mais pas les applications qui traitent certains rendus. Les fonctionnalités d’API génériques telles que l’authentification, l’analyse et la limitation de débit sont gérées à l’aide de la passerelle API d’Adobe en lien avec le service et gèrent toutes les requêtes adressées à Runtime [!DNL Adobe I/O]. Le routage de l’application est effectué dynamiquement par la couche d’orchestration. Les clientes et clients définissent des applications personnalisées pour des rendus spécifiques, qui s’accompagnent de leur propre ensemble de paramètres uniques. Il est possible de mettre intégralement en parallèle l’exécution des applications, car il s’agit de fonctions distinctes sans serveur dans Adobe [!DNL `I/O Runtime`].

* **Applications de traitement des ressources** spécialisées dans certains types de formats de fichiers ou de rendus de cible. D’un point de vue conceptuel, une application ressemble au concept pipe d’Unix® : un fichier d’entrée est transformé en un ou plusieurs fichiers de sortie.

* **Une [bibliothèque d’applications courantes](https://github.com/adobe/asset-compute-sdk)** gère les tâches courantes. Par exemple, le téléchargement du fichier source, le chargement des rendus, les rapports d’erreur, l’envoi d’événements et la surveillance. Cette conception garantit que le développement des applications reste simple, conforme au concept sans serveur, avec des interactions limitées au système de fichiers local.

<!-- TBD:

* About the YAML file?
* minimize description to custom applications
* remove all internal stuff (e.g. Photoshop application, API Gateway) from text and diagram
* update diagram to focus on 3rd party custom applications ONLY
* Explain important transactions/handshakes?
* Flow of assets/control? See the illustration on the Nui diagrams wiki.
* Illustrations. See the SVG shared by Alex.
* Exceptions? Limitations? Call-outs? Gotchas?
* Do we want to add what basic processing is not available currently, that is expected by existing AEM customers?
-->
