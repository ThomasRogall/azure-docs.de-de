---
title: Festlegen des Layouts von Dateien für die Konvertierung
description: Empfehlungen zur optimalen Platzierung von Dateien im Eingabecontainer.
author: MalcolmTyrrell
ms.author: matyrr
ms.date: 09/03/2020
ms.topic: how-to
ms.openlocfilehash: 8acd19408c328ef3f534ba7bc41c96e395f768be
ms.sourcegitcommit: 655e4b75fa6d7881a0a410679ec25c77de196ea3
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 09/07/2020
ms.locfileid: "89510891"
---
# <a name="laying-out-files-for-conversion"></a>Festlegen des Layouts von Dateien für die Konvertierung

Um eine Ressource ordnungsgemäß zu verarbeiten, muss der Konvertierungsdienst alle Eingabedateien finden können.
Diese bestehen aus der zu konvertierenden Hauptressourcendatei und in der Regel aus anderen Dateien, auf die in den Pfaden innerhalb der Ressourcendatei verwiesen wird.
Der Anforderung zum Konvertieren einer Ressource werden zwei Parameter übergeben, die bestimmen, wie der Konvertierungsdienst diese Dateien findet: Der `input.folderPath` (optional) und der `input.inputAssetPath`.
Sie sind vollständig auf der Seite [Verwenden der REST-API für die Modellkonvertierung](conversion-rest-api.md) dokumentiert.
Für das Dateienlayout ist wichtig zu beachten, dass der `folderPath` einen kompletten Satz von Dateien festlegt, die bei der Verarbeitung der Ressource für den Konvertierungsdienst verfügbar sind.

## <a name="placing-files-so-they-can-be-found"></a>Platzieren von Dateien, sodass diese gefunden werden können

Wenn eine Quellressource externe Dateien nutzt, werden Pfade zu diesen Dateien in der Ressource gespeichert.
Der Konvertierungsdienst muss diese Pfade in ein Dateisystem übersetzen, das sich vom ursprünglichen Dateisystem der Ressource unterscheidet.
Wenn die Pfade als relative Pfade gespeichert werden und die relative Position zwischen der Quellressource und der Datei, auf die sie verweist, unverändert ist, ist es für den Konvertierungsdienst leicht, die referenzierte Datei zu finden.

> [!Note]
> Sie sollten Dateien im Eingabecontainer platzieren, damit die relativen Speicherorte der Dateien dieselben sind wie beim Erstellen der Ressource.

## <a name="finding-textures"></a>Suchen von Texturen

Aufgrund der zahlreichen Möglichkeiten, Ressourcen zu generieren, muss der Konvertierungsdienst flexibel sein.
Insbesondere muss er Situationen bewältigen, in denen die Pfade in der Ressource und im Speicherort von Texturen nicht exakt zueinander passen.
Ein Beispiel hierfür ist das Generieren von Ressourcen, die absolute Pfade enthalten, da diese Pfade niemals dem vom Konvertierungsdienst verwendeten Dateisystem entsprechen.
Um unter anderem diese Situation zu bewältigen, verwenden wir einen Ansatz mit bestmöglicher Leistung für die Suche nach Texturen.

Der Algorithmus für die Suche nach Texturen lautet wie folgt: Vorausgesetzt, ein Pfad ist in einer Ressource gespeichert, suchen Sie das längste Unterpfadsuffix, das bei Verwendung als relativer Pfad vom Speicherort zur Quellressource auf eine vorhandene Datei weist.
Wenn kein solcher Unterpfad (einschließlich des gesamten Pfads) auf eine Datei weist, wird die Textur als fehlend betrachtet.

Sehen Sie sich das folgende erfundene Dateisystem an: 
```
G:\CONVERSION
├───Assets
│   │   myAsset.fbx
│   │   myTexture.png            <- A
│   │
│   └───Textures
│       │   myTexture.png        <- B
│       │
│       └───MyAssetTextures
│               myTexture.png    <- C
│
└───Textures
    │   myTexture.png            <- D
    │
    └───MyAssetTextures
            myTexture.png        <- E
```
Wenn „myAsset.fbx“ auf eine Textur mit dem relativen Pfad `..\Textures\MyAssetTextures\myTexture.png` verweist, verwendet der Konvertierungsdienst Datei „E“. Tatsächlich kann er alle Dateien „A“, „C“ und „E“ verwenden, wenn sie vorhanden sind, wobei Datei „E“ bevorzugt wird, da sie mit dem längsten Suffix gefunden wurde.
Die Dateien „B“ und „D“ werden niemals verwendet, da `Textures\myTexture.png` nicht Teil eines Suffixes des gespeicherten Pfads ist.
Wenn die Ressource stattdessen die Pfade `H:\Foo\Bar\Textures\MyAssetTextures\myTexture.png` oder `..\..\..\Foo\Bar\Textures\MyAssetTextures\myTexture.png` enthielte, kann der Konvertierungsdienst die Dateien „A“ und „C“ finden, wenn sie vorhanden sind (wobei „C“ bevorzugt wird). „E“ kann jedoch nicht auf diese Weise gefunden werden, und die Datei muss verschoben werden.
Dies könnte durch Verschieben des Ordners „Textures“ neben die Ressource korrigiert werden.

> [!Note]
> Wenn Texturen nicht gefunden werden, kann zur Problembehandlung sichergestellt werden, dass die Ressource ein Element ist, das einer Teilstruktur gleich geordnet ist, die die Texturen enthält.

## <a name="next-steps"></a>Nächste Schritte

- [Modellkonvertierung](model-conversion.md)
