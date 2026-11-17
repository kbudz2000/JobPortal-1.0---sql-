# Job Portal — SQL Injection in Job Creation

**Vulnerability class:** SQL Injection — CWE-89  
**Affected endpoint:** `/jobportal/Employer/InsertJob.php`  
**Parameter:** `txtTitle` (POST)  
**Entry page:** `/jobportal/Employer/ManageJob.php`  
**Affected release:** Not established. No CVE identifier has been assigned.

## Description

A SQL injection vulnerability in the Job Portal application's job creation endpoint allows an attacker using an employer session to inject SQL expressions through the `txtTitle` POST parameter. The application concatenates user-controlled input into an SQL statement without parameterization. The supplied test output demonstrates unauthorized extraction of database contents, including administrator credentials and job seeker personal information.

## Simple explanation

The job title should be treated only as text. Instead, the application mixes it with database instructions, allowing specially crafted input to change how the database query behaves.

## Short proof of concept

1. Sign in with a test employer account in an authorized, disposable installation.
2. Open **Manage Job** and capture the job creation POST request using Burp Suite.
3. Confirm that the request targets `InsertJob.php` and contains the `txtTitle` field. Save it locally as `request.txt`; keep its session cookie private.
4. To reproduce detection specifically for this field, run:

```sh
sqlmap -r request.txt -p txtTitle --technique=E --batch
```

This command requests error-based detection without a database dump. The endpoint creates jobs, so even detection can insert test records. Use a disposable database and restore its snapshot afterward.

## Evidence from the supplied test

The reporter's sqlmap output identifies `txtTitle` as injectable using:

- Error-based SQL injection through `EXTRACTVALUE`.
- Boolean-based blind SQL injection through `RLIKE`.
- Time-based blind SQL injection through `SLEEP`.

The supplied output also shows database enumeration and extraction of records from multiple tables. Credentials, personal data, session cookies, and raw database dumps are omitted from this public example. These results were supplied by the reporter; no additional live test was performed when preparing this README.

## Impact

Unauthorized disclosure of database records, including credentials and personal information. Exposure of plaintext administrator passwords can enable account takeover. This evidence does not establish remote code execution or operating-system compromise. The supplied request used a session cookie; anonymous exploitation was not demonstrated by that test.

## Remediation

Use prepared statements with bound parameters for every field. Validate numeric fields on the server, require the appropriate employer role, restrict database account permissions, and store passwords using a password hashing function. Hiding SQL errors alone does not fix SQL injection.
