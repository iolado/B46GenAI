# KYC and AML Compliance Testing

This assignment documents high-level functional requirements, synthetic test data generation, test scenarios, a requirements traceability matrix (RTM), and a sample test report for Know Your Customer (KYC) and Anti-Money Laundering (AML) validation.

## 1. Functional Requirements

### KYC Requirements

- Capture and validate customer identity data during onboarding.
- Collect supporting documents such as passport, driver's license, national ID, and proof of address.
- Validate mandatory fields before submission.
- Verify customer identity against trusted external or internal sources.
- Classify customers by risk level based on jurisdiction, occupation, customer type, and transaction profile.
- Screen customers against sanctions, watchlists, PEP lists, and adverse media sources.
- Trigger enhanced due diligence for high-risk customers.
- Support periodic KYC review and re-verification.
- Collect source of funds and source of wealth when required.

### AML Requirements

- Monitor transactions for suspicious, unusual, or high-risk activity.
- Detect high-value transactions and structuring behavior.
- Flag cross-border transfers involving high-risk jurisdictions.
- Screen customers and counterparties against sanctions lists.
- Route suspicious activity to compliance teams for manual review.
- Maintain an audit trail of checks, alerts, and decisions.
- Enforce role-based access to sensitive compliance data.
- Support compliance reporting and case management workflows.
- Integrate with third-party verification and screening services.

## 2. Compliance Test Use Cases

- Validate complete and incomplete KYC onboarding flows.
- Verify document upload, format validation, and expired document rejection.
- Confirm sanctions, PEP, and adverse media screening.
- Validate low-risk, medium-risk, and high-risk customer handling.
- Verify periodic KYC review reminders and re-KYC processing.
- Detect high-value transactions and unusual transaction patterns.
- Validate structuring, sanctions-linked transfers, and cross-border risk alerts.
- Confirm escalation, manual review, and case closure workflows.
- Verify audit logging, role-based access, and integration behavior.

## 3. Synthetic Data Generators

### 3.1 KYC Synthetic Data Generator

```js
// synthetic-kyc-data.js
// Usage: node synthetic-kyc-data.js 10

const crypto = require("crypto");

const firstNames = ["John", "Jane", "Alex", "Sarah", "Michael", "Emma"];
const lastNames = ["Smith", "Johnson", "Brown", "Taylor", "Anderson", "Thomas"];
const countries = ["USA", "UK", "Canada", "Germany", "India", "UAE", "Singapore"];
const occupations = ["Engineer", "Teacher", "Consultant", "Trader", "Doctor", "Analyst"];
const riskLevels = ["Low", "Medium", "High"];
const documentTypes = ["Passport", "DriverLicense", "NationalID"];

function pick(arr) {
  return arr[Math.floor(Math.random() * arr.length)];
}

function randInt(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

function randomDate(startYear = 1960, endYear = 2003) {
  const year = randInt(startYear, endYear);
  const month = String(randInt(1, 12)).padStart(2, "0");
  const day = String(randInt(1, 28)).padStart(2, "0");
  return `${year}-${month}-${day}`;
}

function randomId(prefix) {
  return `${prefix}-${crypto.randomBytes(4).toString("hex")}`;
}

function generateCustomer(index) {
  const firstName = pick(firstNames);
  const lastName = pick(lastNames);
  const country = pick(countries);

  return {
    customerId: `CUST-${String(index + 1).padStart(5, "0")}`,
    fullName: `${firstName} ${lastName}`,
    dateOfBirth: randomDate(),
    nationality: country,
    occupation: pick(occupations),
    riskLevel: pick(riskLevels),
    pepFlag: Math.random() < 0.1,
    sanctionsFlag: Math.random() < 0.05,
    document: {
      type: pick(documentTypes),
      documentNumber: randomId("DOC"),
      expiryDate: `${randInt(2026, 2035)}-12-31`
    },
    kycStatus: pick(["Pending", "Verified", "Rejected", "ReviewRequired"])
  };
}

const count = Number(process.argv[2] || 5);
const dataset = Array.from({ length: count }, (_, i) => generateCustomer(i));

console.log(JSON.stringify(dataset, null, 2));
```

### 3.2 AML Synthetic Data Generator

```js
// synthetic-aml-data.js
// Usage: node synthetic-aml-data.js 20

const crypto = require("crypto");

const firstNames = ["John", "Jane", "Ava", "Liam", "Noah", "Emma"];
const lastNames = ["Smith", "Johnson", "Brown", "Taylor", "Martin", "Clark"];
const countries = ["US", "UK", "CA", "DE", "AE", "SG", "NG", "IN"];
const highRiskCountries = ["NG", "AE"];
const txnTypes = ["wire_transfer", "cash_deposit", "cash_withdrawal", "crypto_purchase", "card_transfer"];
const currencies = ["USD", "EUR", "GBP", "AED", "SGD"];

function pick(arr) {
  return arr[Math.floor(Math.random() * arr.length)];
}

function randInt(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

function randAmount(min, max) {
  return Number((Math.random() * (max - min) + min).toFixed(2));
}

function id(prefix) {
  return `${prefix}_${crypto.randomBytes(4).toString("hex")}`;
}

function name() {
  return `${pick(firstNames)} ${pick(lastNames)}`;
}

function buildRecord(index) {
  const originCountry = pick(countries);
  const destinationCountry = pick(countries);
  const amount = randAmount(100, 50000);
  const isCrossBorder = originCountry !== destinationCountry;
  const isHighRiskJurisdiction = highRiskCountries.includes(destinationCountry);
  const isStructuring = amount < 10000 && Math.random() < 0.2;
  const isSanctionsHit = Math.random() < 0.05;
  const isPepLinked = Math.random() < 0.1;

  const flagged =
    amount > 10000 ||
    isCrossBorder ||
    isHighRiskJurisdiction ||
    isStructuring ||
    isSanctionsHit ||
    isPepLinked;

  return {
    transactionId: `TXN-${String(index + 1).padStart(6, "0")}`,
    customerId: `CUST-${String(randInt(1, 999)).padStart(5, "0")}`,
    customerName: name(),
    transactionType: pick(txnTypes),
    amount,
    currency: pick(currencies),
    originCountry,
    destinationCountry,
    isCrossBorder,
    isHighRiskJurisdiction,
    isStructuring,
    isSanctionsHit,
    isPepLinked,
    counterpartyId: id("CP"),
    timestamp: new Date().toISOString(),
    alertStatus: flagged ? pick(["FLAGGED", "UNDER_REVIEW", "ESCALATED"]) : "CLEAR"
  };
}

const count = Number(process.argv[2] || 10);
const dataset = Array.from({ length: count }, (_, i) => buildRecord(i));

console.log(JSON.stringify(dataset, null, 2));
```

## 4. Validation Test Cases and Scenarios

### KYC Scenarios

- Valid onboarding with complete identity and address proof.
- Rejection of expired or invalid identity documents.
- Blocking submission when mandatory fields are missing.
- Screening and flagging sanctions or PEP matches.
- Triggering enhanced due diligence for high-risk customers.
- Triggering periodic review for customers due for re-KYC.

### AML Scenarios

- Alert generation for high-value transactions.
- Detection of repeated low-value transfers indicating structuring.
- Flagging cross-border transfers to high-risk jurisdictions.
- Detection of sanctions-linked counterparties.
- Escalation of suspicious activity for manual compliance review.
- Audit logging of investigation and closure actions.

## 5. Combined Node.js Validation Script

```js
// kyc-aml-validation-tests.js
// Run: node kyc-aml-validation-tests.js

const assert = require("assert");

const kycData = [
  {
    customerId: "CUST-001",
    fullName: "John Smith",
    nationality: "USA",
    riskLevel: "Low",
    pepFlag: false,
    sanctionsFlag: false,
    document: { type: "Passport", documentNumber: "P123456", expiryDate: "2030-12-31" }
  },
  {
    customerId: "CUST-002",
    fullName: "Sarah Johnson",
    nationality: "UAE",
    riskLevel: "High",
    pepFlag: true,
    sanctionsFlag: false,
    document: { type: "Passport", documentNumber: "P654321", expiryDate: "2022-01-01" }
  }
];

const amlData = [
  {
    transactionId: "TXN-001",
    customerId: "CUST-001",
    amount: 5000,
    isCrossBorder: false,
    isHighRiskJurisdiction: false,
    isStructuring: false,
    isSanctionsHit: false
  },
  {
    transactionId: "TXN-002",
    customerId: "CUST-002",
    amount: 25000,
    isCrossBorder: true,
    isHighRiskJurisdiction: true,
    isStructuring: false,
    isSanctionsHit: false
  }
];

function isDocumentValid(document) {
  return Boolean(
    document &&
      document.type &&
      document.documentNumber &&
      new Date(document.expiryDate) > new Date()
  );
}

function requiresEDD(customer) {
  return customer.riskLevel === "High" || customer.pepFlag;
}

function shouldFlagTransaction(txn) {
  return (
    txn.amount > 10000 ||
    txn.isCrossBorder ||
    txn.isHighRiskJurisdiction ||
    txn.isStructuring ||
    txn.isSanctionsHit
  );
}

assert.strictEqual(isDocumentValid(kycData[0].document), true);
assert.strictEqual(isDocumentValid(kycData[1].document), false);
assert.strictEqual(requiresEDD(kycData[1]), true);
assert.strictEqual(shouldFlagTransaction(amlData[0]), false);
assert.strictEqual(shouldFlagTransaction(amlData[1]), true);

console.log("All KYC and AML validation tests passed.");
```

## 6. Requirements Traceability Matrix

| Req ID | Requirement | Test Case ID | Scenario | Expected Result | Status |
| --- | --- | --- | --- | --- | --- |
| KYC-01 | Capture and validate customer identity data | TC-KYC-01 | Complete onboarding submission | Customer data is accepted and validated | Pass |
| KYC-02 | Validate identity documents | TC-KYC-02 | Valid passport upload | Document is accepted | Pass |
| KYC-03 | Reject incomplete KYC submission | TC-KYC-03 | Missing mandatory fields | Submission is blocked | Pass |
| KYC-04 | Detect expired documents | TC-KYC-04 | Expired passport upload | Resubmission is requested | Pass |
| KYC-05 | Screen sanctions and PEP matches | TC-KYC-05 | Sanctions or PEP hit | Account is flagged for review | Pass |
| KYC-06 | Trigger enhanced due diligence | TC-KYC-06 | High-risk onboarding | EDD workflow starts | Pass |
| AML-01 | Detect suspicious transaction patterns | TC-AML-01 | Unusual transaction activity | Alert is raised | Pass |
| AML-02 | Detect high-value transactions | TC-AML-02 | Transfer above threshold | Transaction is flagged | Pass |
| AML-03 | Detect structuring behavior | TC-AML-03 | Repeated low-value transfers | Structuring alert is raised | Pass |
| AML-04 | Flag cross-border high-risk transfers | TC-AML-04 | Transfer to high-risk jurisdiction | AML alert is generated | Pass |
| AML-05 | Support compliance escalation | TC-AML-05 | Suspicious activity detected | Case is routed for manual review | Pass |
| SEC-01 | Enforce role-based access | TC-SEC-01 | Unauthorized access attempt | Access is denied | Pass |

## 7. Sample Test Report

**Project:** KYC and AML Compliance Testing  
**Test Scope:** Functional validation, synthetic data validation, scenario coverage, and traceability verification  
**Overall Result:** Partial Pass

### Summary

- Total test cases executed: 12
- Passed: 10
- Failed: 2
- Blocked: 0

### Execution Results

| Test Case ID | Description | Result | Remarks |
| --- | --- | --- | --- |
| TC-KYC-01 | Validate complete onboarding | Pass | Customer details captured correctly |
| TC-KYC-02 | Validate valid document upload | Pass | Passport accepted |
| TC-KYC-03 | Validate missing mandatory fields | Pass | Submission blocked as expected |
| TC-KYC-04 | Validate expired document rejection | Pass | Expired passport rejected |
| TC-KYC-05 | Validate sanctions and PEP screening | Pass | Match flagged for review |
| TC-KYC-06 | Validate enhanced due diligence trigger | Pass | Workflow initiated |
| TC-AML-01 | Validate suspicious transaction detection | Pass | Unusual transaction flagged |
| TC-AML-02 | Validate high-value transaction alert | Pass | Alert created successfully |
| TC-AML-03 | Validate structuring detection | Fail | Linked transactions were not grouped correctly |
| TC-AML-04 | Validate high-risk cross-border transfer | Pass | Transaction flagged |
| TC-AML-05 | Validate compliance escalation workflow | Pass | Case routed to analyst |
| TC-SEC-01 | Validate role-based access control | Fail | Access rule exception observed for one test user |

### Defects Identified

- `DEF-01`: Structuring logic did not correctly aggregate related small transactions within the monitoring window.
- `DEF-02`: One unauthorized user was incorrectly granted access to a compliance review view.

### Conclusion

The system meets most high-level KYC and AML compliance expectations. The main gaps are in AML structuring detection and one access-control rule, both of which should be corrected and retested before final approval.
