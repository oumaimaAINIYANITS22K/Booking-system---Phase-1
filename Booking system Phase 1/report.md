# **Penetration Testing Report - Booking System (Phase 1)**  

## **📌 Introduction**  
This report documents the penetration testing performed on the **Booking System’s registration page**. The testing focused on identifying security vulnerabilities related to **authentication, input validation, and user role assignment**.  

Testing was conducted using:  
- **Kali Linux**
- **OWASP ZAP**
- **Burp Suite**
- **Manual SQL Injection Testing**

---

## **🔍 Summary of Findings**  

| Severity | Issue Found | Description | Suggested Fix |
|----------|------------|-------------|---------------|
| 🔴 **Critical** | **Plaintext Password Storage** | User passwords are stored in plaintext in the database. | Implement bcrypt hashing before storing passwords. |
| ⚠ **Medium** | **SQL Injection (SQLi)** | The registration form is vulnerable to SQL Injection. | Use **prepared statements** and **input validation**. |
| ⚠ **Medium** | **User Role Manipulation** | Users can escalate privileges via request parameters. | Enforce **server-side role validation**. |

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

## **📄 Conclusion**
The penetration test revealed **multiple vulnerabilities**, including **plaintext password storage, SQL Injection, and privilege escalation risks**. Fixing these issues will improve **security and compliance**.

📌 **Next Steps:**
✔ **Fix vulnerabilities using recommended solutions.**  
✔ **Re-scan the system after fixes.**  
✔ **Implement security best practices to prevent future risks.**  

🚀 **This report serves as a security assessment of the Booking System Phase 1.**
