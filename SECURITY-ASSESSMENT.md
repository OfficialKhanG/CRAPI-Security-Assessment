```markdown
# 🔒 CRAPI Security Assessment: Complete Vulnerability Analysis

**Author:** Security Researcher  
**Date:** March 2026  
**Target:** CRAPI (Comprehensive REST API)  

---

## 📋 Executive Summary

This report documents **4 critical vulnerabilities** discovered in CRAPI.

| Severity | Count |
|----------|-------|
| 🟠 High | 4 |

---

## 🔴 Vulnerability #1: Order Return BOLA

CVSS Score: 7.6 (High)
Vector: CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:L/A:L

### Affected Endpoint
POST /workshop/api/shop/orders/return_order?order_id={id}

text

### Proof of Concept
```bash
# Login as Sasuke
curl -X POST http://localhost:8888/identity/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"sasuke@test.com","password":"Sasuke@1234"}' \
  -c cookies.txt

# Return Itachi's order
curl -X POST "http://localhost:8888/workshop/api/shop/orders/return_order?order_id=11" \
  -b cookies.txt

Impact

An authenticated attacker can manipulate the order_id parameter to return orders belonging to other users without authorization. This exposes victim order details including purchase history, product information, and transaction records. BOLA represents the OWASP API Security Top 10 #1 risk precisely because server-side object-level authorization checks are frequently absent or incomplete, making this vulnerability endemic across real-world APIs.
In a production environment, this vulnerability allows an attacker to impersonate legitimate users, manipulate financial transactions, and potentially redirect refunds to unauthorized accounts — causing direct monetary loss to both users and the organization. The business risk includes regulatory liability under India's DPDP Act 2023 and reputational damage if exploited at scale. This class of vulnerability is routinely missed in standard security assessments because most testing occurs at the UI level rather than the API layer, leaving authorization logic completely unvalidated.

🔴 Vulnerability #2: Vehicle Location Tracking

CVSS Score: 7.1 (High)
Vector: CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:L/A:N

Affected Endpoints

GET /community/api/v2/community/posts/recent
GET /identity/api/v2/vehicle/{vehicle_id}/location

Proof of Concept

# Extract vehicle ID
VEHICLE_ID=$(curl -s http://localhost:8888/community/api/v2/community/posts/recent \
  | jq -r '.posts[0].author.vehicleid')

# Track vehicle
curl -X GET "http://localhost:8888/identity/api/v2/vehicle/$VEHICLE_ID/location" \
  -b cookies.txt

Impact

An authenticated attacker can exploit missing object-level authorization by enumerating vehicle identifiers from publicly accessible community data and directly querying backend APIs to retrieve real-time location information of other users’ vehicles. This exposes precise live location data of vehicle owners whose identifiers were harvested, putting their physical safety and privacy at risk. In a production environment, such a flaw can enable stalking, theft, or other illegal activities, leading to severe business consequences including regulatory penalties under frameworks like the Digital Personal Data Protection Act, 2023 in India, as well as significant reputational damage. This vulnerability is often missed during testing because assessments are typically limited to front-end workflows and do not rigorously validate server-side authorization controls at the API level.

🔴 Vulnerability #3: Mass Assignment - Admin Creation

CVSS Score: 8.8 (High)
Vector: CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H

Affected Endpoint

POST /identity/api/auth/signup

Proof of Concept

curl -X POST http://localhost:8888/identity/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{
    "name":"attacker",
    "email":"attacker@test.com",
    "password":"Pass@1234",
    "number":"90999999999",
    "role":"admin"
  }'

Impact

An authenticated attacker can exploit a mass assignment flaw by injecting privileged fields such as `role=admin` in the signup request to escalate their privileges during account creation. This grants the attacker unauthorized administrative access, allowing full control over system functionality, user management, and sensitive operations. In a production environment, this can lead to complete system compromise, data breaches, service disruption, and serious legal consequences including regulatory liability under the Digital Personal Data Protection Act, 2023 in India, along with loss of customer trust. This vulnerability is often missed because developers rely on client-side restrictions or assume implicit field validation, while failing to enforce strict server-side allowlisting of assignable attributes.

🔴 Vulnerability #4: Negative Price Exploit

CVSS Score: 7.1 (High)
Vector: CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:L/I:H/A:N

Affected Endpoint

POST /workshop/api/shop/products

Proof of Concept

# Create negative-priced product
curl -X POST http://localhost:8888/workshop/api/shop/products \
  -b cookies.txt \
  -H "Content-Type: application/json" \
  -d '{
    "name":"Money Printer",
    "price":"-20000",
    "image_url":"https://evil.com/money.png"
  }'

Impact

An authenticated attacker can exploit insufficient server-side validation by submitting a negative value in the `price` field when creating a product, effectively manipulating the application’s business logic. This allows the attacker to generate financial gain by abusing transactions involving negative-priced items, potentially leading to unauthorized credits or balance manipulation. In a production environment, this can result in severe financial losses, abuse of the platform’s economic model, and legal consequences including regulatory liability under the Digital Personal Data Protection Act, 2023 in India, along with reputational damage. This vulnerability is often missed because testing typically focuses on valid input ranges and UI constraints, while failing to enforce strict validation and business rules at the backend level.

🛡️ Remediation Recommendations

1. Fix BOLA

def return_order(order_id, current_user):
    order = Order.query.filter_by(
        id=order_id, 
        user_id=current_user.id
    ).first()
    if not order:
        return {"error": "Unauthorized"}, 403

2. Fix BOLA (Vehicle Access Control)

def get_vehicle_location(vehicle_id, current_user):
    vehicle = Vehicle.query.filter_by(
        id=vehicle_id,
        owner_id=current_user.id
    ).first()
    if not vehicle:
        return {"error": "Unauthorized"}, 403

Do Not Expose Vehicle IDs

post_data = {
    "author": {
        "name": user.name
        # remove vehicle_id from response
    }
}

Add Server-Side Authorization Check

if vehicle.owner_id != current_user.id:
    return {"error": "Forbidden"}, 403

3. Fix Mass Assignment

class UserRegistrationDTO:
    allowed_fields = ['name', 'email', 'password', 'number']
    role = 'user'  # Always default

4. Fix Negative Price

def create_product(data):
    if float(data.get('price', 0)) <= 0:
        return {"error": "Invalid price"}, 400

📈 Testing Methodology (UPTV)

Phase	           Activities
Understand	     Map endpoints, analyze business logic
Prioritize	     Assess impact
Test	           Functional, negative, edge cases
Validate	       Response analysis

🛠️ Tools Used
curl - API testing
jq - JSON parsing
CRAPI - Target application
