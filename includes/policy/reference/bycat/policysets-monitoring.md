---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 09/16/2020
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: f039b88dd629b437c17a2f08fe04faff0a993074
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 09/22/2020
ms.locfileid: "90984472"
---
|Name |BESCHREIBUNG |Richtlinien |Version |
|---|---|---|---|
|[Aktivieren von Azure Monitor für VM-Skalierungsgruppen](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Monitoring/AzureMonitor_VMSS.json) |Hiermit aktivieren Sie Azure Monitor für die VM-Skalierungsgruppen im angegebenen Bereich (Verwaltungsgruppe, Abonnement oder Ressourcengruppe). Akzeptiert den Log Analytics-Arbeitsbereich als Parameter. Hinweis: Wenn „upgradePolicy“ für Ihre Skalierungsgruppe auf „Manuell“ festgelegt ist, müssen Sie die Erweiterung auf alle VMs in der Gruppe anwenden, indem Sie dafür ein Upgrade aufrufen. Führen Sie dazu in der CLI „az vmss update-instances“ aus. |6 |1.0.1 |
|[Azure Monitor für VMs aktivieren](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Monitoring/AzureMonitor_VM.json) |Hiermit aktivieren Sie Azure Monitor für die virtuellen Computer (VMs) in dem angegebenen Bereich (Verwaltungsgruppe, Abonnement oder Ressourcengruppe). Akzeptiert den Log Analytics-Arbeitsbereich als Parameter. |10 |2.0.0 |
