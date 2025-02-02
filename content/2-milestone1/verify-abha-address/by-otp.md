+++
title = "By OTP"
date = 2023-03-16T09:30:25+05:30
weight = 3
chapter = true
pre = "<b>2.2.3 </b>"
+++

# By OTP

|  Applicable To                             |   HMIS / LMIS (PVT)  |   Government Health App  |    
|-------------------------------|:----------------------:|:--------------------:|
|   By OTP                      |  {{% badge style="blue" %}} Mandatory{{% /badge %}}       |  {{% badge style="blue" %}} Mandatory{{% /badge %}}        |  

## Functionality Overview

When user visits the hospital, and shares ABHA address with Hospital staff. The staff verifies it with OTP on the mobile number attached to that ABHA address.

**Steps to initiate verification of ABHA Address by OTP:**
- Patient must have a valid ABHA address.
- Patient verbally shares the ABHA address to the health facility during registration.
- Facility initiates a mobile OTP authentication for this ABHA address.
- Then patient will receive an OTP on their mobile, which they must share with the operator.
- The facility sends the OTP to the HIE-CM. On successful validations it gets the demographic details of the user and a new linking token.
- The HRP software must store this access (linking) token in their system for the purpose of linking care contexts to the patient.

![Sharing PHR Verbally](/abdm-docs/img/share_phr_verbally.PNG)

## Test Cases

Functionality|Test Case|Steps to execute|
| ----- | ----- | ----- |
{{% badge style="blue"%}}Mandatory{{% /badge %}} ABHA Verification Using Mobile Number and Mobile OTP (VRFY_ABHA_401)|System must allow ABHA retrieval and verification using communication mobile number and OTP|**1.** Share Mobile Number. **2.** Receive OTP on Mobile Number. **3.** Enter OTP. **4.** Get list of ABHA Numbers linked to the Mobile Number. **5.** Select one ABHA Number|


## API Sequence Diagram

{{< mermaid >}}
%%{init:{"fontSize": "1.0rem", "sequence":{"showSequenceNumbers":true}}}%%
sequenceDiagram
title Verify User's ABHA Address by User shared OTP
actor User
User-->>HIP System:Share the health ID and preferred auth Mode
HIP System->>Repository:User shares Health ID & selects Auth mode as MOBILE_OTP
activate Repository
Repository->>Gateway:/v0.5/users/auth/init
Gateway->>CM:/v0.5/users/auth/init
activate CM
CM-->>User:Sends OTP
CM->>Gateway:/v0.5/users/auth/on-init
deactivate CM
Gateway->>Repository:/v0.5/users/auth/on-init
Repository-->>HIP System:Notification
deactivate Repository
User-->>HIP System:Shares the received OTP
HIP System->>Repository:Forwrads OTP
activate Repository
Repository->>Gateway:/v0.5/users/auth/confirm
Gateway->>CM:/v0.5/users/auth/confirm
activate CM
note left of CM:Shares a new auth.accessToken specific for Linking
CM->>Gateway:/v0.5/users/auth/on-confirm
deactivate CM
Gateway->>Repository:/v0.5/users/auth/on-confirm
Repository-->>HIP System:Notification
deactivate Repository
{{< /mermaid >}}


## API Information Request Response 


**1. Get a patient's authentication modes**

**URL:** https://dev.abdm.gov.in/gateway/v0.5/users/auth/fetch-modes

{{< swaggermin src="/abdm-docs/Yaml/ndhm-hip.yml" api="POST /v0.5/users/auth/fetch-modes$" >}}

**2. Accept callback with supported authentication modes for this PHR Address**

**URL:** {HRP CALLBACK URL}/v0.5/users/auth/on-fetch-modes

{{< swaggermin src="/abdm-docs/Yaml/ndhm-hip.yml" api="POST /v0.5/users/auth/on-fetch-modes$" >}}

**3. Initialize authentication from HIP**

**URL:** https://dev.abdm.gov.in/gateway/v0.5/users/auth/init

{{< swaggermin src="/abdm-docs/Yaml/ndhm-hip.yml" api="POST /v0.5/users/auth/init$" >}}


**4. Response to user authentication initialization**

**URL:** {HRP CALLBACK URL}/v0.5/users/auth/on-init

{{< swaggermin src="/abdm-docs/Yaml/ndhm-hip.yml" api="POST /v0.5/users/auth/on-init$" >}}

User will get a SMS on their mobile phone with a OTP from the HIE-CM. Collect this OTP from the user and send it via the next API

**5. Confirmation request by sending token, otp**

**URL:** https://dev.abdm.gov.in/gateway/v0.5/users/auth/confirm

{{< swaggermin src="/abdm-docs/Yaml/ndhm-hip.yml" api="POST /v0.5/users/auth/confirm$" >}}

Ensure the transaction ID is the same as obtained in the on-init callback

**6. Callback from HIE-CM with KYC and Token information**

**URL:** {HRP CALLBACK URL}/v0.5/users/auth/on-confirm

{{< swaggermin src="/abdm-docs/Yaml/ndhm-hip.yml" api="POST /v0.5/users/auth/on-confirm$" >}}


