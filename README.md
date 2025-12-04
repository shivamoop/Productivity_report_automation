# 🧠 Productivity Report Automation (n8n Workflow)

### **Overview**
The **Productivity Report Automation** workflow automates performance tracking by collecting, processing, and summarizing productivity data submitted through Google Forms.  
It monitors a Google Sheet for new responses, extracts uploaded CSV reports, summarizes team performance (Accepted, Pending, Rejected), and automatically generates a new summary sheet with email notifications.

---

## ⚙️ **Workflow Summary**

| **Feature** | **Description** |
|--------------|-----------------|
| **Platform** | n8n |
| **Trigger** | Google Sheets (on new row added) |
| **Input** | Google Form responses linked to a Google Sheet |
| **Expected Upload** | CSV report via Google Drive link |
| **Output** | Time-stamped productivity summary sheet |
| **Notifications** | Gmail alerts on success or errors |

---

## 🧩 **Node-by-Node Breakdown**

### 1. **Google Sheets Trigger**
- Watches for new responses on:  
  [Performance Tracker Report (Responses)](https://docs.google.com/spreadsheets/d/1pCpPyRRMhlnRPd0i-0tBYxgOkgGYQa7Vok43RMQQ_5w)
- Executes whenever a new form entry is added.

---

### 2. **If Node — File Link Validation**
- Checks if the *“Productivity Report”* field contains a Google Drive link.
- If missing → Sends an email alert:  
  **Subject:** “File missing”  
  **Body:** “Please upload the report file.”

---

### 3. **Sort → Loop Nodes**
- **Sort:** Orders entries by timestamp (latest first).  
- **Loop:** Ensures each row is processed individually, avoiding data overlap when multiple form responses arrive quickly.

---

### 4. **Google Drive — Download File**
- Extracts the file ID from the Drive link.
- Downloads the CSV file for further processing.

---

### 5. **If Node — File Type Validation**
- Checks whether the downloaded file is in `.csv` format.
- If not → Sends an error email:  
  **Subject:** “File format error”  
  **Body:** “Please upload the file in CSV format.”

---

### 6. **Extract From File**
- Reads data from the CSV into structured JSON format for analysis.

---

### 7. **Code Node — Generate Summary**
Processes all rows to calculate:
- Total time per **status** (Accepted, Pending, Rejected)
- Groups results by **Recorder**
- Produces summarized rows such as:
  ```
  Recorder | Accepted | Pending | Rejected | Grand Total
  ```

---

### 8. **Create Sheet**
- Creates a new Google Sheet tab named:
  ```
  Productivity_Report_YYYYMMDD_HHmm
  ```
  inside the master Google Spreadsheet.

---

### 9. **Append Row in Sheet**
- Inserts summarized data into the newly created sheet.

---

### 10. **Code Node — Add Sheet URL**
- Generates a direct link to the created sheet:
  ```
  https://docs.google.com/spreadsheets/d/{spreadsheetId}/edit#gid={sheetId}
  ```

---

### 11. **Gmail Notification — Success**
- Sends confirmation email with sheet name and URL:
  ```
  ✅ New Productivity Report generated!
  Sheet created: <Sheet Name>
  Open it here: <Sheet URL>
  ```

---

## ⚠️ **Error Handling**

| **Error Condition** | **Action Taken** |
|----------------------|------------------|
| Missing file link | Sends “File missing” email |
| Non-CSV upload | Sends “File format error” email |
| Any other error | Redirects to fallback workflow (`MYbra9Ggkp2CoclE`) |

---

## 🧠 **Development & Debugging Notes**

### **Issue 1: Localhost Port Failure**
During initial testing, the default port **5678** for n8n did not respond, preventing workflow execution.  
**Fix:** Changed the localhost port to **5680**, restoring the execution environment.

### **Issue 2: Sheet Creation Without Data**
The workflow was creating a new sheet but not appending any data afterward.  
**Fix:** Split logic into two nodes —  
1️⃣ **Create Sheet**, followed by  
2️⃣ **Append Row**,  
ensuring data writes only after the new sheet is successfully created.

### **Optimization: Sorting and Looping**
Added **Sort** and **Loop** nodes to:
- Maintain processing order by timestamp.  
- Handle one record per cycle, preventing data mix-ups or concurrent overwrites.

---

## 🕒 **Execution Configuration**

| **Setting** | **Value** |
|--------------|-----------|
| **Trigger Frequency** | Every 1 minute |
| **Execution Order** | Sequential (Batch size = 1) |
| **Auto-save Progress** | Enabled |
| **Estimated Time Saved** | ~20 minutes per report |

---

## 🔐 **Connected Services**

| **Service** | **Usage** |
|--------------|-----------|
| **Google Sheets** | Trigger, sheet creation, and data append |
| **Google Drive** | Download uploaded CSV reports |
| **Gmail** | Notifications (success/error) |

---

## 🚀 **How to Use**

1. **Import Workflow**  
   Upload `Productivity_report_automation.json` to your n8n instance.

2. **Configure Credentials**  
   - Add credentials for:
     - Google Sheets OAuth2
     - Google Drive OAuth2
     - Gmail OAuth2

3. **Activate Workflow**  
   Enable it to run automatically on each new form submission.

4. **Verify Output**  
   - Check Gmail for notifications.  
   - Access generated summary sheets in the Google Spreadsheet.

---

## 📊 **Sample Output**

| Recorder | Accepted | Pending | Rejected | Grand Total |
|-----------|-----------|----------|-----------|--------------|
| Alice | 120 | 15 | 10 | 145 |
| Bob | 90 | 20 | 5 | 115 |

---

## 👨‍💻 **Created By**

**Shivam Yadav**  
Automation Developer | Data Annotation Lead  
📧 [shivamyadavsm09@gmail.com](mailto:shivamyadavsm09@gmail.com)
