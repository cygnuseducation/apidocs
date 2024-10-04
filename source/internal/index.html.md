---
title: VEGA API docs - Internal APIs

language_tabs: # must be one of https://github.com/rouge-ruby/rouge/wiki/List-of-supported-languages-and-lexers
  - shell

search: false

code_clipboard: true

meta:
  - name: description
    content: VEGA Internal API documentation
---

# Introduction

This document enumerates the most commonly-used APIs called internally within the VEGA web application.

# Authentication

All API requests need to be authenticated with either a session cookie (when accessed via the VEGA web app) or passed in a header named `X-API-Key`:

`X-API-Key: YOUR_API_KEY`

Unless otherwise specified, the requests and responses of these APIs use [JSON:API](https://jsonapi.org/).

# Core types

## Time

All times are represented in RFC3339 format, e.g. `2024-01-02T03:04:05Z`. APIs that aggregate data by day, such as the Performance API, require a timezone to be passed.

Timezones are specified using [tzdb](https://www.iana.org/time-zones) identifiers, e.g. `America/New_York`.

## Public IDs

Many entities in VEGA have a unique *public ID* in addition to a serial numeric id. This public ID is a longer random hexadecimal string. While the numeric id is often used within the internal APIs documented here, all external-facing APIs (e.g. the lead posting endpoint used by third-party vendors) exclusively use public IDs in order to reduce the risk of unintentional (or malicious) substitution of one valid ID for another.

The public ID is often prefixed with a type designator; e.g. `CAMP-abc1234` identifies the campaign with public id `abc1234`. The following type designators currently exist:

Prefix | Type
------ | ----
`VEGA` | [Lead](#leads)
`CAMP` | [Client Campaign](#client-campaigns)
`PROG` | [Degree Program](#degree-programs)

# Clients

A *client* represents an entity that collects leads from vendors, and is
typically equivalent to a school/institution.

> Example client:

```json
{
  "id": "12345",
  "type": "clients",
  "attributes": {
    "name": "Sample Client",
    "time-zone": "America/New_York",
    "created-at": "2020-11-10T02:42:10.653Z",
    "updated-at": "2024-10-03T07:00:00.234Z",
    "tcpa-disclosure": "Custom TCPA Disclosure Text",
    "contact-name": "John Smith",
    "contact-email": "client@example.com",
    "contact-address": "123 Main St",
    "contact-city": "Anytown",
    "contact-state": "MN",
    "contact-zip": "12345",
    "contact-phone": "2125551234",
    "metadata": {
      "statuses": [
      {
        "key": "rejected",
        "value": "Bad Lead"
      },
      {
        "key": "interviewed",
        "value": "Qualified"
      }
      ],
      "dispositions": [
      {
        "key": "not_qualified",
        "value": "Unqualified"
      },
      {
        "key": "duplicate",
        "value": "Duplicate Lead"
      }
      ]
    }
  },
  "relationships": {
    "leads-endpoint": {
      "data": {
        "id": "456",
        "type": "endpoints"
      }
    }
  }
}
```

Attribute | Type | Description
--------- | ------- | -----------
`name` | String | The name of the client
`time-zone` | String | Time zone used for determining start-of-day for allocations
`created-at` | String | Created timestamp
`updated-at` | String | Updated timestamp
`metadata` | [ClientMetadata](#clientmetadata) | Client-specific names for statuses and dispositions
`tcpa-disclosure` | String | (Optional) TCPA disclosure text shown on landing pages
`contact-name` | String | (Optional) default contact name
`contact-email` | String | (Optional) default contact email address
`contact-address` | String | (Optional) default contact street address
`contact-city` | String | (Optional) default contact city
`contact-state` | String | (Optional) default contact state
`contact-zip` | String | (Optional) default contact ZIP code
`contact-phone` | String | (Optional) default contact phone number

Relationship | Type | Description
--------- | ------- | -----------
`leads-endpoint` | LeadsEndpoint | Custom lead-posting endpoint used by third-party vendors posting leads to this client


## GET /api/v1/clients

```shell
curl --get 'https://api.vegaforeducation.com/api/v1/clients' \
  -H 'X-API-Key: YOUR_API_KEY'
```

Returns a list of clients visible to the current user.

## GET /api/v1/clients/:id

```shell
curl --get 'https://api.vegaforeducation.com/api/v1/clients/123' \
  -H 'X-API-Key: YOUR_API_KEY'
```

Returns a specific client.

## PATCH /api/v1/clients/:id

```shell
curl -X PATCH 'https://api.vegaforeducation.com/api/v1/clients/123' \
  -H 'X-API-Key: YOUR_API_KEY' \
  -H 'Content-type: application/json' \
  -d '{
        "data": {
          "attributes": {
            "name": "New client name"
          }
        }
      }'
```

Updates a specific client.

## DELETE /api/v1/clients/:id

```shell
curl -X DELETE 'https://api.vegaforeducation.com/api/v1/clients/123' \
  -H 'X-API-Key: YOUR_API_KEY'
```

Deletes a specific client.

# Vendors

A *vendor* represents an entity that generates leads for clients.

> Example vendor:

```json
{
  "id": "1337",
  "type": "vendors",
  "attributes": {
    "name": "Leads R Us",
    "created-at": "2020-11-10T02:42:10.653Z",
    "updated-at": "2024-10-03T07:00:00.234Z",
    "time-zone-name": "America/New_York"
    "contact-name": "John Smith",
    "contact-email": "vendor@example.com",
    "street-address": "123 Main St",
    "city": "Anytown",
    "state": "MN",
    "zip-code": "12345",
    "phone": "2125551234",
  }
}
```

Attribute | Type | Description
--------- | ------- | -----------
`name` | String | The name of the client
`created-at` | String | Created timestamp
`updated-at` | String | Updated timestamp
`time-zone-name` | String | Time zone used for determining start-of-day for landing page rules
`contact-name` | String | (Optional) default contact name
`contact-email` | String | (Optional) default contact email address
`street-address` | String | (Optional) default contact street address
`city` | String | (Optional) default contact city
`state` | String | (Optional) default contact state
`zip` | String | (Optional) default contact ZIP code
`phone` | String | (Optional) default contact phone number


## GET /api/v1/vendors

```shell
curl --get 'https://api.vegaforeducation.com/api/v1/vendors' \
  -H 'X-API-Key: YOUR_API_KEY'
```

Returns a list of vendors visible to the current user.

## GET /api/v1/vendors/:id

```shell
curl --get 'https://api.vegaforeducation.com/api/v1/vendors/123' \
  -H 'X-API-Key: YOUR_API_KEY'
```

Returns a specific vendor.

## PATCH /api/v1/vendors/:id

```shell
curl -X PATCH 'https://api.vegaforeducation.com/api/v1/vendors/123' \
  -H 'X-API-Key: YOUR_API_KEY' \
  -H 'Content-type: application/json' \
  -d '{
        "data": {
          "attributes": {
            "name": "New vendor name"
          }
        }
      }'
```

Updates a specific vendor.

## DELETE /api/v1/vendors/:id

```shell
curl -X DELETE 'https://api.vegaforeducation.com/api/v1/vendors/123' \
  -H 'X-API-Key: YOUR_API_KEY'
```

Deletes a specific vendor.

# Contracts

A *contract* represents an association between a client and a vendor, and must
exist in order to have VEGA leads associated to the pair.

> Example contract:

```json
{
  "id": "12345",
  "type": "clients",
  "attributes": {
    "payout": "cost_per_lead",
    "can-post-leads": true
  },
  "relationships": {
    "client": {
      "data": {
        "id": "123",
        "type": "clients"
      }
    },
    "vendor": {
      "data": {
        "id": "456",
        "type": "vendors"
      }
    }
  }
}
```

Attribute | Type | Description
--------- | ------- | -----------
`payout` | [Payout](#payout) | Payout type for this contract (default `cost_per_lead`)
`can-post-leads` | Boolean | If true, then vendors are allowed to post leads (default `true`)

Relationship | Type | Description
--------- | ------- | -----------
`client` | [Client](#clients) | Client of this contract
`vendor` | [Vendor](#vendors) | Vendor of this contract

## GET /api/v1/clients/:clientId/contracts

```shell
curl --get 'https://api.vegaforeducation.com/api/v1/clients/123/contracts' \
  -H 'X-API-Key: YOUR_API_KEY'
```

Returns a list of contracts for the specified client.

# Degree Programs

Each client has a number of *degree programs* (often just "programs"),
organized into [program groups](#program-groups).

> Example degree program:

```json
{
  "id": "234",
  "type": "degree-programs",
  "attributes": {
    "name": "Medical Coding and Billing",
    "program_code": "MBC",
    "status": "active",
    "public-id": "cafe123"
  },
  "relationships": {
    "client": {
      "data": {
        "id": "123",
        "type": "clients"
      }
    },
    "program-group": {
      "data": {
        "id": "345",
        "type": "program-groups"
      }
    }
  }
}
```

Attribute | Type | Description
--------- | ------- | -----------
`name` | String | Name of the program
`program-code` | String | Client's proprietary ID for the program, used for lead posting
`status` | DegreeProgramStatus | Program status
`public-id` | Public ID | ID used for non-internal API calls

Relationship | Type | Description
--------- | ------- | -----------
`client` | [Client](#clients) | Client to which this program belongs
`program-group` | [ProgramGroup](#program-groups) | Program group to which this program belongs

# Program Groups

Each degree program belongs to a *program group*.

> Example program group:

```json
{
  "id": "345",
  "type": "program-groups",
  "attributes": {
    "description": "Healthcare Programs",
  },
  "relationships": {
    "client": {
      "data": {
        "id": "123",
        "type": "clients"
      }
    }
  }
}
```

Attribute | Type | Description
--------- | ------- | -----------
`description` | String | Name of the program group

Relationship | Type | Description
--------- | ------- | -----------
`client` | [Client](#clients) | Client to which this program belongs


# Client Campaigns

A *client campaign* (often referred to as simply a "campaign") is essentially a
bucket of leads for a specific contract and a specific program group. Each
campaign has [allocations](#allocations) set for each calendar month.

> Example client campaign:

```json
{
  "id": "42",
  "type": "client-campaigns",
  "attributes": {
    "public-id": "abc1234",
    "name": "Healthcare",
    "campaign-type": "exclusive",
    "status": "active",
    "smart-score": "green",
    "smart-score-explanation": "Cost Per Enroll < $2000"
  },
  "relationships": {
    "contract": {
      "data": {
        "id": "111",
        "type": "contracts"
      }
    },
    "program-group": {
      "data": {
        "id": "222",
        "type": "program-groups"
      }
    }
  }
}
```

Attribute | Type | Description
--------- | ------- | -----------
`public-id` | Public ID | ID used for non-internal API calls
`name` | String | Name of the campaign
`campaign-type` | CampaignType | Type of the campaign (shared or exclusive)
`status` | CampaignStatus | Current status of the campaign
`smart-score` | SmartScore | Smart score shown in the Allocations page
`smart-score-explanation` | String | Description of score generated by SmartScorer service

Relationship | Type | Description
--------- | ------- | -----------
`contract` | [Contract](#contracts) | Contract to which this client campaign belongs
`program-group` | [ProgramGroup](#program-groups) | Program group to which this client campaign belongs

# Allocations

An *allocation* specifies the CPL (cost per lead), cap (monthly allocation, in
number of leads), and vendor projection (estimated lead delivery target, in
number of leads) for a given client campaign, year, and month.

> Example allocation:

```json
{
  "id": "12345",
  "type": "allocations",
  "attributes": {
      "year": 2024,
      "month": 1,
      "cap": 1000,
      "cpl-in-cents": 2000,
      "vendor-projection": 1200
    }
  },
  "relationships": {
    "client-campaign": {
      "data": {
        "id": "789",
        "type": "client-campaigns"
      }
    }
  }
}
```

Attribute | Type | Description
--------- | ------- | -----------
`year` | Integer | Calendar year (e.g. 2024)
`month` | Integer | Calendar month (1-12)
`cap` | Integer | Monthly cap, in number of leads
`cpl-in-cents` | Integer | CPL in cents (e.g. 2000 for $20.00)
`vendor-projection` | Integer | Vendor projection, in number of leads



## GET /api/v1/allocations

```shell
curl --get 'https://api.vegaforeducation.com/api/v1/allocations' \
  -H 'X-API-Key: YOUR_API_KEY' \
  -d 'year=2024' \
  -d 'month=1'
```

Returns allocations, across all campaigns, for a given year and month.

Parameters:

Parameter | Type | Description
--------- | ---- | -----------
`year` | Integer | Year for which to show allocations
`month` | Integer | Month for which to show allocations

## GET /api/v1/clients/:clientId/allocations

```shell
curl --get 'https://api.vegaforeducation.com/api/v1/clients/123/allocations' \
  -H 'X-API-Key: YOUR_API_KEY' \
  -d 'year=2024' \
  -d 'month=1'
```

Returns allocations, across all campaigns, for a given client, year, and month.

Parameters:

Parameter | Type | Description
--------- | ---- | -----------
`clientId` | ID | Client ID
`contractId` | ID | Contract ID
`year` | Integer | Year for which to show allocations
`month` | Integer | Month for which to show allocations

## GET /api/v1/clients/:clientId/contracts/:contractId/allocations

```shell
curl --get 'https://api.vegaforeducation.com/api/v1/clients/123/contracts/321/allocations' \
  -H 'X-API-Key: YOUR_API_KEY' \
  -d 'year=2024' \
  -d 'month=1'
```

Returns allocations, across all campaigns, for a given contract, year, and month.

Parameters:

Parameter | Type | Description
--------- | ---- | -----------
`clientId` | ID | Client ID
`contractId` | ID | Contract ID
`year` | Integer | Year for which to show allocations
`month` | Integer | Month for which to show allocations

# Leads

Lead records can be retrieved with this endpoint.

<aside class="warning">
Depending on the role assigned to the user making the request, PII (e.g. first name, last name, etc) may be masked and substituted with the value `***`.
</aside>

## GET /api/v1/leads

> Sample request

```shell
curl --get 'https://api.vegaforeducation.com/api/v1/leads' \
  -H 'X-API-Key: YOUR_API_KEY' \
  -d 'filter[client-id]=123' \
  -d 'filter[created-at]=2024-01-01T04:00:00Z,2024-01-02T04:00:00Z'
```

The following parameters are supported:

Parameter | Description
--------- | -----------
`per_page` | Number of records to return
`page` | Index of the page number to return (first page is `1`)
`search` | Free-text search on lead data
`filter` | Filters (see below)
`order` | Sort order (see below)

### Filtering

Filters are specified with query-string parameters of the form
`filter[KEY]=VALUE`, where each `KEY` can be an attribute of the lead or one of
the following:

Key | Description
--- | -----------
`client-id` | Client ID to which the lead belongs
`landing-page-id` | Landing Page ID from which the lead originated
`campaign-type` | Type of the client campaign of the lead (e.g. `shared` or `exclusive`)

The `VALUE` of the filters is handled based on the type of the attribute:

Attribute type | Handling
-------------- | --------
Boolean        | `VALUE` must be `true` or `false`
Integer or ID  | `VALUE` may be a comma-separated list of values
String         | `VALUE` is a substring to search for in the field
Time           | `VALUE` is a comma-separated pair of RFC3339 timestamps

By default, only non-test leads are returned. To show test leads instead, pass
`filter[test]=true`.

### Ordering

By default, leads are returned in ascending `id` order. To override the order,
pass `order[KEY]=asc` or `order[KEY]=desc` to order by attribute `KEY` in
ascending or descending order, respectively.

# Performance

The Performance API is used for generating aggregate data for the Performance
Table in VEGA.

## POST /api/v1/performance

> Sample request

```shell
curl -X POST 'https://api.vegaforeducation.com/api/v1/performance' \
  -H 'X-API-Key: YOUR_API_KEY' \
  -H 'Content-type: application/json' \
  -d '{
        "timezone": "America/New_York",
        "reporting_mode": "cohort",
        "start_time": "2024-01-01T04:00:00Z",
        "end_time": "2024-02-01T03:59:59Z",
        "group_by": ["client_id", "vendor_id"]
      }'
```

The performance API accepts a POST request with the following payload:

Key | Description
--- | -----------
`timezone` | The timezone used for aggregating data, as a TZ identifier (e.g. `America/New_York`)
`reporting_mode` | (Optional) The reporting mode, either `cohort` or `snapshot` (defaults to `cohort`)
`start_time` | (Optional) The start time over which to aggregate leads, in RFC3339 format
`end_time` | (Optional) The end time (inclusive) over which to aggregate leads, in RFC3339 format
`group_by` | Grouping columns (see below)
`filter` | Filters (see below)
`aggregates` | (Optional) Additional aggregates (see below)
`limit` | (Optional) Limit number of records to return

The API will aggregate leads, selected based on the time range and filtering
criteria, by the specified grouping columns.

### Grouping

The `group_by` parameter must contain an array of strings which defines the
grouping dimensions. The following grouping columns are allowed:

Column | Description
------ | -----------
`ad_id` | Ad id from marketing platform
`ad_account_id` | Ad account id from marketing platform
`ad_campaign_id` | Campaign id from marketing platform
`ad_set_id` | Adset id from marketing platform
`marketing_platform_id` | Marketing platform id
`client_id` | Client to which the lead belongs
`campus_id` | Campus to which the lead belongs
`degree_program_id` | Degree program for which the lead was originally submitted
`vendor_id` | Vendor to which the lead belongs
`source` | Source name
`program_group_id` | Program Group for which the lead was originally submitted
`client_campaign_id` | Client campaign
`campaign_type` | Campaign Type (`shared` or `exclusive`)
`landing_page_public_id`\* | Landing Page from which the lead originated
`complyed_creative_id` | Creative ID from which the lead originated
`template_id`\* | Landing Page template id
`primary_client_id`\* | Identifies the client whose landing page the lead originated from
`subid` | Subid of the lead (client/vendor-specific)
`year` | Calendar year
`quarter` | Quarter of calendar year (1-4)
`month` | Calendar month (1-12)
`hour` | Hour of day, in the specified timezone
`day` | Day of week (Sunday=0, Saturday=6)
`city` | City
`state` | State
`original_standard_fields:city` | Original city, if lead data has changed
`original_standard_fields:state` | Original state, if lead data has changed
`current_degree_program_id` | Degree program to which the lead is currently assigned

\* Only applies to leads orgiinated from VEGA-hosted landing pages

### Filtering

The optional `filter_by` parameter is an optional JSON object to filter the
leads prior to aggregation. The keys are the same as the grouping columns
above, and the value is an array containing the matching values.

### Additional aggregates

The optional `aggregates` parameter specifies additional aggregates to be
computed on each group. It is an array of JSON objects containing the following
entries:

Key | Description
--- | -----------
`name` | Name of the aggregate column to return
`field` | Name of the [field](#fields) to aggregate on
`type` | The data type (`bigint` or `float`)
`function` | Aggregation function (e.g. `sum`)
`filter` | An optional [lead status](#leadstatus), to filter only leads matching that status

### Response

> Sample response

```json
{
  "data": [
    {
      "client_id": 123,
      "vendor_id": 456,
      "lead_count": 10,
      "billable_lead_count": 8,
      "good_lead_count": 8,
      "pending_lead_count": 0,
      "filtered_lead_count": 0,
      "rejected_lead_count": 1,
      "error_lead_count": 0,
      "returned_lead_count": 1,
      "new_lead_lead_count": 8,
      "contacted_lead_count": 4,
      "interviewed_lead_count": 4,
      "qualified_lead_count": 4,
      "application_started_lead_count": 4,
      "applied_lead_count": 4,
      "admitted_lead_count": 3,
      "confirmed_acceptance_lead_count": 3,
      "enrolled_lead_count": 3,
      "started_lead_count": 0,
      "matriculated_lead_count": 0,
      "class_passed_lead_count": 0,
      "graduated_lead_count": 0,
      "custom_good_1_lead_count": 0,
      "custom_good_2_lead_count": 0,
      "custom_good_3_lead_count": 0,
      "custom_good_4_lead_count": 0,
      "custom_good_5_lead_count": 0,
      "total_cost_in_cents": 1000,
      "impressions": 100,
      "clicks": 50,
      "total_spend_in_cents": 1000,
    }
  ]
}
```

The response contains an array of rows, where each row contains a value for
each of the grouping columns along with the following aggregates:

Column | Description
------ | -----------
`lead_count` | Number of leads
`billable_lead_count` | Number of billable leads
`good_lead_count` | Number of leads with a "good" status (see [LeadStatus](#leadstatus))
`{status}_lead_count` | For each [LeadStatus](#leadstatus), the number of leads with this status. Note that for funnel statuses, the count includes all leads with this specific status *or* a later funnel status. For example, `interviewed_lead_count` includes leads with statuses `interviewed`, `enrolled`, `started`, etc.
`total_cost_in_cents` | Total of the billable leads' `cost_in_cents`, which is based on the `cpl_in_cents` of the [allocation](#allocations)
`impressions` | Number of impressions from the marketing platform integration, if available
`clicks` | Number of clicks from the marketing platform integration, if available
`total_spend_in_cents` | Total spend from the marketing platform integration, if available


# Other types

## ClientMetadata

Metadata for clients. This is a JSON object containing the following:

Key | Description
----- | -----------
`statuses` | List of client-specific status names. Array of objects of the form `{key: "standard-name", value: "client-specific-name"}`, where `standard-name` is one of the [LeadStatus](#leadstatus) values
`dispositions` | List of client-specific disposition names. Array of objects of the form `{key: "standard-name", value: "client-specific-name"}`

## Payout

Payout type for a contract. Can be one of the following:

Value | Description
----- | -----------
`cost_per_lead` | Campaigns are billed on a cost-per-lead (CPL / PPL) basis
`cost_per_enroll` | Campaigns are billed on a cost-per-enroll (CPE) basis
`percentage_of_spend` | Campaigns are billed based on a fixed percentage of marketing spend

## LeadStatus

Lead statuses, which can be customized via [ClientMetadata](#clientmetadata).

Statuses can be categorized as "good" leads versus "bad" leads, as indicated by
the "Good" column below.

A subset of "good" statuses are considered to be part of the enrollment funnel,
as indicated by "Funnel" below. These statuses are special in that every funnel
status is considered to be a subset of preceding statuses, e.g. a lead that has
the status `enrolled` will be included in counts of interviewed, qualified, and
applied leads as well.

Value                                | Good?  | Funnel? | Description
------------------------------------ | ------ | ------- | -----------
`pending`                            |        |         | Lead has been submitted to VEGA and is being processed
`filtered`                           |        |         | Lead has been filtered without being sent downstream to the client's LMS (when VEGA is not the primary LMS). The lead's `filtered_reason` will be set accordingly.
`rejected`                           |        |         | Lead has been rejected by either VEGA (when VEGA is the primary LMS) or the client's downstream system. The lead's `rejection_reason` will be set accordingly.
`error`                              |        |         | An error occurred while attempting to post the lead to the client's downstream system. The lead's `rejection_reason` will be set accordingly.
`returned`                           |        |         | A previously-accepted lead was scrubbed after initial review.
`new_lead`                           |   ✔    |    ✔    | Accepted lead.
`contacted`                          |   ✔    |    ✔    | Lead was successfully contacted.
`interviewed`                        |   ✔    |    ✔    | Lead has been interviewed.
`qualified`                          |   ✔    |    ✔    | Lead was successfully qualified during the interview.
`application_started`                |   ✔    |    ✔    | Lead has started the application process.
`applied`                            |   ✔    |    ✔    | Lead has completed the application process.
`admitted`                           |   ✔    |    ✔    | Lead was admitted to the institution.
`confirmed_acceptance`               |   ✔    |    ✔    | Lead has confirmed their acceptance.
`enrolled`                           |   ✔    |    ✔    | Lead has enrolled with the institution.
`started`                            |   ✔    |    ✔    | Lead has started classes.
`matriculated`                       |   ✔    |    ✔    | Lead has matriculated.
`class_passed`                       |   ✔    |    ✔    | Lead has passed at least one class.
`graduated`                          |   ✔    |    ✔    | Lead has graduated.
`custom_good_1`, ... `custom_good_5` |   ✔    |         | Custom statuses that can be used by clients for miscellaneous states
