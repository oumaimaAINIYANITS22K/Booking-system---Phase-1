# **Penetration Testing Report - Booking System (Phase 1 & Phase 2)**

## **📌 Introduction**  
This report documents the penetration testing performed on the **Booking System** across **two phases**. The testing focused on identifying security vulnerabilities related to **authentication, input validation, API security, and potential misconfigurations**.

Testing was conducted using:  
- **Kali Linux (Virtual Machine)**
- **OWASP ZAP by Checkmarx**
- **Burp Suite**
- **Manual SQL Injection Testing**
- **Docker-based deployment for testing**

---

# **Phase 1: Security Testing Findings**

## **🔍 Summary of Findings (Phase 1)**  

| Severity | Issue Found | Description | Suggested Fix |
|----------|------------|-------------|---------------|
| 🔴 **Critical** | **Plaintext Password Storage** | User passwords are stored in plaintext in the database. | Implement bcrypt hashing before storing passwords. |
| ⚠ **Medium** | **SQL Injection (SQLi)** | The registration form is vulnerable to SQL Injection. | Use **prepared statements** and **input validation**. |
| ⚠ **Medium** | **User Role Manipulation** | Users can escalate privileges via request parameters. | Enforce **server-side role validation**. |
| 🔹 **Informational** | **User-Agent Fuzzer Alert** | The system responded differently to User-Agent header variations. | Ensure uniform response handling for all User-Agents. |

---

## **1️⃣ Critical Issue: Plaintext Password Storage (🔴 Critical)**
- **Description:**  
  - Passwords are stored **in plaintext** in the `xyz123_users` table.
- **How It Was Found:**  
  - A simple SQL query revealed unencrypted passwords:
    ```sql
    SELECT * FROM xyz123_users;
    ```
- **Risk:**  
  - Attackers can **steal credentials** and reuse them for other accounts.  
  - This violates **GDPR and security best practices**.  
- **Suggested Fix:**  
  - Use **bcrypt hashing** before storing passwords:
    ```python
    from flask_bcrypt import Bcrypt
    bcrypt = Bcrypt()
    hashed_pw = bcrypt.generate_password_hash("password123").decode('utf-8')
    ```
  - **Ensure all passwords are hashed and never stored in plaintext.**

---

## **2️⃣ SQL Injection in Registration Form (⚠ Medium)**
- **Description:**  
  - The form allows SQL Injection, which could be used to access the database.
- **How It Was Found:**  
  - Tested with this SQL payload:
    ```sql
    ' OR 1=1 --
    ```
  - This allowed login **without valid credentials**.
- **Suggested Fix:**  
  - Use **prepared statements**:
    ```python
    cursor.execute("SELECT * FROM users WHERE username = ? AND password = ?", (user_input, password))
    ```

---

## **3️⃣ User Role Manipulation (⚠ Medium)**
- **Description:**  
  - Users can **escalate privileges** by modifying the request parameters.
- **How It Was Found:**  
  - Changed role from `user` to `admin` in a **POST request**.
- **Suggested Fix:**  
  - **Enforce role-based authentication on the server.**

---

# **Phase 2: Security Testing Findings**

## **🔍 Summary of Findings (Phase 2)**  

| Severity | Issue Found | Description | Suggested Fix |
|----------|------------|-------------|---------------|
| ⚠ **Medium** | **User-Agent Fuzzer Alert** | The system responded differently to various User-Agent headers, which could lead to information leakage. | Ensure uniform response handling for all User-Agent headers. |
| 🔹 **Informational** | **ZAP Scan Findings** | No high or medium severity vulnerabilities found, but response inconsistencies were noted. | Review and standardize API responses. |

---

## **4️⃣ User-Agent Fuzzer Alert (⚠ Medium) - Phase 2**
- **Description:**  
  - The system responded **differently** when tested with multiple **User-Agent headers**, indicating possible behavioral inconsistencies.
- **How It Was Found:**  
  - A fuzzing attack was conducted with various User-Agent headers (e.g., **Googlebot, iPhone Safari, Internet Explorer**), and variations in the response were observed.
- **Risk:**  
  - Attackers could **spoof their User-Agent** to gain unintended access or scrape sensitive information.
- **Suggested Fix:**  
  - Ensure **uniform response handling** for all User-Agent headers to avoid behavior inconsistencies.
  - Implement **bot detection mechanisms** that rely on behavioral analysis rather than simple User-Agent validation.

---

## **📄 Lessons Learned & Reflections**

### **Challenges Faced**
1. **Docker Port Conflicts:**
   - Encountered **port 8000 and 5432 conflicts**, requiring manual intervention to stop existing containers before starting new ones.
   - Commands used to resolve:
     ```bash
     sudo docker ps
     sudo docker stop <container_id>
     sudo netstat -tulnp | grep <port>
     ```

2. **ZAP Scan Execution & Analysis:**
   - Learned how to interpret **User-Agent fuzzer alerts** and their implications for security.
   - Discovered potential **bot detection weaknesses** that could be exploited by attackers.
   
3. **Security Testing in Virtualized Environments:**
   - The test environment was **fully contained within a Kali Linux VM**, ensuring safe testing.
   - Encountered **network connectivity issues** due to virtual machine isolation, which were resolved by configuring the VM’s network adapter properly.

### **Key Takeaways**
✅ **Testing should include multiple attack vectors:** While the ZAP scan found no critical vulnerabilities, manual testing revealed response inconsistencies.  
✅ **Security scanners help detect subtle behavior changes:** Automated tools can highlight potential weaknesses that require further investigation.  
✅ **Understanding Docker and Linux troubleshooting is crucial:** Hands-on experience in resolving container conflicts is essential for effective penetration testing.  

---

## **📌 Conclusion & Next Steps**
The penetration tests across **Phase 1 and Phase 2** revealed **multiple vulnerabilities** in authentication, input validation, and API response behavior. While **Phase 1** had critical security flaws like **plaintext passwords and SQL Injection**, **Phase 2** focused more on **inconsistent User-Agent handling** and **API response issues**.

📌 **Next Steps:**
✔ **Fix vulnerabilities using recommended solutions.**  
✔ **Re-run penetration testing after fixes are applied.**  
✔ **Perform in-depth manual API testing beyond automated scanning.**  
✔ **Investigate additional security hardening measures for authentication and input validation.**  

🚀 **This report serves as a comprehensive security assessment of the Booking System across both phases.**
