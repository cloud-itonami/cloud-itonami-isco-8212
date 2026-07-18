# Security Policy

This project handles electrical and electronic equipment assembler
operating workflows. Treat vulnerabilities as potentially high impact even
when the demo data is synthetic — this domain's failure modes include real
electrical-shock risk from live circuit-board and electrical-equipment
assembly work, and component-damage/data-integrity risk from
ESD (electrostatic discharge)-sensitive component handling, alongside
physical worker-safety risk.

## Do Not Disclose Publicly

Report privately before opening public issues for:

- credential exposure
- real worker, line or operator data exposure
- authorization bypass
- Electrical/Electronic Assembly Line Scheduling Coordination Governor
  bypass
- audit-ledger tampering
- over-disclosure in reports or exports
- unsafe robot action dispatch
- any path that lets a proposal reach an assembly-execution decision, a
  line-safety-clearance decision, or a plant-safety-officer-override
  decision

## Reporting

Use GitHub private vulnerability reporting when available for the repository.
If that is unavailable, contact the repository maintainers through the
cloud-itonami organization before publishing details.

Include:

- affected commit or version
- reproduction steps
- expected and actual behavior
- impact on worker/line data, policy enforcement or audit logging
- suggested fix, if known

## Production Guidance

- Store secrets outside Git.
- Keep real worker/line/operator data outside this repository.
- Run policy tests before deployment.
- Export and review audit logs regularly.
- Use least privilege for operators and service accounts.
