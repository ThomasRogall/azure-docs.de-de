---
title: Konfigurieren eines VNN mit Azure Load Balancer für HADR
description: Erfahren Sie, wie Sie einen Azure Load Balancer zum Weiterleiten von Datenverkehr an den virtuellen Netzwerknamen (VNN) für Ihre Verfügbarkeitsgruppe oder Failoverclusterinstanz (FCI) mit SQL Server auf Azure-VMs für Hochverfügbarkeit und Notfallwiederherstellung (High Availability und Disaster Recovery, HADR) konfigurieren.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
manager: jroth
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 06/02/2020
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 2f36e568603ded5a89f88cf11627a09a5a240fac
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 09/25/2020
ms.locfileid: "91316986"
---
# <a name="configure-vnn-with-azure-load-balancer-sql-server-on-azure-vms"></a>Konfigurieren eines VNN mit Azure Load Balancer (SQL Server auf Azure-VMs)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

In Azure Virtual Machines wird für Cluster ein Lastenausgleich für eine IP-Adresse verwendet, die zu einem bestimmten Zeitpunkt auf einem Clusterknoten vorhanden sein muss. In dieser Lösung enthält der Load Balancer die IP-Adresse für den Namen des virtuellen Netzwerks (VNN), der von der Clusterressource in Azure verwendet wird. 

In diesem Artikel erfahren Sie, wie Sie Lastenausgleich mit dem Azure Load Balancer-Dienst konfigurieren. Der Load Balancer leitet Datenverkehr an den [Verfügbarkeitsgruppenlistener](availability-group-overview.md) oder an [Failoverclusterinstanzen (FCIs)](failover-cluster-instance-overview.md) mit SQL Server auf Azure-VMs für Hochverfügbarkeit und Notfallwiederherstellung (High Availability und Disaster Recovery, HADR) weiter. 



## <a name="prerequisites"></a>Voraussetzungen

Bevor Sie die in diesem Artikel aufgeführten Schritte ausführen, sollten Sie über Folgendes verfügen:

- Sie sollten entschieden haben, dass Azure Load Balancer die geeignete [Konnektivitätsoption für die HADR-Lösung](hadr-cluster-best-practices.md#connectivity) ist.
- Ihre [Verfügbarkeitsgruppenlistener](availability-group-overview.md) oder [Failoverclusterinstanzen](failover-cluster-instance-overview.md) müssen konfiguriert sein. 
- Sie müssen die neueste Version von [PowerShell](/powershell/azure/install-az-ps?view=azps-4.2.0) installiert haben. 


## <a name="create-load-balancer"></a>Erstellen eines Load Balancers

Verwenden Sie das [Azure-Portal](https://portal.azure.com) zum Erstellen des Lastenausgleichs:

1. Navigieren Sie im Azure-Portal zu der Ressourcengruppe mit den virtuellen Computern.

1. Wählen Sie **Hinzufügen**. Durchsuchen Sie Azure Marketplace nach **Load Balancer**. Wählen Sie **Load Balancer** aus.

1. Klicken Sie auf **Erstellen**.

1. Richten Sie den Lastenausgleich mit den folgenden Werten ein:

   - **Abonnement**: Ihr Azure-Abonnement.
   - **Ressourcengruppe**: Die Ressourcengruppe, die Ihre virtuellen Computer enthält.
   - **Name**: Ein Name, mit dem der Lastenausgleich identifiziert wird.
   - **Region**: Der Azure-Standort, der Ihre virtuellen Computer enthält.
   - **Typ:** Öffentlich oder privat. Der Zugriff auf einen privaten Lastenausgleich ist innerhalb des virtuellen Netzwerks möglich. Für die meisten Azure-Anwendungen kann ein privater Lastenausgleich verwendet werden. Verwenden Sie einen öffentlichen Lastenausgleich, wenn Ihre Anwendung direkten Zugriff auf SQL Server über das Internet benötigt.
   - **SKU**: Standard.
   - **Virtuelles Netzwerk:** Dies ist dasselbe Netzwerk wie für die virtuellen Computer.
   - **IP-Adresszuweisung**: Statisch. 
   - **Private IP-Adresse**: Die IP-Adresse, die Sie der Clusternetzwerkressource zugewiesen haben.

   Die folgende Abbildung zeigt die Benutzeroberfläche zum **Erstellen des Lastenausgleichs**:

   ![Einrichten des Lastenausgleichs](./media/failover-cluster-instance-premium-file-share-manually-configure/30-load-balancer-create.png)
   

## <a name="configure-backend-pool"></a>Konfigurieren des Back-End-Pools

1. Wechseln Sie zurück zur Azure-Ressourcengruppe mit den virtuellen Computern, und suchen Sie nach dem neuen Lastenausgleich. Möglicherweise müssen Sie die Ansicht in der Ressourcengruppe aktualisieren. Wählen Sie den Load Balancer aus.

1. Wählen Sie **Back-End-Pools** und dann **Hinzufügen** aus.

1. Ordnen Sie den Back-End-Pool der Verfügbarkeitsgruppe mit den virtuellen Computern zu.

1. Aktivieren Sie unter **Zielnetzwerk-IP-Konfigurationen** die Option **VIRTUELLER COMPUTER**, und wählen Sie die virtuellen Computer aus, die als Clusterknoten eingeschlossen werden. Schließen Sie dabei alle virtuellen Computer ein, die die FCI oder Verfügbarkeitsgruppe hosten.

1. Wählen Sie **OK** aus, um den Back-End-Pool zu erstellen.

## <a name="configure-health-probe"></a>Konfigurieren des Integritätstests

1. Wählen Sie im Bereich für den Lastenausgleich **Integritätstests** aus.

1. Wählen Sie **Hinzufügen**.

1. Legen Sie im Bereich **Integritätstest hinzufügen** <span id="probe"> </span>die folgenden Parameter für den Integritätstest fest:

   - **Name**: Ein Name für den Integritätstest.
   - **Protokoll:** TCP.
   - **Port:** Der Port, den Sie [beim Vorbereiten der VM](failover-cluster-instance-prepare-vm.md#uninstall-sql-server-1) in der Firewall für den Integritätstest erstellt haben. Im Beispiel dieses Artikels wird der TCP-Port `59999` verwendet.
   - **Intervall**: 5 Sekunden.
   - **Fehlerschwellenwert**: Zwei aufeinanderfolgende Fehler.

1. Klicken Sie auf **OK**.

## <a name="set-load-balancing-rules"></a>Festlegen von Lastenausgleichsregeln

1. Wählen Sie im Bereich für den Lastenausgleich **Lastenausgleichsregeln** aus.

1. Wählen Sie **Hinzufügen**.

1. Legen Sie die Parameter für die Lastenausgleichsregeln fest:

   - **Name**: Ein Name für die Lastenausgleichsregeln.
   - **Front-End-IP-Adresse**: Die IP-Adresse für die Clusternetzwerkressource der SQL Server-FCI oder des Verfügbarkeitsgruppenlisteners.
   - **Port:** Der SQL Server-TCP-Port. Der Standardport der Instanz lautet 1433.
   - **Back-End-Port**: Der gleiche Port, den Sie als Wert für **Port** angeben, wenn Sie **Floating IP (Direct Server Return)** aktivieren.
   - **Back-End-Pool**: Der Name des Back-End-Pools, den Sie zuvor konfiguriert haben.
   - **Integritätstest**: Der Integritätstest, den Sie zuvor konfiguriert haben.
   - **Sitzungspersistenz**: Keine.
   - **Leerlaufzeitüberschreitung (Minuten)** : 4.
   - **Floating IP (Direct Server Return)** : Aktiviert.

1. Klicken Sie auf **OK**.

## <a name="configure-cluster-probe"></a>Konfigurieren des Clustertests

Legen Sie den Parameter für den Clustertestport in PowerShell fest.

Aktualisieren Sie die Variablen im folgenden Skript mit Werten aus Ihrer Umgebung, um den Parameter für den Clustertestport festzulegen. Entfernen Sie die spitzen Klammern (`<` und `>`) aus dem Skript.

```powershell
$ClusterNetworkName = "<Cluster Network Name>"
$IPResourceName = "<SQL Server FCI / AG Listener IP Address Resource Name>" 
$ILBIP = "<n.n.n.n>" 
[int]$ProbePort = <nnnnn>

Import-Module FailoverClusters

Get-ClusterResource $IPResourceName | Set-ClusterParameter -Multiple @{"Address"="$ILBIP";"ProbePort"=$ProbePort;"SubnetMask"="255.255.255.255";"Network"="$ClusterNetworkName";"EnableDhcp"=0}
```

Die Werte, die Sie aktualisieren können, werden in der folgenden Tabelle beschrieben:


|**Wert**|**Beschreibung**|
|---------|---------|
|`Cluster Network Name`| Der Name des Windows Server-Failoverclusters für das Netzwerk. Klicken Sie in **Failovercluster-Manager** > **Netzwerke** mit der rechten Maustaste auf das Netzwerk, und wählen Sie **Eigenschaften** aus. Der richtige Wert befindet sich auf der Registerkarte **Allgemein** unter **Name**.|
|`SQL Server FCI/AG listener IP Address Resource Name`|Der Ressourcenname für die IP-Adresse der SQL Server-FCI oder des Verfügbarkeitsgruppenlisteners. Klicken Sie in **Failovercluster-Manager** > **Rollen** unter der Rolle „SQL Server-FCI“ unter **Servername** mit der rechten Maustaste auf die IP-Adressressource, und wählen Sie dann **Eigenschaften** aus. Der richtige Wert befindet sich auf der Registerkarte **Allgemein** unter **Name**.|
|`ILBIP`|Die IP-Adresse des internen Load Balancers (ILB). Diese Adresse wird als Front-End-Adresse des ILB im Azure-Portal konfiguriert. Dies ist auch die FCI-IP-Adresse von SQL Server. Sie finden sie im **Failovercluster-Manager** auf der gleichen Eigenschaftenseite, auf der sich der `<SQL Server FCI/AG listener IP Address Resource Name>` befindet.|
|`nnnnn`|Der Testport, den Sie im Integritätstest des Lastenausgleichs konfiguriert haben. Alle nicht verwendeten TCP-Ports sind zulässig.|
|„SubnetMask“| Die Subnetzmaske für den Clusterparameter. Dabei muss es sich um die TCP/IP-Broadcastadresse handeln: `255.255.255.255`.| 


Nach dem Festlegen des Clustertests werden in PowerShell alle Clusterparameter angezeigt. Führen Sie dieses Skript aus:

```powershell
Get-ClusterResource $IPResourceName | Get-ClusterParameter
```


## <a name="test-failover"></a>Testfailover


Testen Sie das Failover der Clusterressource, um die Clusterfunktionalität zu validieren. 

# <a name="failover-cluster-instance"></a>[Failoverclusterinstanz](#tab/fci)

Führen Sie die folgenden Schritte aus:

1. Stellen Sie mithilfe von RDP eine Verbindung mit einem der SQL Server-Clusterknoten her.
1. Öffnen Sie den **Failovercluster-Manager**. Wählen Sie **Rollen** aus. Achten Sie darauf, welcher Knoten im Besitz der SQL Server-FCI-Rolle ist.
1. Klicken Sie mit der rechten Maustaste auf die SQL Server-FCI-Rolle. 
1. Wählen Sie **Verschieben** aus, und wählen Sie dann **Bestmöglicher Knoten** aus.

Unter **Failovercluster-Manager** wird die Rolle angezeigt, und die Ressourcen werden in den Offlinezustand versetzt. Die Ressourcen werden dann verschoben und auf dem anderen Knoten dann wieder in den Onlinezustand versetzt.



# <a name="ag-listener"></a>[Verfügbarkeitsgruppenlistener](#tab/ag)

Führen Sie die folgenden Schritte aus:

1. Öffnen Sie [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms), und stellen Sie eine Verbindung mit Ihrem Verfügbarkeitsgruppenlistener her. 

1. Erweitern Sie **Always On-Verfügbarkeitsgruppe** im **Objekt-Explorer**. 
1. Klicken Sie mit der rechten Maustaste auf die Verfügbarkeitsgruppe, und wählen Sie **Failover** aus. 
1. Befolgen Sie die Anweisungen des Assistenten, um ein Failover der Verfügbarkeitsgruppe auf ein sekundäres Replikat auszuführen. 

Das Failover ist erfolgreich, wenn die Replikate ihre Rollen tauschen und beide synchronisiert werden. 

---

## <a name="test-connectivity"></a>Testen der Konnektivität

Melden Sie sich zum Testen der Konnektivität an einem anderen virtuellen Computer in demselben virtuellen Netzwerk an. Öffnen Sie **SQL Server Management Studio**, und stellen Sie eine Verbindung mit dem SQL Server-FCI-Namen oder dem Verfügbarkeitsgruppenlistener her.

>[!NOTE]
>Bei Bedarf können Sie [SQL Server Management Studio herunterladen](/sql/ssms/download-sql-server-management-studio-ssms).



## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen zu SQL Server-HADR-Features in Azure finden Sie unter [Verfügbarkeitsgruppen](availability-group-overview.md) und [Failoverclusterinstanz](failover-cluster-instance-overview.md). Sie können sich auch mit den [bewährten Methoden](hadr-cluster-best-practices.md) zum Konfigurieren Ihrer Umgebung für Hochverfügbarkeit und Notfallwiederherstellung vertraut machen. 



