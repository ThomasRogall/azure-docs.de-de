---
title: Installation des Azure Cosmos DB-Emulators für die lokale Entwicklung
description: Erfahren Sie, wie der Azure Cosmos DB-Emulator für Windows, Linux, macOS und Docker-Umgebungen unter Windows installiert und verwendet wird. Mit dem Emulator können Sie Ihre Anwendung kostenlos lokal entwickeln und testen, ohne ein Azure-Abonnement erstellen zu müssen.
ms.service: cosmos-db
ms.topic: how-to
author: markjbrown
ms.author: mjbrown
ms.date: 09/17/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 16448706b7167f55f31c7603676010e4ad30166f
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 09/22/2020
ms.locfileid: "90985851"
---
# <a name="install-and-use-the-azure-cosmos-emulator-for-local-development-and-testing"></a>Installieren und Verwenden des Azure Cosmos-Emulators für lokale Entwicklung und Tests

Der Azure Cosmos-Emulator stellt eine lokale Umgebung bereit, die zu Entwicklungszwecken den Dienst Azure Cosmos DB emuliert. Mit dem Azure Cosmos-Emulator können Sie Ihre Anwendung lokal entwickeln und testen, ohne ein Azure-Abonnement erstellen zu müssen oder sonstige Kosten zu verursachen. Wenn Sie mit der Funktionsweise der Anwendung im Azure Cosmos-Emulator zufrieden sind, können Sie auf ein Azure Cosmos-Konto in der Cloud umsteigen. Laden Sie zunächst die neueste Version des [Azure Cosmos-Emulators](https://aka.ms/cosmosdb-emulator) auf Ihren lokalen Computer herunter, und installieren Sie sie. In diesem Artikel erfahren Sie, wie der Azure Cosmos DB-Emulator für Windows, Linux, macOS und in Docker-Umgebungen unter Windows installiert und verwendet wird.

Sie können mit dem Azure Cosmos-Emulator Anwendungen mit Konten für die APIs [SQL](local-emulator.md#sql-api), [Cassandra](local-emulator.md#cassandra-api), [MongoDB](local-emulator.md#azure-cosmos-dbs-api-for-mongodb), [Gremlin](local-emulator.md#gremlin-api) und [Tabelle](local-emulator.md#table-api) entwickeln. Derzeit unterstützt der Daten-Explorer im Emulator nur die Anzeige von SQL-Daten. Die Daten, die mit MongoDB-, Gremlin/Graph- und Cassandra-Clientanwendungen erstellt werden, können gegenwärtig nicht angezeigt werden. Weitere Informationen finden Sie unter [Herstellen einer Verbindung mit dem Emulatorendpunkt](#connect-with-emulator-apis) über verschiedene APIs.

## <a name="how-does-the-emulator-work"></a>Wie funktioniert der Emulator?

Der Azure Cosmos-Emulator stellt eine originalgetreue Emulation des Azure Cosmos DB-Diensts bereit. Er unterstützt die gleichen Funktionen wie Azure Cosmos DB, was das Erstellen und Abfragen von Daten, Bereitstellen und Skalieren von Containern sowie das Ausführen von gespeicherten Prozeduren und Triggern einschließt. Sie können Anwendungen mithilfe des Azure Cosmos-Emulators entwickeln und testen und diese durch Aktualisierung des Azure Cosmos DB-Verbindungsendpunkts global in Azure bereitstellen.

Die Emulation des Azure Cosmos DB-Diensts entspricht exakt dem Dienst, die Implementierung des Emulators weist allerdings einige Unterschiede zum Dienst auf. Der Emulator verwendet Standardkomponenten des Betriebssystems, wie z.B. das lokale Dateisystem für Persistenz und den HTTPS-Protokollstapel für Konnektivität. Bei Verwendung des Emulators werden Funktionen, die die Azure-Infrastruktur benötigen, sind nicht unterstützt. Hierzu gehören beispielsweise die globale Replikation, Lese-/Schreibvorgänge mit Latenz im einstelligen Millisekundenbereich und optimierbare Konsistenzebenen.

Sie können Daten mit dem [Migrationstool für Azure Cosmos DB-Daten](https://github.com/azure/azure-documentdb-datamigrationtool) zwischen dem Azure Cosmos-Emulator und dem Dienst Azure Cosmos DB migrieren.

## <a name="differences-between-the-emulator-and-the-cloud-service"></a>Unterschiede zwischen Emulator und Clouddienst

Da der Azure Cosmos-Emulator eine emulierte Umgebung bereitstellt, die auf einer lokalen Entwicklerarbeitsstation läuft, gibt es einige funktionelle Unterschiede zwischen dem Emulator und einem Azure Cosmos-Konto in der Cloud:

* Der Bereich **Daten-Explorer** im Emulator unterstützt derzeit nur SQL-API-Clients vollständig. Die Ansicht **Daten-Explorer** und Vorgänge für Azure Cosmos DB-APIs wie MongoDB-, Tabellen-, Graph- und Cassandra-APIs werden nicht vollständig unterstützt.

* Der Emulator unterstützt nur ein einziges festgelegtes Konto und einen bekannten Primärschlüssel. Bei Verwenden des Azure Cosmos-Emulators können Sie den Schlüssel nicht neu generieren. Sie können jedoch den Standardschlüssel mit der [Befehlszeilenoption](emulator-command-line-parameters.md) ändern.

* Mit dem Emulator können Sie ein Azure Cosmos-Konto nur im Modus [Bereitgestellter Durchsatz](set-throughput.md) erstellen. Der Modus [Serverlos](serverless.md) wird derzeit nicht unterstützt.

* Der Emulator ist kein skalierbarer Speicherdienst und unterstützt keine große Anzahl von Containern. Bei Verwenden des Azure Cosmos-Emulators können Sie standardmäßig bis zu 25 Container mit fester Größe mit 400 RU/s (nur unterstützt bei Verwenden von Azure Cosmos DB SDKs) oder 5 unbegrenzte Container erstellen. Weitere Informationen zum Ändern dieses Werts finden Sie im Artikel [Festlegen des PartitionCount-Werts]emulator-command-line-parameters.md#set-partitioncount).

* Der Emulator bietet im Vergleich zum Clouddienst keine unterschiedlichen [Azure Cosmos DB-Konsistenzebenen](consistency-levels.md).

* Der Emulator bietet keine [Replikation in mehrere Regionen](distribute-data-globally.md).

* Ggf. ist Ihre Kopie des Azure Cosmos-Emulators nicht auf dem neuesten Stand der Änderungen im Dienst Azure Cosmos DB. Verwenden Sie daher stets den [Azure Cosmos DB-Kapazitätsplaner](estimate-ru-with-capacity-planner.md), um den erforderlichen Durchsatz (RUs, Request Units = Anforderungseinheiten) für Ihre Anwendung richtig einzuschätzen.

* Der Emulator unterstützt für die ID-Eigenschaft maximal 254 Zeichen.

## <a name="install-the-emulator"></a>Installieren des Emulators

Vergewissern Sie sich vor der Installation des Emulators, dass die folgenden Hardware- und Softwareanforderungen erfüllt sind:

* Softwareanforderungen:
  * Derzeit werden als Hostbetriebssystem Windows Server 2012 R2, Windows Server 2016, Windows Server 2019 oder Windows 8 und Windows 10 unterstützt. Hostbetriebssysteme mit aktiviertem Active Directory werden derzeit nicht unterstützt.
  * 64-Bit-Betriebssystem

* Mindestanforderungen für Hardware:
  * 2 GB RAM
  * 10 GB verfügbarer Speicherplatz

* Zum Installieren, Konfigurieren und Ausführen des Azure Cosmos-Emulators benötigen Sie Administratorrechte auf dem Computer. Der Emulator fügt ein Zertifikat hinzu und legt auch die Firewallregeln fest, um seine Dienste auszuführen. Daher benötigt der Emulator Administratorrechte, um solche Vorgänge ausführen zu können.

Laden Sie zunächst die neueste Version des [Azure Cosmos-Emulators](https://aka.ms/cosmosdb-emulator) auf Ihren lokalen Computer herunter, und installieren Sie sie. Wenn Sie bei der Installation des Emulators auf Probleme stoßen, lesen Sie den Artikel zum [Behandeln von Emulatorproblemen](troubleshoot-local-emulator.md).

Je nach Ihren Systemanforderungen können Sie den Emulator, wie in den nächsten Abschnitten dieses Artikels beschrieben, unter [Windows](#run-on-windows), [Docker für Windows](#run-on-windows-docker), Linux oder [macOS](#run-on-linux-macos) ausführen.

## <a name="check-for-emulator-updates"></a>Prüfen auf Emulatorupdates

Jede Version des Emulators enthält eine Reihe von Featureupdates oder Fehlerkorrekturen. Informationen zu den verfügbaren Versionen finden Sie im Artikel mit den [Anmerkungen zur Emulatorversion](local-emulator-release-notes.md).

Wenn Sie nach der Installation die Standardeinstellungen verwendet haben, werden die dem Emulator entsprechenden Daten am Speicherort %LOCALAPPDATA%\CosmosDBEmulator gespeichert. Sie können einen anderen Speicherort konfigurieren, indem Sie die optionalen Datenpfadeinstellungen, also `/DataPath=PREFERRED_LOCATION` als [Befehlszeilenparameter](emulator-command-line-parameters.md), verwenden. Der Zugriff auf Daten, die in einer bestimmten Version des Azure Cosmos-Emulators erstellt wurden, ist bei Verwendung einer anderen Version nicht gewährleistet. Wenn Sie die Daten langfristig beibehalten müssen, wird empfohlen, sie in einem Azure Cosmos-Konto statt im Azure Cosmos-Emulator zu speichern.

## <a name="use-the-emulator-on-windows"></a><a id="run-on-windows"></a>Verwenden des Emulators unter Windows

Der Azure Cosmos-Emulator wird standardmäßig am Speicherort `C:\Program Files\Azure Cosmos DB Emulator` installiert. Wählen Sie zum Starten des Azure Cosmos-Emulators unter Windows die Schaltfläche **Start** aus, oder drücken Sie die WINDOWS-TASTE. Beginnen Sie mit der Eingabe von **Azure Cosmos-Emulator**, und wählen Sie den Emulator in der Liste mit den Anwendungen aus.

:::image type="content" source="./media/local-emulator/database-local-emulator-start.png" alt-text="Wählen Sie die Schaltfläche „Start“ aus, oder drücken Sie die WINDOWS-TASTE. Beginnen Sie mit der Eingabe von „Azure Cosmos-Emulator“, und wählen Sie den Emulator in der Liste der Anwendungen aus.":::

Nach Start des Emulators wird im Infobereich der Windows-Taskleiste ein Symbol angezeigt. Darüber wird der Daten-Explorer von Azure Cosmos automatisch an der URL `https://localhost:8081/_explorer/index.html` in Ihrem Browser geöffnet.

:::image type="content" source="./media/local-emulator/database-local-emulator-taskbar.png" alt-text="Wählen Sie die Schaltfläche „Start“ aus, oder drücken Sie die WINDOWS-TASTE. Beginnen Sie mit der Eingabe von „Azure Cosmos-Emulator“, und wählen Sie den Emulator in der Liste der Anwendungen aus.":::

Sie können den Emulator auch über die Befehlszeile oder PowerShell-Befehle starten und beenden. Weitere Informationen finden Sie im Artikel mit der [Referenz zum Befehlszeilentool](emulator-command-line-parameters.md).

Der Azure Cosmos-Emulator wird standardmäßig auf dem lokalen Computer (localhost) ausgeführt und lauscht an Port 8081. Die Adresse wird als `https://localhost:8081/_explorer/index.html` angezeigt. Wenn Sie den Explorer schließen und ihn später erneut öffnen möchten, können Sie entweder die URL in Ihrem Browser öffnen oder (wie unten gezeigt) über den Azure Cosmos-Emulator im Windows-Taskleistensymbol starten.

:::image type="content" source="./media/local-emulator/database-local-emulator-data-explorer-launcher.png" alt-text="Wählen Sie die Schaltfläche „Start“ aus, oder drücken Sie die WINDOWS-TASTE. Beginnen Sie mit der Eingabe von „Azure Cosmos-Emulator“, und wählen Sie den Emulator in der Liste der Anwendungen aus.":::

## <a name="use-the-emulator-on-docker-for-windows"></a><a id="run-on-windows-docker"></a>Verwenden des Emulators in Docker für Windows

Sie können den Azure Cosmos-Emulator im Windows-Docker-Container ausführen. Informationen zum Befehl „docker pull“ finden Sie auf [Docker Hub](https://hub.docker.com/r/microsoft/azure-cosmosdb-emulator/). Auf [GitHub](https://github.com/Azure/azure-cosmos-db-emulator-docker) erfahren Sie mehr zu `Dockerfile` und weitere Informationen. Derzeit funktioniert der Emulator nicht in Docker für Oracle Linux. Verwenden Sie die folgenden Anweisungen, um den Emulator in Docker für Windows auszuführen:

1. Nachdem [Docker für Windows](https://www.docker.com/docker-windows) installiert wurde, wechseln Sie zu Windows-Containern, indem Sie mit der rechten Maustaste auf der Symbolleiste auf das Docker-Symbol klicken und **Switch to Windows containers** (Zu Windows-Containern wechseln) auswählen.

1. Rufen Sie als Nächstes das Emulator-Image per Pullvorgang aus dem Docker-Hub ab, indem Sie in Ihrer bevorzugten Shell den folgenden Befehl ausführen.

   ```bash
   docker pull mcr.microsoft.com/cosmosdb/windows/azure-cosmos-emulator
   ```

1. Um das Image zu starten, führen Sie die folgenden Befehle abhängig von der Befehlszeile oder PowerShell-Umgebung aus:

   # <a name="command-line"></a>[Befehlszeile](#tab/cli)

   ```cmd

   md %LOCALAPPDATA%\CosmosDBEmulator\bind-mount

   docker run --name azure-cosmosdb-emulator --memory 2GB --mount "type=bind,source=%LOCALAPPDATA%\CosmosDBEmulator\bind-mount,destination=C:\CosmosDB.Emulator\bind-mount" --interactive --tty -p 8081:8081 -p 8900:8900 -p 8901:8901 -p 8902:8902 -p 10250:10250 -p 10251:10251 -p 10252:10252 -p 10253:10253 -p 10254:10254 -p 10255:10255 -p 10256:10256 -p 10350:10350 mcr.microsoft.com/cosmosdb/windows/azure-cosmos-emulator
   ```

   # <a name="powershell"></a>[PowerShell](#tab/powershell)

   ```powershell

   md $env:LOCALAPPDATA\CosmosDBEmulator\bind-mount 2>null

   docker run --name azure-cosmosdb-emulator --memory 2GB --mount "type=bind,source=$env:LOCALAPPDATA\CosmosDBEmulator\bind-mount,destination=C:\CosmosDB.Emulator\bind-mount" --interactive --tty -p 8081:8081 -p 8900:8900 -p 8901:8901 -p 8902:8902 -p 10250:10250 -p 10251:10251 -p 10252:10252 -p 10253:10253 -p 10254:10254 -p 10255:10255 -p 10256:10256 -p 10350:10350 mcr.microsoft.com/cosmosdb/windows/azure-cosmos-emulator

   ```

   Die Antwort kann wie folgt aussehen:

   ```cmd
   Starting emulator
   Emulator Endpoint: https://172.20.229.193:8081/
   Master Key: C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==
   Exporting SSL Certificate
   You can import the SSL certificate from an administrator command prompt on the host by running:
   cd /d %LOCALAPPDATA%\CosmosDBEmulatorCert
   powershell .\importcert.ps1
   --------------------------------------------------------------------------------------------------
   Starting interactive shell
   ```
   ---

   > [!NOTE]
   > Wenn bei der Ausführung des Befehls `docker run` ein Portkonfliktfehler auftritt (d. h. der angegebene Port bereits verwendet wird), übergeben Sie einen benutzerdefinierten Port, indem Sie die Portnummern ändern. Beispielsweise können Sie den Parameter „-p 8081:8081“ in „-p 443:8081“ ändern.

1. Verwenden Sie nun den Emulatorendpunkt und Primärschlüssel aus der Antwort, und importieren Sie das TLS-/SSL-Zertifikat in Ihren Host. Führen Sie an einer Eingabeaufforderung mit Administratorberechtigungen folgende Schritte aus, um das TLS-/SSL-Zertifikat zu importieren:

   # <a name="command-line"></a>[Befehlszeile](#tab/cli)

   ```cmd
   cd  %LOCALAPPDATA%\CosmosDBEmulator\bind-mount
   powershell .\importcert.ps1
   ```

   # <a name="powershell"></a>[PowerShell](#tab/powershell)

   ```powershell
   cd $env:LOCALAPPDATA\CosmosDBEmulator\bind-mount
   .\importcert.ps1
   ```
   ---

1. Wenn Sie die interaktive Shell schließen, nachdem der Emulator gestartet wurde, wird der Container des Emulators geschlossen. Navigieren Sie in Ihrem Browser zu folgender URL, um den Daten-Explorer erneut zu öffnen. Der Emulator-Endpunkt wird in der oben gezeigten Antwortnachricht bereitgestellt.

   `https://<emulator endpoint provided in response>/_explorer/index.html`

Wenn Sie über eine in einem Docker-Container für Linux ausgeführte .NET-Clientanwendung verfügen und den Azure Cosmos-Emulator auf einem Hostcomputer ausführen, befolgen Sie die Anweisungen im nächsten Abschnitt, um das Zertifikat in den Docker-Container für Linux zu importieren.

### <a name="regenerate-the-emulator-certificates-when-running-on-a-docker-container"></a>Erneutes Generieren der Emulatorzertifikate bei Ausführung in einem Docker-Container

Wenn der Emulator in einem Docker-Container ausgeführt wird, werden die dem Emulator zugeordneten Zertifikate jedes Mal neu generiert, wenn Sie den jeweiligen Container beenden und neu starten. Aus diesem Grund müssen Sie die Zertifikate nach jedem Containerstart neu importieren. Um diese Einschränkung zu umgehen, können Sie eine Docker Compose-Datei verwenden, um den Docker-Container an eine bestimmte IP-Adresse und ein Containerimage zu binden.

Sie können z. B. die folgende Konfiguration innerhalb der Docker Compose-Datei verwenden. Achten Sie darauf, sie entsprechend Ihren Anforderungen zu formatieren: 

```yml
version: '2.4' # Do not upgrade to 3.x yet, unless you plan to use swarm/docker stack: https://github.com/docker/compose/issues/4513

networks:
  default:
    external: false
    ipam:
      driver: default
      config:
        - subnet: "172.16.238.0/24"

services:

  # First create a directory that will hold the emulator traces and certificate to be imported
  # set hostDirectory=C:\emulator\bind-mount
  # mkdir %hostDirectory%

  cosmosdb:
    container_name: "azurecosmosemulator"
    hostname: "azurecosmosemulator"
    image: 'mcr.microsoft.com/cosmosdb/windows/azure-cosmos-emulator'
    platform: windows
    tty: true
    mem_limit: 3GB
    ports:
        - '8081:8081'
        - '8900:8900'
        - '8901:8901'
        - '8902:8902'
        - '10250:10250'
        - '10251:10251'
        - '10252:10252'
        - '10253:10253'
        - '10254:10254'
        - '10255:10255'
        - '10256:10256'
        - '10350:10350'
    networks:
      default:
        ipv4_address: 172.16.238.246
    volumes:
        - '${hostDirectory}:C:\CosmosDB.Emulator\bind-mount'
```

## <a name="use-the-emulator-on-linux-or-macos"></a><a id="run-on-linux-macos"></a>Verwenden des Emulators unter Linux oder macOS

Der Azure Cosmos-Emulator kann derzeit nur unter Windows ausgeführt werden. Bei Verwenden von Linux oder macOS können Sie den Emulator in einer Windows-VM ausführen, die in einem Hypervisor wie Parallels oder VirtualBox gehostet wird.

> [!NOTE]
> Bei jedem Neustart des virtuellen Windows-Computers, der in einem Hypervisor gehostet wird, müssen Sie das Zertifikat neu importieren, da sich die IP-Adresse des virtuellen Computers ändert. Der Import des Zertifikats ist nicht erforderlich, wenn Sie den virtuellen Computer so konfiguriert haben, dass die IP-Adresse beibehalten wird.

Führen Sie die folgenden Schritte aus, um den Emulator in Linux- oder macOS-Umgebungen zu verwenden:

1. Führen Sie den folgenden Befehl auf dem virtuellen Windows-Computer aus, und notieren Sie sich die IPv4-Adresse:

   ```cmd
   ipconfig.exe
   ```

1. Ändern Sie in der Anwendung die Endpunkt-URL so, dass anstelle von `localhost` die von `ipconfig.exe` zurückgegebene IPv4-Adresse verwendet wird.

1. Starten Sie in der Windows-VM den Azure Cosmos-Emulator mit den folgenden Optionen über die Befehlszeile. Einzelheiten zu den von der Befehlszeile unterstützten Parametern finden Sie in der [Referenz zum Befehlszeilentool für den Emulator](emulator-command-line-parameters.md):

   ```cmd
   Microsoft.Azure.Cosmos.Emulator.exe /AllowNetworkAccess /Key=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM +4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==
   ```

1. Schließlich müssen Sie den Zertifikatsvertrauensprozess zwischen der Anwendung, die in der Linux- oder Mac-Umgebung ausgeführt wird, und dem Emulator auflösen. Sie können eine der beiden folgenden Optionen wählen, um das Zertifikat aufzulösen:

   1. [Importieren des TLS-/SSL-Zertifikats des Emulators in die Linux- oder Mac-Umgebung](#import-certificate) oder
   2. [Deaktivieren der TLS-/SSL-Überprüfung in der Anwendung](#disable-ssl-validation)

### <a name="option-1-import-the-emulator-tlsssl-certificate"></a><a id="import-certificate"></a>Option 1: Importieren des TLS-/SSL-Zertifikats des Emulators

In den folgenden Abschnitten wird gezeigt, wie Sie das TLS-/SSL-Zertifikat des Emulators in eine Linux- oder macOS-Umgebung importieren.

#### <a name="linux-environment"></a>Linux-Umgebung

Unter Linux verwendet .NET OpenSSL für die Überprüfung:

1. [Exportieren Sie das Zertifikat im PFX-Format](local-emulator-export-ssl-certificates.md#export-emulator-certificate). Die Option „PFX“ ist verfügbar, wenn Sie den privaten Schlüssel exportieren.

1. Kopieren Sie die PFX-Datei in Ihre Linux-Umgebung.

1. Konvertieren Sie die PFX-Datei in eine CRT-Datei.

   ```bash
   openssl pkcs12 -in YourPFX.pfx -clcerts -nokeys -out YourCTR.crt
   ```

1. Kopieren Sie die CRT-Datei in den Ordner, der in Ihrer Linux-Distribution benutzerdefinierte Zertifikate enthält. In Debian-Distributionen befindet er sich üblicherweise unter `/usr/local/share/ca-certificates/`.

   ```bash
   cp YourCTR.crt /usr/local/share/ca-certificates/
   ```

1. Aktualisieren Sie die TLS-/SSL-Zertifikate, wodurch der Ordner `/etc/ssl/certs/` aktualisiert wird.

   ```bash
   update-ca-certificates
   ```

#### <a name="macos-environment"></a>macOS-Umgebung

Führen Sie in einer Mac-Umgebung die folgenden Schritte aus:

1. [Exportieren Sie das Zertifikat im PFX-Format](local-emulator-export-ssl-certificates.md#export-emulator-certificate). Die Option „PFX“ ist verfügbar, wenn Sie den privaten Schlüssel exportieren.

1. Kopieren Sie die PFX-Datei in Ihre Mac-Umgebung.

1. Öffnen Sie die Anwendung *Keychain Access*, und importieren Sie die PFX-Datei.

1. Öffnen Sie die Liste mit den Zertifikaten, und suchen Sie nach dem Zertifikat `localhost`.

1. Wählen Sie im Kontextmenü für dieses Element die Option *Element abrufen* und anschließend unter *Vertrauensstellung* > *Bei Verwendung dieses Zertifikats* die Option *Immer vertrauen* aus. 

   :::image type="content" source="./media/local-emulator/mac-trust-certificate.png" alt-text="Wählen Sie die Schaltfläche „Start“ aus, oder drücken Sie die WINDOWS-TASTE. Beginnen Sie mit der Eingabe von „Azure Cosmos-Emulator“, und wählen Sie den Emulator in der Liste der Anwendungen aus.":::
  
### <a name="option-2-disable-the-ssl-validation-in-the-application"></a><a id="disable-ssl-validation"></a>Option 2: Deaktivieren der SSL-Überprüfung in der Anwendung

Die Deaktivierung der SSL-Überprüfung wird nur zu Entwicklungszwecken empfohlen und sollte bei der Ausführung in einer Produktionsumgebung nicht durchgeführt werden. In den folgenden Beispielen wird gezeigt, wie die SSL-Validierung für .NET-und Node.js-Anwendungen deaktiviert wird.

# <a name="net-standard-21"></a>[.NET Standard 2.1+](#tab/ssl-netstd21)

Für alle Anwendungen, die unter einem Framework mit .NET Standard 2.1-Kompatibilität (oder höher) ausgeführt werden, können wir `CosmosClientOptions.HttpClientFactory` nutzen:

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/HttpClientFactory/Program.cs?name=DisableSSLNETStandard21)]

# <a name="net-standard-20"></a>[.NET-Standard 2.0](#tab/ssl-netstd20)

Für alle Anwendungen, die unter einem mit .NET Standard 2.0 kompatiblen Framework ausgeführt werden, können wir `CosmosClientOptions.HttpClientFactory` nutzen:

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/HttpClientFactory/Program.cs?name=DisableSSLNETStandard20)]

# <a name="nodejs"></a>[Node.js](#tab/ssl-nodejs)

Für Node.js-Anwendungen können Sie Ihre Datei `package.json` ändern, um für das Starten der Anwendung `NODE_TLS_REJECT_UNAUTHORIZED` festzulegen:

```json
"start": NODE_TLS_REJECT_UNAUTHORIZED=0 node app.js
```
--- 

## <a name="enable-access-to-emulator-on-a-local-network"></a>Aktivieren des Zugriffs auf den Emulator in einem lokalen Netzwerk

Wenn Sie mehrere Computer in einem einzigen Netzwerk haben und den Emulator auf einem der Computer einrichten und von einem anderen Computer darauf zugreifen möchten. In diesem Fall müssen Sie den Zugriff auf den Emulator in einem lokalen Netzwerk aktivieren.

Sie können den Emulator in einem lokalen Netzwerk ausführen. Um Netzwerkzugriff zu ermöglichen, geben Sie in der [Befehlszeile](emulator-command-line-parameters.md)`/AllowNetworkAccess` an, wodurch Sie gleichzeitig auch `/Key=key_string` oder `/KeyFile=file_name` angeben müssen. Sie können mit `/GenKeyFile=file_name` im Vorfeld eine Datei mit einem zufälligen Schlüssel generieren. Dann übergeben Sie diese an `/KeyFile=file_name` oder `/Key=contents_of_file`.

Wenn der Netzwerkzugriff zum ersten Mal aktiviert wird, muss der Benutzer den Emulator herunterfahren und das Datenverzeichnis des Emulators *%LOCALAPPDATA%\CosmosDBEmulator* löschen.

## <a name="authenticate-connections-when-using-emulator"></a><a id="authenticate-requests"></a>Authentifizieren von Verbindungen bei Verwendung des Emulators

Wie bei Azure Cosmos DB in der Cloud muss jede Anforderung, die Sie an den Azure Cosmos-Emulator richten, authentifiziert werden. Der Azure Cosmos-Emulator unterstützt ausschließlich die sichere Kommunikation über TLS. Der Azure Cosmos-Emulator unterstützt ein einziges festgelegtes Konto und einen bekannten Authentifizierungsschlüssel für die Authentifizierung per Primärschlüssel. Dieses Konto und dieser Schlüssel sind die einzigen Anmeldeinformationen, die für den Azure Cosmos-Emulator verwendet werden dürfen. Sie lauten wie folgt:

```bash
Account name: localhost:<port>
Account key: C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==
```

> [!NOTE]
> Der vom Azure Cosmos-Emulator unterstützte Primärschlüssel ist nur für die Verwendung mit dem Emulator gedacht. Es ist nicht möglich, Ihr Azure Cosmos DB-Konto und den dazugehörigen Schlüssel mit dem Azure Cosmos-Emulator zu verwenden.

> [!NOTE]
> Wenn Sie den Emulator mit der Option „/Key“ gestartet haben, verwenden Sie den generierten Schlüssel anstelle des Standardschlüssels `C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==`. Weitere Informationen zur Option „/Key“ finden Sie in der [Referenz zum Befehlszeilentool](emulator-command-line-parameters.md).

## <a name="connect-to-different-apis-with-the-emulator"></a><a id="connect-with-emulator-apis"></a>Herstellen einer Verbindung mit verschiedenen APIs mit dem Emulator

### <a name="sql-api"></a>SQL-API

Sobald der Azure Cosmos-Emulator auf Ihrem Desktop ausgeführt wird, können Sie jedes unterstützte [Azure Cosmos DB SDK](sql-api-sdk-dotnet-standard.md) oder die [Azure Cosmos DB-REST-API](/rest/api/cosmos-db/) für die Interaktion mit dem Emulator verwenden. Der Azure Cosmos-Emulator enthält auch einen integrierten Daten-Explorer, mit dem Sie Container für die SQL-API und die Mongo DB-API für Azure Cosmos DB erstellen können. Mithilfe des Daten-Explorers können Sie Elemente anzeigen und bearbeiten, ohne Code schreiben zu müssen.

```csharp
// Connect to the Azure Cosmos emulator running locally
CosmosClient client = new CosmosClient(
   "https://localhost:8081", 
    "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==");

```

### <a name="azure-cosmos-dbs-api-for-mongodb"></a>Azure Cosmos DB-API für MongoDB

Sobald der Azure Cosmos-Emulator auf Ihrem Desktop ausgeführt wird, können Sie die [Azure Cosmos DB-API für MongoDB](mongodb-introduction.md) für die Interaktion mit dem Emulator verwenden. Starten Sie den Emulator über die [Eingabeaufforderung](emulator-command-line-parameters.md) als Administrator mit „/EnableMongoDbEndpoint“. Verwenden Sie dann die folgende Verbindungszeichenfolge zum Herstellen einer Verbindung mit dem MongoDB-API-Konto:

```bash
mongodb://localhost:C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==@localhost:10255/admin?ssl=true
```

### <a name="table-api"></a>Tabelle-API

Sobald der Azure Cosmos-Emulator auf Ihrem Desktop ausgeführt wird, können Sie das [SDK für die Tabellen-API von Azure Cosmos DB](table-storage-how-to-use-dotnet.md) für die Interaktion mit dem Emulator verwenden. Starten Sie den Emulator über die [Eingabeaufforderung](emulator-command-line-parameters.md) als Administrator mit „/EnableTableEndpoint“. Führen Sie anschließend den folgenden Code aus, um eine Verbindung mit dem Tabellen-API-Konto herzustellen:

```csharp
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Table;
using CloudTable = Microsoft.WindowsAzure.Storage.Table.CloudTable;
using CloudTableClient = Microsoft.WindowsAzure.Storage.Table.CloudTableClient;

string connectionString = "DefaultEndpointsProtocol=http;AccountName=localhost;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==;TableEndpoint=http://localhost:8902/;";

CloudStorageAccount account = CloudStorageAccount.Parse(connectionString);
CloudTableClient tableClient = account.CreateCloudTableClient();
CloudTable table = tableClient.GetTableReference("testtable");
table.CreateIfNotExists();
table.Execute(TableOperation.Insert(new DynamicTableEntity("partitionKey", "rowKey")));
```

### <a name="cassandra-api"></a>Cassandra-API

Starten Sie den Emulator über die [Eingabeaufforderung](emulator-command-line-parameters.md) als Administrator mit „/EnableCassandraEndpoint“. Alternativ können Sie auch die Umgebungsvariable `AZURE_COSMOS_EMULATOR_CASSANDRA_ENDPOINT=true` festlegen.

1. [Installieren Sie Python 2.7](https://www.python.org/downloads/release/python-2716/).

1. [Installieren Sie Cassandra CLI/CQLSH](https://cassandra.apache.org/download/).

1. Führen Sie die folgenden Befehle in einem regulären Eingabeaufforderungsfenster aus:

   ```bash
   set Path=c:\Python27;%Path%
   cd /d C:\sdk\apache-cassandra-3.11.3\bin
   set SSL_VERSION=TLSv1_2
   set SSL_VALIDATE=false
   cqlsh localhost 10350 -u localhost -p C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw== --ssl
   ```

1. Führen Sie in der CQLSH Shell die folgenden Befehle aus, um eine Verbindung zum Cassandra-Endpunkt herzustellen:

   ```bash
   CREATE KEYSPACE MyKeySpace WITH replication = {'class':'MyClass', 'replication_factor': 1};
   DESCRIBE keyspaces;
   USE mykeyspace;
   CREATE table table1(my_id int PRIMARY KEY, my_name text, my_desc text);
   INSERT into table1 (my_id, my_name, my_desc) values( 1, 'name1', 'description 1');
   SELECT * from table1;
   EXIT
   ```

### <a name="gremlin-api"></a>Gremlin-API

Starten Sie den Emulator über die [Eingabeaufforderung](emulator-command-line-parameters.md) als Administrator mit „/EnableGremlinEndpoint“. Alternativ können Sie auch die Umgebungsvariable `AZURE_COSMOS_EMULATOR_GREMLIN_ENDPOINT=true` festlegen.

1. [Installieren Sie „apache-tinkerpop-gremlin-console-3.3.4“.](https://archive.apache.org/dist/tinkerpop/3.3.4)

1. Erstellen Sie im Daten-Explorer des Emulators die Datenbank „db1“ und die Sammlung „coll1“, und wählen Sie „/name“ als Partitionsschlüssel aus.

1. Führen Sie die folgenden Befehle in einem regulären Eingabeaufforderungsfenster aus:

   ```bash
   cd /d C:\sdk\apache-tinkerpop-gremlin-console-3.3.4-bin\apache-tinkerpop-gremlin-console-3.3.4
  
   copy /y conf\remote.yaml conf\remote-localcompute.yaml
   notepad.exe conf\remote-localcompute.yaml
     hosts: [localhost]
     port: 8901
     username: /dbs/db1/colls/coll1
     password: C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==
     connectionPool: {
     enableSsl: false}
     serializer: { className: org.apache.tinkerpop.gremlin.driver.ser.GraphSONMessageSerializerV1d0,
     config: { serializeResultToString: true  }}

   bin\gremlin.bat
   ```

1. Führen Sie in der Gremlin-Shell die folgenden Befehle aus, um eine Verbindung zum Gremlin-Endpunkt herzustellen:

   ```bash
   :remote connect tinkerpop.server conf/remote-localcompute.yaml
   :remote console
   :> g.V()
   :> g.addV('person1').property(id, '1').property('name', 'somename1')
   :> g.addV('person2').property(id, '2').property('name', 'somename2')
   :> g.V()
   ```

## <a name="uninstall-the-local-emulator"></a><a id="uninstall"></a>Deinstallieren des lokalen Emulators

Führen Sie die folgenden Schritte aus, um den Emulator zu deinstallieren:

1. Schließen Sie alle geöffneten Instanzen des lokalen Emulators, indem Sie mit der rechten Maustaste auf der Taskleiste auf das Symbol **Azure Cosmos-Emulator** klicken und anschließend **Beenden** auswählen. Bis alle Instanzen beendet wurden, kann unter Umständen eine Minute vergehen.

1. Geben Sie in das Windows-Suchfeld **Apps & Features** ein, und wählen Sie das Ergebnis **Apps & Features (Systemeinstellungen)** aus.

1. Scrollen Sie in der Liste der Apps zu **Azure Cosmos DB-Emulator**. Wählen Sie den Eintrag aus, und klicken Sie auf **Deinstallieren**. Bestätigen Sie den Vorgang, und wählen Sie erneut **Deinstallieren** aus.

## <a name="next-steps"></a>Nächste Schritte

In diesem Artikel haben Sie erfahren, wie Sie den lokalen Emulator für die kostenlose lokale Entwicklung verwenden. Sie können jetzt mit dem nächsten Artikel fortfahren:

* [Exportieren der Azure Cosmos-Emulatorzertifikate zur Verwendung bei Java-, Python- und Node.js-Apps](local-emulator-export-ssl-certificates.md)
* [Verwenden von Befehlszeilenparametern und PowerShell-Befehlen zum Steuern des Emulators](emulator-command-line-parameters.md)
* [Debuggen von Problemen mit dem Emulator](troubleshoot-local-emulator.md)
