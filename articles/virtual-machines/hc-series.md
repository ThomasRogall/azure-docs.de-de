---
title: 'HC-Serie: Azure Virtual Machines'
description: Spezifikationen für die VMs der HC-Serie
author: ju-shim
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 09/09/2020
ms.author: amverma
ms.reviewer: jushiman
ms.openlocfilehash: bb76dbf975ce15f72d3ad339853c407de42cf507
ms.sourcegitcommit: 1b320bc7863707a07e98644fbaed9faa0108da97
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 09/09/2020
ms.locfileid: "89594389"
---
# <a name="hc-series"></a>HC-Serie

Die VMs der HC-Serie sind für Anwendungen mit hohen Anforderungen an die Rechenleistung optimiert, z. B. implizite Finite-Element-Analysen, Molekulardynamik oder Chemoinformatik. HC-VMs bieten 44 Intel-Prozessorkerne des Modells Xeon Platinum 8168, 8 GB RAM pro CPU-Kern und kein Hyperthreading. Die Intel Xeon Platinum-Plattform unterstützt Intels umfangreiches Ökosystem von Softwaretools, etwa die Intel Math Kernel Library, und erweiterte Vektorverarbeitungsfunktionen, z. B. AVX-512.

VMs der HC-Serie unterstützen Mellanox EDR InfiniBand mit 100 Gbit/s. Diese VMs sind für eine optimierte und konsistente RDMA-Leistung in einer FAT-Struktur ohne Blocks verbunden. Diese VMs unterstützen adaptives Routing und DCT (Dynamic Connected Transport, zusätzlich zum standardmäßigen RC- und UD-Transport). Diese Features verbessern die Anwendungsleistung, Skalierbarkeit und Konsistenz, und ihre Verwendung wird dringend empfohlen.

ACU: 297-315

Storage Premium Unterstützt

Storage Premium-Zwischenspeicherung: Unterstützt

Livemigration: Nicht unterstützt

Updates mit Speicherbeibehaltung: Nicht unterstützt

| Size | vCPU | Prozessor | Arbeitsspeicher (GB) | Speicherbandbreite GB/s | Basis-CPU-Frequenz (GHz) | Frequenz für alle Kerne (GHz, Spitze) | Frequenz für Einzelkern (GHz, Spitze) | RDMA-Leistung (Gbit/s) | MPI-Unterstützung | Temporärer Speicher (GB) | Max. Anzahl Datenträger | Max. Ethernet-Karten |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_HC44rs | 44 | Intel Xeon Platinum 8168 | 352 | 191 | 2.7 | 3.4 | 3,7 | 100 | All | 700 | 4 | 1 |

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="other-sizes"></a>Andere Größen

- [Allgemeiner Zweck](sizes-general.md)
- [Arbeitsspeicheroptimiert](sizes-memory.md)
- [Speicheroptimiert](sizes-storage.md)
- [GPU-optimiert](sizes-gpu.md)
- [High Performance Computing](sizes-hpc.md)
- [Vorherige Generationen](sizes-previous-gen.md)

## <a name="next-steps"></a>Nächste Schritte

- Erfahren Sie mehr über das [Konfigurieren Ihrer VMs](./workloads/hpc/configure.md), das [Aktivieren von InfiniBand](./workloads/hpc/enable-infiniband.md), das [Einrichten von MPI](./workloads/hpc/setup-mpi.md) und das Optimieren von HPC-Anwendungen für Azure mit [HPC-Workloads](./workloads/hpc/overview.md).
- Informieren Sie sich in den [Tech Community-Blogs zu Azure Compute](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute) über die neuesten Ankündigungen, und machen Sie sich mit einigen Beispielen und Ergebnissen zu HPC vertraut.
- Eine allgemeine Übersicht über die Architektur für die Ausführung von HPC-Workloads finden Sie unter [High Performance Computing (HPC) in Azure](/azure/architecture/topics/high-performance-computing/).
- Weitere Informationen dazu, wie Sie mit [Azure-Computeeinheiten (ACU)](acu.md) die Computeleistung von Azure-SKUs vergleichen können.
