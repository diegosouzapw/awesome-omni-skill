---
name: ethereum-proposal-researcher
description: Deep research and analysis of Ethereum proposals from ethresear.ch, evaluating technical impacts, improvement opportunities, and technical debt implications for existing projects.
---

# Ethereum Proposal Researcher

Comprehensive analysis of Ethereum proposals from ethresear.ch forum, providing technical impact assessment, improvement opportunities, technical debt analysis, and implementation recommendations.

## When to Use This Skill

Use this skill when the user wants to:
- Research and understand Ethereum proposals from ethresear.ch
- Evaluate technical impact on existing projects
- Assess improvement opportunities from proposal adoption
- Identify technical debt implications
- Get implementation recommendations and roadmaps
- Make informed decisions about proposal adoption

**Trigger phrases:**
- "研究以太坊提案"
- "analyze ethereum proposal"
- "evaluate proposal impact"
- "提案可行性分析"
- "assess technical debt from proposal"
- Any mention of ethresear.ch URLs

## Prerequisites

Ensure the following are available:
- Python with requests, beautifulsoup4 packages
- Internet access for fetching proposals
- Utility: `utils/proposal_fetcher.py`
- Optional: Foundry cast for on-chain queries

## Analysis Workflow

### Stage 1: Proposal Acquisition & Understanding

**Objective:** Extract and understand the proposal content.

**Process:**

1. **Get the proposal URL from user**
   - If ethresear.ch URL provided, use it directly
   - If EIP number mentioned, search ethresear.ch or use eips.ethereum.org
   - If proposal text provided, analyze directly

2. **Fetch proposal content**
   ```python
   from utils.proposal_fetcher import fetch_ethresearch_proposal, generate_proposal_summary

   proposal = fetch_ethresearch_proposal(url)
   summary = generate_proposal_summary(proposal)
   ```

3. **Present proposal summary to user**
   - Show title, author, date, tags
   - Display content preview
   - List referenced EIPs
   - Highlight key community feedback

4. **Identify proposal category**
   - Protocol change (consensus, execution, networking)
   - Smart contract standard (ERC)
   - DeFi mechanism
   - Security/cryptography improvement
   - Developer tooling
   - Economic mechanism

**Output:** Structured understanding of the proposal with category classification.

---

### Stage 2: Technical Analysis

**Objective:** Analyze technical implications across multiple dimensions.

**Analysis Dimensions:**

#### 2.1 Protocol Layer Impact

Evaluate changes to Ethereum protocol layers:

- **Consensus Layer**
  - Does it change consensus mechanism?
  - Impact on validators/staking?
  - Fork choice modifications?

- **Execution Layer**
  - New opcodes or precompiles?
  - EVM behavior changes?
  - Gas cost model updates?
  - State management changes?

- **Networking Layer**
  - P2P protocol changes?
  - Transaction propagation?
  - Block gossip modifications?

**Scoring:** Rate 1-5 for each:
- 1: No impact
- 2: Minor impact, easy adjustments
- 3: Moderate impact, significant work
- 4: High impact, major refactoring
- 5: Critical impact, fundamental redesign

#### 2.2 Smart Contract Impact

Analyze impact on smart contracts:

- **Compatibility**
  - Backwards compatible with existing contracts?
  - Requires contract upgrades?
  - New security considerations?

- **Development Patterns**
  - New patterns enabled?
  - Deprecated patterns?
  - Best practices changes?

- **Gas Costs**
  - Operations cheaper/more expensive?
  - Overall contract cost impact?

#### 2.3 Implementation Complexity

Assess implementation difficulty:

- **Client Implementation** (Geth, Nethermind, Besu, etc.)
  - Lines of code estimate
  - Complexity level (low/medium/high)
  - Testing requirements

- **Smart Contract Migration**
  - Number of contracts affected
  - Migration effort per contract
  - Risk level

- **Tooling Updates**
  - Compilers (solc, vyper)
  - Frameworks (Foundry, Hardhat)
  - Testing tools
  - Block explorers

#### 2.4 Security Analysis

Identify security implications:

- **New Attack Vectors**
  - Novel vulnerabilities introduced?
  - Known attack patterns affected?

- **Security Model Changes**
  - Trust assumptions modified?
  - Cryptographic primitives changed?
  - Access control implications?

- **Risk Assessment**
  - Critical security risks?
  - Mitigation strategies?

**Commands for Analysis:**

```bash
# If proposal includes contract code, analyze it
# Use existing ethereum-contract-analyzer skill

# Check gas costs if applicable
cast estimate <FUNCTION_CALL> --rpc-url $ETH_RPC_URL

# Review related EIPs
curl https://eips.ethereum.org/EIPS/eip-<NUMBER>
```

**Output:** Technical analysis report with scores (1-5) and detailed explanations for each dimension.

---

### Stage 3: Impact Assessment

**Objective:** Evaluate benefits, risks, and ecosystem impact.

#### 3.1 Improvement Opportunities

Identify positive impacts:

- **Performance**
  - TPS increase?
  - Latency reduction?
  - Block time improvements?

- **Security**
  - Attack surface reduction?
  - Stronger cryptography?
  - Better access control?

- **User Experience**
  - Simpler transactions?
  - Lower costs?
  - Better privacy?

- **Developer Experience**
  - Easier development?
  - Better tooling?
  - Clearer patterns?

- **New Capabilities**
  - Previously impossible features?
  - Better interoperability?
  - Enhanced functionality?

#### 3.2 Risk Analysis

Identify challenges and risks:

- **Backwards Compatibility**
  - Breaking changes?
  - Migration path complexity?
  - Legacy system support?

- **Adoption Barriers**
  - Technical complexity?
  - Economic costs?
  - Coordination requirements?

- **Security Risks**
  - Vulnerabilities in proposal?
  - Implementation risks?
  - Economic attack vectors?

- **Centralization Concerns**
  - Does it favor certain actors?
  - Trust assumptions?
  - Censorship resistance impact?

#### 3.3 Quantitative Metrics

Provide concrete numbers where possible:

- **Gas Cost Changes:** +/- X%
- **Performance:** X TPS improvement
- **Storage:** +/- X bytes per transaction
- **Network:** +/- X% bandwidth usage

**Use existing skills for deep dives:**

```
# For DeFi-related proposals
→ Chain to ethereum-defi-analyzer

# For contract security
→ Chain to ethereum-security-auditor

# For transaction analysis
→ Chain to ethereum-transaction-inspector
```

**Output:** Impact assessment matrix with opportunities, risks, and quantified metrics.

---

### Stage 4: Technical Debt Analysis

**Objective:** Calculate the total cost of adoption.

#### 4.1 Code Migration Debt

**Estimate work required:**

- **Smart Contracts**
  - Number of contracts to update: X
  - Hours per contract: Y
  - Total: X × Y hours

- **Backend Services**
  - API changes: Z hours
  - Database migrations: W hours

- **Frontend/Wallets**
  - UI updates: P hours
  - Library updates: Q hours

**Complexity Score:** 1-5 (aggregate of all changes)

#### 4.2 Testing Debt

**Testing requirements:**

- **Unit Tests:** X new tests, Y hours
- **Integration Tests:** A new scenarios, B hours
- **Testnet Deployment:** C hours
- **Audit Requirements:** D hours (if security-critical)

#### 4.3 Documentation Debt

**Documentation updates:**

- **Technical Specs:** X hours
- **User Guides:** Y hours
- **API Docs:** Z hours
- **Tutorials/Examples:** W hours

#### 4.4 Operational Debt

**Ongoing maintenance:**

- **Monitoring:** New alerts, dashboards
- **Infrastructure:** Server upgrades, new services
- **Incident Response:** Updated runbooks

#### 4.5 Dependency Debt

**External dependencies:**

- Library updates required
- Breaking changes in dependencies
- Version compatibility issues

**Debt Estimation Table:**

| Debt Type | Short-term (0-3mo) | Mid-term (3-6mo) | Long-term (6mo+) |
|-----------|-------------------|------------------|------------------|
| Code Migration | X hours | Y hours | Z hours/month |
| Testing | A hours | B hours | C hours/month |
| Documentation | D hours | E hours | F hours/month |
| Operations | G hours | H hours | I hours/month |
| **TOTAL** | **sum** | **sum** | **sum/month** |

**Output:** Detailed technical debt inventory with time/cost estimates.

---

### Stage 5: Implementation Roadmap

**Objective:** Provide actionable adoption strategy.

#### 5.1 Adoption Strategy Recommendation

Based on analysis, recommend:

- **Full Adoption:** Implement all features (high value, acceptable debt)
- **Partial Adoption:** Cherry-pick beneficial components
- **Wait and See:** Monitor development, defer decision
- **Reject:** Incompatible with project goals, too much debt

#### 5.2 Phased Implementation Plan

**Phase 1: Preparation (Week 1-2)**
- [ ] Team training on proposal concepts
- [ ] Proof-of-concept development
- [ ] Risk assessment validation
- [ ] Stakeholder alignment

**Phase 2: Development (Week 3-6)**
- [ ] Core implementation
- [ ] Testing infrastructure
- [ ] Code review and refactoring
- [ ] Documentation drafting

**Phase 3: Testing (Week 7-8)**
- [ ] Unit and integration tests
- [ ] Testnet deployment
- [ ] Security audit (if needed)
- [ ] Performance benchmarking

**Phase 4: Deployment (Week 9-10)**
- [ ] Mainnet deployment
- [ ] Monitoring setup
- [ ] Gradual rollout
- [ ] Documentation finalization

**Phase 5: Maintenance (Ongoing)**
- [ ] Bug fixes and optimizations
- [ ] Community feedback integration
- [ ] Technical debt paydown

#### 5.3 Dependencies & Blockers

Identify prerequisites:
- Required EIP implementations
- Network upgrade schedules
- Third-party tool availability
- Community consensus

#### 5.4 Resource Requirements

Estimate resources needed:
- **Development:** X engineer-hours (by specialty)
- **Infrastructure:** $Y/month
- **Audit/Security:** $Z one-time
- **Training:** W hours

#### 5.5 Risk Mitigation

Plan for issues:
- Fallback mechanisms
- Rollback procedures
- Feature flags for gradual rollout
- Compatibility layers

**Output:** Comprehensive implementation roadmap with timeline and resources.

---

## Final Report Generation

### Report Structure

Generate a comprehensive markdown report with these sections:

**1. Executive Summary**
- 2-3 paragraph overview
- Key findings (bullet points)
- Final recommendation: ADOPT / WAIT / REJECT
- Critical decision factors

**2. Proposal Details**
- Title, author, date, URL
- Current status
- Referenced EIPs
- Community sentiment

**3. Technical Analysis**
- Protocol layer impacts (with scores)
- Smart contract implications
- Security considerations
- Implementation complexity

**4. Impact Assessment Matrix**

| Category | Opportunities | Risks | Net Impact |
|----------|--------------|-------|-----------|
| Performance | [list] | [list] | +/neutral/- |
| Security | [list] | [list] | +/neutral/- |
| UX | [list] | [list] | +/neutral/- |
| DX | [list] | [list] | +/neutral/- |
| Economics | [list] | [list] | +/neutral/- |

**5. Technical Debt Analysis**
- Debt inventory table
- Effort estimates
- Priority ranking

**6. Implementation Roadmap**
- Adoption strategy
- Phased plan with milestones
- Resource requirements

**7. Decision Framework**

| Dimension | Score (1-5) | Weight | Weighted Score |
|-----------|-------------|--------|----------------|
| Technical Feasibility | X | 25% | X |
| Value Proposition | X | 30% | X |
| Risk Level (inverted) | X | 20% | X |
| Technical Debt (inverted) | X | 15% | X |
| Ecosystem Alignment | X | 10% | X |
| **TOTAL** | - | 100% | **X.X/5** |

**Decision Criteria:**
- Score ≥ 4.0: Strong adoption recommendation
- Score 3.0-3.9: Conditional adoption
- Score 2.0-2.9: Wait and monitor
- Score < 2.0: Reject or deprioritize

**8. References & Resources**
- Original proposal link
- Related EIP links
- Code repositories
- Community discussions

### Save Report

Save the final report to:
```
<proposal-name>_Research_Report.md
```

---

## Error Handling

**Invalid URL:**
- Verify URL format
- Offer manual input mode
- Ask user to provide proposal text

**Proposal Too Early/Conceptual:**
- Tag as "Conceptual Stage"
- Reduce quantitative analysis depth
- Focus on theoretical impacts
- Mark uncertainty clearly

**Insufficient Technical Details:**
- Request clarification from user
- Infer from similar proposals (with disclaimer)
- Focus on high-level analysis

**Network/Fetch Errors:**
- Retry with timeout
- Fallback to manual input
- Cache previously fetched content

---

## Integration with Other Skills

**Chain to related skills as needed:**

- **@ethereum-contract-analyzer** - For contract code in proposals
- **@ethereum-security-auditor** - For security deep dives
- **@ethereum-defi-analyzer** - For DeFi protocol impacts
- **@ethereum-transaction-inspector** - For transaction examples
- **@ethereum-chain-explorer** - For on-chain data analysis

**Example Chaining:**
```
User: "Analyze Lucid encrypted mempool proposal"
  1. ethereum-proposal-researcher: Fetch and categorize
  2. Identify MEV-related contracts
     → ethereum-security-auditor: Analyze encryption security
  3. Identify transaction ordering changes
     → ethereum-chain-explorer: Check current mempool patterns
  4. Reference DEX examples
     → ethereum-defi-analyzer: Analyze DEX impact
  5. Generate comprehensive report
```

---

## Tips for Effective Analysis

1. **Start with context** - Understand problem proposal solves
2. **Be thorough** - Cover all analysis dimensions
3. **Quantify where possible** - Concrete numbers > vague statements
4. **Consider stakeholders** - Impact on users, developers, validators
5. **Mark uncertainty** - If data is missing or inferred, say so
6. **Be practical** - Focus on actionable recommendations
7. **Reference existing patterns** - Compare to known EIPs/proposals

---

## Resources

- **ethresear.ch:** https://ethresear.ch
- **EIPs:** https://eips.ethereum.org
- **Ethereum Docs:** https://ethereum.org/developers
- **Foundry:** https://book.getfoundry.sh
