# 30MHz

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
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

30MHz is a Rotterdam-based horticulture technology company that builds wireless in-crop sensors and the
ZENSIE data platform for greenhouse and controlled-environment growers. Sensors capture temperature,
humidity, PAR/light, CO2, substrate moisture and EC, stream it to ZENSIE over LoRa gateways, and surface
it as dashboards, maps, alerts, cultivation strategies and AI-generated growing advice.

## The API

30MHz publishes the **ZENSIE API**, a Swagger 2.0 REST contract served by the API host itself at
<https://api.30mhz.com/api/swagger.json> — **425 paths, 558 operations, 300 schema definitions**. It is
not linked from the marketing site; the support centre's developer docs point at the Swagger UI at
<https://api.30mhz.com/api/swagger>. Authentication is a bearer JWT API key created in the ZENSIE web app
under Account Settings > Developer.

- Developer docs: <https://support.30mhz.com/developer-docs>
- Create an API key: <https://support.30mhz.com/create-an-api-key>
- Company: <https://30mhz.com/> · Pricing: <https://www.30mhz.com/pricing/>

## What this profile found

Probed 2026-09-05. No `/.well-known/` document of any kind on any of the five hosts, no security.txt,
no agent card, no MCP server, no AsyncAPI and no advertised webhooks, no public SDK on any registry
(a first-party Python client exists but is only available on request through support), no CLI, no
sandbox, no status page, no SLA, no deprecation policy, and no published rate limits. The contract
itself is strong — every operation carries an operationId, 556 of 558 carry a summary, and 19
operations are correctly marked deprecated as the `location` vocabulary migrates to `site` — but it
ships zero in-spec examples, no `application/problem+json`, no idempotency key on any of its 240
mutating operations, and pagination on only 2 of 558.
