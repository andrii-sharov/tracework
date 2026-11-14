# Tracework — Themes

**Silent Waters Light** is the default. **Silent Waters Dark** uses the same palette on deeper slate surfaces.

## Theme palette

| Palette color | Hex       | Role                                                              |
| ------------- | --------- | ----------------------------------------------------------------- |
| Light Cyan    | `#E0FBFC` | Toolbar highlights, selected surfaces, and dark-theme button fill |
| Light Blue    | `#C2DFE3` | Soft accent surfaces, including the Gmail banner                  |
| Cool Steel    | `#9DB4C0` | Dividers and dark-theme secondary text                            |
| Blue Slate    | `#5C6B73` | Light-theme secondary text, input borders, and hover accents      |
| Jet Black     | `#253237` | Light-theme text and primary buttons, dark-theme cards            |

Use shadcn/ui semantic tokens and shared Tailwind utilities. Near-white surfaces and a deeper dark canvas extend the original five-color palette. Reserve green, amber, and rose for statuses and feedback.

| Token                    | Silent Waters · Light | Silent Waters · Dark |
| ------------------------ | --------------------- | -------------------- |
| `background`             | `#F1F8F9`             | `#182429`            |
| `card`, `popover`        | `#FCFEFE`             | `#253237`            |
| `foreground`             | `#253237`             | `#E0FBFC`            |
| `muted`, `secondary`     | `#E0FBFC`             | `#32434A`            |
| `accent`                 | `#C2DFE3`             | `#32434A`            |
| `muted-foreground`       | `#5C6B73`             | `#9DB4C0`            |
| `border`                 | `#9DB4C0`             | `#5C6B73`            |
| `input`                  | `#5C6B73`             | `#9DB4C0`            |
| `primary`                | `#253237`             | `#E0FBFC`            |
| `primary-foreground`     | `#E0FBFC`             | `#253237`            |
| `ring`                   | `#5C6B73`             | `#C2DFE3`            |
| `destructive`            | `#B42332`             | `#FDA4AF`            |
| `destructive-foreground` | `#FFFFFF`             | `#3A101B`            |

Use foreground for text on cards, popovers, secondary controls, and accent surfaces. Use foreground for banner text on Light Blue. Persist the selected theme in profile settings and apply it to dialogs, dropdowns, and overlays.

### Status and reply colors

Each cell lists **text / badge fill**. Custom stages start with the neutral Applied pair. Keep the text label visible in every theme. No Answer and New Reply are secondary indicators beside the current stage.

| Status or indicator | Light               | Dark                |
| ------------------- | ------------------- | ------------------- |
| Applied             | `#253237 / #E0FBFC` | `#C2DFE3 / #32434A` |
| In Review           | `#1E40AF / #DBEAFE` | `#BFDBFE / #1E365D` |
| Screen Interview    | `#3730A3 / #E0E7FF` | `#C7D2FE / #2C305E` |
| Test Task           | `#5B21B6 / #EDE9FE` | `#DDD6FE / #372A58` |
| Tech Interview      | `#115E59 / #CCFBF1` | `#99F6E4 / #173D3B` |
| Offer               | `#166534 / #DCFCE7` | `#BBF7D0 / #1C3D2C` |
| Rejected            | `#9F1239 / #FFE4E6` | `#FDA4AF / #4A2330` |
| Withdrawn           | `#374151 / #E5E7EB` | `#D1D5DB / #30363F` |
| No Answer           | `#78350F / #FEF3C7` | `#FDE68A / #423417` |
| New Reply           | `#075985 / #E0F2FE` | `#BAE6FD / #19384D` |
