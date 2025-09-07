# Class Functions Documentation

## Telecom SIM Activation System - Class Responsibilities

This document provides detailed explanations of each class's functions and responsibilities based on the PlantUML class diagram design.

---

## **Core Domain Classes**

### **1. Customer Class**
**Primary Function**: Customer data management and eligibility validation

#### **Attributes**:
- `int id` - Unique customer identifier (primary key)
- `string firstName` - Customer's first name
- `string lastName` - Customer's last name
- `string email` - Unique contact email address
- `string phone` - Customer's phone number
- `DateTime dateOfBirth` - Date of birth for age verification
- `DateTime createdAt` - Account creation timestamp

#### **Responsibilities**:
- Store and manage customer personal information
- Validate customer eligibility for SIM activation
- Provide customer identification methods

#### **Key Methods**:
- `__construct(firstName, lastName, email, dateOfBirth)` - Initialize customer with essential data
- `getId() : int` - Retrieve unique customer identifier
- `getFullName() : string` - Concatenate first and last name for display
- `getEmail() : string` - Get customer contact email
- `isEligibleForActivation() : bool` - Business rule validation for activation eligibility (age verification, email uniqueness)

#### **Business Logic**:
- Age verification based on date of birth
- Email uniqueness enforcement
- Foundation entity for all activation requests

---

### **2. SimCard Class**
**Primary Function**: SIM card inventory and lifecycle management

#### **Attributes**:
- `int id` - Unique SIM card identifier (primary key)
- `string iccid` - International Circuit Card Identifier (20 digits)
- `string pukCode` - Personal Unblocking Key (8 digits)
- `string pinCode` - Personal Identification Number (4 digits)
- `SimStatus status` - Current SIM status (enum: AVAILABLE, ASSIGNED, ACTIVE)
- `DateTime createdAt` - SIM creation timestamp

#### **Responsibilities**:
- Track individual SIM card details and security codes
- Manage SIM card status throughout lifecycle
- Provide availability checking for assignment

#### **Key Methods**:
- `__construct(iccid, pukCode, pinCode)` - Initialize SIM with unique identifiers
- `getId() : int` - Retrieve SIM card database ID
- `getIccid() : string` - Get International Circuit Card Identifier
- `isAvailable() : bool` - Check if SIM can be assigned to customer
- `activate() : void` - Change status to active when activation completes
- `getStatus() : SimStatus` - Retrieve current SIM status

#### **Business Logic**:
- ICCID uniqueness enforcement
- Status progression: AVAILABLE → ASSIGNED → ACTIVE
- Security code management (PUK/PIN)

---

### **3. MobileNumber Class**
**Primary Function**: Mobile number pool management and assignment

#### **Attributes**:
- `int id` - Unique number identifier (primary key)
- `string number` - The actual mobile phone number (15 digits max)
- `NumberStatus status` - Current number status (enum: AVAILABLE, ASSIGNED)
- `DateTime createdAt` - Number creation timestamp

#### **Responsibilities**:
- Manage available mobile number inventory
- Handle number assignment to customers
- Track number availability status

#### **Key Methods**:
- `__construct(number)` - Initialize with phone number
- `getId() : int` - Retrieve number database ID
- `getNumber() : string` - Get the actual phone number
- `isAvailable() : bool` - Check if number can be assigned to a customer
- `assign() : void` - Mark number as assigned to customer

#### **Business Logic**:
- Phone number uniqueness enforcement
- Simple status management: AVAILABLE → ASSIGNED
- Number pool inventory control

---

### **4. DocumentType Class**
**Primary Function**: Identity document type definitions and requirements

#### **Attributes**:
- `int id` - Unique document type identifier (primary key)
- `string name` - Human-readable document type name (e.g., "Passport", "National ID")
- `string code` - Unique document type code (e.g., "PASS", "NIC", "DL", "SIG")
- `bool isRequired` - Whether this document type is mandatory for activation

#### **Responsibilities**:
- Define accepted identity document categories
- Specify document requirements for activation
- Standardize document validation criteria

#### **Key Methods**:
- `__construct(name, code, isRequired)` - Initialize document type
- `getId() : int` - Retrieve document type ID
- `getName() : string` - Get human-readable document name
- `getCode() : string` - Get unique document type code
- `isRequired() : bool` - Check if document is mandatory for activation

#### **Business Logic**:
- Document type standardization (passport, national identity card, driver's license, signature)
- Requirement specification for different document categories
- Code uniqueness for system integration

---

### **5. CustomerDocument Class**
**Primary Function**: Document storage and regulatory compliance validation

#### **Attributes**:
- `int id` - Unique document identifier (primary key)
- `int customerId` - Foreign key linking to customer
- `int documentTypeId` - Foreign key linking to document type
- `string documentNumber` - Official document number (passport, NIC, etc.)
- `string filePath` - File storage location for uploaded document
- `VerificationStatus verificationStatus` - Current verification state (enum: PENDING, VERIFIED, REJECTED)
- `DateTime verifiedAt` - Timestamp when document was verified
- `string verifiedBy` - Staff member who verified the document
- `string rejectionReason` - Reason for rejection if applicable
- `bool regulatoryCheckPassed` - Compliance validation flag
- `string complianceNotes` - Regulatory validation notes
- `DateTime createdAt` - Document submission timestamp

#### **Responsibilities**:
- Store customer-submitted identity documents
- Manage document verification workflow
- Track regulatory compliance validation
- Handle document approval/rejection process

#### **Key Methods**:
- `__construct(customerId, documentTypeId, documentNumber)` - Link document to customer
- `getId() : int` - Retrieve document database ID
- `getDocumentNumber() : string` - Get official document number
- `verify(string verifiedBy) : void` - Mark document as verified with auditor info
- `reject(string reason, string verifiedBy) : void` - Reject document with reason and auditor
- `getVerificationStatus() : VerificationStatus` - Get current verification state
- `performRegulatoryCheck() : bool` - Execute compliance validation
- `isCompliant() : bool` - Check if document meets regulatory requirements

#### **Business Logic**:
- Document verification workflow management
- Regulatory compliance tracking with boolean flag
- Audit trail for verification decisions
- File path management for document storage

---

### **6. SimActivation Class**
**Primary Function**: Central workflow orchestrator with audit tracking

#### **Attributes**:
- `int id` - Unique activation identifier (primary key)
- `int customerId` - Foreign key linking to customer
- `int simCardId` - Foreign key linking to SIM card
- `int mobileNumberId` - Foreign key linking to mobile number
- `ActivationStatus status` - Current activation status (enum: PENDING, VERIFIED, ACTIVATED, REJECTED)
- `DateTime requestDate` - When activation was requested
- `DateTime activationDate` - When activation was completed
- `string rejectionReason` - Reason for rejection if applicable
- `string processedBy` - Staff member handling the request
- `string processNotes` - Processing notes for audit trail
- `DateTime lastStatusChange` - Timestamp of last status update
- `DateTime createdAt` - Activation request creation timestamp

#### **Responsibilities**:
- Coordinate entire SIM activation process
- Link customer, SIM card, and mobile number
- Manage activation status progression
- Provide audit trail for process tracking
- Handle activation approval/rejection workflow

#### **Key Methods**:
- `__construct(customerId, simCardId, mobileNumberId)` - Initialize activation request
- `getId() : int` - Retrieve activation request ID
- `processActivation(string processedBy) : void` - Execute activation workflow with auditor
- `activate(string processedBy) : void` - Complete activation with audit tracking
- `reject(string reason, string processedBy) : void` - Reject activation with reason and auditor
- `getStatus() : ActivationStatus` - Get current activation status
- `validateDocuments() : bool` - Check if all required documents are verified
- `addAuditNote(string note, string processedBy) : void` - Add process notes for audit trail
- `getAuditTrail() : string` - Retrieve complete audit history

#### **Business Logic**:
- Central coordination of activation workflow
- Status progression: PENDING → VERIFIED → ACTIVATED or REJECTED
- Audit trail maintenance with timestamps and responsible parties
- Document validation orchestration
- Business rule enforcement for activation completion

---

## **Service Layer**

### **7. ActivationService Class**
**Primary Function**: Business logic coordination and process management

#### **Responsibilities**:
- Orchestrate complete SIM activation workflow
- Coordinate between domain entities
- Provide high-level business operations
- Manage regulatory compliance checks
- Generate audit reports

#### **Key Methods**:
- `initiateActivation(customerId, simCardId, mobileNumberId) : SimActivation` - Start new activation process
- `processActivation(activationId, string processedBy) : bool` - Execute activation workflow with audit
- `validateDocuments(customerId) : bool` - Verify all customer documents are compliant
- `performRegulatoryCompliance(customerId) : bool` - Execute regulatory validation checks
- `activateSim(activationId, string processedBy) : bool` - Complete SIM activation with audit
- `getActivationAuditTrail(activationId) : array` - Generate audit report for activation

#### **Business Logic**:
- High-level workflow coordination
- Cross-entity business rule enforcement
- Regulatory compliance orchestration
- Audit trail generation and reporting
- Transaction boundary management for activation process

---

## **Design Patterns Applied**

### **Entity Pattern**
- Each core class represents a distinct business entity
- Clear separation of data and behavior
- Encapsulation of entity-specific business rules

### **Service Layer Pattern**
- `ActivationService` encapsulates complex business workflows
- Coordinates multiple entities for complete business operations
- Provides transaction boundaries and cross-cutting concerns

### **Value Object Pattern**
- Enums provide type-safe status management
- Immutable status values prevent invalid state transitions

---

## **Class Interaction Flow**

### **Typical Activation Workflow**:
1. **ActivationService.initiateActivation()** creates new **SimActivation**
2. **Customer.isEligibleForActivation()** validates customer eligibility
3. **SimCard.isAvailable()** and **MobileNumber.isAvailable()** check resource availability
4. **CustomerDocument.performRegulatoryCheck()** validates compliance
5. **SimActivation.validateDocuments()** ensures all documents are verified
6. **ActivationService.activateSim()** completes the process
7. **SimActivation.addAuditNote()** records final audit information

This design ensures clear separation of concerns while maintaining the essential SIM activation workflow with proper audit tracking and regulatory compliance.

---

## **Detailed Workflow Explanation**

### **Complete SIM Activation Process Flow**

#### **Phase 1: Customer Registration & Document Submission**

1. **Customer Registration**
   - `Customer` object is created with `__construct(firstName, lastName, email, dateOfBirth)`
   - System validates `email` uniqueness in database
   - `isEligibleForActivation()` checks age requirements (must be 18+ typically)
   - Customer data stored in `customers` table

2. **Document Submission**
   - For each required `DocumentType` (Passport, NIC, Driver's License, Signature):
   - `CustomerDocument` created with `__construct(customerId, documentTypeId, documentNumber)`
   - Document file uploaded and `filePath` stored
   - Initial `verificationStatus` set to `PENDING`
   - Record stored in `customer_documents` table

#### **Phase 2: Resource Assignment**

3. **SIM Card Assignment**
   - System queries available `SimCard` objects where `status = AVAILABLE`
   - `SimCard.isAvailable()` validates SIM can be assigned
   - Selected SIM status changed from `AVAILABLE` → `ASSIGNED`
   - SIM's `iccid`, `pukCode`, `pinCode` reserved for customer

4. **Mobile Number Assignment**
   - System queries available `MobileNumber` objects where `status = AVAILABLE`
   - `MobileNumber.isAvailable()` validates number can be assigned
   - `MobileNumber.assign()` changes status from `AVAILABLE` → `ASSIGNED`
   - Number reserved for customer activation

#### **Phase 3: Activation Request Creation**

5. **Activation Initialization**
   - `ActivationService.initiateActivation(customerId, simCardId, mobileNumberId)` called
   - Creates new `SimActivation` object linking all three entities e.g. customer, SIM card, and mobile number
   - Initial `status` set to `PENDING`
   - `requestDate` timestamp recorded
   - Record stored in `sim_activations` table

#### **Phase 4: Document Verification & Regulatory Compliance**

6. **Document Verification Process**
   - For each `CustomerDocument`:
   - Staff member reviews uploaded document
   - `CustomerDocument.performRegulatoryCheck()` validates against requirements e.g. age verification, document authenticity, etc.
   - `regulatoryCheckPassed` flag set to `true/false`
   - `complianceNotes` added explaining validation results
   - If valid: `verify(verifiedBy)` sets status to `VERIFIED`
   - If invalid: `reject(reason, verifiedBy)` sets status to `REJECTED`

7. **Overall Document Validation**
   - `ActivationService.validateDocuments(customerId)` checks all documents
   - Ensures all required document types are submitted and verified
   - `SimActivation.validateDocuments()` returns `true` only if all documents pass
   - `ActivationService.performRegulatoryCompliance(customerId)` performs final compliance check

#### **Phase 5: Activation Processing**

8. **Activation Decision**
   - If all documents verified: `SimActivation` status changes `PENDING` → `VERIFIED`
   - `processActivation(processedBy)` method called with staff member identifier
   - `processNotes` and `lastStatusChange` updated for audit trail
   - `addAuditNote(note, processedBy)` records processing decision

9. **Final Activation**
   - `ActivationService.activateSim(activationId, processedBy)` called
   - `SimActivation.activate(processedBy)` changes status to `ACTIVATED`
   - `activationDate` timestamp recorded
   - `SimCard.activate()` changes SIM status to `ACTIVE`
   - Customer receives activated SIM with assigned mobile number

#### **Phase 6: Audit Trail & Completion**

10. **Audit Documentation**
    - `getAuditTrail()` provides complete process history
    - `getActivationAuditTrail(activationId)` generates compliance report
    - All status changes, timestamps, and responsible parties tracked
    - Regulatory compliance flags maintained for reporting

### **Alternative Flow: Rejection Process**

#### **Document Rejection**
- If document fails verification:
  - `CustomerDocument.reject(reason, verifiedBy)` called
  - `verificationStatus` set to `REJECTED`
  - `rejectionReason` stored explaining failure
  - Customer notified to resubmit correct documents

#### **Activation Rejection**
- If overall process fails:
  - `SimActivation.reject(reason, processedBy)` called
  - Status changed to `REJECTED`
  - `rejectionReason` stored
  - Resources (SIM, number) released back to available pool
  - `SimCard` status reverts to `AVAILABLE`
  - `MobileNumber` status reverts to `AVAILABLE`

### **Key Workflow Characteristics**

#### **Status Progression Tracking**
- **Customer Documents**: `PENDING` → `VERIFIED` or `REJECTED`
- **SIM Cards**: `AVAILABLE` → `ASSIGNED` → `ACTIVE`
- **Mobile Numbers**: `AVAILABLE` → `ASSIGNED`
- **Activations**: `PENDING` → `VERIFIED` → `ACTIVATED` or `REJECTED`

#### **Audit Trail Features**
- Every status change recorded with timestamp
- Responsible party tracked for all decisions
- Process notes maintained for business context
- Complete audit trail available for compliance reporting

#### **Regulatory Compliance Integration**
- Document validation against regulatory requirements
- Compliance flags and notes maintained
- Rejection reasons tracked for regulatory reporting
- Audit trail supports compliance investigations

This workflow ensures complete traceability, regulatory compliance, and proper resource management while maintaining simplicity in the core activation process.
