# Classify findings and check dependencies

Grade impact and confidence separately. Before emitting an issue, establish a credible trigger, its reachability, what the changed code does, and the practical consequence. Check whether the issue predates the diff. A pre-existing problem is relevant when this change introduces a new path to it or makes its impact worse.

| Impact | Typical handling |
| --- | --- |
| Critical | Block for a verified release-stopping impact, such as likely data loss, a severe security exposure, or a broad outage. |
| High | Block for a verified regression on a supported path, an important broken contract, or a high-impact failure under credible conditions. |
| Medium | Use an issue when a bounded defect is established. Choose blocking when merging would violate an applicable contract or create an unacceptable operational cost. |
| Low | Use an issue for a small confirmed defect, a chore for an applicable requirement, or a non-blocking suggestion for an optional improvement. |

An incomplete proof stays a non-blocking suggestion or question when the concern warrants follow-up. Report clear local standards fixes as completed edits. A standards preference, a deferred part of a broader spec, or a missing test by itself gives insufficient basis to block. If the review tool requires priorities, map them to the repository's convention.

## Dependencies

Compare dependency manifests and lockfiles against the base. Identify new packages and upgrades, including transitive changes introduced by the diff. Record the ecosystem, exact resolved version, and where the dependency runs. Consider runtime, build, and install paths. If you cannot recover the resolved version, check the declared range and report that limit.

For each changed dependency, check current maintainer advisories and a trusted advisory database. Match advisory ranges to the resolved version. Investigate material related risks with a concrete signal, such as a compromised release or a breaking security default. Tie findings to the affected configuration and execution path. A CVE identifier or score alone gives insufficient evidence of local impact. [GitHub Advisory Database](https://github.com/advisories) · [OSV](https://osv.dev/)

Apply the organization reuse exception when you can verify that another microservice in the organization already uses the same resolved version under materially comparable conditions. Cite the service and its manifest or lockfile, and record that you skipped the fresh lookup. Still account for known advisories or any exposure new to this service. If you cannot verify the exception, perform the online check. If a lookup is unavailable, report the verification gap.
