**CIS Ubuntu Linux 24.04 LTS Benchmark - Package Management**

# **1.2 Package Management**
Proper package management is critical for maintaining system security and stability. Ensuring secure package repositories and timely updates helps mitigate vulnerabilities, prevent software supply chain attacks, and keep the system in compliance with security best practices.

---

## **1.2.1 Configure Package Repositories**  

### **1.2.1.1 Ensure GPG keys are configured (Manual)**  
**Description:**  
GPG (GNU Privacy Guard) keys are used to verify the integrity and authenticity of software packages before they are installed. Ensuring that GPG keys are properly configured prevents malicious or tampered packages from being installed.

**Rationale:**  
If the package manager does not have valid GPG keys, it cannot verify the authenticity of software updates. Attackers could inject malicious software into repositories, leading to unauthorized access or data breaches.

**Audit Procedure:**  
Run the following command to check if GPG keys are properly configured:
```bash
apt-key list
```
Alternatively, on newer versions of Ubuntu:
```bash
gpg --list-keys --keyring /etc/apt/trusted.gpg
```

**Remediation:**  
To add a missing or valid GPG key, use:
```bash
curl -fsSL https://download.example.com/pubkey.gpg | sudo gpg --dearmor -o /usr/share/keyrings/example.gpg
```
Then update your repository to use the keyring:
```bash
echo "deb [signed-by=/usr/share/keyrings/example.gpg] http://download.example.com/ubuntu focal main" | sudo tee /etc/apt/sources.list.d/example.list
```

**Real-World Example:**  
Imagine an attacker compromises a software repository and injects a backdoored package. Without proper GPG key verification, the system might install this malicious software, leading to potential data theft or ransomware infections. Ensuring that packages are signed with a trusted GPG key mitigates such risks.

---

### **1.2.1.2 Ensure package manager repositories are configured (Manual)**  
**Description:**  
Ubuntu uses APT (Advanced Package Tool) for managing repositories. Ensuring repositories are correctly configured prevents the use of untrusted sources and ensures that only verified software is installed.

**Rationale:**  
Untrusted repositories may contain outdated, vulnerable, or malicious software. Configuring repositories securely ensures that the system only retrieves packages from trusted sources.

**Audit Procedure:**  
Check the current repository configuration:
```bash
cat /etc/apt/sources.list
ls -l /etc/apt/sources.list.d/
```

**Remediation:**  
To remove an untrusted repository:
```bash
sudo rm /etc/apt/sources.list.d/untrusted-repo.list
sudo apt update
```
To add a trusted repository:
```bash
echo "deb http://security.ubuntu.com/ubuntu focal-security main" | sudo tee -a /etc/apt/sources.list
```

**Real-World Example:**  
A company unknowingly adds an unverified third-party repository to install software. Later, the repository is compromised, and users start receiving malicious updates. Ensuring repositories are from trusted sources helps prevent such supply chain attacks.

---

## **1.2.2 Configure Package Updates**  

### **1.2.2.1 Ensure updates, patches, and additional security software are installed (Manual)**  
**Description:**  
Regular updates and security patches fix known vulnerabilities and enhance system stability. Ensuring updates are applied promptly is essential for maintaining a secure system.

**Rationale:**  
Unpatched systems are an easy target for attackers. Delayed updates can leave critical vulnerabilities exposed, allowing attackers to exploit them.

**Audit Procedure:**  
Check for available updates:
```bash
sudo apt update && sudo apt list --upgradable
```

**Remediation:**  
To apply updates manually:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt autoremove -y
```
To enable automatic security updates, install and configure **unattended-upgrades**:
```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure unattended-upgrades
```

**Real-World Example:**  
The WannaCry ransomware exploited an unpatched Windows vulnerability, affecting thousands of businesses worldwide. Similarly, unpatched Linux systems are at risk of cyberattacks. A company that delays installing security updates could face serious data breaches.

---

### **Key Takeaways:**  
✅ Always verify software packages using GPG keys.  
✅ Use only trusted repositories to prevent malicious package installations.  
✅ Regularly update and patch the system to mitigate security vulnerabilities.  

---

**End of Document**

