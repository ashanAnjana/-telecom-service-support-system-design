# Simplified Telecom SIM Activation System

## Overview

A streamlined design for **SIM card activation workflow** focusing on the essential components. The system handles the core process: customer purchases SIM → provides identity documents → system validates → SIM is activated.

## Core Workflow
1. **Customer Registration** - Customer provides personal information
2. **Document Submission** - Customer uploads proof-of-identity documents  
3. **SIM & Number Assignment** - System maps SIM card to mobile number
4. **Document Verification** - System validates documents against regulatory requirements
5. **Activation** - SIM card is activated and ready for use

## Database Design

### Key Entities

#### **Customers**
- Essential customer information (name, email, phone, DOB)
- Links to documents and activation requests

#### **SIM Cards** 
- Physical SIM inventory with ICCID, PUK/PIN codes
- Simple status tracking (available, assigned, active)

#### **Mobile Numbers**
- Available number pool
- Basic assignment status (available, assigned)

#### **Document Management**
- **DocumentTypes**: Accepted identity document types
- **CustomerDocuments**: Uploaded customer documents with verification status

#### **Activation Process**
- **SimActivations**: Central workflow entity linking customer, SIM, and number
- Simple status progression (pending → verified → activated)

### Design Principles
- **Clarity**: Clear entity relationships and responsibilities
- **Maintainability**: Minimal complexity while meeting requirements

## Object-Oriented Design

### Core Domain Classes

#### **Entity Classes**
- **Customer**: Customer data and eligibility validation
- **SimCard**: SIM inventory and status management  
- **MobileNumber**: Number pool and assignment
- **DocumentType**: Identity document definitions
- **CustomerDocument**: Document storage and verification
- **SimActivation**: Central activation workflow coordinator

#### **Service Layer**
- **ActivationService**: Main business logic for SIM activation process

### Design Patterns Applied

#### **Service Layer Pattern**  
- Encapsulates business logic in `ActivationService`
- Coordinates between entities for activation workflow
- Provides clear API for activation operations

## Key Design Assumptions

### **Business Rules**
1. **One SIM per activation** - Each activation links one customer, one SIM, one number
2. **Document verification required** - Identity documents must be verified before activation
3. **Simple status progression** - Clear workflow states (pending → verified → activated)

### **Technical Assumptions**
1. **MySQL database** - Relational data with referential integrity
2. **PHP object-oriented design** - Modern OOP principles
3. **File upload capability** - Document storage for verification

## System Components

### **Database Schema** (`schema.dbml`)
- **6 core tables**: customers, sim_cards, mobile_numbers, document_types, customer_documents, sim_activations
- **Audit tracking**: Basic fields in sim_activations (processed_by, process_notes, last_status_change)
- **Regulatory compliance**: Document validation fields (verified_by, regulatory_check_passed, compliance_notes)
- **Essential relationships**: Foreign keys linking core workflow entities

### **Class Diagram** (`class_diagram.puml`)
- **6 domain classes**: Core entities with audit and compliance capabilities
- **1 service class**: `ActivationService` with regulatory and audit methods
- **4 enums**: Type-safe status management
- **Enhanced methods**: Audit tracking and regulatory compliance support

---

## File Structure
```
/
├── schema.dbml                    # Database schema design
├── class_diagram.puml             # Object-oriented class diagram
├── README.md                      # Explanation of design & rationale
└── diagrams/                      # Optional: rendered images of diagrams
```
