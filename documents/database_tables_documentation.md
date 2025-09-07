# Database Tables Documentation

## Telecom SIM Activation System - Database Schema Analysis

This document provides a detailed explanation of each table's function in the telecom SIM activation system database schema based on the DBML specification.

---

## **Core Entity Tables**

### **1. `customers`**
**Function**: Stores essential customer information for SIM activation

#### **Schema Definition**:
```sql
CREATE TABLE customers (
    id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(20) NOT NULL,
    date_of_birth DATE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Indexes
    UNIQUE INDEX idx_customers_email (email),
    INDEX idx_customers_phone (phone)
);
```

#### **Field Specifications**:
- `id` - Primary key, auto-increment integer
- `first_name` - VARCHAR(100), NOT NULL - Customer's first name
- `last_name` - VARCHAR(100), NOT NULL - Customer's last name  
- `email` - VARCHAR(255), UNIQUE, NOT NULL - Unique contact identifier
- `phone` - VARCHAR(20), NOT NULL - Customer's phone number
- `date_of_birth` - DATE, NOT NULL - Age verification for eligibility
- `created_at` - TIMESTAMP, DEFAULT now() - Account creation timestamp

#### **Business Role**: 
- Foundation for all SIM activation requests
- Validates customer eligibility through age verification
- Ensures unique customer identification via email

#### **Relationships**: 
- One-to-many with `customer_documents` (customers can submit multiple documents)
- One-to-many with `sim_activations` (customers can request multiple activations)

#### **Sample Insert**:
```sql
INSERT INTO customers (first_name, last_name, email, phone, date_of_birth)
VALUES ('John', 'Doe', 'john.doe@example.com', '1234567890', '1990-01-01');
``` 

### **2. `sim_cards`**
**Function**: SIM card inventory and lifecycle management

#### **Schema Definition**:
```sql
CREATE TABLE sim_cards (
    id INT PRIMARY KEY AUTO_INCREMENT,
    iccid VARCHAR(20) UNIQUE NOT NULL,
    puk_code VARCHAR(8) NOT NULL,
    pin_code VARCHAR(4) NOT NULL,
    status ENUM('available', 'assigned', 'active') DEFAULT 'available',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Indexes
    UNIQUE INDEX idx_sim_cards_iccid (iccid),
    INDEX idx_sim_cards_status (status)
);
```

#### **Field Specifications**:
- `id` - Primary key, auto-increment integer
- `iccid` - VARCHAR(20), UNIQUE, NOT NULL - International Circuit Card Identifier
- `puk_code` - VARCHAR(8), NOT NULL - Personal Unblocking Key
- `pin_code` - VARCHAR(4), NOT NULL - Personal Identification Number
- `status` - ENUM('available', 'assigned', 'active'), DEFAULT 'available' - SIM lifecycle state
- `created_at` - TIMESTAMP, DEFAULT now() - SIM creation timestamp

#### **Business Role**:
- Manages SIM card inventory and availability
- Tracks SIM lifecycle: available → assigned → active
- Stores security codes (PUK/PIN) for SIM activation

#### **Relationships**:
- One-to-many with `sim_activations` (SIMs can be used in multiple activation attempts)

#### **Status Progression**:
- `available` - SIM ready for assignment
- `assigned` - SIM reserved for customer activation
- `active` - SIM successfully activated and in use

### **3. `mobile_numbers`**
**Function**: Mobile number pool management and assignment

#### **Schema Definition**:
```sql
CREATE TABLE mobile_numbers (
    id INT PRIMARY KEY AUTO_INCREMENT,
    number VARCHAR(15) UNIQUE NOT NULL,
    status ENUM('available', 'assigned') DEFAULT 'available',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Indexes
    UNIQUE INDEX idx_mobile_numbers_number (number),
    INDEX idx_mobile_numbers_status (status)
);
```

#### **Field Specifications**:
- `id` - Primary key, auto-increment integer
- `number` - VARCHAR(15), UNIQUE, NOT NULL - The actual mobile phone number
- `status` - ENUM('available', 'assigned'), DEFAULT 'available' - Number availability status
- `created_at` - TIMESTAMP, DEFAULT now() - Number creation timestamp

#### **Business Role**:
- Manages mobile number inventory pool
- Ensures unique number assignment to customers
- Tracks number availability for assignment

#### **Relationships**:
- One-to-many with `sim_activations` (numbers can be reassigned if activation fails)

#### **Status Management**:
- `available` - Number ready for assignment
- `assigned` - Number reserved for customer activation

## **Document Management Tables**

### **4. `document_types`**
**Function**: Defines accepted identity document categories

#### **Schema Definition**:
```sql
CREATE TABLE document_types (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    code VARCHAR(20) UNIQUE NOT NULL,
    is_required BOOLEAN DEFAULT TRUE,
    
    -- Indexes
    UNIQUE INDEX idx_document_types_code (code)
);
```

#### **Field Specifications**:
- `id` - Primary key, auto-increment integer
- `name` - VARCHAR(100), NOT NULL - Human-readable document type name
- `code` - VARCHAR(20), UNIQUE, NOT NULL - Unique document type identifier
- `is_required` - BOOLEAN, DEFAULT TRUE - Whether document is mandatory for activation

#### **Business Role**:
- Configures accepted identity document categories
- Standardizes document requirements for activation
- Enables flexible document validation rules

#### **Relationships**:
- One-to-many with `customer_documents` (document types can have multiple customer instances)

#### **Standard Document Types**:
```sql
INSERT INTO document_types (name, code, is_required) VALUES
('Passport', 'PASS', TRUE),
('National Identity Card', 'NIC', TRUE),
('Driver''s License', 'DL', TRUE),
('Signature', 'SIG', TRUE);
```

### **5. `customer_documents`**
**Function**: Stores uploaded customer identity documents with regulatory compliance

#### **Schema Definition**:
```sql
CREATE TABLE customer_documents (
    id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT NOT NULL,
    document_type_id INT NOT NULL,
    document_number VARCHAR(100) NOT NULL,
    file_path VARCHAR(500),
    verification_status ENUM('pending', 'verified', 'rejected') DEFAULT 'pending',
    verified_at TIMESTAMP NULL,
    verified_by VARCHAR(100),
    rejection_reason TEXT,
    regulatory_check_passed BOOLEAN DEFAULT FALSE,
    compliance_notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Foreign Keys
    FOREIGN KEY (customer_id) REFERENCES customers(id),
    FOREIGN KEY (document_type_id) REFERENCES document_types(id),
    
    -- Indexes
    INDEX idx_customer_documents_customer_id (customer_id),
    INDEX idx_customer_documents_verification_status (verification_status),
    INDEX idx_customer_documents_regulatory_check (regulatory_check_passed),
    UNIQUE INDEX idx_customer_documents_unique (customer_id, document_type_id)
);
```

#### **Field Specifications**:
- `id` - Primary key, auto-increment integer
- `customer_id` - INT, NOT NULL, FK to customers.id
- `document_type_id` - INT, NOT NULL, FK to document_types.id
- `document_number` - VARCHAR(100), NOT NULL - Official document number
- `file_path` - VARCHAR(500) - File storage location for uploaded document
- `verification_status` - ENUM('pending', 'verified', 'rejected'), DEFAULT 'pending'
- `verified_at` - TIMESTAMP NULL - When document was verified
- `verified_by` - VARCHAR(100) - Staff member who processed verification
- `rejection_reason` - TEXT - Reason for rejection if applicable
- `regulatory_check_passed` - BOOLEAN, DEFAULT FALSE - Compliance validation flag
- `compliance_notes` - TEXT - Regulatory validation notes
- `created_at` - TIMESTAMP, DEFAULT now() - Document submission timestamp

#### **Business Role**:
- Central hub for document verification with regulatory compliance
- Links customers to their submitted identity documents
- Tracks verification workflow and compliance status

#### **Relationships**:
- Many-to-one with `customers` (customers can submit multiple documents)
- Many-to-one with `document_types` (documents belong to specific types)
- One-to-many with `document_validations` (documents can have multiple validation checks)

#### **Unique Constraint**:
- Each customer can have only one document per document type: `(customer_id, document_type_id)`
## **Process Management Tables**

### **6. `sim_activations`** ⭐ **CENTRAL TABLE**
**Function**: Central workflow orchestrator for SIM activation process with audit tracking

#### **Schema Definition**:
```sql
CREATE TABLE sim_activations (
    id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT NOT NULL,
    sim_card_id INT NOT NULL UNIQUE,
    mobile_number_id INT NOT NULL UNIQUE,
    status ENUM('pending', 'verified', 'activated', 'rejected') DEFAULT 'pending',
    request_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    activation_date TIMESTAMP NULL,
    rejection_reason TEXT,
    processed_by VARCHAR(100),
    process_notes TEXT,
    last_status_change TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Foreign Keys (One-to-One relationships)
    FOREIGN KEY (customer_id) REFERENCES customers(id),
    FOREIGN KEY (sim_card_id) REFERENCES sim_cards(id),
    FOREIGN KEY (mobile_number_id) REFERENCES mobile_numbers(id),
    
    -- Indexes
    INDEX idx_sim_activations_customer_id (customer_id),
    UNIQUE INDEX idx_sim_activations_sim_card_id (sim_card_id),
    UNIQUE INDEX idx_sim_activations_mobile_number_id (mobile_number_id),
    INDEX idx_sim_activations_status (status),
    INDEX idx_sim_activations_last_status_change (last_status_change)
);
```

#### **Field Specifications**:
- `id` - Primary key, auto-increment integer
- `customer_id` - INT, NOT NULL, FK to customers.id
- `sim_card_id` - INT, NOT NULL, UNIQUE, FK to sim_cards.id (one-to-one)
- `mobile_number_id` - INT, NOT NULL, UNIQUE, FK to mobile_numbers.id (one-to-one)
- `status` - ENUM('pending', 'verified', 'activated', 'rejected'), DEFAULT 'pending'
- `request_date` - TIMESTAMP, DEFAULT now() - When activation was requested
- `activation_date` - TIMESTAMP NULL - When activation was completed
- `rejection_reason` - TEXT - Reason for rejection if applicable
- `processed_by` - VARCHAR(100) - Staff member handling the request
- `process_notes` - TEXT - Processing notes for audit trail
- `last_status_change` - TIMESTAMP, DEFAULT now() - Last status update timestamp
- `created_at` - TIMESTAMP, DEFAULT now() - Activation request creation

#### **Business Role**:
- **MOST CRITICAL TABLE** - coordinates entire activation workflow
- Links customer, SIM card, and mobile number in one-to-one relationships
- Manages activation status progression with audit tracking
- Provides complete audit trail for regulatory compliance

#### **Relationships**:
- Many-to-one with `customers` (customers can have multiple activations)
- One-to-one with `sim_cards` (each activation uses exactly one SIM)
- One-to-one with `mobile_numbers` (each activation assigns exactly one number)
- One-to-many with `activation_audit_log` (detailed audit trail)

#### **Status Workflow**:
- `pending` → `verified` → `activated` (success path)
- `pending` → `rejected` (failure path)
- `verified` → `rejected` (late failure)

### **7. `users`**
**Function**: System operator authentication and authorization

#### **Schema Definition**:
```sql
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(100) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    role ENUM('admin', 'operator', 'supervisor') NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    last_login TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

#### **Field Specifications**:
- `id` - Primary key, auto-increment integer
- `username` - VARCHAR(100), UNIQUE, NOT NULL - Login username
- `email` - VARCHAR(255), UNIQUE, NOT NULL - User email address
- `password_hash` - VARCHAR(255), NOT NULL - Hashed password for security
- `first_name` - VARCHAR(100), NOT NULL - Staff member's first name
- `last_name` - VARCHAR(100), NOT NULL - Staff member's last name
- `role` - ENUM('admin', 'operator', 'supervisor'), NOT NULL - Permission level
- `is_active` - BOOLEAN, DEFAULT TRUE - Account status
- `last_login` - TIMESTAMP NULL - Last login activity tracking
- `created_at` - TIMESTAMP, DEFAULT now() - Account creation
- `updated_at` - TIMESTAMP, DEFAULT now() ON UPDATE now() - Last modification

#### **Business Role**:
- Role-based access control for activation processing
- Authentication and authorization for system operations
- Audit trail attribution for all system actions

#### **Role Hierarchy**:
- `admin` - Full system access, user management
- `supervisor` - Activation oversight, reporting access
- `operator` - Document verification, activation processing

#### **Relationships**:
- Referenced by `activation_audit_log` for audit attribution
- Referenced by `document_validations` for validation tracking
---

## **Compliance & Audit Tables**

### **8. `activation_audit_log`**
**Function**: Complete audit trail for all activation process changes

#### **Schema Definition**:
```sql
CREATE TABLE activation_audit_log (
    id INT PRIMARY KEY AUTO_INCREMENT,
    activation_id INT,
    user_id INT,
    action VARCHAR(100) NOT NULL,
    old_status VARCHAR(50),
    new_status VARCHAR(50),
    comments TEXT,
    ip_address VARCHAR(45),
    user_agent TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Foreign Keys
    FOREIGN KEY (activation_id) REFERENCES sim_activations(id),
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

#### **Field Specifications**:
- `id` - Primary key, auto-increment integer
- `activation_id` - INT, FK to sim_activations.id - Links to specific activation
- `user_id` - INT, FK to users.id - Staff member who made the change
- `action` - VARCHAR(100), NOT NULL - Description of action performed
- `old_status` - VARCHAR(50) - Previous status before change
- `new_status` - VARCHAR(50) - New status after change
- `comments` - TEXT - Additional notes about the action
- `ip_address` - VARCHAR(45) - IP address for security tracking
- `user_agent` - TEXT - Browser/client information
- `created_at` - TIMESTAMP, DEFAULT now() - When action occurred

#### **Business Role**:
- Ensures full traceability for regulatory compliance
- Provides detailed audit trail for all activation changes
- Supports compliance reporting and investigations

#### **Audit Events Tracked**:
- Status changes (pending → verified → activated)
- Document verification actions
- Rejection decisions with reasons
- Administrative overrides

### **9. `regulatory_rules`**
**Function**: Configurable validation rules for document verification

#### **Schema Definition**:
```sql
CREATE TABLE regulatory_rules (
    id INT PRIMARY KEY AUTO_INCREMENT,
    rule_name VARCHAR(200) NOT NULL,
    rule_code VARCHAR(50) UNIQUE NOT NULL,
    description TEXT,
    validation_logic TEXT,
    is_active BOOLEAN DEFAULT TRUE,
    country_code VARCHAR(5),
    effective_date DATE NOT NULL,
    expiry_date DATE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

#### **Field Specifications**:
- `id` - Primary key, auto-increment integer
- `rule_name` - VARCHAR(200), NOT NULL - Descriptive rule name
- `rule_code` - VARCHAR(50), UNIQUE, NOT NULL - Unique rule identifier
- `description` - TEXT - Human-readable rule explanation
- `validation_logic` - TEXT - Technical implementation details
- `is_active` - BOOLEAN, DEFAULT TRUE - Whether rule is currently active
- `country_code` - VARCHAR(5) - Geographic applicability (optional)
- `effective_date` - DATE, NOT NULL - When rule becomes effective
- `expiry_date` - DATE - When rule expires (optional)
- `created_at` - TIMESTAMP, DEFAULT now() - Rule creation
- `updated_at` - TIMESTAMP, DEFAULT now() ON UPDATE now() - Last modification

#### **Business Role**:
- Implements country-specific regulatory requirements
- Enables flexible compliance with different regional regulations
- Supports configurable document validation logic

#### **Relationships**:
- One-to-many with `document_validations` (rules can validate multiple documents)

#### **Example Rules**:
```sql
INSERT INTO regulatory_rules (rule_name, rule_code, description, validation_logic, country_code, effective_date) VALUES
('Age Verification', 'AGE_CHECK', 'Verify customer is 18+ years old', 'date_of_birth validation', 'LK', '2023-01-01'),
('Document Authenticity', 'DOC_AUTH', 'Verify document authenticity', 'document format validation', 'LK', '2023-01-01');
```

### **10. `document_validations`**
**Function**: Individual validation results for each document against regulatory rules

#### **Schema Definition**:
```sql
CREATE TABLE document_validations (
    id INT PRIMARY KEY AUTO_INCREMENT,
    customer_document_id INT,
    regulatory_rule_id INT,
    validation_status ENUM('pending', 'passed', 'failed') DEFAULT 'pending',
    validation_result TEXT,
    validated_at TIMESTAMP NULL,
    validated_by INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Foreign Keys
    FOREIGN KEY (customer_document_id) REFERENCES customer_documents(id),
    FOREIGN KEY (regulatory_rule_id) REFERENCES regulatory_rules(id),
    FOREIGN KEY (validated_by) REFERENCES users(id)
);
```

#### **Field Specifications**:
- `id` - Primary key, auto-increment integer
- `customer_document_id` - INT, FK to customer_documents.id - Document being validated
- `regulatory_rule_id` - INT, FK to regulatory_rules.id - Rule being applied
- `validation_status` - ENUM('pending', 'passed', 'failed'), DEFAULT 'pending'
- `validation_result` - TEXT - Detailed validation outcome and notes
- `validated_at` - TIMESTAMP NULL - When validation was performed
- `validated_by` - INT, FK to users.id - Staff member who performed validation
- `created_at` - TIMESTAMP, DEFAULT now() - Validation record creation

#### **Business Role**:
- Provides granular validation tracking for regulatory compliance
- Links specific documents to applicable regulatory rules
- Tracks detailed validation outcomes for audit purposes

#### **Relationships**:
- Many-to-one with `customer_documents` (documents can have multiple validations)
- Many-to-one with `regulatory_rules` (rules can validate multiple documents)
- Many-to-one with `users` (users can perform multiple validations)

#### **Validation Workflow**:
- `pending` - Validation queued but not yet performed
- `passed` - Document meets regulatory requirements
- `failed` - Document does not meet regulatory requirements
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
