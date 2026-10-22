# Data model and privacy rules

This is what the dashboard expects from each source. These are design assumptions, not a confirmed schema. Check them against Samaritans' live Salesforce org and ad accounts before building anything.

## Sources

Every source is reduced to counts before it reaches the dashboard.

| Source | Refresh | Brings in | Never brings in |
| --- | --- | --- | --- |
| **Salesforce** | Every 15 min | Conversation timestamps and tags, supporter shifts and hours (V4S), onboarding stages, survey responses, campaigns | Message content, phone numbers, texters' names |
| **TikTok Ads** | Daily | Spend, impressions, reach, clicks, video views, age / gender / state audience reports | Individual profiles, custom audiences built from texters |
| **Google Analytics 4** | Hourly | Sessions, UTM campaign tags, "Text Hey Sam" clicks, device and state | IP addresses, User-ID, cross-site tracking |
| **Meta & Google Ads** | Daily | Instagram and Search spend, reach, clicks, age / gender / region breakdowns | Pixel data tied to individuals |
| **Excel uploads** | Weekly | Shift rosters, school partnership log, donation budget, cohort plan | Names or contact details (stripped on import) |

## Assumed Salesforce fields

The design assumes the texting platform writes one `Conversation__c` record per conversation, with timestamps and tags only. No message body, no phone number. Volunteers are managed with Volunteers for Salesforce (V4S). Survey responses are a separate object with no lookup to `Conversation__c`.

| Field / object | Type | Used for |
| --- | --- | --- |
| `Conversation__c.Start_Time__c` | Datetime (ET) | Demand heatmap, weekly volume, hourly match to campaigns |
| `Conversation__c.First_Response_Seconds__c` | Number | Median first response, share waiting over 10 minutes |
| `Conversation__c.After_Hours__c` | Checkbox | After-hours texts (auto-reply with 988 referral) |
| `Conversation__c.Is_First_Contact__c` | Checkbox | Share of first-time texters, set by the platform with no identifier stored |
| `Conversation__c.Topic_Tags__c` | Multi-select picklist | Conversation topics |
| `Conversation__c.Safety_Protocol__c` | Checkbox | Weekly count of conversations using the safety protocol |
| `GW_Volunteers__Volunteer_Shift__c` | V4S object | Desired vs filled slots, giving shift fill rate |
| `GW_Volunteers__Volunteer_Hours__c.Hours_Worked__c` | Number | Supporter hours per week, giving capacity |
| `Contact.Onboarding_Stage__c` | Picklist | Pipeline: Applied → Screened → Training → Shadowing → Active |
| `Survey_Response__c` | Custom object | Felt heard, feel better, would text again, how they heard, optional banded demographics |
| `Campaign.UTM_Campaign__c`, `ActualCost` | Standard + custom | Joins ad spend and GA4 traffic to one campaign record |

## De-identification rules

1. **No message content.** Conversations never leave the texting platform. Only timestamps and supporter-applied topic tags reach Salesforce.
2. **Small-group suppression.** Any figure based on fewer than 10 people shows as `<10` and is left out of totals that could reveal it.
3. **Banded demographics.** Age in platform bands (13–17, 18–24, 25+). Location at state level. Survey demographics are optional and self-reported.
4. **Aggregate attribution.** Campaigns are linked to demand by matching hourly counts. A person is never followed from an ad to a conversation.
5. **Unlinked surveys.** `Survey_Response__c` has no lookup to `Conversation__c`, so feedback can't be traced to a conversation.
6. **Role-based access.** Only program, marketing and leadership roles see the dashboard. Access is reviewed every quarter, and source logs are deleted after 90 days.

## Campaign verdicts

The Marketing tab labels each campaign using two numbers: cost per conversation, and sustained lift (the share of peak-week demand still there in week 6).

- **Scale.** Sustained lift of 70% or more and under $30 per conversation. Grow spend as capacity grows.
- **Hold.** Moderate retention. Keep current spend and retest the creative.
- **Burst only.** Demand fades fast. Run only in weeks the forecast shows spare capacity.
