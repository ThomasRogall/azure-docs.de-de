---
title: Konfigurieren von Serverparametern – Azure PowerShell – Azure Database for MariaDB
description: In diesem Artikel wird beschrieben, wie Sie mit PowerShell die Dienstparameter in Azure Database for MariaDB konfigurieren.
author: ajlam
ms.author: andrela
ms.service: mariadb
ms.devlang: azurepowershell
ms.topic: how-to
ms.date: 5/26/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: be9e61363beeb2f040aba44e67076c3d66997eee
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/31/2020
ms.locfileid: "87490114"
---
# <a name="configure-server-parameters-in-azure-database-for-mariadb-using-powershell"></a>Konfigurieren von Serverparametern in Azure Database for MariaDB mit PowerShell

Sie können Konfigurationsparameter für einen Azure Database for MariaDB-Server mithilfe von PowerShell auflisten, anzeigen und aktualisieren. Auf Serverebene ist eine Teilmenge der Engine-Konfigurationen verfügbar und kann geändert werden.

## <a name="prerequisites"></a>Voraussetzungen

Zum Durcharbeiten dieses Leitfadens benötigen Sie Folgendes:

- Lokale Installation des [Moduls „Az PowerShell“](https://docs.microsoft.com/powershell/azure/install-az-ps) oder von [Azure Cloud Shell](https://shell.azure.com/) im Browser
- [Azure Database for MariaDB-Server](quickstart-create-mariadb-server-database-using-azure-powershell.md)

> [!IMPORTANT]
> Solange nur eine Vorschauversion des PowerShell-Moduls „Az.MariaDb“ verfügbar ist, müssen Sie es separat über das Azure PowerShell-Modul installieren. Verwenden Sie dazu den folgenden Befehl: `Install-Module -Name Az.MariaDb -AllowPrerelease`.
> Sobald das PowerShell-Modul Az.MariaDb allgemein verfügbar ist, wird es in zukünftige Releases des Az-PowerShell-Moduls integriert und in Azure Cloud Shell nativ zur Verfügung gestellt.

Wenn Sie PowerShell lieber lokal verwenden möchten, stellen Sie mithilfe des Cmdlets [Connect-AzAccount](https://docs.microsoft.com/powershell/module/az.accounts/connect-azaccount) eine Verbindung mit Ihrem Azure-Konto her.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="list-server-configuration-parameters-for-azure-database-for-mariadb-server"></a>Auflisten der Serverkonfigurationsparameter für Azure Database for MariaDB

Führen Sie das Cmdlet `Get-AzMariaDbConfiguration` aus, um alle änderbaren Parameter eines Servers mit ihren Werten aufzulisten.

Mit dem folgenden Beispiel werden die Serverkonfigurationsparameter für den Server **mydemoserver** in der Ressourcengruppe **myresourcegroup** aufgelistet.

```azurepowershell-interactive
Get-AzMariaDbConfiguration -ResourceGroupName myresourcegroup -ServerName mydemoserver
```

Die Definition der einzelnen aufgeführten Parameter finden Sie in der MariaDB-Referenz im Abschnitt [Server System Variables](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html).

## <a name="show-server-configuration-parameter-details"></a>Anzeigen von Details zu Serverkonfigurationsparametern

Um Details zu einem bestimmten Konfigurationsparameter für einen Server anzuzeigen, führen Sie das Cmdlet `Get-AzMariaDbConfiguration` aus und geben dabei den Parameter **Name** an.

Mit diesem Beispiel werden Details des Serverkonfigurationsparameters **slow\_query\_log** für den Server **mydemoserver** in der Ressourcengruppe **myresourcegroup** angezeigt.

```azurepowershell-interactive
Get-AzMariaDbConfiguration -Name slow_query_log -ResourceGroupName myresourcegroup -ServerName mydemoserver
```

## <a name="modify-a-server-configuration-parameter-value"></a>Ändern des Werts von Serverkonfigurationsparametern

Sie können den Wert eines bestimmten Serverkonfigurationsparameters ändern und dadurch den zugrunde liegenden Konfigurationswert für die MariaDB-Server-Engine aktualisieren. Zum Aktualisieren der Konfiguration verwenden Sie das Cmdlet `Update-AzMariaDbConfiguration`.

So aktualisieren Sie den Serverkonfigurationsparameter **slow\_query\_log** für den Server **mydemoserver** in der Ressourcengruppe **myresourcegroup**

```azurepowershell-interactive
Update-AzMariaDbConfiguration -Name slow_query_log -ResourceGroupName myresourcegroup -ServerName mydemoserver -Value On
```

## <a name="next-steps"></a>Nächste Schritte

> [!div class="nextstepaction"]
> [Automatische Speichervergrößerung für einen Azure Database for MariaDB-Server mithilfe von PowerShell](howto-auto-grow-storage-powershell.md)
