---
title: Verwalten der Indizierung in der Azure Cosmos DB-API für MongoDB
description: Dieser Artikel enthält eine Übersicht über die Indizierungsfunktionen von Azure Cosmos DB über die Azure Cosmos DB-API für MongoDB.
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: how-to
ms.date: 08/07/2020
author: timsander1
ms.author: tisande
ms.custom: devx-track-js
ms.openlocfilehash: abd6d6379fba1efac20255ca97e66e6b2d7e72ee
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 09/25/2020
ms.locfileid: "91324407"
---
# <a name="manage-indexing-in-azure-cosmos-dbs-api-for-mongodb"></a>Verwalten der Indizierung in der Azure Cosmos DB-API für MongoDB

Die Azure Cosmos DB-API für MongoDB nutzt die Kernfunktionen für automatische Indexverwaltung von Azure Cosmos DB. Dieser Artikel konzentriert sich auf das Hinzufügen von Indizes mithilfe der Azure Cosmos DB-API für MongoDB. Sie können auch den Artikel [Indizierung in Azure Cosmos DB: Übersicht](index-overview.md) lesen, der für alle APIs relevant ist.

## <a name="indexing-for-mongodb-server-version-36"></a>Indizierung für MongoDB-Serverversion 3.6

Die Azure Cosmos DB-API für die MongoDB-Serverversion 3.6 indiziert automatisch das Feld `_id`, das nicht gelöscht werden kann. Sie erzwingt automatisch die Eindeutigkeit des Felds `_id` durch einen Shardschlüssel. In der Azure Cosmos DB-API für MongoDB sind horizontale Partitionierung und Indizierung separate Konzepte. Sie müssen Ihren Shardschlüssel nicht indizieren. Wie bei jeder anderen Eigenschaft in Ihrem Dokument gilt jedoch: Wenn diese Eigenschaft ein allgemeiner Filter in Ihren Abfragen ist, empfiehlt es sich, den Shardschlüssel zu indizieren.

Um zusätzliche Felder zu indizieren, verwenden Sie die MongoDB-Indexverwaltungsbefehle. Wie bei MongoDB indiziert die Azure Cosmos DB-API für MongoDB automatisch auch nur das Feld `_id`. Diese Standardindizierungsrichtlinie unterscheidet sich von der Azure Cosmos DB-SQL-API, die standardmäßig alle Felder indiziert.

Um eine Abfrage zu sortieren, müssen Sie einen Index für die Felder erstellen, die beim Sortiervorgang verwendet werden.

## <a name="index-types"></a>Indextypen

### <a name="single-field"></a>Einzelfeld

Sie können für jedes einzelne Feld Indizes erstellen. Die Sortierreihenfolge des Einzelfeldindexes ist unerheblich. Der folgende Befehl erstellt einen Index für das Feld `name`:

`db.coll.createIndex({name:1})`

Bei einer Abfrage werden mehrere Einzelfeldindizes verwendet, soweit verfügbar. Sie können pro Container bis zu 500 Einzelfeldindizes erstellen.

### <a name="compound-indexes-mongodb-server-version-36"></a>Zusammengesetzte Indizes (MongoDB-Serverversion 3.6)

Die Azure Cosmos DB-API für MongoDB unterstützt zusammengesetzte Indizes für Konten, die Version 3.6 des Wire-Protokolls verwenden. Sie können bis zu acht Felder in einen zusammengesetzten Index einschließen. **Anders als in MongoDB sollten Sie nur dann einen zusammengesetzten Index erstellen, wenn die Abfrage über mehrere Felder gleichzeitig effizient sortiert werden muss.** Für Abfragen mit mehreren Filtern, die nicht sortiert werden müssen, sollten Sie anstelle eines einzelnen zusammengesetzten Indexes mehrere Einzelfeldindizes erstellen.

Der folgende Befehl erstellt einen zusammengesetzten Index für die Felder `name` und `age`:

`db.coll.createIndex({name:1,age:1})`

Sie können zusammengesetzte Indizes zum gleichzeitigen Sortieren anhand mehrerer Felder verwenden, wie im folgenden Beispiel gezeigt:

`db.coll.find().sort({name:1,age:1})`

Sie können obigen zusammengesetzten Index auch für die effiziente Sortierung einer Abfrage mit der umgekehrten Sortierreihenfolge anhand aller Felder verwenden. Hier sehen Sie ein Beispiel:

`db.coll.find().sort({name:-1,age:-1})`

Allerdings muss die Reihenfolge der Pfade im zusammengesetzten Index exakt mit der Abfrage übereinstimmen. Hier sehen Sie ein Beispiel für eine Abfrage, die einen zusätzlichen zusammengesetzten Index erfordern würde:

`db.coll.find().sort({age:1,name:1})`

> [!NOTE]
> Für geschachtelte Eigenschaften oder Arrays können Sie keine zusammengesetzten Indizes erstellen.

### <a name="multikey-indexes"></a>Indizes mit mehreren Schlüsseln

Azure Cosmos DB erstellt Indizes mit mehreren Schlüsseln, um in Arrays gespeicherte Inhalte zu indizieren. Wenn Sie ein Feld mit einem Arraywert indizieren, indiziert Azure Cosmos DB automatisch alle Elemente im Array.

### <a name="geospatial-indexes"></a>Räumliche Indizes

Viele räumliche Operatoren profitieren von räumlichen Indizes. Derzeit unterstützt die API für MongoDB von Azure Cosmos DB `2dsphere`-Indizes. Die API unterstützt `2d`-Indizes noch nicht.

Hier sehen Sie ein Beispiel für das Erstellen eines räumlichen Indexes für das Feld `location`:

`db.coll.createIndex({ location : "2dsphere" })`

### <a name="text-indexes"></a>Textindizes

Derzeit unterstützt die Azure Cosmos DB-API für MongoDB Textindizes noch nicht. Bei Textsuchabfragen für Zeichenfolgen sollten Sie die [Azure Cognitive Search](https://docs.microsoft.com/azure/search/search-howto-index-cosmosdb)-Integration mit Azure Cosmos DB verwenden.

## <a name="wildcard-indexes"></a>Platzhalterindizes

Sie können Platzhalterindizes verwenden, um Abfragen für unbekannte Felder zu unterstützen. Angenommen, Sie verfügen über eine Sammlung mit Daten über Familien.

Hier sehen Sie einen Auszug eines Beispieldokuments in dieser Sammlung:

```json
  "children": [
     {
         "firstName": "Henriette Thaulow",
         "grade": "5"
     }
  ]
```

Hier sehen Sie ein weiteres Beispiel, diesmal mit etwas anderen Eigenschaften in `children`:

```json
  "children": [
      {
        "familyName": "Merriam",
        "givenName": "Jesse",
        "pets": [
            { "givenName": "Goofy" },
            { "givenName": "Shadow" }
      },
      {
        "familyName": "Merriam",
        "givenName": "John",
      }
  ]
```

In dieser Sammlung können Dokumente viele verschiedene Eigenschaften haben. Wenn Sie alle Daten im Array `children` indizieren möchten, haben Sie zwei Möglichkeiten: Sie können separate Indizes für jede einzelne Eigenschaft erstellen oder einen einzelnen Platzhalterindex für das gesamte Array `children`.

### <a name="create-a-wildcard-index"></a>Erstellen eines Platzhalterindex

Mit dem folgenden Befehl wird ein Platzhalterindex für alle Eigenschaften in `children`erstellt:

`db.coll.createIndex({"children.$**" : 1})`

**Anders als in MongoDB können von Platzhalterindizes mehrere Felder in Abfrageprädikaten unterstützt werden.** Für die Abfrageleistung ist es unerheblich, ob Sie einen einzelnen Platzhalterindex verwenden oder einen separaten Index für jede Eigenschaft erstellen.

Folgende Indextypen lassen sich mit Platzhaltersyntax erstellen:

- Einzelfeld
- Geodaten

### <a name="indexing-all-properties"></a>Indizieren aller Eigenschaften

So erstellen Sie einen Platzhalterindex für alle Felder:

`db.coll.createIndex( { "$**" : 1 } )`

Zu Beginn der Entwicklung kann es hilfreich sein, einen Platzhalterindex für alle Felder zu erstellen. Wenn nach und nach mehr Eigenschaften in einem Dokument indiziert werden, erhöht sich die RU-Gebühr (Request Unit, Anforderungseinheit) für das Schreiben und Aktualisieren des Dokuments. Bei schreibintensiven Workloads empfiehlt es sich daher, Pfade einzeln zu indizieren, anstatt Platzhalterindizes zu verwenden.

### <a name="limitations"></a>Einschränkungen

Folgende Indextypen oder Eigenschaften werden von Platzhalterindizes nicht unterstützt:

- Verbund
- TTL
- Eindeutig

**Anders als in MongoDB** können Platzhalterindizes in der Azure Cosmos DB-API für MongoDB **nicht** für Folgendes verwendet werden:

- Erstellen eines Platzhalterindex, der mehrere spezifische Felder einschließt

`db.coll.createIndex(
    { "$**" : 1 },
    { "wildcardProjection " :
        {
           "children.givenName" : 1,
           "children.grade" : 1
        }
    }
)`

- Erstellen eines Platzhalterindex, der mehrere spezifische Felder ausschließt

`db.coll.createIndex(
    { "$**" : 1 },
    { "wildcardProjection" :
        {
           "children.givenName" : 0,
           "children.grade" : 0
        }
    }
)`

Alternativ können Sie mehrere Platzhalterindizes erstellen.

## <a name="index-properties"></a>Indexeigenschaften

Die folgenden Vorgänge können für Konten mit der Wire-Protokollversion 3.6 und Konten mit früheren Versionen verwendet werden. Sie können noch mehr über [unterstützte Indizes und indizierte Eigenschaften](mongodb-feature-support-36.md#indexes-and-index-properties) erfahren.

### <a name="unique-indexes"></a>Eindeutige Indizes

[Eindeutige Indizes](unique-keys.md) sind nützlich, um zu erzwingen, dass zwei oder mehr Dokumente nicht denselben Wert für indizierte Felder enthalten dürfen.

> [!IMPORTANT]
> Eindeutige Indizes können nur erstellt werden, wenn die Sammlung leer ist (keine Dokumente enthält).

Mit dem folgenden Befehl wird ein eindeutiger Index mit dem Feld `student_id` erstellt:

```shell
globaldb:PRIMARY> db.coll.createIndex( { "student_id" : 1 }, {unique:true} )
{
        "_t" : "CreateIndexesResponse",
        "ok" : 1,
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 4
}
```

Bei partitionierten Sammlungen muss für die Erstellung eines eindeutigen Indexes der Shardschlüssel (Partitionsschlüssel) angegeben werden. Anders ausgedrückt: Alle eindeutigen Indizes einer Sammlung mit Shards sind zusammengesetzte Indizes, bei denen eines der Felder der Partitionsschlüssel ist.

Mit den folgenden Befehlen erstellen Sie die partitionierte Sammlung ```coll``` (der Shardschlüssel lautet ```university```) mit einem eindeutigen Index für die Felder `student_id` und `university`:

```shell
globaldb:PRIMARY> db.runCommand({shardCollection: db.coll._fullName, key: { university: "hashed"}});
{
        "_t" : "ShardCollectionResponse",
        "ok" : 1,
        "collectionsharded" : "test.coll"
}
globaldb:PRIMARY> db.coll.createIndex( { "student_id" : 1, "university" : 1 }, {unique:true})
{
        "_t" : "CreateIndexesResponse",
        "ok" : 1,
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 3,
        "numIndexesAfter" : 4
}
```

Im vorherigen Beispiel wird beim Weglassen der ```"university":1```-Klausel folgende Fehlermeldung zurückgegeben:

```"cannot create unique index over {student_id : 1.0} with shard key pattern { university : 1.0 }"```

### <a name="ttl-indexes"></a>TTL-Indizes

Um den Dokumentablauf in einer bestimmten Sammlung zu aktivieren, müssen Sie einen [TTL-Index](../cosmos-db/time-to-live.md) (Time To Live, Gültigkeitsdauer) erstellen. Ein TTL-Index ist ein Index für das Feld `_ts` mit einem Wert für `expireAfterSeconds`.

Beispiel:

```JavaScript
globaldb:PRIMARY> db.coll.createIndex({"_ts":1}, {expireAfterSeconds: 10})
```

Der obige Befehl löscht alle Dokumente in der Sammlung ```db.coll```, die nicht innerhalb der letzten zehn Sekunden geändert wurden.

> [!NOTE]
> **_ts** ist ein Azure Cosmos DB-spezifisches Feld, auf das nicht über MongoDB-Clients zugegriffen werden kann. Dies ist eine reservierte Eigenschaft (Systemeigenschaft), die den Zeitstempel der letzten Änderung eines Dokuments enthält.

## <a name="track-index-progress"></a>Nachverfolgen des Indizierungsfortschritts

Version 3.6 der Azure Cosmos DB-API für MongoDB unterstützt den Befehl `currentOp()` zum Nachverfolgen des Indizierungsfortschritts für eine Datenbankinstanz. Dieser Befehl gibt ein Dokument zurück, das Informationen zu den aktuell in Bearbeitung befindlichen Vorgängen in einer Datenbankinstanz enthält. Verwenden Sie den Befehl `currentOp`, um alle laufenden Vorgänge in der nativen MongoDB-Datenbank zu verfolgen. In der Azure Cosmos DB-API für MongoDB unterstützt dieser Befehl nur die Nachverfolgung des Indizierungsvorgangs.

Im Folgenden werden einige Beispiele zur Verwendung des Befehls `currentOp` zur Nachverfolgung des Indizierungsfortschritts gezeigt:

* Abrufen des Indexfortschritts für eine Sammlung:

   ```shell
   db.currentOp({"command.createIndexes": <collectionName>, "command.$db": <databaseName>})
   ```

* Abrufen des Indizierungsfortschritts für alle Sammlungen in einer Datenbank:

  ```shell
  db.currentOp({"command.$db": <databaseName>})
  ```

* Abrufen des Indizierungsfortschritts für alle Datenbanken und Sammlungen in einem Azure Cosmos-Konto:

  ```shell
  db.currentOp({"command.createIndexes": { $exists : true } })
  ```

### <a name="examples-of-index-progress-output"></a>Beispiele für die Ausgabe des Indizierungsfortschritts

Die Details zum Indizierungsfortschritt zeigen den Fortschritt des aktuellen Indizierungsvorgangs in Prozent an. Im folgenden Beispiel wird das Format des ausgegebenen Dokuments für verschiedene Phasen des Indizierungsfortschritts veranschaulicht:

- Wenn ein Indizierungsvorgang für die Sammlung „foo“ und die Datenbank „bar“ einen Fortschritt von 60 % aufweist, wird das folgende Dokument ausgegeben. Im Feld `Inprog[0].progress.total` wird 100 als Zielprozentsatz für den Abschluss angezeigt.

   ```json
   {
        "inprog" : [
        {
                ………………...
                "command" : {
                        "createIndexes" : foo
                        "indexes" :[ ],
                        "$db" : bar
                },
                "msg" : "Index Build (background) Index Build (background): 60 %",
                "progress" : {
                        "done" : 60,
                        "total" : 100
                },
                …………..…..
        }
        ],
        "ok" : 1
   }
   ```

- Bei einem Indizierungsvorgang, der erst für die Sammlung „foo“ und die Datenbank „bar“ gestartet wurde, zeigt das Ausgabedokument einen Fortschritt von 0 % an, bis ein messbarer Wert erreicht wurde.

   ```json
   {
        "inprog" : [
        {
                ………………...
                "command" : {
                        "createIndexes" : foo
                        "indexes" :[ ],
                        "$db" : bar
                },
                "msg" : "Index Build (background) Index Build (background): 0 %",
                "progress" : {
                        "done" : 0,
                        "total" : 100
                },
                …………..…..
        }
        ],
       "ok" : 1
   }
   ```

- Wenn der aktive Indizierungsvorgang abgeschlossen wird, zeigt das Ausgabedokument leere `inprog`-Vorgänge an.

   ```json
   {
      "inprog" : [],
      "ok" : 1
   }
   ```

## <a name="background-index-updates"></a>Indexaktualisierungen im Hintergrund

Unabhängig von dem für die **Hintergrund**-Indexeigenschaft angegebenen Wert werden Indexaktualisierungen immer im Hintergrund durchgeführt. Da Indexaktualisierungen Anforderungseinheiten (Request Units, RUs) mit einer niedrigeren Priorität als andere Datenbankvorgänge nutzen, führen Indexänderungen nicht zu Ausfallzeiten bei Schreib-, Update- oder Löschvorgängen.

Das Hinzufügen eines neuen Indexes hat keine Auswirkung auf die Leseverfügbarkeit. Abfragen verwenden neue Indizes erst dann, wenn die Indextransformation abgeschlossen ist. Während der Indextransformation werden von der Abfrage-Engine weiterhin vorhandene Indizes verwendet, sodass Sie während der Indextransformation eine ähnliche Leseleistung beobachten werden wie vor dem Einleiten der Indexänderung. Beim Hinzufügen neuer Indizes besteht auch kein Risiko, unvollständige oder inkonsistente Abfrageergebnisse zu erhalten.

Wenn Indizes entfernt und sofort Abfragen ausgeführt werden, die nach den gelöschten Indizes filtern, können die Ergebnisse inkonsistent und unvollständig sein, solange die Indextransformation nicht abgeschlossen ist. Wenn Sie Indizes entfernen, gewährleistet die Abfrage-Engine keine konsistenten oder vollständigen Ergebnisse, falls Abfragen nach diesen soeben entfernten Indizes filtern. Die meisten Entwickler löschen keine Indizes und versuchen dann sofort, Abfragen dafür auszuführen, sodass diese Situation in der Praxis eher unwahrscheinlich ist.

> [!NOTE]
> Sie können [den Indizierungsfortschritt nachverfolgen](#track-index-progress).

## <a name="migrate-collections-with-indexes"></a>Migrieren von Sammlungen mit Indizes

Derzeit ist die Erstellung von eindeutigen Indizes nur möglich, wenn die Sammlung keine Dokumente enthält. Bei gängigen MongoDB-Migrationstools wird versucht, die eindeutigen Indizes nach dem Importieren der Daten zu erstellen. Um dieses Problem zu umgehen, können Sie die entsprechenden Sammlungen und eindeutigen Indizes manuell erstellen, damit das Migrationstool dies nicht versucht. (Sie können dieses Verhalten für ```mongorestore``` erzielen, indem Sie das `--noIndexRestore`-Flag an der Befehlszeile verwenden.)

## <a name="indexing-for-mongodb-version-32"></a>Indizierung für MongoDB-Version 3.2

Bei Azure Cosmos-Konten, die mit Version 3.2 des MongoDB-Wire-Protokolls kompatibel sind, weichen die verfügbaren Indizierungsfeatures und Standardwerte ab. Sie können [die Version Ihres Kontos überprüfen](mongodb-feature-support-36.md#protocol-support). Sie können ein Upgrade auf die Version 3.6 durchführen, indem Sie eine [Supportanfrage](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) einreichen.

Wenn Sie Version 3.2 verwenden, beachten Sie die in diesem Abschnitt erläuterten wichtigen Unterschiede zu Version 3.6.

### <a name="dropping-default-indexes-version-32"></a>Löschen der Standardindizes (Version 3.2)

Anders als bei der Version 3.6 der Azure Cosmos DB-API für MongoDB werden in der Version 3.2 standardmäßig alle Eigenschaften indiziert. Mit dem folgenden Befehl können Sie diese Standardindizes für eine Sammlung (```coll```) löschen:

```JavaScript
> db.coll.dropIndexes()
{ "_t" : "DropIndexesResponse", "ok" : 1, "nIndexesWas" : 3 }
```

Nachdem Sie die Standardindizes gelöscht haben, können Sie zusätzliche Indizes wie in Version 3.6 hinzufügen.

### <a name="compound-indexes-version-32"></a>Zusammengesetzte Indizes (Version 3.2)

Zusammengesetzte Indizes enthalten Verweise auf mehrere Felder eines Dokuments. Wenn Sie einen zusammengesetzten Index erstellen möchten, führen Sie ein Upgrade auf Version 3.6 durch, indem Sie eine [Supportanfrage](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) erstellen.

### <a name="wildcard-indexes-version-32"></a>Platzhalterindizes (Version 3.2)

Wenn Sie einen Platzhalterindex erstellen möchten, erstellen Sie eine [Supportanfrage](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade), um ein Upgrade auf die Version 3.6 durchzuführen.

## <a name="next-steps"></a>Nächste Schritte

* [Indizierung in Azure Cosmos DB](../cosmos-db/index-policy.md)
* [Festlegen einer Gültigkeitsdauer für den automatischen Ablauf von Daten in Azure Cosmos DB](../cosmos-db/time-to-live.md)
* Informationen zur Beziehung zwischen Partitionierung und Indizierung finden Sie im Artikel [Abfragen eines Azure Cosmos-Containers](how-to-query-container.md).
