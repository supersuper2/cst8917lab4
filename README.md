# CST8917 Lab 4 – Real-Time Trip Event Analysis

## Overview
This project uses Azure services to monitor taxi trip events for suspicious or notable patterns. It detects:

1. Short cash rides

2. High passenger count trips

3. Vendor anomalies

## Youtube link: https://youtu.be/nwdRoY-5eBI

##  Architecture

```
Event Hub 
    ↓
Logic App
    ↓
Azure Function (analyze_trip)
    ↓
Logic App (For Each trip result)
    ├─ Not interesting → Teams: "No Issues"
    ├─ Suspicious → Teams: "Suspicious Trip"
    └─ Interesting → Teams: "Interesting Trip"
```

## Logic App Architecture



##  Azure Function Logic

Analyzes trips and returns:

1. insights: flags like CashPayment, GroupRide, SuspiciousVendorActivity

2. isInteresting: true if any insights

3. summary: quick description

###  Logic:
```python
if distance > 10: insights.append("LongTrip")
if passenger_count > 4: insights.append("GroupRide")
if payment == "2": insights.append("CashPayment")
if payment == "2" and distance < 1: insights.append("SuspiciousVendorActivity")
```
---

##  Logic App Flow

### Actions
1. **Call Azure Function `analyze_trip`**

2. **For Each loop** on Function results:
   - Add `Condition`:  
     Expression:  
     ```text
     equals(items('For_each')?['isInteresting'])
     ```

   - **If True (Interesting)**:
     - Inner Condition:  
       ```text
       items('For_each')?['insights']?['SuspiciousVendorActivity']
       ```
       - If its True → Post  Suspicious Trip Card
       - If its False → Post Interesting Trip Card

   - **Else **:
     - Post "Trip Analyzed – No Issues" card

---

##  Example Input & Output

### Sample Input (Event Payload to Event Hub):
```json
{
  "vendorID": "4",
  "tpepPickupDateTime": 1528119858000,
  "tpepDropoffDateTime": 1528121148000,
  "passengerCount": 3,
  "tripDistance": 9.09,
  "puLocationId": "186",
  "doLocationId": "230",
  "startLon": null,
  "startLat": null,
  "endLon": null,
  "endLat": null,
  "rateCodeId": 1,
  "storeAndFwdFlag": "N",
  "paymentType": 3,
  "fareAmount": 13.5,
  "extra": 0,
  "mtaTax": 0.5,
  "improvementSurcharge": "0.3",
  "tipAmount": 2.86,
  "tollsAmount": 0,
  "totalAmount": 17.16
}
```

### Sample Output (from Azure Function):
```json
{
  "statusCode": 200,
  "headers": {
    "Date": "Sat, 02 Aug 2025 03:37:18 GMT",
    "Server": "Kestrel",
    "Transfer-Encoding": "chunked",
    "Request-Context": "appId=cid-v1:44ebe048-cbf2-4152-b6d6-578670b8491a",
    "Content-Type": "application/json",
    "Content-Length": "148"
  },
  "body": [
    {
      "vendorID": "4",
      "tripDistance": 9.09,
      "passengerCount": 3,
      "paymentType": "3",
      "insights": [],
      "isInteresting": false,
      "summary": "Trip normal"
    }
  ]
}
```
