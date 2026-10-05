# Choosing React and JavaScript test layers

## Pure unit tests

Use for deterministic transformations, parsers, schemas, reducers, selectors, formatters, and policies. Assert exact results/invariants and boundaries. Do not render React for pure logic.

Not proved: component wiring, accessibility markup, user interaction, providers, browser APIs, or network integration.

## Component tests with Testing Library

Use for rendering, forms, accessible interaction, state transitions, conditional UI, and request-driven states. Render with the same essential providers as production using fresh instances per test.

Interact with `userEvent`; observe the DOM/accessibility surface. Prefer network interception so the real component, hooks, query/cache library, client, normalization, and error mapping participate.

Good proof: component-level user workflow and request/UI contract under the test environment. Not proved: actual browser layout, native browser validation, downloads, service worker behavior, or full deployment.

## Hook tests

Test a hook directly when it is a reusable library/public unit with a meaningful API. Product-specific hooks are often better tested through the component that gives them user meaning. Do not mock a hook in a component test and claim the integrated behavior works.

## Network/client tests

Exercise the real client/adapter against the project's interception layer. Assert method, URL, relevant headers/body, decoding, error mapping, cancellation, and retry semantics. Mock the server, not the client under test.

For component tests, configure handlers per scenario and fail on unexpected/unhandled requests when supported. A handler must represent the state claimed: success-empty, validation error, permission denial, server failure, delayed/deferred response, and so on.

## Browser/component-browser/E2E tests

Use the existing browser layer for risks JSDOM does not faithfully represent:

- real navigation and cross-page state;
- focus/keyboard behavior dependent on browser implementation;
- layout, overflow, responsive behavior, or visual regression;
- downloads/uploads and native file controls;
- storage/cookies across contexts;
- service workers, SSE/WebSocket, streaming, or browser cancellation;
- critical end-to-end journeys and deployed API compatibility.

Use isolated browser contexts and provision unique server-side data. Prefer role/label locators and web-first assertions. Avoid manual sleeps.

## Accessibility

Semantic queries provide useful accessibility pressure but are not a complete accessibility audit. Assert relevant accessible name, description, role, error association, focus, expanded/selected/checked state, and keyboard interaction when contractual. Use automated accessibility tooling only as a complement.

## Cross-layer changes

When an API contract changes, cover frontend parsing/rendering and at least one real mocked-network payload matching the backend contract. When routing/storage/browser behavior changes, component tests alone are insufficient. State the evidence boundary explicitly.
