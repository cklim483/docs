---
ms.openlocfilehash: dcb31ab7b27f6f3ebe89699a3a2e96dd1e78a5db
ms.sourcegitcommit: 872c4751a3fc255671295a5dea6a2081c66b7b71
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/30/2022
ms.locfileid: "145066502"
---
{% data reusables.enterprise-accounts.access-enterprise %} {% data reusables.enterprise-accounts.policies-tab %} {% data reusables.enterprise-accounts.actions-tab %} {%- ifversion ghec or ghes > 3.3 or ghae-issue-5091 %} {% data reusables.enterprise-accounts.actions-runner-groups-tab %}
1. Cliquez sur **Nouveau groupe d’exécuteurs**.
{%- elsif ghes < 3.4 or ghae %} {% data reusables.enterprise-accounts.actions-runners-tab %}
1. Utilisez la liste déroulante **Ajouter nouveau**, puis sélectionnez **Nouveau groupe**.
{%- endif %}
1. Sous « Nom du groupe », tapez un nom pour votre groupe d’exécuteurs.
