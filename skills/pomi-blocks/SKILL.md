---
name: pomi-blocks
description: >-
  Guide Pomi to compose Orchard Core pages with BagPart and Block content types,
  nested editable components, useful field placement, and matching Liquid templates.
  Use alongside Pomi when creating a website from a brief or supplied materials,
  or designing editable pages and sections, even when the request does not mention blocks.
---

# Pomi Blocks

Apply this composition preference alongside Pomi's normal agent and skills.
For generated page sections, replace their FlowPart/Widget defaults with
BagPart/Block throughout definitions, content payloads, and templates. Let Pomi
handle site planning, installation, scaffolding, authentication, assets,
navigation, and the rest of the build using its usual procedures and the user's
brief. Tenant-stored Liquid templates remain a normal way to build the site.

Use built-in Orchard Core features. This skill requires no Content Studio,
custom module, private package feed, or other personal skill. Inspect Pomi's
live schemas and help for the target tenant instead of pinning CLI commands,
versions, project layouts, or a fixed catalogue of section types.

## Compose pages with blocks

- Use an ordered `BagPart` for page sections, named `Sections` when that name
  helps. Set its attachment's `ContentTypePartSettings.Editor` to `Blocks`.
  `OrchardCore.Flows` provides BagPart; keep that feature enabled.
- Give section content types the exact `ContentTypeSettings.Stereotype: "Block"`.
  A Block is an ordinary content type with this stereotype, not a `BlockPart`.
  The Blocks editor is a separate BagPart setting; selecting it does not set
  the children's stereotypes.
- Keep embedded types normally non-creatable and non-listable, with useful
  display text in the editor. Keep the page routable with its usual content
  stereotype. The page owns the embedded items' save/publish lifecycle.
- Constrain each Bag with `BagPartSettings.ContainedContentTypes`. Alternatively,
  use `ContainedStereotypes: ["Block"]` for a picker intentionally open to all
  blocks. Choose one strategy: stereotype selection can take precedence over
  the type list, so setting both is not an intersection.
- Set `BagPartSettings.DisplayType` explicitly, normally `Detail` for sections
  and components. The built-in Bag template can otherwise default to `Summary`.

Example attachment in a Pomi content-type definition; adapt names and allowed
types to the site and confirm the live schema:

```json
{
  "name": "Sections",
  "partName": "BagPart",
  "settings": {
    "ContentTypePartSettings": { "Editor": "Blocks" },
    "BagPartSettings": {
      "ContainedContentTypes": ["HeroBlock", "FeatureGridBlock"],
      "DisplayType": "Detail"
    }
  }
}
```

Store children in the attachment's `ContentItems` array: `Sections.ContentItems`,
or `BagPart.ContentItems` for an unnamed Bag. Do not generate `FlowPart.Widgets`
or `FlowMetadata` for this composition. On existing sites, preserve unrelated
content and legitimate module widgets, such as forms or widgets in Layers.

## Model the section's meaning

Derive reusable section types from the brief and supplied material. Use fields
for editable values: text for headings, HTML/Markdown for authored prose, media
for images, links for actions, and pickers for references. Layout markup belongs
in templates. Supply helpful labels, hints, defaults, and suitable editors.

Use nested BagParts for ordered repeated children, for example:

```text
Page -> Sections -> HeroBlock -> Actions -> Action
                -> FeatureGridBlock -> Cards -> FeatureCard
```

Named Bags make roles clear and allow several collections on one section;
an unnamed Bag is fine for a single unambiguous collection. Constrain the
child types. An Action might contain a `LinkField` and an emphasis choice.
Avoid numbered fields such as `Button1`/`Button2`, JSON in text fields, and
fixed repeated markup. Nested components can be ordinary content types without
a stereotype; use `Block` when they should themselves behave as reusable blocks.
Reserve independently stored items and relationships for content needing its
own route, reuse, or lifecycle.

Reuse one section type when only its appearance changes. Use bounded choices
such as `SectionStyle = light|dark|accent` or `Layout = split|centered`, with
stable stored values and friendly labels. A `TextField` with the `PredefinedList`
editor and `TextFieldPredefinedListEditorSettings` is sufficient. A reusable
appearance part can share these fields across blocks when useful. Map every
choice to markup/classes in the template, with a safe default; do not expose
arbitrary CSS classes to editors or create a new type for each color variation.

## Render through the block's template

Continue using Pomi's Templates workflow. Choose the template name from the
item's actual stereotype and display type:

| Item | Tenant-stored template name |
| --- | --- |
| Page with no custom stereotype | `Content__Page` |
| HeroBlock with stereotype Block, Detail | `Block__HeroBlock` |
| HeroBlock with stereotype Block, Summary | `Block_Summary__HeroBlock` |
| Action with no custom stereotype, Detail | `Content__Action` |

`Widget__HeroBlock` and `Content__HeroBlock` do not target a Block item.
Use the double-underscore shape name for stored templates, without an extension.
If working in theme files, `Block__HeroBlock` maps to `Block-HeroBlock.liquid`
or `.cshtml`.

Keep the page template structural and let each block render its own design.
Prefer rendering the existing Bag shape, for example
`{{ Model.Content.Sections | shape_render }}` on the page and
`{{ Model.Content.Actions | shape_render }}` inside the HeroBlock template.
Rendering `Model.Content` is also appropriate when all its shapes belong there.

When a custom collection wrapper is needed, build each child's display shape
so its stereotype, template, and field drivers still apply:

```liquid
<div class="hero-actions">
  {% for action in Model.ContentItem.Content.Actions.ContentItems %}
    {{ action | shape_build_display: "Detail" | shape_render }}
  {% endfor %}
</div>
```

Do not render the same collection both ways. Keep custom Bag overrides scoped
to their owning type/attachment. In a BagPart Liquid template, iterate
`Model.ContentItems`; an item template instead starts at
`Model.ContentItem.Content`. Use field shapes for rich content and formatting;
escape direct text and resolve media through Orchard's filters. If a template
reads a field directly, placement cannot move that manually emitted markup.

## Make field placement useful

Design the editor as deliberately as the page: group copy and media, give
appearance choices a clear home, place nested Actions near the hero content,
and move occasional settings out of the main editing path. Use tabs, cards,
columns, and ordering where they improve the specific form; avoid unnecessary
groups for a tiny component.

Use Pomi's Placements workflow (`OrchardCore.Placements`) for tenant-stored
rules. Target the actual editor shape and exact differentiator, scoped to the
content type. Example body for `TextField_Edit`, assuming these fields belong
to the `HeroBlock` part:

```json
{
  "shapeType": "TextField_Edit",
  "nodes": [
    { "contentType": "HeroBlock", "differentiator": "HeroBlock-Heading", "place": "Parts:0#Content;0" },
    { "contentType": "HeroBlock", "differentiator": "HeroBlock-SectionStyle", "place": "Parts:0#Appearance;1" }
  ]
}
```

Placement modifiers are `#Tab;order`, `%Card;order`, and `|Column_width;order`;
for example `Parts:1#Content;0%Media;1` or `Parts:2#Content;0|Image_4;1`.
Use the native editor's grouping behavior and verify it at narrow widths.
Move a whole Bag editor with `ContentPart_Edit` and differentiator
`HeroBlock-Actions` (`ContentType-PartName`), including its label/wrapper.
`BagPart_Edit` only targets the inner editor. Field differentiators normally
use `PartName-FieldName`; custom editors/display modes may change shape names.

For public placement, move field/part shapes into zones the block template
actually renders. Keep admin editor rules separate from public display rules.
Read existing placement before updating: Pomi replaces the rule array for a
shape type, so retain unrelated rules.

## Check the result

Alongside Pomi's normal validation, verify a representative page: the allowed
blocks can be added and reordered, nested children can be edited and reordered,
save/reopen preserves their values, and the public page uses the correct Block
templates and style choices. Check an empty collection and missing optional
media, field grouping on a narrow editor, and that each field/Bag renders once.

For version-specific details, consult the official [BagPart](https://docs.orchardcore.net/en/latest/reference/modules/Flow/BagPart/),
[Templates](https://docs.orchardcore.net/en/latest/reference/modules/Templates/), and
[Placement](https://docs.orchardcore.net/en/latest/reference/modules/Placement/) references.
