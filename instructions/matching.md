# Matching instructions

Load this module only after discovery and deduplication.

## Candidate evidence

Use `candidate/profile.md` for verified qualifications and `candidate/preferences.md` for desired roles and constraints. Read `candidate/resume.pdf` only when a material requirement cannot be checked from the profile. If the files disagree, report the conflict and do not resolve it by assumption.

Never invent experience, technologies, dates, authorization, location, language ability, or willingness to relocate. Distinguish:

- **verified**: explicitly supported by candidate data;
- **uncertain**: not stated or ambiguous;
- **not met**: candidate data explicitly conflicts with a requirement.

Separate mandatory requirements from preferences or nice-to-haves in the job description. Treat vague statements as uncertain, not verified.

## Qualitative classifications

Do not assign scores or weights.

- `strong_match`: target work is relevant; mandatory requirements, remote/location rules, authorization, and contract preferences are verified; no significant gap is known.
- `potential_match`: no known mandatory blocker, but role relevance is partial or a meaningful non-eligibility qualification is uncertain/gapped.
- `eligibility_review`: technical fit may be good, but remote-from-candidate-location, work authorization, sponsorship, residence, or contract eligibility is unresolved. This takes precedence over `strong_match` and `potential_match`.
- `not_recommended`: an exclusion applies, a mandatory requirement is not met, the role is outside configured scope, or remote/location/contract facts explicitly conflict with preferences.

An ambiguous “remote” label is insufficient. Verify the permitted countries or report eligibility as unresolved. Never equate European time-zone compatibility with permission to employ or contract someone in the candidate's country.

## Match result contract

```json
{
  "run_id": "string",
  "candidate_sources": ["candidate/profile.md", "candidate/preferences.md"],
  "resume_read": false,
  "results": [
    {
      "job_id": "string",
      "classification": "strong_match|potential_match|eligibility_review|not_recommended",
      "mandatory_requirements": [
        {
          "requirement": "string",
          "status": "verified|uncertain|not_met",
          "candidate_evidence": "string|null",
          "job_evidence": "string"
        }
      ],
      "matching_reasons": ["string"],
      "significant_gaps": ["string"],
      "unresolved_eligibility": ["string"],
      "exclusion_reason": "string|null"
    }
  ]
}
```

Rank within categories using direct role relevance and verified preferred qualifications, expressed in prose. Preserve category order: strong matches, potential matches, then eligibility-review jobs. Never obscure an eligibility issue because the technical fit is strong. Apply `maximum_daily_results` only after mandatory exclusions and classification.
