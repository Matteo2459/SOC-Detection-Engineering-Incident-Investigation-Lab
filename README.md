# SOC Detection Engineering & Incident Investigation Lab

## Descrizione del Progetto
Questo progetto documenta la creazione e la validazione di un laboratorio SOC orientato alla **Detection Engineering** e all'**Incident Investigation**. 

L'obiettivo principale è passare da un approccio di difesa tradizionale basato sulla semplice "ricerca di file malevoli"  a un monitoraggio avanzato di tipo **comportamentale** , analizzando le tattiche e le tecniche utilizzate dagli attaccanti nel framework **MITRE ATT&CK**.

---

##  Architettura del Laboratorio e Componenti
Il laboratorio è strutturato con un'architettura Client-Server scalabile:

*   **SIEM / Log Manager:** Server Wazuh centralizzato installato su macchina virtuale **Debian**.
*   **Target Machine (Vittima):** VM **Windows** monitorata in tempo reale.
*   **Endpoint Telemetry:** **Microsoft Sysmon** configurato sulla macchina Windows per registrare eventi granulari di sistema (creazione processi, connessioni di rete, modifiche al registro).
*   **Wazuh Agent** su Windows per l'inoltro sicuro e immediato della telemetria verso il SIEM.
*   **Attack Simulator:** Framework **Atomic Red Team**  installato sul target per emulare attacchi reali in modo sicuro e controllato.

---

##  Il Flusso di Rilevamento 
1. **Emulazione della Minaccia:** Tramite PowerShell, viene lanciato un test specifico di *Atomic Red Team*.
2. **Generazione della Telemetria:** *Sysmon* intercetta il comportamento anomalo a basso livello e genera un Event ID specifico sul Registro Eventi di Windows.
3. **Inoltro dei Log:** Il *Wazuh Agent* raccoglie l'evento in tempo reale e lo spedisce al *Wazuh Manager*.
4. **Analisi e Alerting:** Il motore di regole di *Wazuh* decodifica il log, correla le informazioni e fa scattare un allarme critico sulla Dashboard Web.

---

## Caso di Studio & Investigazione: OS Credential Dumping (MITRE T1003.005)

Per convalidare l'efficacia del laboratorio, è stata simulata una delle tecniche più diffuse nelle fasi di post-compromissione: il furto delle credenziali dal Windows Credential Manager.

### 1. Esecuzione del Test d'Attacco
È stato eseguito  **T1003-6** tramite PowerShell:

Invoke-AtomicTest T1003-6
### 2. Analisi della Detection (Wazuh Dashboard)
Il SIEM ha rilevato l'attività generando molteplici avvisi ad altissima criticità:

**Severità dell'Allarme**: Livello 15 (Il massimo livello di criticità in Wazuh).

**Rule ID**: 92213 (Rilevamento di attività sospette legate a processi di sistema ).

### 3. Triage e Investigation dei Log (Indicatori di Compromissione)
L'analisi forense del log catturato ha rivelato dettagli cruciali sulle azioni svolte in background :

Abuso di Binari Legittimi (LOLBins): È stata rilevata l'esecuzione di C:\Windows\System32\rundll32.exe utilizzata per richiamare in modo anomalo la libreria di sistema keymgr.dll al fine di esportare le password.

Compile After Delivery (MITRE T1027.004): La telemetria di Sysmon ha intercettato il processo csc.exe (C# Compiler) mentre creava e compilava dinamicamente una DLL sul disco (RuleName: DLL) pochi istanti prima dell'attacco, evidenziando il tentativo del framework di generare il payload direttamente sull'host.

