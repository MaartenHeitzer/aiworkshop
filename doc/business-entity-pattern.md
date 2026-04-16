# ABL Business Entity Architecture Pattern

## Overview

The Business Entity pattern provides a standardized, maintainable approach to data access in OpenEdge ABL applications. It separates UI logic from database operations through a layered architecture that promotes reusability, testability, and consistency across the application.

## Architecture Layers

### 1. UI Layer (Windows/Forms)
- **Responsibility**: User interaction and presentation
- **Access**: Never directly accesses database tables
- **Communication**: Calls Business Entity methods with datasets

### 2. Business Entity Layer
- **Responsibility**: Data access, business rules, validation
- **Inheritance**: Extends `OpenEdge.BusinessLogic.BusinessEntity`
- **Management**: Instantiated through EntityFactory (singleton pattern)

### 3. Database Layer
- **Responsibility**: Persistent storage
- **Access**: Only through data-sources attached to business entities

## Key Components

### EntityFactory (Singleton Pattern)

**Purpose**: Centralized management of business entity lifecycle

**Pattern**:
```abl
CLASS business.EntityFactory:
    VAR PRIVATE STATIC EntityFactory objInstance.
    VAR PRIVATE CustomerEntity objCustomerEntityInstance.
    
    CONSTRUCTOR PRIVATE EntityFactory():
    END CONSTRUCTOR.
    
    METHOD PUBLIC STATIC EntityFactory GetInstance():
        IF objInstance = ? THEN
            objInstance = NEW EntityFactory().
        RETURN objInstance.
    END METHOD.
    
    METHOD PUBLIC CustomerEntity GetCustomerEntity():
        IF objCustomerEntityInstance = ? THEN
            objCustomerEntityInstance = NEW CustomerEntity().
        RETURN objCustomerEntityInstance.
    END METHOD.
END CLASS.
```

### Dataset Definition (.i Include Files)

**Purpose**: Defines temp-tables and datasets for data transfer

**Pattern**:
```abl
DEFINE TEMP-TABLE ttCustomer BEFORE-TABLE bttCustomer
    FIELD CustNum AS INTEGER INITIAL "0" LABEL "Cust Num"
    FIELD Name AS CHARACTER LABEL "Name"
    INDEX CustNum IS PRIMARY UNIQUE CustNum ASCENDING.

DEFINE DATASET dsCustomer FOR ttCustomer.
```

**Key Points**:
- `BEFORE-TABLE` enables change tracking for updates
- Temp-table fields match database table structure
- Primary index mirrors database primary key

### Business Entity Class

**Purpose**: Encapsulates all data operations for a specific entity

**Pattern**:
```abl
CLASS business.CustomerEntity INHERITS BusinessEntity USE-WIDGET-POOL:
    {business/CustomerDataset.i}
    DEFINE DATA-SOURCE srcCustomer FOR Customer.
    
    CONSTRUCTOR PUBLIC CustomerEntity():
        SUPER(DATASET dsCustomer:HANDLE).
        VAR HANDLE[1] hDataSourceArray = DATA-SOURCE srcCustomer:HANDLE.
        VAR CHARACTER[1] cSkipListArray = [""].
        THIS-OBJECT:ProDataSource = hDataSourceArray.
        THIS-OBJECT:SkipList = cSkipListArray.
    END CONSTRUCTOR.
END CLASS.
```

## Standard CRUD Operations

### Read (Query) Operations

**Pattern**:
```abl
METHOD PUBLIC LOGICAL GetCustomerByNumber(INPUT ipiCustNum AS INTEGER,
                                          OUTPUT DATASET dsCustomer):
    VAR CHARACTER cFilter = "WHERE Customer.CustNum = " + STRING(ipiCustNum).
    THIS-OBJECT:ReadData(cFilter).
    RETURN CAN-FIND(FIRST ttCustomer).
END METHOD.
```

**Key Points**:
- Use `OUTPUT DATASET` parameter (NOT `BY-REFERENCE`)
- Build filter as string with WHERE clause
- Call parent's `ReadData()` method

### Update Operations

**Pattern**:
```abl
METHOD PUBLIC VOID UpdateCustomer(INPUT-OUTPUT DATASET dsCustomer):
    THIS-OBJECT:UpdateData(DATASET dsCustomer BY-REFERENCE).
END METHOD.
```

**Usage from UI**:
```abl
lFound = objCustomerEntity:GetCustomerByNumber(iCustNum, OUTPUT DATASET dsCustomer).
IF lFound THEN DO:
    TEMP-TABLE ttCustomer:TRACKING-CHANGES = TRUE.
    FIND FIRST ttCustomer.
    ttCustomer.Name = "Updated Name".
    objCustomerEntity:UpdateCustomer(INPUT-OUTPUT DATASET dsCustomer BY-REFERENCE).
END.
```

## Validation Pattern

```abl
METHOD PUBLIC LOGICAL ValidateCustomer(INPUT-OUTPUT DATASET dsCustomer,
                                     OUTPUT errorMessage AS CHARACTER):
    VAR LOGICAL isValid = TRUE.
    FIND FIRST ttCustomer NO-ERROR.
    IF AVAILABLE ttCustomer THEN DO:
        IF ttCustomer.Name = "" THEN DO:
            isValid = FALSE.
            errorMessage = "Customer name cannot be empty".
        END.
    END.
    RETURN isValid.
END METHOD.
```

## Common Pitfalls

### Pitfall 1: Using BY-REFERENCE on OUTPUT DATASET for Read Operations

**Problem**: `lFound = objEntity:GetCustomerByNumber(iNum, OUTPUT DATASET dsCustomer BY-REFERENCE).`

**Solution**: `lFound = objEntity:GetCustomerByNumber(iNum, OUTPUT DATASET dsCustomer).`

### Pitfall 2: Forgetting Change Tracking for Updates

**Problem**: Modifying temp-table without enabling TRACKING-CHANGES.

**Solution**: Enable `TEMP-TABLE ttCustomer:TRACKING-CHANGES = TRUE` before modifications.

### Pitfall 3: Direct Database Access from UI

**Problem**: UI directly accessing database tables.

**Solution**: Always use business entity methods for data access.

## Benefits

- **Maintainability**: Centralized data access logic
- **Reusability**: Business entities shared across multiple UI components
- **Testability**: Business logic isolated from UI
- **Consistency**: Standard error handling and validation approach
- **Scalability**: Easy to add new entities and business rules

## References

- OpenEdge Documentation: Business Entity Class
- ABL Reference: Dataset and Temp-Table Definitions
- Project Examples: `src/business/CustomerEntity.cls`, `src/business/EntityFactory.cls`, `src/CustomerWin.w`