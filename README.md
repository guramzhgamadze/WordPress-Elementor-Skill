# 🚀 WordPress & Elementor Pro Development Standards

Welcome! This repository serves as a centralized knowledge base and procedural guide for writing production-grade WordPress and Elementor Pro code. 

Whether you're scaffolding a new custom plugin, building advanced Elementor widgets, or optimizing a site's frontend performance, these documents provide the modern practices, strict standards, and compatibility rules needed to keep your codebase clean, secure, and highly maintainable. 

The goal here isn't just to make things work, but to build robust, SEO-friendly, and performant projects that scale beautifully.

## 📦 Installation Guide (Adding as an AI Skill)

This repository is designed to be ingested by Claude (or similar AI platforms) to help it generate perfectly formatted, standardized code for you. To add this as a custom Skill, follow these exact steps:

1. **Package it:** Download or clone this entire repository to your local machine, then compress the main folder into a single `.zip` file.
2. **Start the Process:** Log into your AI dashboard, navigate to the **customize/Skills** menu, and click the **Upload a skill** button. drag and drop your `.zip` file and it's done.
## 🗂️ What's Inside

The documentation is split into focused, easy-to-digest guides. Start with the core philosophy in the Skill Router, then dive into the specific topics you need for your current task:

* **[`SKILL.md`](./SKILL.md) — The Skill Router:** The main entry point. Start here to understand the default assumptions, golden rules, and core philosophy of the repository.
* **[`scaffolding.md`](./scaffolding.md) — Plugin & Theme Structure:** Best practices for code placement, organizing child themes, registering Custom Post Types (CPTs), and setting up AJAX handlers.
* **[`php-standards.md`](./php-standards.md) — PHP Best Practices:** The non-negotiables of backend development. Covers data sanitization, escaping, nonce verification, error handling with `WP_Error`, and transient management.
* **[`js-css-standards.md`](./js-css-standards.md) — Frontend Standards:** Guidelines for modern ES6+ JavaScript, leveraging native WordPress enqueue APIs, BEM methodology for CSS, and targeting Elementor's specific DOM structure.
* **[`elementor-patterns.md`](./elementor-patterns.md) — Elementor Pro Integration:** Rules for ensuring V4 compatibility, creating Custom Widgets, building Dynamic Tags, setting up Loop Grids, and managing Theme Builder conditions.
* **[`woocommerce.md`](./woocommerce.md) — WooCommerce Mastery:** Everything you need for modern eCommerce, including High-Performance Order Storage (HPOS) compatibility, working with Order APIs, and overriding templates safely.
* **[`rest-api.md`](./rest-api.md) — Custom Endpoints:** How to build secure REST API endpoints, validate schemas, and properly handle permission callbacks.
* **[`offcanvas-ui.md`](./offcanvas-ui.md) — Accessible UI Components:** Patterns for building accessible off-canvas menus and popups, including focus traps, scroll locks, and proper ARIA attributes.
* **[`performance.md`](./performance.md) — Speed & Accessibility:** Checklists for both frontend and backend performance, speculative loading techniques, and hitting WCAG 2.2 AA accessibility standards.

## 🌟 Our Core Philosophy

If you take nothing else away from this repository, keep these four principles in mind whenever you write a line of code:

1. **Native APIs First:** Always look to WordPress core hooks before writing custom plugin logic, and utilize Elementor's built-in APIs before overriding templates manually. Don't reinvent the wheel.
2. **Sanitize In, Escape Out:** Trust absolutely no one. Every piece of input must be sanitized the moment it hits the server. Every output must be escaped exactly where it is rendered to the screen. No exceptions.
3. **Prefix Everything:** The WordPress ecosystem is crowded. Consistently prefix all of your functions, classes, constants, hooks, and CSS classes to prevent nasty conflicts.
4. **Know Your State Placement:** Be intentional about where your code lives. Does this belong in the child theme's `functions.php`, a standalone utility plugin, or an Elementor Custom Code snippet? Plan before you code.

## ⚙️ Environment Assumptions

Unless a specific guide notes otherwise, the patterns in this repo assume you are building for a modern stack:

* **WordPress:** 6.9.3+ (Built with an eye toward WP 7.0).
* **PHP:** 8.3+ (We love typed class constants and readonly classes!).
* **Elementor:** Core 3.35.7+ / Pro 3.35.1+ (Using the V3 `Widget_Base` API and prepared for V4 DOM changes).
* **WooCommerce:** HPOS is enabled by default (WC 8.2+).

## 🛠️ How to Use This Repo (Day-to-Day)

Once you have this installed as a skill in Claude:

1. **Ask for what you need:** Tell Claude what you are trying to build (e.g., "Build me a custom Elementor widget that pulls in WooCommerce products").
2. **Let Claude check the docs:** Claude will automatically consult the `SKILL.md` router and the relevant sub-files to ensure the output matches our strict guidelines.
3. **Review and implement:** Drop the generated, production-ready code straight into your WordPress environment!