## Validating requests

- show examples from the mobile-phone-add implementation. (hibernate annotation + records)

## Code format

Add code format gradle plugin to Radar, or lint failure step (can we run a quick lint pipeline to fail a branch commit)?
- RADAR Item - try to automate this via CI/CD, a gradle plugin https://github.com/google/google-java-format or project level IDE settings and take the onus away from the developer.


## Definition of Done

- Acceptance criteria met; no open critical defects; increment is usable/releasable
- Code peer-reviewed and approved.
- Tests added/updated (unit, integration, and e2e where relevant); all pass in CI.
- CI is green, including build and security/vulnerability scans.
- Documentation updated as needed (user/operational notes, README/config).
- Integrated and validated: merged to `develop` branch and, when applicable, deployed to a staging environment and tested.
- DoD checklist completed on the issue/PR and visible; the team agrees it’s done.
