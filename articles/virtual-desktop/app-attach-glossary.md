---
title: 'Windows Virtual Desktop: Glossar zum MSIX-Feature zum Anfügen von Apps – Azure'
description: Ein Glossar der Begriffe und Konzepte des MSIX-Features zum Anfügen von Apps.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: conceptual
ms.date: 08/17/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: f3cc8495f673c8b428aa9e6ace2747a70c5b0847
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 08/18/2020
ms.locfileid: "88556152"
---
# <a name="msix-app-attach-glossary"></a>Glossar des MSIX-Features zum Anfügen von Apps

Dieser Artikel ist eine Liste der Definitionen für wichtige Begriffe und Konzepte im Zusammenhang mit dem MSIX-Feature zum Anfügen von Apps.

## <a name="msix-container"></a>MSIX-Container

Ein MSIX-Container ist der Ort, an dem MSIX-Apps ausgeführt werden. Weitere Informationen finden Sie unter [MSIX-Container](/windows/msix/msix-container).

## <a name="msix-application"></a>MSIX-Anwendung 

Eine in einer MSIX-Datei gespeicherte Anwendung.

## <a name="msix-package"></a>MSIX-Paket 

Ein MSIX-Paket ist eine MSIX-Datei oder -Anwendung.

## <a name="msix-share"></a>MSIX-Freigabe

Eine MSIX0-Freigabe ist eine Netzwerkfreigabe, die erweiterte MSIX-Pakete enthält. MSIX-Freigaben unterstützen SMB 3 oder höher. Anwendungen werden aus dieser MSIX-Freigabe bereitgestellt, ohne dass Anwendungsdateien auf das Systemlaufwerk verschoben werden müssen.

## <a name="repackage"></a>Neu paketieren

Beim Neupaketieren wird eine Nicht-MSIX-Anwendung mithilfe des MSIX-Pakettools (MPT) in MSIX konvertiert. Weitere Informationen finden Sie unter [MSIX-Pakettool: Übersicht](/windows/msix/packaging-tool/tool-overview).

## <a name="expand"></a>Expand

Das Erweitern eines MSIX-Pakets ist ein Prozess, der aus mehreren Schritten besteht. Dabei wird der Inhalt der MSIX-Datei in einer VHD(x)- oder CIM-Datei abgelegt. 

So erweitern Sie ein MSIX-Paket

1. Rufen Sie ein MSIX-Paket (eine MSIX-Datei) ab.
2. Benennen Sie die MSIX-Datei in eine ZIP-Datei um.
3. Entpacken Sie die resultierende ZIP-Datei in einen Ordner.
4. Erstellen Sie eine virtuelle Festplatte (VHD) mit derselben Größe wie der Ordner.
5. Binden Sie die VHD ein.
6. Initialisieren Sie einen Datenträger.
7. Erstellen Sie eine Partition.
8. Formatieren Sie die Partition.
9. Kopieren Sie den entpackten Inhalt in die VHD.
10. Verwenden Sie das MSIXMGR-Tool, um Zugriffssteuerungslisten (ACLs) auf den Inhalt der VHD anzuwenden.
11. Heben Sie die Einbindung der VHD(x) oder [CIM](#cim) auf.

## <a name="upload-an-msix-package"></a>Hochladen eines MSIX-Pakets 

Das Hochladen eines MSIX-Pakets umfasst das Hochladen der VHD(x) oder [CIM](#cim), die ein erweitertes MSIX-Paket enthält, auf die MSIX-Freigabe.

In Windows Virtual Desktop erfolgen Uploads einmalig pro MSIX-Freigabe. Nachdem Sie ein Paket hochgeladen haben, können alle Hostpools im selben Abonnement darauf verweisen.

## <a name="publish-an-msix-package"></a>Veröffentlichen eines MSIX-Pakets

In Windows Virtual Desktop wird beim Veröffentlichen eines MSIX-Pakets dieses mit einer Remote-App oder einem Remotedesktop verknüpft.

## <a name="assign-an-msix-package"></a>Zuweisen eines MSIX-Pakets 

In Windows Virtual Desktop muss ein veröffentlichtes MSIX-Paket einem Active Directory Domain Service (AD DS) oder einem Benutzer bzw. einer Benutzergruppe in Azure Active Directory (Azure AD) zugewiesen werden.

## <a name="staging"></a>Staging

Das Staging umfasst zwei Dinge:

- Einbinden der VHD(x) oder [CIM](#cim) in der VM.
- Benachrichtigen des Betriebssystems, dass das MSIX-Paket für die Registrierung verfügbar ist.

## <a name="registration"></a>Registrierung

Die Registrierung stellt ein bereitgestelltes MSIX-Paket für Ihre Benutzer zur Verfügung. Die Registrierung erfolgt pro Benutzer. Wenn Sie eine App nicht explizit für diesen spezifischen Benutzer registriert haben, kann dieser die App nicht ausführen.

Es gibt zwei Arten der Registrierung: regulär und verzögert.

### <a name="regular-registration"></a>Reguläre Registrierung

Bei der regulären Registrierung ist jede Anwendung, die einem Benutzer zugewiesen ist, vollständig registriert. Die Registrierung erfolgt in der Zeit, in der sich der Benutzer bei der Sitzung anmeldet, was sich auf die Dauer auswirken kann, die dieser benötigt, um mit der Verwendung von Windows Virtual Desktop zu beginnen.

### <a name="delayed-registration"></a>Verzögerte Registrierung

Bei der verzögerten Registrierung ist jede dem Benutzer zugewiesene Anwendung nur teilweise registriert. Die teilweise Registrierung bedeutet, dass die Kachel des Startmenüs und Doppelklick-Dateizuordnungen registriert sind. Die Registrierung erfolgt, während sich der Benutzer bei seiner Sitzung anmeldet, sodass sich dies nur minimal auf die Dauer auswirkt, die benötigt wird, um mit der Verwendung von Windows Virtual Desktop zu beginnen. Die Registrierung wird nur abgeschlossen, wenn der Benutzer die Anwendung im MSIX-Paket ausführt.

Die verzögerte Registrierung ist derzeit die Standardkonfiguration für das MSIX-Feature zum Anfügen von Apps.

## <a name="deregistration"></a>Aufheben der Registrierung

Beim Aufheben der Registrierung wird ein registriertes, aber nicht ausgeführtes MSIX-Paket für einen Benutzer entfernt. Das Aufheben der Registrierung erfolgt, während sich der Benutzer bei seiner Sitzung abmeldet. Während des Aufhebens der Registrierung pusht das MSIX-Feature zum Anfügen von Apps Anwendungsdaten, die für den Benutzer spezifisch sind, in das lokale Benutzerprofil.

## <a name="destage"></a>Staging aufheben

Das Aufheben des Stagings benachrichtigt das Betriebssystem, dass die Einbindung eines MSIX-Pakets oder einer Anwendung, das/die zurzeit nicht ausgeführt wird und für keinen Benutzer bereitgestellt ist, aufgehoben werden kann. Dadurch werden alle Verweise darauf im Betriebssystem entfernt.

## <a name="cim"></a>CIM

.CIM ist eine neue Dateierweiterung, die mit dem CimFS (Composite Image Files System) verknüpft ist. Das Einbinden und Aufheben der Einbindung von CIM-Dateien erfolgt schneller als bei VHD-Dateien. CIM verbraucht außerdem weniger CPU und Arbeitsspeicher als VHD.

In der folgenden Tabelle finden Sie einen Leistungsvergleich zwischen VHD und CimFS. Diese Zahlen waren das Ergebnis eines Testlaufs mit fünfhundert 300-MB-Dateien in jedem der Formate, die auf einem DSv4-Computer ausgeführt wurden.

|  Spezifikationen                          | VHD                    | CimFS   |
|---------------------------------|--------------------------|-----------|
| Durchschnittliche Einbindungszeit     | 356 ms                     | 255 ms      |
| Durchschnittliche Zeit zum Aufheben der Bereitstellung   | 1615 ms                    | 36 ms       |
| Arbeitsspeicherverbrauch | 6 % (von 8 GB)                      | 2 % (von 8 GB)       |
| CPU (Zählerspitze)          | Mehrfaches Erreichen des Maximums | Keine Auswirkungen |

## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen über das MSIX-Feature zum Anfügen von Apps finden Sie in unserer [Übersicht](what-is-app-attach.md) und in den [Häufig gestellten Fragen](app-attach-faq.md). Oder legen Sie direkt mit dem [Einrichten des MSIX-Features zum Anfügen von Apps](app-attach.md) los.

