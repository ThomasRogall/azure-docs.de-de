---
title: Azure Defender für Server – Vorteile und Features
description: Erfahren Sie etwas über die Vorteile und Features von Azure Defender für Server.
author: memildin
ms.author: memildin
ms.date: 9/23/2020
ms.topic: overview
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: 8757399329f3a9bd9f4d7b914b12b2a0f7e85603
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 10/05/2020
ms.locfileid: "91448291"
---
# <a name="introduction-to-azure-defender-for-servers"></a>Einführung in Azure Defender für Server

Azure Defender für Server stattet Ihre Windows- und Linux-Computer mit Bedrohungserkennung und erweiterten Schutzmaßnahmen aus.

Unter Windows wird Azure Defender zur Überwachung und zum Schutz Ihrer Windows-basierten Computer in Azure-Dienste integriert. In Security Center werden die Warnungen und Behandlungsvorschläge aus allen diesen Diensten in einem benutzerfreundlichen Format präsentiert.

Unter Linux sammelt Azure Defender Überwachungsdatensätze von Linux-Computern mithilfe von **auditd**, einem der am häufigsten verwendeten Linux-Überwachungsframeworks. auditd befindet sich im Mainline-Kernel. 


## <a name="what-are-the-benefits-of-azure-defender-for-servers"></a>Welche Vorteile bietet die Nutzung von Azure Defender für Server?

Zu den Funktionen für Bedrohungserkennung und Schutz von Azure Defender für Server gehören:

- **Überprüfung auf Sicherheitsrisiken für VMs:** Die in Azure Security Center enthaltene Überprüfung auf Sicherheitsrisiken basiert auf Qualys. 

    Der Scanner von Qualys ist eines der führenden Tools für die Echtzeiterkennung von Sicherheitsrisiken auf Ihren virtuellen Azure-Computern. Sie benötigen keine Qualys-Lizenz und auch kein Qualys-Konto – alles erfolgt nahtlos innerhalb von Security Center. [Weitere Informationen](deploy-vulnerability-assessment-vm.md)

- **Just-In-Time-Zugriff (JIT) auf VMs:** Bedrohungsakteure suchen aktiv nach zugänglichen Computern mit offenen Verwaltungsports wie RDP oder SSH. Alle Ihre virtuellen Computer sind potenzielle Ziele für Angriffe. Wenn eine VM erfolgreich kompromittiert wurde, wird sie als Einstiegspunkt verwendet, um weitere Ressourcen in Ihrer Umgebung anzugreifen.

    Wenn Sie Azure Defender für Server aktivieren, können Sie Just-In-Time-VM-Zugriff verwenden, um eingehenden Datenverkehr auf Ihren VMs zu blockieren und dadurch die Gefährdung durch Angriffe zu verringern und bei Bedarf einen einfachen Zugriff für das Verbinden mit VMs bereitzustellen. [Weitere Informationen](just-in-time-explained.md)

- **Überwachung der Dateiintegrität:** Die Überwachung der Dateiintegrität (File Integrity Monitoring, FIM), auch als Änderungsüberwachung bezeichnet, untersucht unter anderem Dateien und Registrierungen des Betriebssystems sowie Anwendungen auf Änderungen, die auf einen Angriff hindeuten. Eine Vergleichsmethode wird verwendet, um zu bestimmen, ob der aktuelle Status der Datei sich von der letzten Überprüfung der Datei unterscheidet. Sie können diesen Vergleich nutzen, um zu bestimmen, ob gültige oder verdächtige Änderungen an Ihren Dateien vorgenommen wurden.

    Wenn Sie Azure Defender für Server aktivieren, können Sie mithilfe von FIM die Integrität von Windows-Dateien, Windows-Registrierungen und Linux-Dateien überprüfen. [Weitere Informationen](security-center-file-integrity-monitoring.md)

- **Adaptive Anwendungssteuerungen:** Adaptive Anwendungssteuerungen sind eine intelligente und automatisierte Lösung zum Definieren von Positivlisten bekannter, sicherer Anwendungen für Ihre Computer.

    Wenn Sie adaptive Anwendungssteuerungen aktiviert und konfiguriert haben, erhalten Sie Sicherheitswarnungen, wenn eine Anwendung ausgeführt wird, die nicht als sicher definiert ist. [Weitere Informationen](security-center-adaptive-application.md)

- **Adaptive Netzwerkhärtung (ANH):** Der Einsatz von Netzwerksicherheitsgruppen (NSGs) zum Filtern von ein- und ausgehendem Datenverkehr für Ressourcen verbessert den Sicherheitsstatus Ihres Netzwerks. Es gibt jedoch Situationen, in denen es sich bei dem Datenverkehr, der die NSG durchläuft, um eine Teilmenge der definierten NSG-Regeln handelt. In diesen Fällen lässt sich der Sicherheitsstatus durch eine Härtung der NSG-Regeln auf der Grundlage tatsächlicher Datenverkehrsmuster noch weiter verbessern.

    Die adaptive Netzwerkhärtung liefert Empfehlungen zur weiteren Härtung der NSG-Regeln. Dabei kommt ein Machine Learning-Algorithmus zum Einsatz, der Faktoren wie tatsächlichen Datenverkehr, bekannte vertrauenswürdige Konfiguration und Bedrohungsinformationen sowie weitere Anzeichen einer Kompromittierung berücksichtigt und auf der Grundlage dieser Faktoren Empfehlungen abgibt, um nur Datenverkehr von bestimmten IP-/Port-Tupeln zuzulassen. [Weitere Informationen](security-center-adaptive-network-hardening.md)

- **Integration in Microsoft Defender Advanced Threat Protection (ATP) (nur Windows):** Azure Defender ist in Microsoft Defender Advanced Threat Protection (ATP) integriert. Dadurch stehen umfassende EDR-Funktionen (Endpoint Detection and Response; Endpunkterkennung und -reaktion) zur Verfügung. [Weitere Informationen](security-center-wdatp.md)

    > [!IMPORTANT]
    > Der Microsoft Defender ATP-Sensor wird automatisch auf Windows-Servern aktiviert, die Security Center verwenden.

    Wenn von Microsoft Defender ATP eine Bedrohung erkannt wird, wird eine Warnung ausgelöst. Die Warnung wird in Security Center angezeigt. Über Security Center können Sie zur Microsoft Defender ATP-Konsole wechseln und eine ausführliche Untersuchung durchführen, um das Ausmaß des Angriffs zu ermitteln. Weitere Informationen zu Microsoft Defender ATP finden Sie unter [Integrieren von Servern in den Microsoft Defender ATP-Dienst](https://docs.microsoft.com/windows/security/threat-protection/microsoft-defender-atp/configure-server-endpoints).

- **Docker-Hosthärtung:** Azure Security Center identifiziert nicht verwaltete Container, die auf IaaS-Linux-VMs oder anderen Linux-Computern gehostet werden, auf denen Docker-Container ausgeführt werden. Security Center bewertet kontinuierlich die Konfigurationen dieser Container. Anschließend werden sie mit dem Docker-Benchmark von Center for Internet Security (CIS) verglichen. Security Center umfasst den gesamten Regelsatz des CIS-Docker-Benchmark und benachrichtigt Sie, sobald Ihre Container eine der Steuerungen nicht erfüllen. [Weitere Informationen](harden-docker-hosts.md)

- **Erkennung dateiloser Angriffe (nur Windows):** Bei dateilosen Angriffen werden schädliche Nutzlasten in den Arbeitsspeicher injiziert, um die Erkennung durch Verfahren zur datenträgerbasierten Überprüfung zu verhindern. Die Nutzlast des Angreifers nistet sich im Arbeitsspeicher von kompromittierten Prozessen ein und führt ein breites Spektrum an schädlichen Aktivitäten aus.

  Die Erkennung dateiloser Angriffe erkennt dank automatisierter forensischer Techniken für den Arbeitsspeicher Toolkits, Techniken und Verhaltensweisen im Zusammenhang mit dateilosen Angriffen. Diese Lösung überprüft zur Laufzeit in regelmäßigen Abständen Ihren Computer und extrahiert Erkenntnisse direkt aus dem Arbeitsspeicher von Prozessen. Zu den spezifischen Erkenntnissen zählt die Ermittlung der folgenden Elemente: 

  - Bekannte Toolkits und Kryptografieminingsoftware 

  - Shellcode, ein kurzer Codeabschnitt, der normalerweise als Nutzlast bei der Ausnutzung eines Softwaresicherheitsrisikos verwendet wird

  - Injektion bösartiger ausführbarer Dateien in den Prozessspeicher

  Bei der Erkennung dateiloser Angriffe werden ausführliche Sicherheitswarnungen generiert, die Beschreibungen mit zusätzlichen Prozessmetadaten (etwa zur Netzwerkaktivität) enthalten. Dadurch werden die Warnungsselektion und die Korrelation beschleunigt und die Downstream-Reaktionszeit verkürzt. Dieser Ansatz dient zur Ergänzung ereignisbasierter EDR-Lösungen und ermöglicht ein größeres Erkennungsspektrum.

  Details der Warnungen bei Erkennung dateiloser Angriffe finden Sie in der [Referenztabelle der Warnungen](alerts-reference.md#alerts-windows).

- **Integration von Linux-auditd-Warnungen mit dem Log Analytics-Agent (nur Linux):** Beim auditd-System handelt es sich um ein Subsystem auf Kernelebene, das Systemaufrufe überwacht. Es filtert sie nach einem angegebenen Regelsatz und schreibt entsprechende Meldungen in einen Socket. Security Center integriert Funktionen aus dem auditd-Paket im Log Analytics-Agent. Diese Integration ermöglicht das Sammeln von auditd-Ereignissen in allen unterstützten Linux-Distributionen ohne weitere Bedingungen.

    auditd-Datensätze werden gesammelt, angereichert und mithilfe des Log Analytics-Agents für Linux zu Ereignissen aggregiert. In Security Center werden ständig neue Analysen hinzugefügt, die Linux-Signale verwenden, um schädliches Verhalten auf cloudbasierten und lokalen Linux-Computern zu erkennen. Diese Analysen umfassen ähnlich wie bei Windows-Funktionen verdächtige Prozesse und Anmeldeversuche, das Laden von Kernmodulen sowie andere Aktivitäten. Diese Aktivitäten können darauf hindeuten, dass ein Computer angegriffen wird oder kompromittiert wurde.  

    Eine Liste der Linux-Warnungen finden Sie in der [Referenztabelle der Warnungen](alerts-reference.md#alerts-linux).


## <a name="simulating-alerts"></a>Simulieren von Warnungen

Sie können Warnungen simulieren, indem Sie eines der folgenden Playbooks herunterladen:

- Windows: [Azure Security Center-Playbook: Sicherheitswarnungen](https://github.com/Azure/Azure-Security-Center/blob/master/Simulations/Azure%20Security%20Center%20Security%20Alerts%20Playbook_v2.pdf)

- Linux: [Azure Security Center-Playbook: Azure Security Center-Playbook: Linux Detections (Linux-Erkennungsfunktionen)](https://github.com/Azure/Azure-Security-Center/blob/master/Simulations/Azure%20Security%20Center%20Linux%20Detections_v2.pdf) herunterladen.




## <a name="next-steps"></a>Nächste Schritte

In diesem Artikel haben Sie etwas über Azure Defender für Server erfahren. 

Weitere Informationen finden Sie in den folgenden Artikeln: 

- Sie können Warnungen exportieren, und zwar unabhängig davon, ob sie von Security Center generiert oder von Security Center über ein anderes Sicherheitsprodukt empfangen wurden. Befolgen Sie die Anweisungen unter [Exportieren von Sicherheitswarnungen und -empfehlungen (Vorschau)](continuous-export.md), um die Warnungen zu Azure Sentinel (oder ein Drittanbieter-SIEM) bzw. ein beliebiges anderes externes Tool zu exportieren.

- > [!div class="nextstepaction"]
    > [Aktivieren von Azure Defender](security-center-pricing.md)