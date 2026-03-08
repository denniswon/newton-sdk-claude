# Mintlify Platform Conventions (`site/`)

Rules for working with the Mintlify documentation framework in the `site/` directory.

## MDX File Structure

Every page is an MDX file with YAML frontmatter:

```yaml
---
title: "Page Title"                    # Required
description: "Short page description"  # Recommended
---

Page content in MDX...
```

- `title` is **required** — displayed as the page heading and in navigation
- `description` is **recommended** — used for SEO meta description
- No other frontmatter properties are used in this project

## Components

### CodeGroup — Multi-tab code examples

Use for showing the same concept in multiple languages or package managers:

```mdx
<CodeGroup>

```bash npm
npm install @magicnewton/newton-protocol-sdk
```

```bash pnpm
pnpm add @magicnewton/newton-protocol-sdk
```

</CodeGroup>
```

### Tabs / Tab — Alternative approaches

Use for mutually exclusive options (e.g., "New Contract" vs "Existing Contract"):

```mdx
<Tabs>
  <Tab title="New Contract">
    Content for creating a new contract...
  </Tab>
  <Tab title="Existing Contract">
    Content for integrating with an existing contract...
  </Tab>
</Tabs>
```

### Steps / Step — Sequential procedures

Use for ordered walkthroughs:

```mdx
<Steps>
  <Step title="Install the SDK">
    ```bash
    pnpm add @magicnewton/newton-protocol-sdk
    ```
  </Step>
  <Step title="Create a client">
    ```typescript
    const client = createPublicClient({ ... });
    ```
  </Step>
</Steps>
```

### Accordion / AccordionGroup — Collapsible sections

Use for optional details, long examples, or FAQ items:

```mdx
<AccordionGroup>
  <Accordion title="Full response example">
    ```json
    { "result": { ... } }
    ```
  </Accordion>
</AccordionGroup>
```

### Card — Navigation cards

```mdx
<Card icon="play" href="https://demo.newton.xyz/" title="Try the demo">
  See a live example of a Newton-powered app
</Card>
```

### Callouts — Note, Warning, Info

```mdx
<Note>Informational callout for helpful context.</Note>
<Warning>Warning for potential pitfalls or breaking changes.</Warning>
<Info>Tip or additional context that aids understanding.</Info>
```

### Expandable — Long content blocks

Use for lengthy type definitions or code blocks that would disrupt reading flow:

```mdx
<Expandable title="Full type definition">
  ```typescript
  interface PolicyConfig { ... }
  ```
</Expandable>
```

## docs.json Structure

The navigation is organized as **tabs -> groups -> pages**:

```json
{
  "navigation": {
    "tabs": [
      {
        "tab": "Developers",
        "groups": [
          {
            "group": "Overview",
            "pages": [
              "developers/overview/about",
              "developers/overview/quickstart"
            ]
          }
        ]
      }
    ]
  }
}
```

### Adding a new page

1. Create the MDX file in the appropriate directory under `site/`
2. Add its path (without `.mdx` extension) to the `pages` array in the correct group in `site/docs.json`

### Moving or renaming a page

1. Move/rename the MDX file
2. Update the path in `site/docs.json`
3. Add a redirect in the `redirects` array in `site/docs.json`
4. Update any internal links in other MDX files that reference the old path

### Removing a page

1. Remove the path from `site/docs.json`
2. Add a redirect to the most relevant replacement page
3. Search for and update/remove internal links to the deleted page

## Images

- Store in `site/images/` directory
- Reference as `/images/filename.png` (absolute from doc root)
- Use descriptive filenames (not `image1.png`)

## Mermaid Diagrams

Supported inline in MDX using standard fenced code blocks with `mermaid` language tag.

## Links

- **Internal links:** Use relative paths without `.mdx` extension: `/developers/reference/rpc-api`
- **External links:** Use full URLs with `https://`
- **Anchor links:** Use `#section-heading` (auto-generated from headings, lowercase, hyphenated)

## Redirects

When moving or removing a page, always add to the `redirects` array in `site/docs.json`:

```json
{
  "source": "/old/path",
  "destination": "/new/path"
}
```

Wildcard redirects are supported: `/old/:path*` -> `/new/:path*`

## Twoslash Code Verification

TypeScript code blocks in docs can be type-checked at CI time using Twoslash:

````mdx
```ts twoslash
import { createPublicClient, http } from 'viem'
import { sepolia } from 'viem/chains'
// Types are verified against the actual SDK
```
````

- Add `twoslash` meta flag to any TypeScript code block that uses SDK types
- Mintlify renders hover types natively; CI verifies correctness via `twoslash-cli`
- Use `// @errors: NNNN` for intentionally-erroring examples
- Use `// @noErrors` to suppress inline error display while still compiling
