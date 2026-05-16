# Ring deployment
- [What is ring deployment?](#what-is-ring-deployment)
- [Why use ring deployment?](#why-use-ring-deployment)
- [Key Concepts of Ring Deployment](#key-concepts-of-ring-deployment)
- [Ring deployment vs Blue-Green deployment](#ring-deployment-vs-blue-green-deployment)
- [Best practices for ring deployment](#best-practices-for-ring-deployment)
- [Reference](#reference)   

## What is ring deployment

Ring deployment is a gradual software release strategy where a new version is rolled out to different regions and environments in a specific order (rings), allowing for progressively wider exposure.
Usually applied in distributed microservices architectures, this approach helps minimize the blast radius of potential issues.

## Why use ring deployment

- Reduce risk
- Feedback and validation
- Fully control rollout order and strategy

## Key Concepts of Ring Deployment

1. **Rings (or Stages)**: Regions and environments are divided into rings, with each ring representing a sequential stage of the rollout.
For example, the first ring might be the developer environment and a QA environment; the second ring could be an automation environment for end-to-end tests; the third ring might be an integration environment for a small group of external users; and the final ring is the full production environment.
2. **Gradual Rollout**: Unlike a "big bang" deployment where the new version is released to all users simultaneously, ring deployment releases the update incrementally.
Between rings, automated gates (or manual approvals) decide whether to proceed based on health metrics and test results.
This controlled exposure allows teams to monitor application health and gather feedback before committing to a full-scale release.
3. **Rollback Capability**: A critical component of ring deployment is the ability to quickly roll back to the previous stable version if issues are detected.
Since changes are introduced in stages, rolling back only affects users in the current ring, making it far less disruptive than a full-scale rollback.
4. **Automated Testing**: Before promoting a service to the next ring, you should run a suite of tests that guarantees the change hasn't introduced regressions.
Recommended tests include: unit, functional, smoke, integration, end-to-end, and contract tests.

## Ring deployment vs Blue-Green deployment

While both strategies aim to minimize downtime, they differ in traffic management.
Blue-Green deployment maintains two identical environments (Blue and Green) and switches 100% of traffic at once.
Ring deployment is more granular, rolling out to subsets of environments over time.
While Blue-Green can be resource-intensive (requiring double the infrastructure), Ring deployment often leverages existing multi-region/multi-environment setups to transition versions progressively.

## Best practices for ring deployment

1. **Automation** - Use automated end-to-end tests and health checks. If a test fails, the rollout should stop immediately.
2. **Quick Rollback** - Ensure the process for reverting to a previous version is fast and reliable, Support manual overrides for hotfixes when necessary.
3. **Continuous Monitoring** - Closely monitor performance, error rates, and latency as the update reaches each new ring.
4. **Communication** - Keep stakeholders and users informed about the rollout status and potential impacts.
5. **Documentation** - Maintain records of the rollout schedule, metrics monitored, and post-mortem notes for any issues encountered.

## Reference

- [MS Ring Deployment](https://microsoft.github.io/PowerApps-TestEngine/context/ring-deployment-model/)
- [Atlassian Ring Deployment](https://blog.atlan.com/engineering/ring-deployment-releases/)
- [Blue-Green Deployment](https://en.wikipedia.org/wiki/Blue%E2%80%93green_deployment)
