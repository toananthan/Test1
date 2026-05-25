# GitHub Copilot Agentic AI Setup for Event Processing Microservices

**Author**: Engineering Lead  
**Team**: Microservices Platform (Financial Firm)  
**Last Updated**: May 2026

## 1. Overview

This document provides a complete guide for using **GitHub Copilot Agentic AI** (Skills + Agents + MCP) to accelerate our repetitive work with Spring Integration Aggregators, Kafka event processing, Avro schemas, Spring Cloud Contracts, and related tools.

### Tech Context
- **Core Stack**: Confluent Kafka, Pivotal Cloud Foundry, Spring Boot, Spring Integration Aggregator, Oracle DB, Spring Cloud Contract
- **Internal Tools**: Megatron, Skywalker, Splunk, Event Store
- **Recurring Tasks**: New aggregators, schema updates, contract tests, Spring Boot upgrades, production analysis

---

## 2. Folder Structure

```bash
.github/
├── skills/
│   ├── create-new-aggregator/
│   │   └── SKILL.md
│   ├── update-avro-schema-mapping/
│   │   └── SKILL.md
│   ├── generate-spring-cloud-contract-tests/
│   │   └── SKILL.md
│   ├── splunk-query-generator/
│   │   └── SKILL.md
│   ├── spring-boot-upgrade-assistant/
│   │   └── SKILL.md
│   ├── jira-ticket-generator/
│   │   └── SKILL.md
│   └── megatron-test-payload-creator/
│       └── SKILL.md
│
├── agents/
│   └── event-processing-agent.md
│
└── mcp/
    └── mcp-config.json

# Optional
templates/
├── aggregator/
├── contract-tests/
└── megatron-payloads/


3. Reusable Skills
Skill 1: create-new-aggregator
Path: .github/skills/create-new-aggregator/SKILL.md

---
name: create-new-aggregator
description: Creates a complete new Spring Integration Aggregator following our financial microservices standards.
version: 1.1
tags: [spring-integration, aggregator, kafka, avro]
---

# Create New Spring Integration Aggregator

You are a senior Spring Integration engineer.

**Mandatory Rules**:
- Correlation key: usually `applicationNumber`
- Timeout: configurable per LOB (default 10 mins)
- Always generate Pend, Success, Failure, Error outbound events
- Include deduplication, logging, Oracle persistence, and Kafka publishing

**Execution**:
1. Ask for LOB, timeout, inbound/outbound schemas if missing
2. Generate: AggregatorConfig, ReleaseStrategy, Mapper, Contract tests
3. Follow our naming conventions

Skill 2: update-avro-schema-mapping
Path: .github/skills/update-avro-schema-mapping/SKILL.md

---
name: update-avro-schema-mapping
description: Updates field mappings and fixes contract tests when Avro schemas change.
version: 1.0
tags: [avro, mapping, schema]
---

# Update Avro Schema Mapping

Handle inbound → outbound mapping updates and regenerate affected Spring Cloud Contract tests.

Skill 3: generate-spring-cloud-contract-tests
Path: .github/skills/generate-spring-cloud-contract-tests/SKILL.md

---
name: generate-spring-cloud-contract-tests
description: Generates Spring Cloud Contract tests for Pend/Success/Failure/Error paths.
version: 1.0
tags: [spring-cloud-contract, testing]
---

# Generate Spring Cloud Contract Tests

Always create tests for all 4 outbound event paths.

Skill 4: splunk-query-generator
Path: .github/skills/splunk-query-generator/SKILL.md

---
name: splunk-query-generator
description: Generates production-ready Splunk queries for inbound/outbound event analysis.
version: 1.0
tags: [splunk, monitoring]
---

# Splunk Query Generator

**Rules**:
- Include time range
- Support analysis by LOB, applicationNumber, status
- Return copy-paste ready queries with explanations

Skill 5: spring-boot-upgrade-assistant
Path: .github/skills/spring-boot-upgrade-assistant/SKILL.md

---
name: spring-boot-upgrade-assistant
description: Guides safe Spring Boot upgrades considering our aggregator and Kafka setup.
version: 1.0
tags: [spring-boot, upgrade]
---

# Spring Boot Upgrade Assistant

Provide pom.xml updates, breaking change analysis, and testing recommendations.

Skill 6: jira-ticket-generator (New)
Path: .github/skills/jira-ticket-generator/SKILL.md

---
name: jira-ticket-generator
description: Creates well-structured Jira tickets for our microservices work.
version: 1.0
tags: [jira, ticket, planning]
---

# Jira Ticket Generator

You are an expert at writing clear Jira tickets for our platform team.

## Required Fields
- Summary (concise)
- Description (detailed with acceptance criteria)
- Technical Details (tech stack, impacted components)
- Testing Steps (Megatron, Skywalker, Contract tests)
- Links to Confluence architecture docs

## When to Use
- New aggregator requests
- Schema changes
- Spring Boot upgrades
- Production issues

## Output Format
Return in Jira markdown format with clear sections: Description, Acceptance Criteria, Technical Notes, Test Plan.

Skill 7: megatron-test-payload-creator (New)
Path: .github/skills/megatron-test-payload-creator/SKILL.md

---
name: megatron-test-payload-creator
description: Creates realistic test payloads for Megatron to test inbound events.
version: 1.0
tags: [megatron, testing, payload]
---

# Megatron Test Payload Creator

You are an expert in creating test events for our financial microservices.

## Rules
- Generate valid Avro-compatible JSON payloads
- Include mandatory fields
- Support different scenarios: Success path, Pend, Failure, Error cases
- Include variations for different LOBs
- Add edge cases (missing fields, duplicates, unsupported products)

## Output
- Full JSON payload ready for Megatron
- Explanation of the test scenario
- Expected outbound behavior

4. Master Agent
Path: .github/agents/event-processing-agent.md

---
name: event-processing-agent
description: Master agent for all event processing and aggregator related work.
version: 1.0
skills: [create-new-aggregator, update-avro-schema-mapping, generate-spring-cloud-contract-tests, splunk-query-generator, spring-boot-upgrade-assistant, jira-ticket-generator, megatron-test-payload-creator]
---

# Event Processing Master Agent

Use appropriate skills automatically based on user request.
Chain tasks when needed (e.g., new aggregator → create ticket → generate test payload).

5. Team Guidelines for Adoption
File: .github/AGENTIC-AI-GUIDELINES.md

# Team Guidelines - GitHub Copilot Agentic AI

## Core Principles
- Use skills first before writing code from scratch
- Always review AI output before committing
- Improve skills collaboratively

## Best Practices
- Be specific in prompts (LOB, timeout, schema names, etc.)
- Use the Master Agent `@event-processing-agent` for complex tasks
- Test all generated code with Megatron + Contract tests
- Create a Jira ticket using the skill for every major change

## MCP Setup
Maintain `.github/mcp/mcp-config.json` with links to Confluence, Splunk, and internal tools.

## Continuous Improvement
- Review skills every sprint
- Share new useful prompts and improvements in team channel

