# 🛡️ Real-Time File Integrity Monitor (FIM)

### **Developed by Skyler Silverio**
*IT Professional | Cybersecurity Specialist | M365 Administrator*

---

## 📌 Project Overview
This project is an **Event-Driven File Integrity Monitor** designed to provide real-time visibility into unauthorized file system changes. Unlike traditional polling-based monitors, this system leverages OS-level signals to detect tampering the millisecond it occurs, ensuring high performance with minimal CPU overhead.

---

## ⚖️ Compliance & Governance Mapping
This tool maps directly to industry-standard security controls, demonstrating how technical automation supports regulatory requirements:

* **NIST Cybersecurity Framework (CSF):** Supports the **Protect** and **Detect** functions by ensuring data integrity and flagging unauthorized changes.
* **ISO/IEC 27001:** Aligns with **Annex A.12.1.2** (Change Management) and **A.12.4.1** (Event Logging).
* **PCI-DSS (Requirement 11.5):** Addresses the specific mandate for deploying file-integrity monitoring to alert personnel to unauthorized modifications of critical system files.
* **SOC2 Integrity:** Validates the **Processing Integrity** principle by ensuring data is not altered in an unauthorized manner.

---

## 🛠️ Technical Architecture

### 1. Hashing Engine (SHA-256)
The core of the system uses the `hashlib` library to generate **SHA-256** checksums.
* **Collision Resistance:** Ensures even a single bit change results in a unique hash.
* **State Management:** Fingerprints are stored in a structured `baseline.json` file for persistent tracking.

### 2. Event-Driven Monitoring
Utilizing the `watchdog` library, the script acts as a listener for Operating System events.
* **Efficiency:** Zero "polling" overhead; the script remains idle until an event occurs.
* **Noise Filtering:** Implemented a logical "cooldown" (debounce) using `time.sleep` to filter out temporary metadata updates created by Windows during file operations.

### 3. Alerting System
* **SMTP Integration:** Securely configured via `smtplib` using Google App Passwords.
* **Instant Notification:** Sends detailed security alerts to the administrator's email upon verified changes.

---

## 🧠 Lessons Learned & Technical Challenges

Building this FIM provided deep insights into the intersection of software development and security auditing. Below are the key technical hurdles overcome during development:

### 1. Handling Operating System "Noise"
**The Challenge:** Initial testing showed that a single "Save" action in Windows triggered 5-7 separate `Modified` alerts. This is because modern OSs perform atomic writes, creating temporary hidden files and updating metadata (timestamps/permissions) in separate bursts.
**The Solution:** Implemented a **logical debounce (cooldown)**. By adding a 0.5-second delay and a secondary `os.path.exists` check, the script waits for the OS to finalize the write operation, ensuring only the final, persistent change is flagged.

### 2. The "Self-Alerting" Infinite Loop
**The Challenge:** Since the `baseline.json` database was stored within the monitored directory, every time the script updated its own database, it triggered a `Modified` event. This created an infinite loop of alerts and emails.
**The Solution:** Integrated an **event filter** within the `FIMHandler` class. By checking `if "baseline.json" not in event.src_path`, the script effectively ignores its own heartbeat while maintaining a high sensitivity to all other files.

### 3. Secure Credential Management
**The Challenge:** Hardcoding email passwords in a script is a critical security vulnerability, especially when pushing code to public repositories like GitHub.
**The Solution:** Adopted the **Environment Variable** pattern using `python-dotenv`. This allowed the separation of sensitive configuration from the source code, facilitating the use of `.gitignore` to keep credentials private while keeping the code portable.

### 4. Evolution of Monitoring Architecture
**The Challenge:** Comparing manual polling (looping `os.listdir`) vs. event-driven monitoring. 
**The Insight:** While polling is simpler to write, it is CPU-intensive and can miss rapid "create-then-delete" actions. Transitioning to the `watchdog` library (Event-Driven) allowed the script to scale to larger directories with near-zero idle CPU usage.

---

## 🤖 Technical Debt & Future Plans

Currently, the FIM does not have hash comparison logic built in. As a result, even if a file was opened and saved without changes made, the system would create a false alert that would be sent to the administrator. By implementing a hash comparison logic, it shoulds decrease the amount of false alerts received. Additionally, the FIM does not have tracking on who made the changes, only when and what event occurred. I plan to revisit this project in the future by implementing all of these features and potential implementation with cloud-based environments.
