# Contribution 2: Support Redis ACL Authentication with Username

**Contribution Number:** 2  
**Student:** Srikanth Medicherla  
**Issue:** https://github.com/graphql-hive/console/issues/8140  
**Status:** Phase I In Progress

---

## Why I Chose This Issue

[1-2 paragraphs explaining why this issue interests you, how it matches your skills/learning goals, what you hope to learn]

I chose this issue because it closely aligns with the type of backend configuration and infrastructure work I have done professionally. The issue involves tracing configuration through multiple services, updating environment validation, and ensuring backward compatibility across a distributed TypeScript codebase. These are skills that I regularly use and would like to strengthen through open source contributions.

I was specifically looking for an issue that solves a real user problem while remaining approachable for a first contribution. The prior selection was too advanced and no one was responding so I choose one that is 3 days old and matches my experience. This issue improves compatibility for organizations using Redis ACL authentication and has a clearly defined scope with a proposed implementation plan. It also provides an opportunity to learn how GraphQL Hive organizes service configuration and manages shared infrastructure concerns across a monorepo. I really liked the open issue as there was enough information for me to do the work needed


---

## Understanding the Issue

### Problem Description

[In your own words, what's broken or missing?]

GraphQL Hive currently supports Redis password authentication but does not support providing a Redis username. This becomes a problem for Redis ACL deployments, where both a username and password are typically required for authentication. Organizations using Redis ACLs currently need to maintain custom modifications or internal forks to support this functionality.

### Expected Behavior

[What should happen?]

Users should be able to optionally provide a `REDIS_USERNAME` environment variable alongside the existing Redis password configuration. When supplied, GraphQL Hive should pass both the username and password to the Redis client while preserving existing behavior for deployments that only use password authentication.

### Current Behavior

[What actually happens?]

The current implementation only exposes Redis password configuration. There is no mechanism to provide a Redis username, which prevents GraphQL Hive from working correctly with some ACL-enabled Redis deployments.

### Affected Components

[Which parts of the codebase are involved?]

Based on the issue description, the affected components include:

- Redis environment configuration models
- Service environment validation
- Redis client initialization logic
- Environment template files
- Shared Redis configuration objects

Likely affected files are included in the issue itself, which I copied here:

- `packages/services/api/src/modules/shared/providers/redis.ts`
- `packages/services/server/src/environment.ts`
- `packages/services/schema/src/environment.ts`
- `packages/services/tokens/src/environment.ts`
- `packages/services/usage/src/environment.ts`
- `packages/services/schema/src/index.ts`
- `packages/services/tokens/src/index.ts`
- `packages/services/usage/src/index.ts`
- Service `.env.template` files

- 
---

## Reproduction Process

### Environment Setup

[Notes on setting up your local development environment - challenges you faced, how you solved them]

I set up the GraphQL Hive repository locally on macOS using the project's documented setup process. The repository uses a pnpm monorepo and Docker-based local infrastructure. I used AI to run the project and iterate as problems came.

Commands used:

```bash
pnpm install
pnpm local:setup
pnpm generate
pnpm build
```

Setup challenges encountered:

- My local shell was using Node v20, while the repository requires Node v24.14.1.
- Docker Desktop was not running initially.
- The root `.env` file was missing and needed to be created with `ENVIRONMENT=local`.
- `pnpm generate` initially failed due to local Postgres access restrictions.
- `pnpm build` initially failed due to a local IPC permission issue.

After resolving these issues:

- Docker services started successfully.
- Postgres and ClickHouse migrations completed.
- `pnpm generate` completed successfully.
- `pnpm build` completed successfully (22/22 tasks passed).

A separate runtime dependency issue (`fakePromise` export error) prevented full `pnpm dev:hive` startup, but it was unrelated to Redis and did not block investigation of issue #8140.

### Steps to Reproduce

1. Confirm Redis is running locally:

```bash
docker exec hive-dev-redis-1 redis-cli PING
```

Observed result:

```text
PONG
```


2. Create a Redis ACL user and disable the default user:

```bash
docker exec hive-dev-redis-1 redis-cli ACL SETUSER hive on '>hivepass' allcommands allkeys
docker exec hive-dev-redis-1 redis-cli ACL SETUSER default off
```

3. Verify username/password authentication succeeds:

```bash
docker exec hive-dev-redis-1 redis-cli -u redis://hive:hivepass@localhost:6379 PING
```

Observed result:

```text
PONG
```

4. Verify password-only authentication fails:

```bash
docker exec hive-dev-redis-1 redis-cli -a hivepass PING
```

Observed result:

```text
AUTH failed: ERR AUTH <password> called without any password configured for the default user.
NOAUTH Authentication required.
```

5. Trace the Redis configuration flow in the GraphQL Hive codebase.

Observed result:

- `REDIS_PASSWORD` is supported.
- `REDIS_USERNAME` does not exist.
- Redis client constructors do not receive a `username` option.

### Reproduction Evidence

- **Branch:** `fix-issue-8140`
- **Screenshots/logs:** Redis CLI output shown above.
- **My findings:** Redis ACL deployments can require both a username and password. Local testing confirmed that username/password authentication succeeds while password-only authentication fails when the default Redis user is disabled. GraphQL Hive currently has no `REDIS_USERNAME` environment variable or configuration path, so Redis usernames cannot be passed into `ioredis`.

---

## Solution Approach

### Analysis

GraphQL Hive local setup completed through `pnpm install`, `pnpm local:setup`, `pnpm generate`, and `pnpm build`. The full `pnpm dev:hive` startup is currently blocked by an unrelated dependency/runtime error:

```text
SyntaxError: The requested module '@graphql-tools/utils' does not provide an export named 'fakePromise'
```

This does not prevent investigating issue `#8140`, because the Redis ACL behavior can be reproduced independently.

Redis ACL reproduction succeeded: a named Redis user `hive` with password `hivepass` could authenticate successfully, while password-only/default-user authentication failed after disabling the default Redis user. This confirms the issue: Redis ACL deployments require both username and password.

In the codebase, `REDIS_PASSWORD` exists across service environment models and Redis config objects, but `REDIS_USERNAME` is missing from `packages/services`. Redis client constructors currently pass `host`, `port`, `password`, and TLS options, but do not pass `username`.


### Proposed Solution

Add support for Redis ACL authentication by allowing GraphQL Hive services to optionally configure and pass a Redis username in addition to a password. The implementation will extend the existing Redis configuration flow while maintaining backward compatibility so that current password-only Redis deployments continue to work without any changes.


### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:**

GraphQL Hive needs to support Redis ACL authentication for deployments where Redis disables the default user and requires clients to authenticate with both username and password.

**Match:**

The project already has a consistent Redis configuration pattern across services. I will extend that pattern by adding optional `REDIS_USERNAME` support wherever Redis config is parsed, resolved, documented, and passed into `ioredis`.

**Plan:**

1. Work on branch `fix-issue-8140`.
2. Review `docs/DEVELOPMENT.md` and `docs/TESTING.md` before implementation.
3. Add optional `REDIS_USERNAME` to affected Redis environment models.
4. Add `username` to resolved Redis config objects.
5. Pass `username` into shared Redis client helpers and direct `new Redis(...)` constructors.
6. Update relevant `.env.template` files.
7. Review affected service README files and update documentation if the new environment variable needs to be exposed to users.
8. Review self-host/deployment configuration surfaces to determine whether additional propagation is required.
9. Add a changeset if required by the project's contribution process.
10. Add or update tests for optional username configuration and Redis client option propagation where practical.


**Implement:**

Implementation will be limited to Redis configuration support for issue `#8140`. I will avoid unrelated dependency fixes, including the unrelated `fakePromise` startup blocker.


**Review:**

- [ ] Follows existing Redis configuration patterns
- [ ] Keeps `REDIS_USERNAME` optional
- [ ] Preserves backward compatibility for password-only Redis deployments
- [ ] Updates all affected Redis config paths consistently
- [ ] Updates `.env.template` files
- [ ] Updates service README documentation where required
- [ ] Adds a changeset if required
- [ ] Avoids unrelated source changes
- [ ] Includes reviewer notes explaining the Redis ACL reproduction



**Evaluate:** 

I will verify the fix by running the project's recommended validation commands:

```bash
pnpm build
pnpm test
pnpm typecheck
pnpm lint
pnpm lint:env-template
pnpm lint:prettier
```

I will also re-run the Redis ACL reproduction scenario to confirm:

- Named Redis user + password authentication succeeds
- Password-only authentication fails when the default user is disabled
- Hive can pass both `REDIS_USERNAME` and `REDIS_PASSWORD` into Redis
- Existing password-only Redis deployments continue working when `REDIS_USERNAME` is unset

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]


# Contribution 1: Filter changes from Slack notifications

**Contribution Number:** 1
**Student:** Srikanth Medicherla
**Issue:** https://github.com/graphql-hive/console/issues/3730 
**Status:** Phase I Complete

---

## Why I Chose This Issue

[1-2 paragraphs explaining why this issue interests you, how it matches your skills/learning goals, what you hope to learn]

I chose this issue since it aligns with my experience working with Typescript and Slack notifications in a professional environment. This request is clearly defined, the scope is clear, and it also deals with real user workflow. It also matches my current skill level and gives me an oppurtunity to learn from open source Typescript codebases. I was looking for a backend or product focused issue with an active maintainer with clear instructions on what the definition of done is. GraphQL Hive has great documentation, an active community, and the feature is an excellent choice for my first contribution since I worked on something very similar at my work. 

---

## Understanding the Issue

### Problem Description

[In your own words, what's broken or missing?]

The current Slack integration sends notifications for all supported schema changes, including description-only changes. Some teams use these notifications to monitor important schema updates such as breaking or dangerous changes, but are not interested in description changes. Currently, there is no way to filter which change types are included in Slack notifications.

### Expected Behavior

[What should happen?]

Users then should be able to filter out specific type of slack notifications so there is less noise and they can focus on the notifications that are important to them.

### Current Behavior

[What actually happens?]

Right now the users are not able to filter slack notifications. They get all of the notifications including the description changes.

### Affected Components

[Which parts of the codebase are involved?]

Affected components include slack integration, notification generation logic, and also any configs or settings that are used to determine which schema changes are included in outgoing notifications.

---

## Reproduction Process

### Environment Setup

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
