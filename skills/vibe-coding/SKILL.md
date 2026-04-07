---
name: vibe-coding
description: A skill for building and shipping products faster based on the 'Vibe Coding 2.0' framework. It provides a set of rules and best practices to avoid common pitfalls and technical debt, enabling developers to be in the top 1% of builders.
license: Complete terms in LICENSE.txt
---

# Vibe Coding 2.0

This skill provides a set of rules and best practices for building and shipping products faster, based on the "Vibe Coding 2.0" framework. The core principle is to focus on what **not** to build, leveraging existing tools and services to avoid technical debt and ship MVPs in weeks, not months.

## Core Principles

*   **The mistake is the decision made before the code.** Every hour spent on solved problems (like auth) is an hour not spent on your core value proposition.
*   **Trust the tools that already exist.** The best builders know what *not* to build.
*   **Ship fast, learn from real users.** Shipped and imperfect beats polished and never launched.

## The 18 Rules (The DOs)

1.  **Use Ready-Made Auth:** Use services like Clerk or Supabase Auth.
2.  **Use Tailwind and shadcn/ui for UI:** Build legitimate-looking UIs quickly and consistently.
3.  **Use Zustand and Server Components for State:** Keep state management simple.
4.  **Use tRPC and Server Actions for APIs:** End-to-end type safety without the boilerplate.
5.  **Deploy with Vercel One-Click:** Automate deployments and avoid manual configuration.
6.  **Use Prisma and Managed Postgres:** A typed ORM and managed database for 95% of MVP needs.
7.  **Validate with Zod and React Hook Form:** Make forms predictable and trustworthy.
8.  **Use Stripe for Payments:** Never build your own payment system.
9.  **Add Sentry or Error Tracking Early:** Know about production errors immediately.
10. **Set Up Analytics From the Beginning:** Use PostHog or Plausible to understand user behavior.
11. **Store Secrets in Environment Files:** Never hardcode API keys.
12. **Use UploadThing or Cloudinary for Files:** Offload file upload complexity.
13. **Set Up Preview Deployments:** Test changes before they hit production.
14. **Use Component Libraries Like Radix and shadcn:** Don't reinvent the wheel for UI components.
15. **Write a README From Day 1:** Document your project for your future self and others.
16. **Keep Folders Clean and Modular:** A clean codebase is a fast codebase.
17. **Add Onboarding and Empty States:** Guide users to value.
18. **Use Lighthouse and Performance Tools:** Performance is a feature, not a nice-to-have.

## The DON'Ts

This skill also provides a list of things to avoid. For a complete list of DOs and DON'Ts, and a more detailed explanation of each point, please refer to the full Vibe Coding 2.0 document.

To access the full document, read the following file:
`/home/ubuntu/skills/vibe-coding/references/vibe_coding_2.0.md`
