---
title: VEGA API docs

language_tabs: # must be one of https://github.com/rouge-ruby/rouge/wiki/List-of-supported-languages-and-lexers
  - shell

search: false

code_clipboard: true

meta:
  - name: description
    content: VEGA API documentation
---

# Introduction

VEGA's lead API enables clients (institutions), lead vendors, and agencies to create prospective student records in the VEGA database.  Each program has a configurable filter/exclusion ruleset.

To make updates to your specific posting requirements or filter rules please login here: <a href="https://app.vegaforeducation.com/" target="_blank">https://app.vegaforeducation.com/</a> and navigate to the management tab.

# Authentication

All API requests need to be authenticated with your API key, passed in a header named `X-API-Key`:

`X-API-Key: YOUR_API_KEY`

Lead submission requires a client or vendor API key, unless the client has
enabled anonymous lead submission.

All other APIs must be authenticated with a client API key.

# Leads

## Data model

Each lead has the following standard fields, in addition to any custom leads
configured by the client in the Fields second of VEGA.

### Lead

> Example lead:

```json
{
  "id": "a43098ab",
  "external_id": "SALESFORCE_ID_12345",
  "campaign_id": "d7e9134",
  "program_code": "PSYCH",
  "status": "interviewed",
  "billable": true,
  "raw_params": {
    ...
  },
  "remote_ip": "1.2.3.4",
  "source": "facebook",
  "subid": "creative_1234",
  "vendor_id": "c867001",
  "cost_in_cents": 7500,
  "first_name": "Jane",
  "last_name": "Doe",
  "phone": "2125551234",
  "email": "jane.doe@example.com",
  "street_address": "123 Fake St",
  "city": "Oak Lawn",
  "state": "IL",
  "zip_code": "60453",
  "GraduationYear": "2018",
  "Military": "1",
  "StartClasses": "within_3_months"
}
```

Parameter | Type | Description
--------- | ------- | -----------
`id` | String | The identifier of the lead
`external_id` | String | The identifier of the lead in the client's CRM, e.g. Salesforce
`campaign_id` | String | The id of the campaign for which the lead was submitted
`program_code` | String | The id of the program for which the lead was submitted
`status` | [Status](#status) | The status of the lead
`billable` | Boolean | Whether or not the lead is billable to the vendor
`rejection_reason` | String | Rejection reason, if the status is `rejected` or `error`
`filtered_reason` | String | Filter reason, if the status is `filtered`
`raw_params` | JSON | Raw parameters, exactly as they were submitted to the POST endpoint
`remote_ip` | String | IP Address from which the POST endpoint was called
`source` | String | The source of the lead (e.g. Facebook, Google, etc). The semantics of this field are client-defined.
`subid` | String | Finer-grained source identifier, e.g. identifying the specific advertising creative that was shown to the lead. The semantics of this field are client-defined.
`vendor_id` | String | Identifier of the vendor that submitted the lead. This may be an **internal vendor** of the client.
`cost_in_cents` | Integer | The cost-per-lead, in cents, assigned to the lead. Note that this value is present whether or not the lead is billable, so in order to compute total outlay, sum the `cost_in_cents` over only those leads where `billable` is true.
`first_name` | String | First name
`last_name` | String | Last name
`phone` | String | Phone number (digits only)
`email` | String | Email address
`street_address` | String | Street address, possibly normalized with Google Maps API
`city` | String | City
`state` | String | State
`zip_code` | String | ZIP code
... | ... | All client-specific custom fields

### Status

The following statuses are supported by default. Other statuses can be configured per-client:

Status | Description
------ | -----------
`filtered` | Filtered before external validation
`rejected` | Rejected by external validation
`error` | Internal error occurred during lead processing
`returned` | Previously accepted lead was invalidated after the fact
`new_lead` | Lead received and accepted
`contacted` | Lead has been contacted by an admissions advisor
`interviewed` | Lead has completed the interview process
`applied` | Lead has applied to the program
`enrolled` | Lead has enrolled
`started` | Lead has started at least one class in the program
`class_passed` | Lead has passed at least one class


## Post a new lead

```shell
curl -X POST 'https://leads.vegaforeducation.com/partner/leads' \
  -H 'X-API-Key: YOUR_API_KEY' \
  -H 'Content-type: application/json' \
  -d '{
        "campaign_id": "d7e9134",
        "program_code": "PSYCH",
        "StreetAddress": "123 Fake St",
        "City": "Oak Lawn",
        "State": "IL",
        "ZIP_Code": "60453",
        "FirstName": "Jane",
        "LastName": "Doe",
        "Phone": "2125551234",
        "Email": "jane.doe@example.com",
        "GraduationYear": "2018",
        "Military": "1",
        "StartClasses": "within_3_months"
      }'
```

> Accepted lead response with status 200:

```json
{
  "status": "new_lead"
}
```

> Rejected lead response with status 200:

```json
{
  "status": "rejected",
  "rejection_reason": "Duplicate lead: jane.doe@example.com."
}
```

Submit a new lead into VEGA.

Leads can be submitted with a vendor API key or a client API key.

To submit a new lead, refer to the vendor-facing auto-generated form posting
instructions in VEGA for the set of parameters to pass, as the set of fields is
client-specific.

### HTTP Request

`POST https://leads.vegaforeducation.com/partner/leads`

<aside class="success">
In both the accepted and rejected cases, the API will return a 200 status.
</aside>

## Get leads

```shell
curl --get 'https://leads.vegaforeducation.com/partner/leads' \
  -H 'X-API-Key: YOUR_API_KEY' \
  -d 'start_time=2021-01-01T05:00:00Z' \
  -d 'end_time=2021-02-01T04:59:59Z' \
  -d 'campaign_id=d7e9134'
```

> Response:

```json
{
  "leads": [
    {
      "id": "a43098ab",
      "external_id": "SALESFORCE_ID_12345",
      "campaign_id": "d7e9134",
      "program_code": "PSYCH",
      "status": "interviewed",
      "billable": true,
      "cost_in_cents": 7500,
      "first_name": "Jane",
      "last_name": "Doe",
      "phone": "2125551234",
      "email": "jane.doe@example.com",
      "street_address": "123 Fake St",
      "city": "Oak Lawn",
      "state": "IL",
      "zip_code": "60453",
      "GraduationYear": "2018",
      "Military": "1",
      "StartClasses": "within_3_months"
    },
    ...
  ]
}
```

Get a list of your leads.

This endpoint is only accessible with a client API key.

Note that the set of fields returned in the lead data is client-specific and
can be seen in the Fields section of the VEGA management dashboard.

### HTTP Request

`GET https://leads.vegaforeducation.com/partner/leads`

Parameter | Type | Description
--------- | ------- | -----------
`start_time` | ISO 8601 timestamp | Only show leads created at or after the specified time
`end_time` | ISO 8601 timestamp | Only show leads created at or before the specified time
`campaign_id` | String | Only show leads associated with the specified campaign
`vendor_id` | String | Only show leads submitted by the specified vendor
`program_code` | String | Only show leads associated with the specified program

## Update a lead

> Request:

```shell
curl -X PATCH 'https://leads.vegaforeducation.com/partner/leads' \
  -H 'X-API-Key: YOUR_API_KEY' \
  -H 'Content-type: application/json' \
  -d '{
        "id": "a43098ab",
        "status": "returned",
        "billable": false
      }'
```

Update the `status` or `billable` fields of a lead.

This endpoint is only accessible with a client API key.

`PATCH https://leads.vegaforeducation.com/partner/leads`

Parameter | Type | Description
--------- | ------- | -----------
`id` | String | The identifier of the lead
`status` | [Status](#status) | The new status for the lead
`billable` | Boolean | Whether or not the lead is billable. If it is not specified, it is inferred from the status.
