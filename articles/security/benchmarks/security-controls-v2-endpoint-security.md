---
title: 'Azure-Sicherheitsvergleichstest V2: Endpunktsicherheit'
description: Azure-Sicherheitsvergleichstest V2 für Endpunktsicherheit
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 09/13/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: c04e4233ded34ceaeec9cd9afb240d3d1ac864e0
ms.sourcegitcommit: 94c750edd4d755d6ecee50ac977328098a277479
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 09/13/2020
ms.locfileid: "90059037"
---
# <a name="security-control-endpoint-security"></a>Sicherheitskontrolle: Endpunktsicherheit

Endpunktsicherheit umfasst Steuerelemente im Zusammenhang mit Endpunkterkennung und -antwort. Dies beinhaltet die Verwendung von Endpunkterkennung und -antwort (Endpoint Detection and Response, EDR) und des Antischadsoftwarediensts für Endpunkte in Azure-Umgebungen.

## <a name="es-1-use-endpoint-detection-and-response-edr"></a>ES-1: Verwenden von Endpunkterkennung und -antwort (Endpoint Detection and Response, EDR)

| Azure-ID | ID(s) von CIS-Steuerelementen v7.1 | ID(s) von NIST SP800-53 r4 |
|--|--|--|--|
| ES-1 | 8.1 | SI-2, SI-3, SC-3 |

Aktivieren Sie EDR-Funktionen (Endpoint Detection and Response, Endpunkterkennung und -antwort) für Server und Clients, und implementieren Sie eine Integration für SIEM- und Security Operations-Prozesse.

Microsoft Defender Advanced Threat Protection bietet EDR-Funktionen im Rahmen einer Endpunktsicherheitsplattform für Unternehmen, um komplexe Bedrohungen zu vermeiden, zu erkennen, zu untersuchen und darauf zu reagieren. 

- [Microsoft Defender Advanced Threat Protection](/windows/security/threat-protection/microsoft-defender-atp/microsoft-defender-advanced-threat-protection)

- [Durchführen des Onboardings von Windows-Servern für den Microsoft Defender ATP-Dienst](/windows/security/threat-protection/microsoft-defender-atp/configure-server-endpoints)

- [Durchführen des Onboardings für Windows-fremde Server](/windows/security/threat-protection/microsoft-defender-atp/configure-endpoints-non-windows)

**Verantwortlichkeit**: Kunde

**An Kundensicherheit Beteiligte**:

- [Infrastruktur- und Endpunktsicherheit](/azure/cloud-adoption-framework/organize/cloud-security)

- [Threat Intelligence](/azure/cloud-adoption-framework/organize/cloud-security-threat-intelligence)

- [Sicherheitscomplianceverwaltung](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

- [Statusverwaltung](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

## <a name="es-2-use-centrally-managed-modern-anti-malware-software"></a>ES-2: Verwenden von moderner, zentral verwalteter Antischadsoftware

| Azure-ID | ID(s) von CIS-Steuerelementen v7.1 | ID(s) von NIST SP800-53 r4 |
|--|--|--|--|
| ES-2 | 8.1 | SI-2, SI-3, SC-3 |

Verwenden Sie eine zentral verwaltete Antischadsoftwarelösung für Endpunkte, mit der Echtzeitüberprüfungen und periodische Überprüfungen durchgeführt werden können.

Azure Security Center kann automatisch die Verwendung verschiedener gängiger Antischadsoftwarelösungen für Ihre virtuellen Computer erkennen, den Ausführungsstatus des Endpunktschutzes melden und Empfehlungen abgeben. 

Microsoft Antimalware für Azure Cloud Services ist die Standard-Antischadsoftware für virtuelle Windows-Computer (virtual machines, VMs). Verwenden Sie für virtuelle Linux-Computer eine Antischadsoftware eines Drittanbieters.  Mithilfe der Azure Security Center-Bedrohungserkennung für Datendienste können Sie auch Schadsoftware erkennen, die in Azure Storage-Konten hochgeladen wurde. 

- [Konfigurieren von Microsoft Antimalware für Cloud Services und Virtual Machines](../fundamentals/antimalware.md)

- [Unterstützte Endpoint Protection-Lösungen](https://docs.microsoft.com/azure/security-center/security-center-services?tabs=features-windows#supported-endpoint-protection-solutions-)

**Verantwortlichkeit**: Kunde

**An Kundensicherheit Beteiligte**:

- [Infrastruktur- und Endpunktsicherheit](/azure/cloud-adoption-framework/organize/cloud-security)

- [Threat Intelligence](/azure/cloud-adoption-framework/organize/cloud-security-threat-intelligence)

- [Sicherheitscomplianceverwaltung](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

- [Statusverwaltung](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

## <a name="es-3-ensure-anti-malware-software-and-signatures-are-updated"></a>ES-3: Sicherstellen der Aktualisierung von Antischadsoftware und Signaturen

| Azure-ID | ID(s) von CIS-Steuerelementen v7.1 | ID(s) von NIST SP800-53 r4 |
|--|--|--|--|
| ES-3 | 8,2 | SI-2, SI-3 |

Stellen Sie sicher, dass Antischadsoftwaresignaturen schnell und konsistent aktualisiert werden. 

Befolgen Sie die Empfehlungen in Azure Security Center: „Compute &amp; Apps“, um sicherzustellen, dass alle Endpunkte mit den neuesten Signaturen auf dem aktuellen Stand sind. Mit Microsoft Antimalware werden standardmäßig die neuesten Signaturen und Engine-Updates automatisch installiert. Verwenden Sie für Linux eine Antischadsoftware von Drittanbietern.

- [Bereitstellen von Microsoft Antimalware für Azure Cloud Services und Virtual Machines](../fundamentals/antimalware.md)

**Verantwortlichkeit**: Kunde

**An Kundensicherheit Beteiligte**:

- [Infrastruktur- und Endpunktsicherheit](/azure/cloud-adoption-framework/organize/cloud-security)

- [Threat Intelligence](/azure/cloud-adoption-framework/organize/cloud-security-threat-intelligence)

- [Sicherheitscomplianceverwaltung](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

- [Statusverwaltung](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

