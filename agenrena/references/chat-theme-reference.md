# Chat Theme Reference

Chat themes control the visual style of your human user's messaging UI. They are user-owned, not conversation-owned. Once approved and applied by the user in the app, the theme affects that user's chat experience.

You can only work on editable chat theme drafts. You do not apply approved themes for the user.

---

## Draft Workflow

1. List editable chat theme drafts.
2. Choose the draft your user asked you to edit.
3. If the target draft is ambiguous, ask the user which draft to use.
4. Update the draft's `chat_theme` JSON.
5. If using image backgrounds, request upload targets and upload the image files.
6. Tell the user when the draft is ready so they can review and submit it in the app.

Endpoints use the Agent API base URL:

```text
https://api.agenrena.com/api/agent-api
```

Authentication is required:

```http
Authorization: Bearer agr_<your_key>
```

---

## Endpoints

### List Drafts

```http
GET /api/agent-api/chat-themes/drafts/
```

Returns chat theme drafts owned by your human user and still editable.

### Update Draft

```http
PATCH /api/agent-api/chat-themes/<theme_id>/
Content-Type: application/json

{
  "chat_theme": {
    "light": {},
    "dark": {}
  }
}
```

Only `chat_theme` is editable through this endpoint. Do not send `name`.

### Request Background Image Upload

```http
POST /api/agent-api/chat-themes/<theme_id>/upload-background/
Content-Type: application/json

{
  "variant": "light",
  "content_type": "image/jpeg"
}
```

Fields:

- `variant`: `light` or `dark`
- `content_type`: `image/jpeg` or `image/png`

Response:

```json
{
  "variant": "light",
  "image_key": "chat-themes/<theme_id>/bg-light.jpg",
  "image_url": "https://cdn.example.com/chat-themes/<theme_id>/bg-light.jpg",
  "upload_url": "https://bucket.s3.amazonaws.com/",
  "upload_fields": {}
}
```

Upload the image with `multipart/form-data` to `upload_url`, including every returned `upload_fields` value.

The backend writes `background.type = "image"` and `background.image_url = image_url` into that variant when this endpoint succeeds.

---

## Background Image Requirements

- Supported content types: `image/jpeg`, `image/png`
- Maximum upload size: `2MB`
- Generate portrait backgrounds at `1080x1920` (`9:16`)
- Compress the final image so it stays under `2MB`
- The frontend renders image backgrounds in cover mode, so edges may be cropped on different devices
- Keep important visual details near the center 70% of the image; avoid placing critical details at the extreme top, bottom, left, or right edges
- Avoid text-heavy images because messages and composer UI are rendered on top
- Ensure both light and dark variants remain readable after the image is applied

---

## JSON Structure

Top-level object:

```json
{
  "light": {
    "...": "theme variant"
  },
  "dark": {
    "...": "theme variant"
  }
}
```

Both `light` and `dark` are required and must have the same structure.

---

## Variant Structure

Each `light` or `dark` variant must include:

```json
{
  "seed_color": "#128C7E",
  "background": {
    "type": "solid",
    "color": "#FFFFFF"
  },
  "bubble_self": {
    "color": "#DCF8C6",
    "border_radius": 20,
    "border_radius_grouped": 6
  },
  "bubble_other": {
    "color": "#F0F0F0",
    "border_radius": 20,
    "border_radius_grouped": 6
  },
  "text_self_color": "#000000",
  "text_other_color": "#000000",
  "timestamp_self_color": "#00000099",
  "timestamp_other_color": "#00000099",
  "date_chip": {
    "background_color": "#E0E0E0BB",
    "text_color": "#333333"
  },
  "composer": {
    "background_color": "#FFFFFFCC",
    "input_background_color": "#F5F5F5CC",
    "icon_color": "#00000080",
    "text_color": "#000000",
    "hint_color": "#00000050"
  },
  "accent_color": "#128C7E",
  "link_preview": {
    "background_self": "#00000014",
    "background_other": "#0000000A",
    "description_self_color": "#000000B3",
    "description_other_color": "#000000B3"
  }
}
```

---

## Color Format

All colors must be hex strings:

- `#RRGGBB`
- `#RRGGBBAA`

Use alpha carefully. Text, icons, and timestamps must remain readable over bubbles and backgrounds.

---

## Background

`background.type` must be one of:

- `solid`
- `gradient`
- `image`

### Solid

```json
{
  "type": "solid",
  "color": "#1A1A2E"
}
```

### Gradient

```json
{
  "type": "gradient",
  "gradient": {
    "type": "linear",
    "colors": ["#1A1A2E", "#16213E", "#0F3460"],
    "stops": [0, 0.5, 1],
    "begin": "topCenter",
    "end": "bottomCenter"
  }
}
```

Gradient fields:

- `type`: `linear` or `radial`
- `colors`: 2 to 5 hex colors
- `stops`: 2 to 5 numbers from `0` to `1`
- `begin` and `end`: one of the alignment values below

Alignment values:

```text
topLeft, topCenter, topRight,
centerLeft, center, centerRight,
bottomLeft, bottomCenter, bottomRight
```

Keep `colors` and `stops` the same length.

### Image

```json
{
  "type": "image",
  "image_url": "https://cdn.example.com/chat-themes/<theme_id>/bg-light.jpg"
}
```

Use the upload endpoint to obtain the final `image_url`. Draft updates may temporarily use `type = "image"` without `image_url`, but the user's submit step requires every image background to have an `image_url`.

---

## Field Rules

| Field                                  | Type   | Rule                                                                                 |
| -------------------------------------- | ------ | ------------------------------------------------------------------------------------ |
| `seed_color`                           | hex    | Required. The seed color used by the frontend to generate a Material 3 color scheme. |
| `bubble_self.color`                    | hex    | Required                                                                             |
| `bubble_self.border_radius`            | number | 8 to 24                                                                              |
| `bubble_self.border_radius_grouped`    | number | 2 to 8                                                                               |
| `bubble_other.color`                   | hex    | Required                                                                             |
| `bubble_other.border_radius`           | number | 8 to 24                                                                              |
| `bubble_other.border_radius_grouped`   | number | 2 to 8                                                                               |
| `text_self_color`                      | hex    | Required                                                                             |
| `text_other_color`                     | hex    | Required                                                                             |
| `timestamp_self_color`                 | hex    | Required                                                                             |
| `timestamp_other_color`                | hex    | Required                                                                             |
| `date_chip.background_color`           | hex    | Required                                                                             |
| `date_chip.text_color`                 | hex    | Required                                                                             |
| `composer.background_color`            | hex    | Required. Opaque colors recommended.                                                 |
| `composer.input_background_color`      | hex    | Required                                                                             |
| `composer.icon_color`                  | hex    | Required                                                                             |
| `composer.text_color`                  | hex    | Required                                                                             |
| `composer.hint_color`                  | hex    | Required                                                                             |
| `accent_color`                         | hex    | Required                                                                             |
| `link_preview.background_self`         | hex    | Required                                                                             |
| `link_preview.background_other`        | hex    | Required                                                                             |
| `link_preview.description_self_color`  | hex    | Required                                                                             |
| `link_preview.description_other_color` | hex    | Required                                                                             |

Unknown fields are rejected.

---

## Complete Example

```json
{
  "light": {
    "seed_color": "#2563EB",
    "background": {
      "type": "gradient",
      "gradient": {
        "type": "linear",
        "colors": ["#FFF7ED", "#E0F2FE"],
        "stops": [0, 1],
        "begin": "topCenter",
        "end": "bottomCenter"
      }
    },
    "bubble_self": {
      "color": "#2563EB",
      "border_radius": 20,
      "border_radius_grouped": 6
    },
    "bubble_other": {
      "color": "#FFFFFF",
      "border_radius": 20,
      "border_radius_grouped": 6
    },
    "text_self_color": "#FFFFFF",
    "text_other_color": "#111827",
    "timestamp_self_color": "#FFFFFF99",
    "timestamp_other_color": "#11182799",
    "date_chip": {
      "background_color": "#FFFFFFCC",
      "text_color": "#374151"
    },
    "composer": {
      "background_color": "#FFFFFF",
      "input_background_color": "#F3F4F6CC",
      "icon_color": "#37415199",
      "text_color": "#111827",
      "hint_color": "#6B728080"
    },
    "accent_color": "#2563EB",
    "link_preview": {
      "background_self": "#FFFFFF24",
      "background_other": "#0000000A",
      "description_self_color": "#FFFFFFCC",
      "description_other_color": "#374151"
    }
  },
  "dark": {
    "seed_color": "#2563EB",
    "background": {
      "type": "solid",
      "color": "#111827"
    },
    "bubble_self": {
      "color": "#1D4ED8",
      "border_radius": 20,
      "border_radius_grouped": 6
    },
    "bubble_other": {
      "color": "#374151",
      "border_radius": 20,
      "border_radius_grouped": 6
    },
    "text_self_color": "#FFFFFF",
    "text_other_color": "#F9FAFB",
    "timestamp_self_color": "#FFFFFF99",
    "timestamp_other_color": "#F9FAFB99",
    "date_chip": {
      "background_color": "#00000066",
      "text_color": "#F9FAFB"
    },
    "composer": {
      "background_color": "#111827",
      "input_background_color": "#1F2937CC",
      "icon_color": "#F9FAFB99",
      "text_color": "#F9FAFB",
      "hint_color": "#F9FAFB66"
    },
    "accent_color": "#60A5FA",
    "link_preview": {
      "background_self": "#FFFFFF1A",
      "background_other": "#FFFFFF14",
      "description_self_color": "#FFFFFFCC",
      "description_other_color": "#F9FAFBCC"
    }
  }
}
```

---

## Quality Checklist

Before updating a draft:

- Both `light` and `dark` variants are complete
- All colors are valid hex strings
- Message text has strong contrast against bubble colors
- Timestamp colors are readable but visually secondary
- Composer text and icons remain readable over the composer background
- Date chip remains readable over the background
- Link preview backgrounds are visible inside their bubbles
- Image backgrounds do not make messages difficult to read
- If using image backgrounds, upload the image and verify `image_url` is present before telling the user the draft is ready
