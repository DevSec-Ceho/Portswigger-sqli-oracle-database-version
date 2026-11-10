
<div align="center">

  <h1>PortSwigger Web Security Academy</h1>
  <h2>SQL Injection: Querying the Database Type and Version on Oracle</h2>

  <p>
    <img src="https://img.shields.io/badge/Platform-PortSwigger-orange?style=for-the-badge" alt="Platform">
    <img src="https://img.shields.io/badge/Category-SQL%20Injection-red?style=for-the-badge" alt="Category">
    <img src="https://img.shields.io/badge/Database-Oracle-critical?style=for-the-badge" alt="Database">
    <img src="https://img.shields.io/badge/Status-Solved-brightgreen?style=for-the-badge" alt="Status">
  </p>

  <p>
    <strong>Hands-on Web Application Security | SQL Injection | Burp Suite</strong>
  </p>

</div>

---

## 📌 Lab Overview

This repository contains my practical walkthrough and exploitation methodology for the PortSwigger Web Security Academy lab:

**SQL injection attack, querying the database type and version on Oracle.**

The objective of this lab was to exploit a SQL injection vulnerability in a product filtering functionality and retrieve information about the underlying Oracle database.

By manipulating the application's `category` parameter, I was able to inject a UNION-based SQL query and extract database version information from Oracle's built-in `v$version` view.

The lab was successfully completed, and the application confirmed the solution.

---

## 🎯 Lab Objectives

- Identify the SQL injection vulnerability in the application.
- Locate the vulnerable parameter through request interception.
- Analyze the application's SQL query behavior.
- Determine the database type and version.
- Exploit the vulnerability using UNION-based SQL injection.
- Retrieve Oracle database version information.
- Successfully complete the lab.

---

## 🛠️ Tools and Technologies

| Tool | Purpose |
|---|---|
| Kali Linux | Testing environment |
| Burp Suite Community Edition | HTTP interception and request manipulation |
| Mozilla Firefox | Browser and application interaction |
| PortSwigger Web Security Academy | Vulnerable lab environment |
| SQL | Database query manipulation |

---

## 🔍 Step 1: Identifying the Vulnerable Parameter

After launching the lab, I navigated through the shopping application's product categories.

The application used a URL parameter named `category` to filter products.

An example of the original HTTP request was:

```http
GET /filter?category=Gifts HTTP/2
Host: YOUR-LAB-ID.web-security-academy.net
```

I identified the `category` parameter as the potential SQL injection entry point.

Using Burp Suite Proxy, I intercepted the request and forwarded it to Burp Repeater for further testing.

### Initial Request Analysis

The parameter was controlled by the user and appeared to influence the backend database query.

This provided an opportunity to investigate whether the application was vulnerable to SQL injection.

---

## 🔬 Step 2: Analyzing the Database Information

The lab description provided a hint indicating that the database should return version strings similar to the following:

- Oracle Database 11g Express Edition Release 11.2.0.2.0
- PL/SQL Release 11.2.0.2.0
- CORE 11.2.0.2.0
- TNS for Linux: Version 11.2.0.2.0
- NLSRTL Version 11.2.0.2.0

Since the target database was Oracle, I identified the `v$version` view as the appropriate source for retrieving database version information.

The `banner` column contains descriptive information about the Oracle database installation.

The SQL statement required for retrieving this information was:

```sql
SELECT banner FROM v$version;
```

However, because the application query used a UNION operation, the injected query needed to return the appropriate number of columns.

---

## 💉 Step 3: Exploiting the SQL Injection Vulnerability

I modified the `category` parameter in Burp Suite Repeater to inject a UNION-based SQL statement.

### Final SQL Injection Payload

```sql
' UNION SELECT banner,'test1' FROM v$version--
```

### HTTP Request

```http
GET /filter?category=' UNION SELECT banner,'test1' FROM v$version-- HTTP/2
Host: YOUR-LAB-ID.web-security-academy.net
```

Replace `YOUR-LAB-ID` with the hostname of your own active lab instance.

### Payload Breakdown

| Payload Component | Explanation |
|---|---|
| `'` | Closes the original SQL string. |
| `UNION SELECT` | Combines the original query results with the injected query results. |
| `banner` | Retrieves database version information. |
| `'test1'` | Supplies a second column to match the original query's column count. |
| `FROM v$version` | Specifies Oracle's database version information view. |
| `--` | Comments out the remaining SQL statement. |

### Why This Payload Works

The application incorporates the category parameter into a backend SQL query.

By closing the original string and introducing a UNION SELECT statement, I was able to append a second query to the original database operation.

The injected query accessed Oracle's `v$version` view and retrieved the contents of its `banner` column.

The additional `'test1'` expression ensured that the injected SELECT statement returned two columns, matching the required structure of the original query.

---

## 📸 Exploitation Evidence

### Screenshot 1: Initial SQL Injection Testing

<p align="center">
<img width="1912" height="946" alt="sql01" src="https://github.com/user-attachments/assets/1d3339d5-6a92-4430-ba05-3449c1b636be" />
</p>

<p align="center">
  <em>Figure 1: Initial investigation using Burp Suite Repeater for the database version information to be displayed by the lab.</em>
</p>

### Screenshot 2: Successful UNION-Based SQL Injection

<p align="center">
  <img width="1919" height="916" alt="sql06" src="https://github.com/user-attachments/assets/00a42b04-aa99-4c9f-8ded-cad130c2c99d" />
</p>

<p align="center">
  <em>Figure 2: Successful execution of the UNION SELECT payload and retrieval of Oracle database version information.</em>
</p>

### Screenshot 3: Final Lab Completion

<p align="center">
  <img width="1868" height="748" alt="sql07" src="https://github.com/user-attachments/assets/caadd36b-4815-44cf-aa3c-7a5c0b247f15" />
</p>

<p align="center">
  <em>Figure 3: PortSwigger Web Security Academy confirming successful completion of the lab.</em>
</p>

---

## 🗄️ Step 4: Extracted Database Information

The injected query successfully retrieved version information from the Oracle database.

The response contained the following information:

```text
CORE 11.2.0.2.0 Production

NLSRTL Version 11.2.0.2.0 - Production

Oracle Database 11g Express Edition
Release 11.2.0.2.0 - 64bit Production

PL/SQL Release 11.2.0.2.0 - Production

TNS for Linux: Version 11.2.0.2.0 - Production
```

### Analysis of the Results

The returned information revealed:

- **Database Vendor:** Oracle Corporation
- **Database Product:** Oracle Database 11g Express Edition
- **Database Version:** 11.2.0.2.0
- **Architecture:** 64-bit
- **SQL Engine:** PL/SQL
- **Additional Components:** CORE, NLSRTL and TNS

This confirmed that the UNION-based SQL injection was successfully executed and that database version information could be retrieved through the vulnerable parameter.

---

## ✅ Step 5: Lab Completion

After successfully retrieving the Oracle database version information, the PortSwigger Web Security Academy interface displayed:

**Congratulations, you solved the lab!**

The successful completion message confirmed that the lab requirements had been satisfied.

---

## ⚠️ Security Impact

SQL injection vulnerabilities can have serious security implications.

Depending on the database permissions and application architecture, an attacker may be able to:

- Retrieve sensitive database information.
- Identify database vendors and software versions.
- Enumerate database structures.
- Access information from unauthorized database tables.
- Potentially modify or delete database records.
- Escalate the attack depending on database privileges.

In this lab, the demonstrated impact was the unauthorized retrieval of Oracle database version information.

---

## 🛡️ Recommended Mitigations

To prevent SQL injection vulnerabilities, developers should implement the following security practices:

**1. Use Parameterized Queries**

Use prepared statements and parameterized queries instead of dynamically concatenating user input into SQL statements.

**2. Implement Proper Input Handling**

Validate user input according to the expected data type and application requirements.

**3. Apply Least Privilege**

Database accounts should only have the permissions required for their intended operations.

**4. Avoid Exposing Database Errors**

Prevent detailed database errors and internal implementation information from being exposed to end users.

**5. Perform Regular Security Testing**

Conduct code reviews, security assessments and penetration testing to identify SQL injection vulnerabilities before deployment.

---

## 📚 Key Takeaways

Through this lab, I gained practical experience with:

- Identifying SQL injection vulnerabilities.
- Intercepting and manipulating HTTP requests using Burp Suite.
- Understanding UNION-based SQL injection.
- Matching the number of columns in UNION SELECT queries.
- Querying Oracle's built-in `v$version` view.
- Retrieving database version information.
- Understanding the security implications of database information disclosure.

---

## 🏁 Conclusion

I successfully completed the PortSwigger Web Security Academy lab:

**SQL injection attack, querying the database type and version on Oracle.**

By identifying the vulnerable `category` parameter and exploiting it with a UNION-based SQL injection payload, I was able to retrieve Oracle database version information from the `v$version` view.

This exercise demonstrated how improperly handled user input can allow attackers to manipulate backend SQL queries and disclose internal database information.

The lab was successfully solved and provided valuable hands-on experience in identifying, exploiting and understanding SQL injection vulnerabilities.

---

## ⚖️ Disclaimer

This repository is intended for educational purposes, cybersecurity research and authorized penetration testing practice.

All exploitation activities documented here were performed within the PortSwigger Web Security Academy's intentionally vulnerable lab environment.

Never attempt these techniques against systems without explicit authorization.

---

<div align="center">

  <h3>Learning • Practicing • Securing</h3>

  <p>
    <strong>PortSwigger Web Security Academy | SQL Injection Series</strong>
  </p>

</div>
