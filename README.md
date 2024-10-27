# Student Registration System Smart Contract Guide
A beginner-friendly guide to understanding and implementing a Clarity smart contract for managing student records.

## What Does This Contract Do?
This smart contract creates a simple student management system that can:
- Register new students with unique IDs and names
- Update existing student names
- Look up student information
- Control access through an owner system

## Core Components

### 1. Contract Owner System
- Each contract has one owner (initially the person who deploys it)
- Only the owner can register or update student records
- Ownership can be transferred to another address

### 2. Student Records
- Each student has:
  - A unique ID (number between 1 and 1,000,000)
  - A name (up to 50 characters)
- Records are stored permanently on the blockchain

### 3. Error Handling
The contract uses specific error codes to help you understand what went wrong:

| Error Code | Meaning |
|------------|---------|
| 403 | You're not authorized (not the contract owner) |
| 400 | Invalid student ID |
| 401 | Invalid student name |
| 409 | Student ID already exists |
| 404 | Student not found |

## How to Use the Contract

### Administrative Functions

#### Setting a New Contract Owner
```clarity
(set-contract-owner new-owner)
```
- Only the current owner can do this
- Example: `(set-contract-owner 'SP1234ABCDEF...)`

### Student Management Functions

#### 1. Register a New Student
```clarity
(register-student student-id student-name)
```
- Only the contract owner can use this
- Example: `(register-student u123 "Alice Smith")`
- Requirements:
  - ID must be between 1 and 1,000,000
  - Name must not be empty and under 50 characters
  - ID must not already exist

#### 2. Update a Student's Name
```clarity
(update-student-name student-id new-name)
```
- Only the contract owner can use this
- Example: `(update-student-name u123 "Alice Johnson")`
- Requirements:
  - Student must exist
  - New name must be valid (not empty, under 50 characters)

### Lookup Functions
Anyone can use these functions to read data (no owner permission needed)

#### 1. Get a Student's Name
```clarity
(get-student-name student-id)
```
- Example: `(get-student-name u123)`
- Returns the student's name or an error if not found

#### 2. Check if a Student Exists
```clarity
(student-exists student-id)
```
- Example: `(student-exists u123)`
- Returns true/false

#### 3. Get Contract Owner
```clarity
(get-contract-owner)
```
- Returns the address of the current contract owner

## Example Workflow

Here's a typical sequence of operations:

1. Deploy the contract (you become the owner)
```clarity
// Contract is deployed to the blockchain
```

2. Register a new student
```clarity
(register-student u123 "Alice Smith")
// Returns success message if everything is okay
```

3. Look up the student
```clarity
(get-student-name u123)
// Returns "Alice Smith"
```

4. Update the student's name
```clarity
(update-student-name u123 "Alice Johnson")
// Returns success message
```

5. Verify the update
```clarity
(get-student-name u123)
// Returns "Alice Johnson"
```

## Need Help?

- Watch the walkthrough video here: 
- Consult the Clarity documentation book
- Try using AI tools like Claude or ChatGPT for code auditing
- Contact: Christopher@NoCodeClarity.com

## Full Contract Code
```clarity
;; Student Registration System

;; Error codes
(define-constant ERR-NOT-AUTHORIZED (err u403))
(define-constant ERR-INVALID-ID (err u400))
(define-constant ERR-INVALID-NAME (err u401))
(define-constant ERR-STUDENT-EXISTS (err u409))
(define-constant ERR-STUDENT-NOT-FOUND (err u404))

;; Contract configuration
(define-data-var contract-owner principal tx-sender)
(define-constant MAX-STUDENT-ID u1000000)
(define-constant MIN-STUDENT-ID u1)

;; Data storage
(define-map students uint (string-ascii 50))

;; Validation functions
(define-private (is-valid-student-id (id uint))
  (and 
    (>= id MIN-STUDENT-ID)
    (<= id MAX-STUDENT-ID)))

(define-private (is-valid-student-name (name (string-ascii 50)))
  (and 
    (> (len name) u0)
    (< (len name) u50)))

(define-private (is-contract-owner)
  (is-eq tx-sender (var-get contract-owner)))

;; Administrative functions
(define-public (set-contract-owner (new-owner principal))
  (begin
    (asserts! (is-contract-owner) ERR-NOT-AUTHORIZED)
    (var-set contract-owner new-owner)
    (ok "Contract owner updated successfully")))

;; Core functions
(define-public (register-student (student-id uint) (student-name (string-ascii 50)))
  (begin
    (asserts! (is-contract-owner) ERR-NOT-AUTHORIZED)
    (asserts! (is-valid-student-id student-id) ERR-INVALID-ID)
    (asserts! (is-valid-student-name student-name) ERR-INVALID-NAME)
    (asserts! (is-none (map-get? students student-id)) ERR-STUDENT-EXISTS)
    
    (map-set students student-id student-name)
    (print {event: "student-registered", id: student-id, name: student-name})
    (ok "Student registered successfully")))

(define-public (update-student-name (student-id uint) (new-name (string-ascii 50)))
  (begin
    (asserts! (is-contract-owner) ERR-NOT-AUTHORIZED)
    (asserts! (is-valid-student-id student-id) ERR-INVALID-ID)
    (asserts! (is-valid-student-name new-name) ERR-INVALID-NAME)
    (asserts! (is-some (map-get? students student-id)) ERR-STUDENT-NOT-FOUND)
    
    (map-set students student-id new-name)
    (print {event: "student-updated", id: student-id, new-name: new-name})
    (ok "Student name updated successfully")))

;; Read functions
(define-read-only (get-student-name (student-id uint))
  (begin
    (asserts! (is-valid-student-id student-id) ERR-INVALID-ID)
    (match (map-get? students student-id)
      student-name (ok student-name)
      ERR-STUDENT-NOT-FOUND)))

(define-read-only (student-exists (student-id uint))
  (is-some (map-get? students student-id)))

(define-read-only (get-contract-owner)
  (ok (var-get contract-owner)))
```
