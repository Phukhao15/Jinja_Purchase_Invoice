# DATE 26/11/2025
- เพิ่ม Signature ให้กับ Purchase Invoice และไม่ได้แก้ไข Logic ในส่วนอื่นๆ  

---

# DATE 27/11/2025

## 🐛 Bug Fix: SQL Syntax Error ใน Print Format

### ปัญหาที่พบ
- **Print Format:** `ITG - Purchase Invoice with Landed Cost`
- **Error:** `pymysql.err.ProgrammingError: (1064, "You have an error in your SQL syntax...")`
- **สาเหตุ:** คำอธิบายสินค้า (description) มีเครื่องหมาย apostrophe (') ในข้อความ เช่น "can't" ทำให้ SQL query error เมื่อใช้วิธีต่อ string โดยตรง

### การแก้ไข
เปลี่ยนจาก **String Concatenation** เป็น **Parameterized Query** เพื่อป้องกัน:
- SQL Syntax Error จาก special characters
- SQL Injection vulnerabilities

#### Before (โค้ดเดิม)
```python
{% set xsql = "Select lci.item_code, lci.qty, lci.receipt_document, lci.description, lci.applicable_charges  
    from `tabLanded Cost Item` as lci 
    where lci.parent = " + "'" + doc.refer_to_landed_cost_voucher + "'" %}  
{% set xitems = frappe.db.sql(xsql, as_dict=1) %}

{% set w_sql = "Select  pri.base_amount from `tabPurchase Receipt Item` pri 
        Where   pri.parent          = '" + item.receipt_document + "'"
            + " and pri.item_code   = '" + item.item_code   + "'" 
            + " and pri.description = '" +  item.description + "'" %}  
{% set PR_items = frappe.db.sql(w_sql, as_dict=1) %}
```

#### After (โค้ดที่แก้ไขแล้ว)
```python
{% set xsql = "Select lci.item_code, lci.qty, lci.receipt_document, lci.description, lci.applicable_charges  
    from `tabLanded Cost Item` as lci 
    where lci.parent = %s" %}  
{% set xitems = frappe.db.sql(xsql, doc.refer_to_landed_cost_voucher, as_dict=1) %}

{% set w_sql = "Select pri.base_amount from `tabPurchase Receipt Item` pri 
        Where pri.parent = %s 
          and pri.item_code = %s 
          and pri.description = %s" %}  
{% set PR_items = frappe.db.sql(w_sql, (item.receipt_document, item.item_code, item.description), as_dict=1) %}
```

### ผลลัพธ์
✅ แก้ไข SQL Syntax Error เมื่อ description มี apostrophe  
✅ เพิ่มความปลอดภัยจาก SQL Injection  
✅ รองรับ special characters อื่นๆ อัตโนมัติ  
✅ โค้ดอ่านง่ายและ maintain ได้ดีขึ้น

### ไฟล์ที่แก้ไข
- Print Format: `ITG - Purchase Invoice with Landed Cost`

---