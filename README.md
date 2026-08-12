# CRM Bonus

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

CRM Bonus (CRMBonus) is a Brazilian retail technology platform for customer acquisition, conversion
and loyalty — Giftback, Vale Bonus, CRMBack, CRMAds, Conversational Commerce, and Oto CRM.

**Public API surface.** The publicly documented machine-readable contract is the **Oto Data API**, an
OpenAPI 3.1.0 upsert-ingestion API for the Oto CRM platform, served at `https://data-api.otocrm.com.br`
with a public ReDoc reference. Oto CRM was acquired outright by CRMBonus from WPP in June 2025; it is
listed as a CRMBonus retail solution in the crmbonus.com.br navigation, and the Oto help center is
titled "Oto CRM | Powered by CRMBonus".

The Giftback / Vale Bonus / CRMAds APIs on `api.crmbonus.com` are partner-token gated with no public
reference. That host answers HTTP 200 with an identical `{"correlation_id":"…","message":"OK!!"}`
envelope for **every** path, so a 200 there is not evidence a document exists — none of those
responses were credited in this profile.

Backed by: bond-capital — https://crmbonus.com.br
