# Attacker Mindset – IDOR

- Where do identifiers appear? (path/query/body/headers/GraphQL)
- Can I obtain a second account or a collaborator to act as victim?
- Which resources are likely enumerable? (orders, invoices, tickets, files)
- Do batch endpoints validate each element?
- Are there hidden fields I can overpost? (`owner_id`, `account_id`)
- Do file handlers authorize on every request (not only on link creation)?
