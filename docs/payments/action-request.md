---
title: Payments in Outlook ActionRequest reference | Microsoft Docs
description: Learn about the payload fields for requests sent to payment webhooks by Outlook
author: jasonjoh
ms.topic: reference
ms.technology: o365-connectors
ms.date: 10/25/2018
ms.author: jasonjoh
---

# ActionRequest markup reference

Payment request messages use `ActionRequest` markup to add a payment request card to an email message. `ActionRequest` markup is a Payments in Outlook-specific markup based on the [actionable message MessageCard format](../actionable-messages/message-card-reference.md).

## A simple ActionRequest example

```json
{
  "@type": "ActionRequest",
  "@context": "http://schema.org/extensions",
  "originator": "SAMPLE_ID",
  "actions": [
    {
      "@type": "PaymentAction",
      "merchantId": "SAMPLE_ID",
      "displayId": "SAMPLE_ID",
      "productContext": {
        "key1": "value1",
        "key2": "value2"
      }
    }
  ],
  "entity": {
    "@type": "invoice",
    "provider": {
      "@type": "Organization",
      "name": "Contoso Company",
      "logo": "https://contoso.com/images/music.png"
    },
    "broker": {
      "@type": "Organization",
      "name": "Relecloud Inc.",
      "logo": "https://contoso.com/images/relecloud.png"
    },
    "identifier": "103032",
    "invoiceDate": "2018-03-21",
    "paymentDueDate": "2018-04-21",
    "totalPaymentDue": {
      "@type": "PriceSpecification",
      "price": "98.87",
      "priceCurrency": "USD"
    },
    "url": "https://contoso.com/pFYDOJXuMi8RWs7E4G7sskVLOonV1QUAf4bbaj7S?utm_source=outlookpay",
    "paymentStatus": "http://schema.org/PaymentDue"
  }
}
```

## ActionRequest fields

An `ActionRequest` payload consists of the following fields.

| Field | Type | Description |
|-------|------|-------------|
| `@type` | String | Required. MUST be set to `ActionRequest` |
| `@context` | String | Required. MUST be set to `http://schema.org/extensions` |
| `originator` | String | Required. MUST be set to the provider ID generated by the [Actionable Email Developer Dashboard](../actionable-messages/actionable-email-dev-dashboard.md). |
| `actions` | Array of [PaymentAction](#paymentaction) | Required. MUST contain at least one `PaymentAction`. |
| `entity` | For invoicing - Schema.org [Invoice](https://schema.org/Invoice) | Required. Represents the invoice. |

### PaymentAction

| Field | Type | Description |
|-------|------|-------------|
| `@type` | String | Required. MUST be set to `PaymentAction` |
| `merchantId` | String | Required. MUST be set to the merchant ID obtained by registering in the [partner dashboard for payments in Outlook](partner-dashboard.md). |
| `displayId` | String | Required. MUST be set to the display ID obtained by registering in the [partner dashboard for payments in Outlook](partner-dashboard.md). |
| `productContext` | Object | Required. Developers may specify any valid JSON object in this field. The value is included in the payloads sent to your payment request and payment complete webhooks. |

## Adding ActionRequest markup to email

### Static sender

If you register to send payment requests from a static email address, such as `invoices@contoso.com`, you add `ActionRequest` markup to email bodies using the same method as the [MessageCard format](../actionable-messages/message-card-reference.md). The JSON payload is inserted in a `<script type="application/ld+json">` tag in the `<head>` of the HTML body.

### Dynamic sender

If you register to send payment requests from any users in your domain, there is an additional step required to add `ActionRequest` markup to email bodies. A JSON payload is inserted in a `<script type="application/ld+json">` tag in the `<head>` of the HTML body as before, but uses the following format:

```json
{
  "@type": "SignedActionRequest",
  "@context": "http://schema.org/extensions",
  "signedActionRequest": "[signed_action_request_encoded]"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `@type` | String | Required. MUST be set to `SignedActionRequest` |
| `@context` | String | Required. MUST be set to `http://schema.org/extensions` |
| `signedActionRequest` | String | Required. MUST be set to a signed [DynamicActionRequest](#dynamicactionrequest), signed using the [JSON Web Signature (JWS)](https://tools.ietf.org/html/rfc7515) standard. |

#### DynamicActionRequest

| Field | Type | Description |
|-------|------|-------------|
| `originator` | String | Required. MUST be set to the ID provided by Microsoft during onboarding. |
| `iat` | TimeStamp (Unix epoch) | Required. The time that the payload was signed. |
| `sender` | String | Required. The email address used to send this payment request. |
| `sub` | String | Required. A unique identifier in your payment system for the recipient. |
| `actionRequestSerialized` | String | Required. The serialized content of the `ActionRequest` JSON. |

#### Example dynamic sender payload

1. Generate an `ActionRequest` payload.

    > [!NOTE]
    > The following example payload has been truncated for brevity.

    ```json
    {
        "@type": "ActionRequest",
        "@context": "http://schema.org/extensions"
    }
    ```

1. Serialize the `ActionRequest` payload to a string.

    ```text
    "{\r\n    \"@type\": \"ActionRequest\",\r\n    \"@context\": \"http://schema.org/extensions\"\r\n}"
    ```

1. Create a `DynamicActionRequest` payload.

    ```json
    {
      "originator": "SAMPLE_ORIGINATOR_ID",
      "actionRequestSerialized": "{\r\n    \"@type\": \"ActionRequest\",\r\n    \"@context\": \"http://schema.org/extensions\"\r\n}",
      "iat": "1532995422",
      "sender": "TEST_SENDER@CONTOSO.COM",
      "sub": "SAMPLE_UNIQUE_CUSTOMER_ID"
    }
    ```

1. Sign the payload.

    ```text
    eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJvcmlnaW5hdG9yIjoiU0FNUExFX09SSUdJTkFUT1JfSUQiLCJhY3Rpb25SZXF1ZXN0U2VyaWFsaXplZCI6IntcclxuICAgIFwiQHR5cGVcIjogXCJBY3Rpb25SZXF1ZXN0XCIsXHJcbiAgICBcIkBjb250ZXh0XCI6IFwiaHR0cDovL3NjaGVtYS5vcmcvZXh0ZW5zaW9uc1wiXHJcbn0iLCJpYXQiOiIxNTMyOTk1NDIyIiwic2VuZGVyIjoiVEVTVF9TRU5ERVJAQ09OVE9TTy5DT00iLCJzdWIiOiJTQU1QTEVfVU5JUVVFX0NVU1RPTUVSX0lEIiwiZXhwIjoxNTMyOTk5MDIyLCJuYmYiOjE1MzI5OTU0MjJ9.Ogr4V3nLsTSeZmrcsV0p0RkGRiZO_sGQ8-bWXLK9G4r88DO4A8yGaIHajrL7OQSKwTkY1yY5Jejb3hkvfWUagQyx2D9zYPW9pz98uR2fB42qvImMWrdngES7C41if4Qfj0r5Q4kAdCkGmBdvQlvdZ1pT0vZ1c3vRImd9mo65W2yvnB3ctwlTfGNmdq6cidUXZ-PNx_-mtrL_ZEqpzTRrkEoZ0y1XsjlsegEvq-tYrBIHgzoH-BANOwPFAFaIjheavKwpKKrSzYWJ7W6FDE8CrNrUN-UQXBJlJXDaJ-V3MbXoQQUPcMSsr9f917V2-8VqlYzSm2ZnXX9gMS6qFkzqJw
    ```

1. Generate the `SignedActionRequest` payload

    ```json
    {
      "@context": "http://schema.org/extensions",
      "@type": "SignedActionRequest",
      "signedActionRequest": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJvcmlnaW5hdG9yIjoiU0FNUExFX09SSUdJTkFUT1JfSUQiLCJhY3Rpb25SZXF1ZXN0U2VyaWFsaXplZCI6IntcclxuICAgIFwiQHR5cGVcIjogXCJBY3Rpb25SZXF1ZXN0XCIsXHJcbiAgICBcIkBjb250ZXh0XCI6IFwiaHR0cDovL3NjaGVtYS5vcmcvZXh0ZW5zaW9uc1wiXHJcbn0iLCJpYXQiOiIxNTMyOTk1NDIyIiwic2VuZGVyIjoiVEVTVF9TRU5ERVJAQ09OVE9TTy5DT00iLCJzdWIiOiJTQU1QTEVfVU5JUVVFX0NVU1RPTUVSX0lEIiwiZXhwIjoxNTMyOTk5MDIyLCJuYmYiOjE1MzI5OTU0MjJ9.Ogr4V3nLsTSeZmrcsV0p0RkGRiZO_sGQ8-bWXLK9G4r88DO4A8yGaIHajrL7OQSKwTkY1yY5Jejb3hkvfWUagQyx2D9zYPW9pz98uR2fB42qvImMWrdngES7C41if4Qfj0r5Q4kAdCkGmBdvQlvdZ1pT0vZ1c3vRImd9mo65W2yvnB3ctwlTfGNmdq6cidUXZ-PNx_-mtrL_ZEqpzTRrkEoZ0y1XsjlsegEvq-tYrBIHgzoH-BANOwPFAFaIjheavKwpKKrSzYWJ7W6FDE8CrNrUN-UQXBJlJXDaJ-V3MbXoQQUPcMSsr9f917V2-8VqlYzSm2ZnXX9gMS6qFkzqJw"
    }
    ```