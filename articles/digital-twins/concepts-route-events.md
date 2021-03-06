---
title: Ereignisrouten
titleSuffix: Azure Digital Twins
description: Erfahren Sie, wie Sie Ereignisse innerhalb von Azure Digital Twins und an andere Azure-Dienste weiterleiten.
author: baanders
ms.author: baanders
ms.date: 3/12/2020
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: 394752792d143a3712d0bb9c50189936f23062f1
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 08/05/2020
ms.locfileid: "87800465"
---
# <a name="route-events-within-and-outside-of-azure-digital-twins"></a>Weiterleiten von Ereignissen innerhalb und außerhalb von Azure Digital Twins

Azure Digital Twins verwendet **Ereignisrouten** zum Senden von Daten an Consumer außerhalb des Diensts. 

Während der Vorschauphase gibt es zwei Hauptfälle für das Senden von Daten aus Azure Digital Twins:
* Der erste Fall ist das Senden von Daten von einem Zwilling im Azure Digital Twins-Graph an einen anderen. Wenn sich beispielsweise eine Eigenschaft in einem digitalen Zwilling ändert, sollten Sie einen anderen digitalen Zwilling entsprechend benachrichtigen und aktualisieren.
* Der zweite Fall ist das Senden von Daten an Downstreamdatendienste zur zusätzlichen Speicherung oder Verarbeitung (auch als *ausgehende Daten* bezeichnet). Beispiel:
  - Ein Krankenhaus möchte möglicherweise Azure Digital Twins-Ereignisdaten an [Time Series Insights (TSI)](../time-series-insights/time-series-insights-update-overview.md) senden, um Zeitreihendaten von Ereignissen im Zusammenhang mit Händewaschen für eine Massenanalyse aufzuzeichnen.
  - Ein Unternehmen, das bereits [Azure Maps](../azure-maps/about-azure-maps.md) verwendet, möchte möglicherweise Azure Digital Twins verwenden, um seine Lösung zu verbessern. Nach dem Einrichten von Azure Digital Twins kann es schnell eine Azure Maps-Instanz aktivieren, Azure Maps-Entitäten in Azure Digital Twins als [digitale Zwillinge](concepts-twins-graph.md) in den Zwillingsgraphen aufnehmen oder leistungsstarke Abfragen ausführen, die sowohl die Daten aus Azure Maps als auch aus Azure Digital Twins nutzen.

Ereignisrouten werden in beiden Szenarios verwendet.

## <a name="about-event-routes"></a>Informationen zu Ereignisrouten

Ereignisrouten ermöglichen Ihnen das Senden von Ereignisdaten von digitalen Zwillingen in Azure Digital Twins an benutzerdefinierte Endpunkte in Ihren Abonnements. Für Endpunkte werden derzeit drei Azure-Dienste unterstützt: [Event Hub](../event-hubs/event-hubs-about.md), [Event Grid](../event-grid/overview.md) und [Service Bus](../service-bus-messaging/service-bus-messaging-overview.md). Jeder dieser Azure-Dienste kann mit anderen Diensten verbunden werden und fungiert als Mittelsmann, der Daten für die von Ihnen benötigte Verarbeitung an endgültige Ziele wie TSI oder Azure Maps sendet.

Das folgende Diagramm veranschaulicht den Fluss von Ereignisdaten durch eine größere IoT-Lösung mit einem Azure Digital Twins-Aspekt:

:::image type="content" source="media/concepts-route-events/routing-workflow.png" alt-text="Weiterleitung von Daten aus Azure Digital Twins über Endpunkte an mehrere Downstreamdienste" border="false":::

Typische Downstreamziele für Ereignisrouten sind Ressourcen wie TSI und Azure Maps sowie Speicher- und Analyselösungen.

### <a name="event-routes-for-internal-digital-twin-events"></a>Ereignisrouten für interne Ereignisse in Bezug auf digitale Zwillinge

In der aktuellen Vorschauversion werden Ereignisrouten auch verwendet, um Ereignisse innerhalb des Zwillingsgraphen zu verarbeiten und Daten von einem digitalen Zwilling an einen anderen zu senden. Dies erfolgt durch Herstellen einer Verbindung zwischen Ereignisrouten und Computeressourcen wie [Azure Functions](../azure-functions/functions-overview.md) über Event Grid. Diese Funktionen definieren dann, wie Zwillinge Ereignisse empfangen und auf diese reagieren sollen. 

Wenn eine Computeressource den Zwillingsgraphen auf Grundlage eines über die Ereignisroute empfangenen Ereignisses ändern möchte, ist es hilfreich, wenn sie im Voraus weiß, welcher Zwilling geändert werden soll. 

Alternativ enthält die Ereignisnachricht auch die ID des Quellzwillings, der diese gesendet hat, sodass die Computeressource Abfragen verwenden oder Beziehungen durchlaufen kann, um einen Zielzwilling für den gewünschten Vorgang zu finden. 

Darüber hinaus muss die Computeressource Sicherheits -und Zugriffsberechtigungen unabhängig einrichten.

Eine Anleitung für das Einrichten einer Azure-Funktion zum Verarbeiten von Ereignissen für digitale Zwillinge finden Sie unter [*Verbinden von Azure Functions-Apps für die Verarbeitung von Daten*](how-to-create-azure-function.md).

## <a name="create-an-endpoint"></a>Erstellen eines Endpunkts

Entwickler müssen erst Endpunkte definieren, um eine Ereignisroute definieren zu können. Ein **Endpunkt** ist ein Ziel außerhalb von Azure Digital Twins, das eine Routenverbindung unterstützt. In der aktuellen Vorschauversion werden folgende Ziele unterstützt:
* Benutzerdefinierte Event Grid-Themen
* Event Hub
* Service Bus

Endpunkte werden mithilfe von Steuerungsebenen-APIs (von der [Azure Digital Twins-CLI](how-to-use-cli.md) unterstützt) oder über das Azure-Portal eingerichtet. Eine Endpunktdefinition enthält die folgenden Angaben:
* Name des Endpunkts
* Endpunkttyp (Event Grid, Event Hub oder Service Bus)
* Primäre und sekundäre Verbindungszeichenfolge für die Authentifizierung 
* Themenpfad des Endpunkts, z. B. *Ihr-Thema.westus2.eventgrid.azure.net*

In der Steuerungsebene sind Endpunkt-APIs für Folgendes verfügbar:
* Endpunkt erstellen
* Abrufen einer Liste mit Endpunkten
* Abrufen von Endpunkten nach Name
* Löschen von Endpunkten nach Name

## <a name="create-an-event-route"></a>Erstellen einer Ereignisroute
 
Ereignisrouten werden in einer Clientanwendung mit dem folgenden [.NET SDK-Aufruf (C#)](how-to-use-apis-sdks.md) erstellt: 

```csharp
await client.EventRoutes.AddAsync("<name-for-the-new-route>", new EventRoute("<endpoint-name>"));
```

* `endpoint-name` identifiziert einen Endpunkt, z. B. einen Event Hub, Event Grid oder Service Bus. Diese Endpunkte müssen vor diesem Registrierungsaufruf in Ihrem Abonnement erstellt und mithilfe von Steuerungsebenen-APIs an Azure Digital Twins angehängt werden.

Das an `EventRoutes.Add` übergebene Ereignisroutenobjekt verwendet auch einen [**Filterparameter**](./how-to-manage-routes-apis-cli.md#filter-events), der zum Einschränken der Arten von Ereignissen für diese Route verwendet werden kann.

Routen können auch über die [Azure Digital Twins-CLI](how-to-use-cli.md) erstellt werden.

### <a name="types-of-event-messages"></a>Arten von Ereignisnachrichten

Verschiedene Arten von Ereignissen in IoT Hub und Azure Digital Twins führen zu verschiedenen Arten von Benachrichtigungsmeldungen, wie unten beschrieben.

[!INCLUDE [digital-twins-notifications.md](../../includes/digital-twins-notifications.md)]

## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen zum Einrichten und Verwalten einer Ereignisroute finden Sie unter:
* [*Verwalten von Endpunkten und Routen in Azure Digital Twins*](how-to-manage-routes-apis-cli.md)

Weitere Informationen zur Verwendung von Azure Functions zum Weiterleiten von Ereignissen innerhalb von Azure Digital Twins finden Sie unter:
* [*Verbinden von Azure Functions-Apps für die Verarbeitung von Daten*](how-to-create-azure-function.md)