---
name: affiliate-compliance-check
description: 'Automated compliance checking for affiliate marketing content. Verifies FTC disclosure requirements, link tracking, and ethical affiliate practices.'
---

# Affiliate Compliance Check

Automated and manual workflows for ensuring affiliate marketing content complies with FTC guidelines and follows ethical best practices.

## When to Use

- Audit content for FTC disclosure compliance
- Verify affiliate links are properly tracked
- Check for broken or missing affiliate links
- Prepare content for publication review

## Quick Compliance Check

```bash
# Find files with affiliate links
grep -r -l -E "(affiliate|aff\.|&id=|\?ref=)" \
  copywriting/ landing_pages/ --include="*.md" --include="*.html"

# Check for disclosures
for file in $(grep -r -l -E "(affiliate|aff\.)" copywriting/ --include="*.md"); do
  if ! grep -q -i "disclosure" "$file"; then
    echo "❌ Missing disclosure: $file"
  fi
done
```

## FTC Requirements

**Required**:
- Clear disclosure before/near first affiliate link
- Plain language (not legal jargon)
- Conspicuous placement (visible on mobile)
- Explains potential compensation

**Acceptable Disclosure**:
```markdown
## Disclosure

This post contains affiliate links. I may earn a commission if you make 
a purchase, at no additional cost to you. I only recommend products I 
personally use or genuinely believe will add value.
```

## Compliance Checklist

- [ ] Disclosure exists
- [ ] Disclosure is above the fold
- [ ] Language is clear and plain
- [ ] Visible on mobile devices
- [ ] All affiliate links work
- [ ] Links in mapping file
- [ ] Content provides genuine value
- [ ] Honest assessment (pros/cons)

## Resources

- Mapping: `docs/affiliate_links/affiliate_links_mapping_FINAL.json`
- FTC Guidelines: https://www.ftc.gov/business-guidance/resources/disclosures-101-social-media-influencers
