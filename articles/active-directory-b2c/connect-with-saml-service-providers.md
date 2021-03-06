---
title: Konfigurieren von Azure AD B2C als SAML-IDP für Ihre Anwendungen
title-suffix: Azure AD B2C
description: Konfigurieren von Azure AD B2C für die Bereitstellung von SAML-Protokollassertionen für Ihre Anwendungen (Dienstanbieter). Azure AD B2C fungiert als einzelner Identitätsanbieter (Identity Provider, IdP) für Ihre SAML-Anwendung.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 09/09/2020
ms.author: mimart
ms.subservice: B2C
ms.custom: fasttrack-edit
ms.openlocfilehash: 09edfc91f98e51a7dce7e98b48f2970ccba33586
ms.sourcegitcommit: f845ca2f4b626ef9db73b88ca71279ac80538559
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 09/09/2020
ms.locfileid: "89611603"
---
# <a name="register-a-saml-application-in-azure-ad-b2c"></a>Registrieren einer SAML-Anwendung in Azure AD B2C

In diesem Artikel erfahren Sie, wie Sie Azure Active Directory B2C (Azure AD B2C) so konfigurieren, dass der Dienst als SAML-Identitätsanbieter (Security Assertion Markup Language) für Ihre Anwendungen fungiert.

## <a name="scenario-overview"></a>Übersicht über das Szenario

Organisationen, die Azure AD B2C als Lösung für die Identitäts- und Zugriffsverwaltung von Kunden verwenden, benötigen möglicherweise Interaktionen mit Identitätsanbietern oder Anwendungen, die für Authentifizierung mit dem SAML-Protokoll konfiguriert sind.

Azure AD B2C erreicht die SAML-Interoperabilität durch eine der zwei folgenden Möglichkeiten:

* Fungieren als *Identitätsanbieter* (Identity Provider, IdP) und Erreichen von einmaligem Anmelden (Single Sign-On, SSO) mit SAML-basierten Dienstanbietern (Ihren Anwendungen)
* Fungieren als *Dienstanbieter* (Service Provider, SP) und Interagieren mit SAML-basierten Identitätsanbietern wie Salesforce und ADFS

![Abbildung mit B2C als Identitätsanbieter auf der linken Seite und B2C als Dienstanbieter auf der rechten Seite](media/saml-identity-provider/saml-idp-diagram-01.jpg)

Zusammenfassung der beiden nicht exklusiven Kernszenarien mit SAML:

| Szenario | Azure AD B2C-Rolle | Vorgehensweise |
| -------- | ----------------- | ------- |
| Meine Anwendung erwartet eine SAML-Assertion, um eine Authentifizierung abzuschließen. | **Azure AD B2C fungiert als Identitätsanbieter (IdP)**<br />Azure AD B2C fungiert als SAML-IdP für die Anwendungen. | Dieser Artikel |
| Meine Benutzer benötigen einmaliges Anmelden (SSO) mit einem SAML-kompatiblen Identitätsanbieter wie ADFS, Salesforce oder Shibboleth.  | **Azure AD B2C fungiert als Dienstanbieter (SP)**<br />Azure AD B2C fungiert als Dienstanbieter, wenn eine Verbindung mit dem SAML-Identitätsanbieter hergestellt wird. Es handelt sich um einen Verbundproxy zwischen Ihrer Anwendung und dem SAML-Identitätsanbieter.  | <ul><li>[Einrichten der Anmeldung mit ADFS als SAML-IdP mithilfe benutzerdefinierter Richtlinien](identity-provider-adfs2016-custom.md)</li><li>[Einrichten der Anmeldung mit einem Salesforce SAML-Anbieter mithilfe benutzerdefinierter Richtlinien](identity-provider-salesforce-custom.md)</li></ul> |

## <a name="prerequisites"></a>Voraussetzungen

* Führen Sie die Schritte unter [Erste Schritte mit benutzerdefinierten Richtlinien in Azure AD B2C](custom-policy-get-started.md) aus. Sie benötigen die benutzerdefinierte Richtlinie *SocialAndLocalAccounts* aus dem Starter Pack für benutzerdefinierte Richtlinien, die in diesem Artikel erläutert wird.
* Grundlegende Kenntnisse des SAML-Protokolls (Security Assertion Markup Language).
* Eine als SAML-Dienstanbieter (SP) konfigurierte Webanwendung. Für dieses Tutorial können Sie eine [SAML-Testanwendung][samltest] verwenden, die wir bereitstellen.

## <a name="components-of-the-solution"></a>Komponenten der Lösung

Für dieses Szenario sind drei Hauptkomponenten erforderlich:

* SAML-**Dienstanbieter** mit der Möglichkeit, SAML-Anforderungen zu senden und SAML-Assertionen von Azure AD B2C zu empfangen, zu decodieren und darauf zu antworten. Dies wird auch als „vertrauende Seite“ bezeichnet.
* Öffentlich verfügbarer SAML-**Metadatenendpunkt** für Ihren Dienstanbieter.
* [Azure AD B2C-Mandant](tutorial-create-tenant.md)

Wenn Sie noch nicht über einen SAML-Dienstanbieter und einen zugehörigen Metadatenendpunkt verfügen, können Sie diese SAML-Beispielanwendung verwenden, die wir zum Testen zur Verfügung stellen:

[SAML-Testanwendung][samltest]

## <a name="1-set-up-certificates"></a>1. Einrichten von Zertifikaten

Um eine Vertrauensstellung zwischen Ihrem Dienstanbieter und Azure AD B2C zu erstellen, müssen Sie X509-Web-App-Zertifikate bereitstellen.

* **Dienstanbieterzertifikate**
  * Zertifikat mit einem privaten Schlüssel, das in Ihrer Web-App gespeichert ist. Dieses Zertifikat wird von Ihrem Dienstanbieter zum Signieren der an Azure AD B2C gesendeten SAML-Anforderung verwendet. Azure AD B2C liest den öffentlichen Schlüssel aus den Dienstanbietermetadaten, um die Signatur zu überprüfen.
  * (Optional) Zertifikat mit einem privaten Schlüssel, das in Ihrer Web-App gespeichert ist. Azure AD B2C liest den öffentlichen Schlüssel aus den Dienstanbietermetadaten, um die SAML-Assertion zu verschlüsseln. Der Dienstanbieter verwendet dann den privaten Schlüssel zum Entschlüsseln der Assertion.
* **Azure AD B2C-Zertifikate**
  * Zertifikat mit einem privaten Schlüssel in Azure AD B2C. Dieses Zertifikat wird von Azure AD B2C zum Signieren der SAML-Antwort verwendet, die an Ihren Dienstanbieter gesendet wird. Der Dienstanbieter liest den öffentlichen Azure AD B2C-Metadatenschlüssel, um die Signatur der SAML-Antwort zu überprüfen.

Sie können ein Zertifikat verwenden, das von einer öffentlichen Zertifizierungsstelle ausgestellt wurde, oder für dieses Tutorial auch ein selbstsigniertes Zertifikat.

### <a name="11-prepare-a-self-signed-certificate"></a>1.1 Vorbereiten eines selbstsignierten Zertifikats

Wenn Sie noch nicht über ein Zertifikat verfügen, können Sie für dieses Tutorial ein selbstsigniertes Zertifikat verwenden. Unter Windows können Sie das PowerShell-Cmdlet [New-SelfSignedCertificate](https://docs.microsoft.com/powershell/module/pkiclient/new-selfsignedcertificate) verwenden, um ein Zertifikat zu generieren.

1. Führen Sie diesen PowerShell-Befehl aus, um ein selbstsigniertes Zertifikat zu generieren. Ändern Sie das Argument `-Subject` entsprechend Ihrer Anwendung und des Azure AD B2C-Mandantennamens. Sie können auch das `-NotAfter`-Datum anpassen, um einen anderen Ablaufzeitpunkt für das Zertifikat anzugeben.

    ```PowerShell
    New-SelfSignedCertificate `
        -KeyExportPolicy Exportable `
        -Subject "CN=yourappname.yourtenant.onmicrosoft.com" `
        -KeyAlgorithm RSA `
        -KeyLength 2048 `
        -KeyUsage DigitalSignature `
        -NotAfter (Get-Date).AddMonths(12) `
        -CertStoreLocation "Cert:\CurrentUser\My"
    ```

1. Öffnen Sie **Benutzerzertifikate verwalten** > **Aktueller Benutzer** > **Persönliche** > **Zertifikate** > *yourappname.yourtenant.onmicrosoft.com*.
1. Wählen Sie das Zertifikat und dann **Aktion** > **Alle Aufgaben** > **Exportieren** aus.
1. Wählen Sie **Ja** > **Weiter** > **Ja, privaten Schlüssel exportieren** > **Weiter** aus.
1. Übernehmen Sie die Standardeinstellungen für **Format der zu exportierenden Datei** aus.
1. Angeben eines Kennworts für das Zertifikat

### <a name="12-upload-the-certificate"></a>1.2 Hochladen des Zertifikats

Laden Sie nun die SAML-Assertion und das Antwortsignaturzertifikat in Azure AD B2C hoch.

1. Melden Sie sich am [Azure-Portal](https://portal.azure.com) an, und navigieren Sie zu Ihrem Azure AD B2C-Mandanten.
1. Wählen Sie unter **Richtlinien** die Option **Identity Experience Framework** und dann **Richtlinienschlüssel** aus.
1. Klicken Sie auf **Hinzufügen** und anschließend auf **Optionen** > **Hochladen**.
1. Geben Sie einen **Namen** ein, z.B. *SamlIdpCert*. Dem Namen Ihres Schlüssels wird automatisch das Präfix *B2C_1A_* hinzugefügt.
1. Laden Sie Ihr Zertifikat mithilfe des Steuerelements zum Hochladen von Dateien hoch.
1. Geben Sie das Kennwort des Zertifikats ein.
1. Klicken Sie auf **Erstellen**.
1. Vergewissern Sie sich, dass der Schlüssel wie erwartet angezeigt wird. Beispielsweise *B2C_1A_SamlIdpCert*.

## <a name="2-prepare-your-policy"></a>2. Vorbereiten Ihrer Richtlinie

### <a name="21-create-the-saml-token-issuer"></a>2.1 Erstellen des SAML-Tokenausstellers

Fügen Sie nun mithilfe der technischen Profile [SAML-Tokenaussteller](saml-issuer-technical-profile.md) und [SAML-Sitzungsanbieter](custom-policy-reference-sso.md#samlssosessionprovider) die Funktion zum Ausstellen von SAML-Token für Ihren Mandanten hinzu.

Öffnen Sie `SocialAndLocalAccounts\`**`TrustFrameworkExtensions.xml`** im Starter Pack für benutzerdefinierte Richtlinien.

Suchen Sie den Abschnitt `<ClaimsProviders>`, und fügen Sie den folgenden XML-Codeausschnitt hinzu.

Sie können den Wert der `IssuerUri`-Metadaten ändern. Dies ist der Aussteller-URI, der in der SAML-Antwort von Azure AD B2C zurückgegeben wird. Die Anwendung der vertrauenden Seite muss so konfiguriert werden, dass während der Überprüfung der SAML-Assertion ein Aussteller-URI akzeptiert wird.

```xml
<ClaimsProvider>
  <DisplayName>Token Issuer</DisplayName>
  <TechnicalProfiles>

    <!-- SAML Token Issuer technical profile -->
    <TechnicalProfile Id="Saml2AssertionIssuer">
      <DisplayName>Token Issuer</DisplayName>
      <Protocol Name="SAML2"/>
      <OutputTokenFormat>SAML2</OutputTokenFormat>
      <Metadata>
        <!-- The issuer contains the policy name; it should be the same name as configured in the relying party application. B2C_1A_signup_signin_SAML is used below. -->
        <!--<Item Key="IssuerUri">https://tenant-name.b2clogin.com/tenant-name.onmicrosoft.com/B2C_1A_signup_signin_SAML</Item>-->
      </Metadata>
      <CryptographicKeys>
        <Key Id="MetadataSigning" StorageReferenceId="B2C_1A_SamlIdpCert"/>
        <Key Id="SamlAssertionSigning" StorageReferenceId="B2C_1A_SamlIdpCert"/>
        <Key Id="SamlMessageSigning" StorageReferenceId="B2C_1A_SamlIdpCert"/>
      </CryptographicKeys>
      <InputClaims/>
      <OutputClaims/>
      <UseTechnicalProfileForSessionManagement ReferenceId="SM-Saml-issuer"/>
    </TechnicalProfile>

    <!-- Session management technical profile for SAML based tokens -->
    <TechnicalProfile Id="SM-Saml-issuer">
      <DisplayName>Session Management Provider</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.SSO.SamlSSOSessionProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null"/>
    </TechnicalProfile>

  </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="3-add-the-saml-relying-party-policy"></a>3. Hinzufügen der SAML-Richtlinie für die vertrauende Seite

Da Ihr nun Mandant SAML-Assertionen ausstellen kann, müssen Sie die SAML-Richtlinie der vertrauenden Seite erstellen und die User Journey so ändern, dass sie eine SAML-Assertion anstelle eines JWT ausgibt.

### <a name="31-create-sign-up-or-sign-in-policy"></a>3.1 Erstellen einer Registrierungs- oder Anmelderichtlinie

1. Erstellen Sie eine Kopie der Datei *SignUpOrSignin.xml* in Ihrem Starter Pack-Arbeitsverzeichnis, und speichern Sie diese unter einem neuen Namen. Beispielsweise unter dem Namen *SignUpOrSigninSAML.xml*. Dies ist die Richtliniendatei der vertrauenden Seite.

1. Öffnen Sie die Datei *SignUpOrSigninSAML.xml* in Ihrem bevorzugten Editor.

1. Ändern Sie `PolicyId` und `PublicPolicyUri` der Richtlinie in _B2C_1A_signup_signin_saml_ und `http://tenant-name.onmicrosoft.com/B2C_1A_signup_signin_saml` wie unten gezeigt.

    ```xml
    <TrustFrameworkPolicy
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
    PolicySchemaVersion="0.3.0.0"
    TenantId="tenant-name.onmicrosoft.com"
    PolicyId="B2C_1A_signup_signin_saml"
    PublicPolicyUri="http://tenant-name.onmicrosoft.com/B2C_1A_signup_signin_saml">
    ```

1. Fügen Sie folgenden XML-Codeausschnitt direkt vor dem `<RelyingParty>`-Element hinzu. Dieser XML-Code überschreibt den Orchestrierungsschritt Nummer 7 der User Journey _SignUpOrSignIn_. Wenn Sie aus einem anderen Ordner im Starter Pack gestartet sind oder die User Journey durch Hinzufügen oder Entfernen von Orchestrierungsschritten angepasst haben, stellen Sie sicher, dass die Anzahl (im `order`-Element) dem im User Journey für den Tokenausstellungsschritt angegebenen Wert entspricht (in den anderen Starter Pack-Ordnern ist es z. B. Schritt 4 für `LocalAccounts`, 6 für `SocialAccounts` und 9 für `SocialAndLocalAccountsWithMfa`).

    ```xml
    <UserJourneys>
      <UserJourney Id="SignUpOrSignIn">
        <OrchestrationSteps>
          <OrchestrationStep Order="7" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="Saml2AssertionIssuer"/>
        </OrchestrationSteps>
      </UserJourney>
    </UserJourneys>
    ```

1. Ersetzen Sie das gesamte `<TechnicalProfile>`-Element im `<RelyingParty>`-Element durch das folgende technische Profil-XML.

    ```xml
    <TechnicalProfile Id="PolicyProfile">
      <DisplayName>PolicyProfile</DisplayName>
      <Protocol Name="SAML2"/>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName" />
        <OutputClaim ClaimTypeReferenceId="givenName" />
        <OutputClaim ClaimTypeReferenceId="surname" />
        <OutputClaim ClaimTypeReferenceId="email" DefaultValue="" />
        <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="" />
        <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="objectId"/>
      </OutputClaims>
      <SubjectNamingInfo ClaimType="objectId" ExcludeAsClaim="true"/>
    </TechnicalProfile>
    ```

1. Ersetzen Sie `tenant-name` durch den Namen Ihres Azure AD B2C-Mandanten.

Die endgültige Richtliniendatei der vertrauenden Seite sollte wie folgt aussehen:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<TrustFrameworkPolicy
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:xsd="http://www.w3.org/2001/XMLSchema"
  xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
  PolicySchemaVersion="0.3.0.0"
  TenantId="contoso.onmicrosoft.com"
  PolicyId="B2C_1A_signup_signin_saml"
  PublicPolicyUri="http://contoso.onmicrosoft.com/B2C_1A_signup_signin_saml">

  <BasePolicy>
    <TenantId>contoso.onmicrosoft.com</TenantId>
    <PolicyId>B2C_1A_TrustFrameworkExtensions</PolicyId>
  </BasePolicy>

  <UserJourneys>
    <UserJourney Id="SignUpOrSignIn">
      <OrchestrationSteps>
        <OrchestrationStep Order="7" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="Saml2AssertionIssuer"/>
      </OrchestrationSteps>
    </UserJourney>
  </UserJourneys>

  <RelyingParty>
    <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
    <TechnicalProfile Id="PolicyProfile">
      <DisplayName>PolicyProfile</DisplayName>
      <Protocol Name="SAML2"/>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName" />
        <OutputClaim ClaimTypeReferenceId="givenName" />
        <OutputClaim ClaimTypeReferenceId="surname" />
        <OutputClaim ClaimTypeReferenceId="email" DefaultValue="" />
        <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="" />
        <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="objectId"/>
      </OutputClaims>
      <SubjectNamingInfo ClaimType="objectId" ExcludeAsClaim="true"/>
    </TechnicalProfile>
  </RelyingParty>
</TrustFrameworkPolicy>
```

### <a name="32-upload-and-test-your-policy-metadata"></a>3.2 Hochladen und Testen der Richtlinienmetadaten

Speichern Sie die Änderungen, und laden Sie die neue Richtliniendatei hoch. Nachdem Sie beide Richtlinien (die Erweiterung und die Dateien der vertrauenden Seite) hochgeladen haben, öffnen Sie einen Webbrowser, und navigieren Sie dann zu den Richtlinienmetadaten.

Die IDP-Metadaten der Azure AD B2C-Richtlinie sind Informationen, die im SAML-Protokoll verwendet werden, um die Konfiguration eines SAML-Identitätsanbieters offenzulegen. Die Metadaten definieren den Ort der Dienste, z.B. Anmelden und Abmelden, Zertifikate und die Anmeldemethode. Die Azure AD B2C-Richtlinienmetadaten sind unter der folgenden URL verfügbar. Ersetzen Sie `tenant-name` durch den Namen Ihres Azure AD B2C-Mandanten und `policy-name` durch den Namen (ID) der Richtlinie:

`https://tenant-name.b2clogin.com/tenant-name.onmicrosoft.com/policy-name/Samlp/metadata`

Ihre benutzerdefinierte Richtlinie und der Azure AD B2C-Mandant sind jetzt bereit. Erstellen Sie als nun eine Anwendungsregistrierung in Azure AD B2C.

## <a name="4-setup-application-in-the-azure-ad-b2c-directory"></a>4. Einrichten der Anwendung im Azure AD B2C-Verzeichnis

### <a name="41-register-your-application-in-azure-ad-b2c"></a>4.1 Registrieren Ihrer Anwendung bei Azure AD B2C

1. Melden Sie sich beim [Azure-Portal](https://portal.azure.com) an.
1. Wählen Sie im oberen Menü den Filter **Verzeichnis und Abonnement** aus, und wählen Sie dann das Verzeichnis aus, das Ihren Azure AD B2C-Mandanten enthält.
1. Wählen Sie im linken Menü die Option **Azure AD B2C** aus. Oder wählen Sie **Alle Dienste** aus, suchen Sie nach dem Eintrag **Azure AD B2C**, und wählen Sie ihn aus.
1. Wählen Sie **App-Registrierungen** aus, und wählen Sie dann **Registrierung einer neuen Anwendung** aus.
1. Geben Sie unter **Name** einen Namen für die Anwendung ein. Beispielsweise *SAMLApp1*.
1. Wählen Sie unter **Unterstützte Kontotypen** die Option **Nur Konten in diesem Organisationsverzeichnis** aus.
1. Wählen Sie unter **Umleitungs-URIs** die Option **Web** aus, und geben Sie dann `https://localhost` ein. Diesen Wert ändern Sie später im Manifest der Anwendungsregistrierung.
1. Wählen Sie **Registrieren**.

### <a name="42-update-the-app-manifest"></a>4.2 Aktualisieren des App-Manifests

Für SAML-Apps müssen Sie im Manifest der Anwendungsregistrierung verschiedene Eigenschaften konfigurieren.

1. Navigieren Sie im [Azure-Portal](https://portal.azure.com) zu der Anwendungsregistrierung, die Sie im vorherigen Abschnitt erstellt haben.
1. Wählen Sie unter **Verwalten** die Option **Manifest** aus, um den Manifest-Editor zu öffnen. Sie ändern mehrere Eigenschaften in den folgenden Abschnitten.

#### <a name="identifieruris"></a>identifierUris

Der `identifierUris` ist eine Zeichenfolgensammlung mit benutzerdefinierten URIs, die eine Web-App innerhalb ihres Azure AD B2C-Mandanten eindeutig identifizieren. Der Dienstanbieter muss diesen Wert im `Issuer`-Element einer SAML-Anforderung festlegen.

#### <a name="samlmetadataurl"></a>samlMetadataUrl

Diese Eigenschaft stellt die öffentlich verfügbare Metadaten-URL des Dienstanbieters dar. Die Metadaten-URL kann auf eine Metadatendatei verweisen, die auf einen anonym zugänglichen Endpunkt hochgeladen wird, z.B. in Blobspeicher.

Die Metadaten sind Informationen, die im SAML-Protokoll verwendet werden, um die Konfiguration einer SAML-Partei (z.B. eines Dienstanbieters) offenzulegen. Metadaten definieren den Standort der Dienste wie An- und Abmeldung, Zertifikate, Anmeldemethode und mehr. Azure AD B2C liest die Metadaten des Dienstanbieters und verhält sich entsprechend. Die Metadaten sind nicht erforderlich. Sie können auch einige Attribute wie den Antwort-URI und den Abmelde-URI direkt im App-Manifest angeben.

Wenn Eigenschaften *sowohl* in der SAML-Metadaten-URL als auch im Manifest der Anwendungsregistrierung angegeben werden, werden diese **gemergt**. Die in der Metadaten-URL angegebenen Eigenschaften werden zuerst verarbeitet und besitzen Vorrang.

Verwenden Sie in diesem Tutorial, in dem die SAML-Testanwendung verwendet wird, für `samlMetadataUrl` den folgenden Wert:

```json
"samlMetadataUrl":"https://samltestapp2.azurewebsites.net/Metadata",
```

#### <a name="replyurlswithtype-optional"></a>replyUrlsWithType (Optional)

Wenn Sie keinen Metadaten-URI angeben, können Sie die Antwort-URL explizit angeben. Diese optionale Eigenschaft stellt die `AssertionConsumerServiceUrl` (`SingleSignOnService`-URL in den Metadaten des Dienstanbieters) dar, und `BindingType` wird als `HTTP POST`angenommen.

Wenn Sie die Antwort-URL und die Abmelde-URL im Anwendungsmanifest konfigurieren, ohne die Metadaten des Dienstanbieters zu verwenden, validiert Azure AD B2C die Signatur der SAML-Anforderung nicht und verschlüsselt auch nicht die SAML-Antwort.

Legen Sie für dieses Tutorial, in dem Sie die SAML-Testanwendung verwenden, die `url`-Eigenschaft von `replyUrlsWithType` auf den Wert fest, der im folgenden JSON-Codeausschnitt gezeigt wird.

```json
"replyUrlsWithType":[
  {
    "url":"https://samltestapp2.azurewebsites.net/SP/AssertionConsumer",
    "type":"Web"
  }
],
```

#### <a name="logouturl-optional"></a>LogoutUrl (optional)

Diese optionale Eigenschaft stellt die `Logout`-URL (`SingleLogoutService`-URL in den Metadaten der vertrauenden Seite) dar, und `BindingType` wird als `Http-Redirect`angenommen.

Behalten Sie für dieses Tutorial, in dem die SAML-Testanwendung verwendet wird, die Festlegung von `logoutUrl` auf `https://samltestapp2.azurewebsites.net/logout` bei:

```json
"logoutUrl": "https://samltestapp2.azurewebsites.net/logout",
```

## <a name="5-update-your-application-code"></a>5. Aktualisieren des Anwendungscodes

Der letzte Schritt besteht darin, Azure AD B2C als SAML-IdP in Ihrer SAML-Anwendung der vertrauenden Seite zu aktivieren. Jede Anwendung ist anders, und die auszuführenden Schritte sind daher unterschiedlich. Weitere Informationen finden Sie in der Dokumentation zu Ihrer App.

In der Regel sind einige oder alle der folgenden Angaben erforderlich:

* **Metadaten**: `https://tenant-name.b2clogin.com/tenant-name.onmicrosoft.com/policy-name/Samlp/metadata`
* **Aussteller:**   Verwenden Sie den entityID-Wert in der Metadatendatei.
* **Anmelde-URL/SAML-Endpunkt/SAML-URL-** : Überprüfen Sie den Wert in der Metadatendatei.
* **Zertifikat**: Dies ist *B2C_1A_SamlIdpCert*, aber ohne den privaten Schlüssel. So rufen Sie den öffentlichen Schlüssel des Zertifikats ab:

    1. Navigieren Sie zur Metadaten-URL, die oben angegeben wurde.
    1. Kopieren Sie den Wert in das `<X509Certificate>`-Element.
    1. Fügen Sie ihn in eine Textdatei ein.
    1. Speichern Sie die Textdatei als *CER*-Datei.

### <a name="51-test-with-the-saml-test-app-optional"></a>5.1 Testen mit der SAML-Test-App (optional)

Um dieses Tutorial mithilfe unserer [SAML-Testanwendung][samltest] abzuschließen, führen Sie Folgendes aus:

* Aktualisieren des Mandantennamens
* Aktualisieren Sie den Richtliniennamen, z.B. *B2C_1A_signup_signin_saml*.
* Geben Sie diesen Aussteller-URI an: `https://contoso.onmicrosoft.com/app-name`.

Wählen Sie **Login** (Anmelden) aus. Ein Bildschirm für die Benutzeranmeldung sollte angezeigt werden. Bei der Anmeldung wird eine SAML-Assertion an die Beispielanwendung zurückgegeben.

## <a name="enable-encrypted-assertions-optional"></a>Aktivieren verschlüsselter Assertionen (optional)

Zum Verschlüsseln von SAML-Assertionen, die an den Dienstanbieter zurückgesendet werden, verwendet Azure AD B2C das Zertifikat für öffentliche Schlüssel von Dienstanbietern. Der öffentliche Schlüssel muss in den SAML-Metadaten vorhanden sein, die in der obigen ["samlMetadataUrl"](#samlmetadataurl)-Eigenschaft als KeyDescriptor mit der Verwendung von „Encryption“ (Verschlüsselung) beschrieben werden.

Nachfolgend finden Sie ein Beispiel für den KeyDescriptor der SAML-Metadaten, bei dem die „use“ auf „Encryption“ festgelegt ist:

```xml
<KeyDescriptor use="encryption">
  <KeyInfo xmlns="https://www.w3.org/2000/09/xmldsig#">
    <X509Data>
      <X509Certificate>valid certificate</X509Certificate>
    </X509Data>
  </KeyInfo>
</KeyDescriptor>
```

Legen Sie im [technischen Profil der vertrauenden Seite](relyingparty.md#technicalprofile) das Metadatenelement **WantsEncryptedAssertion** auf `true` fest, um Azure AD B2C das Senden verschlüsselter Assertionen zu ermöglichen. Sie können auch den zum Verschlüsseln der SAML-Assertion verwendeten Algorithmus konfigurieren. Weitere Informationen finden Sie unter [Metadaten für das technische Profil der vertrauenden Seite](relyingparty.md#metadata). 

```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
  <TechnicalProfile Id="PolicyProfile">
    <DisplayName>PolicyProfile</DisplayName>
    <Protocol Name="SAML2"/>
    <Metadata>
      <Item Key="WantsEncryptedAssertions">true</Item>
    </Metadata>
   ..
  </TechnicalProfile>
</RelyingParty>
```

## <a name="enable-identity-provider-initiated-flow-optional"></a>Aktivieren des vom Identitätsanbieter initiierten Flows (optional)

Beim vom Identitätsanbieter initiierten Flow wird der Anmeldevorgang vom Identitätsanbieter (Azure AD B2C) initiiert, der eine nicht angeforderte SAML-Antwort an den Dienstanbieter (Ihre Anwendung der vertrauenden Seite) sendet. Um den vom Identitätsanbieter initiierten Flow zu aktivieren, legen Sie im [technischen Profil der vertrauenden Seite](relyingparty.md#technicalprofile) das Metadatenelement **IdpInitiatedProfileEnabled** auf `true` fest.

```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
  <TechnicalProfile Id="PolicyProfile">
    <DisplayName>PolicyProfile</DisplayName>
    <Protocol Name="SAML2"/>
    <Metadata>
      <Item Key="IdpInitiatedProfileEnabled">true</Item>
    </Metadata>
   ..
  </TechnicalProfile>
</RelyingParty>
```

Verwenden Sie die folgende URL, um einen Benutzer über den vom Identitätsanbieter initiierten Flow anzumelden oder zu registrieren:

```
https://tenant-name.b2clogin.com/tenant-name.onmicrosoft.com/policy-name/generic/login
```

Ersetzen Sie die folgenden Werte:

* **tenant-name** durch den Namen Ihres Mandanten
* **policy-name** durch den Namen der SAML-Richtlinie für die vertrauende Seite

## <a name="sample-policy"></a>Beispielrichtlinie

Wir stellen eine vollständige Beispielrichtlinie zur Verfügung, die Sie zum Testen mit der SAML-Test-App verwenden können.

1. Laden Sie die [Anmeldebeispielrichtlinie „SAML-SP-initiated“](https://github.com/azure-ad-b2c/saml-sp/tree/master/policy/SAML-SP-Initiated) herunter.
1. Aktualisieren Sie `TenantId` entsprechend Ihrem Mandantennamen, z.B. *contoso.b2clogin.com*.
1. Behalten Sie den Richtliniennamen *B2C_1A_SAML2_signup_signin* bei.

## <a name="supported-and-unsupported-saml-modalities"></a>Unterstützte und nicht unterstützte SAML-Modalitäten

Die folgenden SAML-Szenarien der vertrauenden Seite (Relying Party, RP) werden über Ihren eigenen Metadatenendpunkt unterstützt:

* Mehrere Abmelde-URLs oder POST-Bindung für die Abmelde-URL im Anwendungs-/Dienstprinzipalobjekt.
* Angeben eines Signaturschlüssels zum Überprüfen von RP-Anforderungen im Anwendungs-/Dienstprinzipalobjekt.
* Angeben eines Tokenverschlüsselungsschlüssels im Anwendungs-/Dienstprinzipalobjekt.
* Die vom Identitätsanbieter initiierte Anmeldung, bei der der Identitätsanbieter Azure AD B2C ist.

Die folgenden SAML-Szenarien der vertrauenden Seite (RP) werden zurzeit nicht unterstützt:
* Die vom Identitätsanbieter initiierte Anmeldung, bei der der Identitätsanbieter ein externer Anbieter ist, z. B. ADFS.

## <a name="next-steps"></a>Nächste Schritte

- Weitere Informationen zum [SAML-Protokoll finden Sie auf der OASIS-Website](https://www.oasis-open.org/).
- Rufen Sie die SAML-Test-Web-App im [Repository der Azure AD B2C-GitHub-Community](https://github.com/azure-ad-b2c/saml-sp-tester) ab.

<!-- LINKS - External -->
[samltest]: https://aka.ms/samltestapp
