# Database Tables Documentation

## Telecom Service Support System - Table Functions

This document provides a detailed explanation of each table's function in the telecom SIM activation system database schema.

---

## **Core Entity Tables**

### **1. `customers`**
**Function**: Stores customer personal information and contact details

- **Primary Purpose**: Customer registration and identity management
- **Key Fields**: 
  - `first_name`, `last_name` - Customer identification
  - `email` - Unique contact identifier
  - `phone` - Contact number
  - `date_of_birth` - Age verification for eligibility
  - `address` - Customer location information
- **Business Role**: Foundation for all SIM activation requests; validates customer eligibility
- **Relationships**: Links to `customer_documents` and `sim_activations`
- **Constraints**: `email` must be unique
- **create table query**: 
```sql
CREATE TABLE customers (
    id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(20),
    date_of_birth DATE NOT NULL,
    address TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```
- **create query**: 
```sql
INSERT INTO customers (first_name, last_name, email, phone, date_of_birth, address)
VALUES ('John', 'Doe', 'john.doe@example.com', '1234567890', '1990-01-01', '123 Main St');
``` 

### **2. `sim_cards`** 
**Function**: Physical SIM card inventory management

- **Primary Purpose**: Track individual SIM cards from manufacturing to activation
- **Key Fields**:
  - `iccid` - Unique SIM card identifier (20 digits)
  - `imsi` - International Mobile Subscriber Identity
  - `puk_code`, `pin_code` - Security codes
  - `status` - Lifecycle state (inactive, active, suspended, terminated)
  - `batch_number` - Manufacturing batch tracking
- **Business Role**: Manages SIM lifecycle and inventory control
- **Relationships**: Links to `sim_activations`
- **create table query**: 
```sql
CREATE TABLE sim_cards (
    id INT PRIMARY KEY AUTO_INCREMENT,
    iccid VARCHAR(20) UNIQUE NOT NULL,
    imsi VARCHAR(15),
    puk_code VARCHAR(8) NOT NULL,
    pin_code VARCHAR(4) NOT NULL,
    status ENUM('inactive', 'active', 'suspended', 'terminated') DEFAULT 'inactive',
    batch_number VARCHAR(50),
    manufactured_date DATE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### **3. `mobile_numbers`**
**Function**: Available phone number pool management

- **Primary Purpose**: Assign and track mobile numbers for customers
- **Key Fields**:
  - `number` - The actual phone number
  - `country_code`, `area_code` - Geographic identifiers
  - `status` - Availability (available, assigned, reserved, blocked)
  - `number_type` - Service type (prepaid, postpaid)
- **Business Role**: Ensures unique number assignment and supports different service plans
- **Relationships**: Links to `sim_activations`
- **create table query**: 
```sql
CREATE TABLE mobile_numbers (
    id INT PRIMARY KEY AUTO_INCREMENT,
    number VARCHAR(15) UNIQUE NOT NULL,
    country_code VARCHAR(5) NOT NULL,
    area_code VARCHAR(10),
    status ENUM('available', 'assigned', 'reserved', 'blocked') DEFAULT 'available',
    number_type ENUM('prepaid', 'postpaid') NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

## **Document Management Tables**

### **4. `document_types`**
**Function**: Defines accepted identity document categories

- **Primary Purpose**: Configure what documents are required for activation
- **Key Fields**:
  - `name` - Document type name (e.g., "Passport", "NIC")
  - `code` - Unique identifier code
  - `is_mandatory` - Whether document is required
  - `regulatory_requirement` - Legal compliance notes
- **Business Role**: Standardizes document requirements across different regions/regulations
- **Relationships**: Links to `customer_documents`
- **create table query**: 
```sql
CREATE TABLE document_types (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    code VARCHAR(20) UNIQUE NOT NULL,
    is_mandatory BOOLEAN DEFAULT FALSE,
    regulatory_requirement TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```
- **insert query**: 
```sql
INSERT INTO document_types (name, code, is_mandatory, regulatory_requirement)
VALUES ('Passport', 'PASS', TRUE, 'Required for international travel'),
('National Identity Card', 'NIC', TRUE, 'Required for local travel'),
('Driver''s License', 'DL', TRUE, 'Required for local travel');
```

### **5. `customer_documents`**
**Function**: Stores uploaded customer identity documents

- **Primary Purpose**: Link customers to their submitted verification documents
- **Key Fields**:
  - `customer_id` - Links to customer
  - `document_type_id` - Type of document
  - `document_number` - Official document number
  - `document_file_path` - File storage location
  - `issue_date`, `expiry_date` - Document validity period
  - `verification_status` - Processing state (pending, verified, rejected, expired)
  - `verified_by` - Staff member who processed
- **Business Role**: Central hub for document verification workflow
- **Relationships**: Links `customers` to `document_types`, connects to `document_validations`

---
- **create table query**: 
```sql
CREATE TABLE customer_documents (
    id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT NOT NULL,
    document_type_id INT NOT NULL,
    document_number VARCHAR(100) NOT NULL,
    document_file_path VARCHAR(500),
    issue_date DATE,
    expiry_date DATE,
    issuing_authority VARCHAR(200),
    verification_status ENUM('pending', 'verified', 'rejected', 'expired') DEFAULT 'pending',
    verified_at TIMESTAMP NULL,
    verified_by INT,
    rejection_reason TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (customer_id) REFERENCES customers(id),
    FOREIGN KEY (document_type_id) REFERENCES document_types(id)
);
```
- **insert query**: 
```sql
INSERT INTO customer_documents (customer_id, document_type_id, document_number, document_file_path, issue_date, expiry_date, issuing_authority, verification_status, verified_at, verified_by, rejection_reason)
VALUES (1, 1, '123456789', 'path/to/document.pdf', '2023-01-01', '2024-01-01', 'Government of Sri Lanka', 'verified', '2023-01-01', 1, 'Document is valid');
```
## **Process Management Tables**

### **6. `sim_activations`** (main table)
**Function**: **Central workflow orchestrator** for SIM activation process

- **Primary Purpose**: Manages complete activation lifecycle from request to completion
- **Key Fields**:
  - `customer_id`, `sim_card_id`, `mobile_number_id` - Core entity links
  - `activation_status` - Workflow state (pending, document_verification, approved, activated, rejected)
  - `regulatory_check_status` - Compliance verification state
  - `request_date`, `activation_date` - Timeline tracking
  - `processed_by` - Staff member handling request
  - `rejection_reason` - Failure explanation
- **Business Role**: **Most critical table** - coordinates entire activation workflow
- **Relationships**: Central hub connecting customers, SIM cards, mobile numbers, and users
- **create table query**: 
```sql
CREATE TABLE sim_activations (
    id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT NOT NULL,
    sim_card_id INT NOT NULL,
    mobile_number_id INT NOT NULL,
    activation_status ENUM('pending', 'document_verification', 'approved', 'activated', 'rejected') DEFAULT 'pending',
    request_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    activation_date TIMESTAMP NULL,
    processed_by INT,
    rejection_reason TEXT,
    regulatory_check_status ENUM('pending', 'passed', 'failed') DEFAULT 'pending',
    regulatory_check_date TIMESTAMP NULL,
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (customer_id) REFERENCES customers(id),
    FOREIGN KEY (sim_card_id) REFERENCES sim_cards(id),
    FOREIGN KEY (mobile_number_id) REFERENCES mobile_numbers(id),
    FOREIGN KEY (processed_by) REFERENCES users(id)
);
```
- **insert query**: 
```sql
INSERT INTO sim_activations (customer_id, sim_card_id, mobile_number_id, activation_status, request_date, processed_by, rejection_reason)
VALUES (1, 1, 1, 'pending', '2023-01-01', 1, 'Document verification failed');
```

### **7. `users`**
**Function**: System operator authentication and authorization

- **Primary Purpose**: Manage staff who process activation requests
- **Key Fields**:
  - `username`, `email` - Login credentials
  - `password_hash` - Secure authentication
  - `first_name`, `last_name` - Staff identification
  - `role` - Permission level (admin, operator, supervisor)
  - `is_active` - Account status
  - `last_login` - Activity tracking
- **Business Role**: Role-based access control for activation processing
- **Relationships**: Links to `sim_activations`, `customer_documents`, `activation_audit_log`
- **create table query**: 
```sql
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    role ENUM('admin', 'operator', 'supervisor') NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    last_login TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (role) REFERENCES roles(id)
);
```
- **insert query**: 
```sql
INSERT INTO users (username, email, password_hash, first_name, last_name, role, is_active)
VALUES ('admin', 'admin@example.com', 'hashed_password', 'Admin', 'User', 'admin', TRUE);
```
---

## **Compliance & Audit Tables**

### **8. `activation_audit_log`**
**Function**: Complete audit trail for all activation process changes

- **Primary Purpose**: Regulatory compliance and process tracking
- **Key Fields**:
  - `activation_id` - Links to specific activation request
  - `user_id` - Staff member who made change
  - `action` - What was done
  - `old_status`, `new_status` - State transition tracking
  - `comments` - Additional notes
  - `ip_address`, `user_agent` - Security tracking
- **Business Role**: Ensures full traceability for compliance reporting
- **Relationships**: Links to `sim_activations` and `users`

### **9. `regulatory_rules`**
**Function**: Configurable validation rules for document verification

- **Primary Purpose**: Implement country-specific regulatory requirements
- **Key Fields**:
  - `rule_name`, `rule_code` - Rule identification
  - `description` - Human-readable explanation
  - `validation_logic` - Technical implementation details
  - `country_code` - Geographic applicability
  - `effective_date`, `expiry_date` - Rule validity period
  - `is_active` - Current status
- **Business Role**: Enables flexible compliance with different regional regulations
- **Relationships**: Links to `document_validations`
- **create table query**: 
```sql
CREATE TABLE regulatory_rules (
    id INT PRIMARY KEY AUTO_INCREMENT,
    rule_name VARCHAR(100) NOT NULL,
    rule_code VARCHAR(20) UNIQUE NOT NULL,
    description TEXT,
    validation_logic TEXT,
    country_code VARCHAR(5) NOT NULL,
    effective_date DATE,
    expiry_date DATE,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (country_code) REFERENCES countries(code)
);
```
- **insert query**: 
```sql
INSERT INTO regulatory_rules (rule_name, rule_code, description, validation_logic, country_code, effective_date, expiry_date, is_active)
VALUES ('web activation', 'RULE1', 'Description of web activation', 'validation logic for web activation', 'LK', '2023-01-01', '2024-01-01', TRUE),
('api activation', 'RULE2', 'Description of api activation', 'validation logic for api activation', 'LK', '2023-01-01', '2024-01-01', TRUE);
```

### **10. `document_validations`**
**Function**: Individual validation results for each document against regulatory rules

- **Primary Purpose**: Track detailed validation outcomes
- **Key Fields**:
  - `customer_document_id` - Document being validated
  - `regulatory_rule_id` - Rule being applied
  - `validation_status` - Result (pending, passed, failed)
  - `validation_result` - Detailed outcome
  - `validated_at`, `validated_by` - Processing details
- **Business Role**: Provides granular validation tracking for compliance
- **Relationships**: Links `customer_documents` to `regulatory_rules`
- **create table query**: 
```sql
CREATE TABLE document_validations (
    id INT PRIMARY KEY AUTO_INCREMENT,
    customer_document_id INT NOT NULL,
    regulatory_rule_id INT NOT NULL,
    validation_status ENUM('pending', 'passed', 'failed') DEFAULT 'pending',
    validation_result TEXT,
    validated_at TIMESTAMP NULL,
    validated_by INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (customer_document_id) REFERENCES customer_documents(id),
    FOREIGN KEY (regulatory_rule_id) REFERENCES regulatory_rules(id)
);
```
---

## **Performance Optimization**

### **11. `indexes`**
**Function**: Database performance optimization

- **Primary Purpose**: Speed up frequently queried operations
- **Key Areas**:
  - Customer lookups (`customers.email`)
  - SIM status checks (`sim_cards.iccid`, `sim_cards.status`)
  - Activation tracking (`sim_activations.activation_status`)
  - Document verification (`customer_documents.verification_status`)
- **Business Role**: Ensures system responsiveness under high load

---

## **Table Relationships Summary**

### **Central Data Flow**
```
customers → customer_documents → document_validations → sim_activations → activation_audit_log
```

### **Key Connection Points**

1. **`sim_activations`** - Central hub connecting:
   - Customers (who is requesting)
   - SIM cards (what is being activated)
   - Mobile numbers (what number is assigned)
   - Users (who is processing)

2. **`document_validations`** - Compliance bridge connecting:
   - Customer documents (what is being checked)
   - Regulatory rules (what standards to apply)

3. **`activation_audit_log`** - Tracking system connecting:
   - Activation requests (what changed)
   - Users (who made the change)

### **Design Benefits**

- **Regulatory Compliance**: Complete audit trails and configurable validation rules
- **Scalability**: Normalized structure with strategic indexing
- **Flexibility**: Configurable document types and regulatory rules
- **Security**: Role-based access control and comprehensive logging
- **Maintainability**: Clear separation of concerns across functional domains

---

## **Usage Patterns**

### **Typical Query Flows**

1. **New Activation Request**:
   - Insert into `customers` → `customer_documents` → `sim_activations`

2. **Document Verification**:
   - Query `customer_documents` → Apply `regulatory_rules` → Insert `document_validations`

3. **Activation Processing**:
   - Update `sim_activations` → Update `sim_cards` status → Insert `activation_audit_log`

4. **Compliance Reporting**:
   - Query `activation_audit_log` → Join with `sim_activations` and `users`

This table structure ensures comprehensive tracking, regulatory compliance, and operational efficiency for telecom SIM activation workflows.
