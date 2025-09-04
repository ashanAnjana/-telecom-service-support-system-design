# Telecom Service Support System Design

## Overview

This project is a system for managing SIM card activation workflow. It ensures regulatory compliance and tracks all operations with audit logs.

**Core Process Flow:**
```
Customer Purchase → Document Submission → Validation → SIM Assignment → Activation
```

## Business Requirements & Rules

### **Core Business Rules**
1. **One-to-One Mapping** - Each activation creates a unique relationship between one customer, one SIM card, and one mobile number
2. **Mandatory Verification** - All identity documents must pass regulatory validation before activation
3. **Status Progression** - Clear, auditable workflow states: `pending → verified → activated → (rejected)`
4. **Audit Trail** - All state changes must be logged with user attribution and timestamps


## System Workflow

### **Detailed Process Flow**

1. **Customer Registration**
   - Capture essential customer information (name, email, phone, DOB)
   - Validate data format and business rules e.g. phone number format, email format, date of birth format
   - Create customer record with unique identifier

2. **Document Submission**
   - Customer uploads required identity documents
   - System stores document metadata and file references
   - Initial document format validation

3. **Resource Assignment**
   - System selects available SIM card from inventory
   - Assigns mobile number from available pool
   - Creates activation record linking all components

4. **Document Verification**
   - Validate documents against regulatory rules
   - Perform compliance checks
   - Update verification status with audit trail

5. **SIM Activation**
   - Final activation of SIM card
   - Update all related entity statuses
   - Generate activation confirmation

### **Error Handling**
- Failed verifications result in `rejected` status with detailed reason codes
- All rejections are logged for compliance reporting
- System supports resubmission workflow for corrected documents

## Database Architecture

### **Core Entity Design**

#### **Customer Management**
- **`customers`**: Core customer data with unique email constraint
- **`customer_documents`**: Document storage with verification workflow
- **`document_types`**: Configurable document type definitions

#### **Inventory Management**
- **`sim_cards`**: Physical SIM inventory with security codes (PUK/PIN)
- **`mobile_numbers`**: Number pool management with assignment tracking

#### **Workflow Management**
- **`sim_activations`**: Central orchestration entity linking all components
- **`activation_audit_log`**: Complete audit trail for all workflow changes

#### **Compliance & Governance**
- **`users`**: System operators with role-based access control
- **`regulatory_rules`**: Configurable compliance validation rules
- **`document_validations`**: Individual validation results against regulatory rules

### **Key Relationships**

```
Customer (1) ←→ (N) CustomerDocuments
Customer (1) ←→ (N) SimActivations
SimCard (1) ←→ (1) SimActivation
MobileNumber (1) ←→ (1) SimActivation
DocumentType (1) ←→ (N) CustomerDocuments
```

## Object-Oriented Architecture

### **Architectural Patterns**

The system implements a **layered architecture** with clear separation of concerns:

```
┌─────────────────────────────────────┐
│           Service Layer             │  ← Business Logic
├─────────────────────────────────────┤
│           Domain Layer              │  ← Core Entities
├─────────────────────────────────────┤
│         Data Access Layer           │  ← Database Operations
└─────────────────────────────────────┘
```

### **Domain Model Design**

#### **Core Entities**

**`Customer`**
- Encapsulates customer data and business rules
- Methods: `isEligibleForActivation()`, `getFullName()`
- Validates customer eligibility before activation

**`SimCard`**
- Manages SIM inventory and lifecycle
- Methods: `isAvailable()`, `activate()`, `getStatus()`
- Handles status transitions: `available → assigned → active`

**`MobileNumber`**
- Controls number pool and assignment logic
- Methods: `isAvailable()`, `assign()`
- Ensures unique number assignment

**`CustomerDocument`**
- Handles document storage and verification workflow
- Methods: `verify()`, `reject()`, `performRegulatoryCheck()`, `isCompliant()`
- Implements regulatory compliance validation

**`SimActivation`**
- Central coordinator for the entire activation process
- Methods: `processActivation()`, `activate()`, `reject()`, `validateDocuments()`, `addAuditNote()`
- Maintains workflow state and audit trail

#### **Service Layer**

**`ActivationService`**
- **Purpose**: Orchestrates the complete activation workflow
- **Responsibilities**:
  - Coordinate between domain entities
  - Enforce business rules and validation
  - Manage transaction boundaries
  - Handle regulatory compliance
- **Key Methods**:
  - `initiateActivation()`: Start new activation process
  - `processActivation()`: Execute workflow steps
  - `validateDocuments()`: Perform document verification
  - `performRegulatoryCompliance()`: Execute compliance checks
  - `getActivationAuditTrail()`: Retrieve audit history

### **Design Patterns Applied**

#### **Service Layer Pattern**
- **Purpose**: Encapsulates business logic and coordinates domain operations
- **Benefits**: Clear API, transaction management, business rule enforcement
- **Implementation**: `ActivationService` as the primary business facade

#### **Aggregate Pattern**
- **Purpose**: Maintain consistency boundaries and business invariants
- **Implementation**: `SimActivation` as aggregate root managing related entities

#### **Status Object Pattern**
- **Purpose**: Type-safe status management with clear transitions
- **Implementation**: Enums for `SimStatus`, `ActivationStatus`, `VerificationStatus`, `NumberStatus`

## Implementation Details

### **Database Schema** (`schema.dbml`)

**Core Tables (9 total):**
- **Primary Entities**: `customers`, `sim_cards`, `mobile_numbers`, `document_types`
- **Workflow Management**: `sim_activations`, `customer_documents`
- **Governance**: `users`, `activation_audit_log`, `regulatory_rules`, `document_validations`

**Key Features:**
- **Audit Tracking**: Comprehensive logging with user attribution and timestamps
- **Regulatory Compliance**: Built-in validation fields and compliance tracking
- **Performance Optimization**: Strategic indexing for high-volume operations
- **Data Integrity**: Foreign key constraints and unique constraints
- **Scalability**: Designed for high-throughput activation processing

### **Class Diagram** (`class_diagram.puml`)

**Architecture Components:**
- **6 Domain Classes**: Complete entity model with business logic
- **1 Service Class**: `ActivationService` with comprehensive workflow management
- **4 Status Enums**: Type-safe state management across all entities
- **Rich Method Set**: Full CRUD operations plus business-specific methods

**Design Features:**
- **Audit Methods**: Built into `SimActivation` for complete traceability
- **Compliance Methods**: Integrated into `CustomerDocument` for regulatory validation
- **Status Management**: Clear state transitions with validation

## Viewing the Design

### **Diagrams Available**
- **ER Diagram**: `schema.png` - Visual database relationships
- **ER Diagram**: `schema.dbml` - Database schema
- **Class Diagram**: `class_diagram.png` - Object-oriented structure
- **PlantUML Source**: `class_diagram.puml` - Editable class diagram

### **How to Use**
1. **Database Implementation**: Use `schema.dbml` with [dbdiagram.io](https://dbdiagram.io) or dbml-cli
2. **class diagram**: Use `class_diagram.puml` with [plantuml](https://plantuml.com)
3. **Code Generation**: Use class diagram as blueprint for implementation
4. **Documentation**: This README provides complete design rationale

---

## Project Structure
```
Assignment/
├──  README.md                   # Complete design documentation
├──  schema.dbml                 # Database schema (DBML format)
├──  class_diagram.puml          # Object-oriented design (PlantUML)
├──  schema.png                  # Database relationships (visual)
├──  class_diagram.png           # Class structure (visual)
```
---
