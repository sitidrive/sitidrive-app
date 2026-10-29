# sitidrive Avatar System: Developer Handoff (for Qwen Coder)

## 1. What this is
A Gen Z rebrand (20 avatars: 10 women, 10 men) of the "people faces" from the sitidrive polo design into a reusable **avatar system** for the sitidrive shared-commuting app (African urban cities). The faces are redrawn as clean, flat, scalable SVGs: they are rounder, they reflect the people who ride in Lagos, Accra, Nairobi and similar cities, and they are legible at 24px.

Use them wherever a person appears: rider/driver profiles, ride-share "who's in this ride" stacks, chat, ratings, empty states, onboarding.

## 2. Files in this pack
| File | Purpose |
|---|---|
| `avatars/sd-avatar-XX-<slug>.svg` | 10 individual avatars, 96x96 viewBox, circular, self-contained |
| `sitidrive-avatars-sprite.svg` | All 10 as `<symbol>` entries for `<use href="#id">` |
| `tokens.json` | Brand colours, skin tones, tint backgrounds, size scale, avatar manifest |
| `preview.html` | Visual reference including the overlapping "ride group" stack |

## 3. Avatar manifest
01 knotless-braids · 02 space-buns · 03 sleek-bob · 04 fluffy-afro-headband · 05 bleached-twa · 06 butterfly-locs · 07 cornrow-puff · 08 bantu-knots · 09 wolf-cut-headphones · 10 modern-hijab

Gen Z look: all ten are young women with current African and diaspora styles: knotless braids with coloured accents, butterfly locs, bantu knots, cornrows into a high puff, bleached tapered curls, sleek bob with sunglasses, wolf cut with chunky headphones, space buns, a big afro with a y2k headband, and a satin hijab. Details include gold hoops, nose studs, scrunchies and blush. 

**Men (ids start `sd-avatar-m`):** m01 taper-curly-top · m02 wave-fade · m03 blonde-twists · m04 locs-half-up · m05 braids-beard · m06 buzz-glasses · m07 afro-shades · m08 snapback-hoodie · m09 bucket-hat · m10 wave-durag. Details: fades, 360 waves, bleached twists, dyed loc tips, beard styles, round glasses, shades, snapback, bucket hat, wave durag, chains and studs.

Each entry in `tokens.json` has a `gender` field (`female` or `male`). Assign by the user's chosen style in Profile > Edit avatar. Show all 20 in one grid and don't tie avatar to gender identity, since users should be free to pick any.

## 4. Brand tokens
- Charcoal `#1F2937` (text, outlines, shirts)
- Orange `#F97316` (primary accent, the "i" dot in the logo)
- Teal `#14B8A6` (secondary, success)
- Magenta `#EC4899` (highlight)
- Muted Red `#F87171` (alerts, SOS)
- Gold accent `#FBBF24` (hoops, braid cuffs, studs only). Avatar tint backgrounds: orange `#FFEDD5`, teal `#CCFBF1`, magenta `#FCE7F3`, red `#FEE2E2`, grey `#E5E7EB`

## 5. Usage rules
1. **Sizes:** xs 24, sm 32, md 40, lg 56, xl 96. Always circular. Never stretch.
2. **Assignment:** on signup, assign `avatarId` at random (stable hash of user id) so no two accounts in a ride look identical where possible. Let users change it in Profile > Edit avatar (grid of 10).
3. **Stack ("ride group"):** overlap by -10px at md size, 2px border in the app background colour, then a teal "+N" pill. Max 3 visible.
4. **Role ring (optional):** driver = 2px orange ring; rider = no ring. Do not rely on colour alone, so add a small car badge for drivers.
5. **Accessibility:** each SVG has `role="img"` and an `aria-label`. In the UI use the user's name as alt text, not the avatar description.
6. **Fallback:** if no `avatarId`, show initials on a tint background (same circle, charcoal text).
7. Do not recolour the skin tones or hair per theme. Dark mode: keep avatars unchanged and use a 1px `#FFFFFF1A` ring.

## 6. Reference implementation (React + TypeScript; adapt to our stack)
```tsx
// Avatar.tsx
import tokens from "./tokens.json";

type Size = keyof typeof tokens.avatarSizes;
type Props = { avatarId?: string; name: string; size?: Size; ring?: boolean };

export function Avatar({ avatarId, name, size = "md", ring }: Props) {
  const px = tokens.avatarSizes[size];
  const style = { width: px, height: px, borderRadius: "50%",
    boxShadow: ring ? `0 0 0 2px ${tokens.color.orange}` : undefined };
  if (!avatarId) {
    const initials = name.split(" ").map(w => w[0]).slice(0, 2).join("").toUpperCase();
    return <div style={{ ...style, background: tokens.avatarBackgrounds.or, color: tokens.color.charcoal,
      display: "grid", placeItems: "center", fontWeight: 600, fontSize: px * 0.4 }}>{initials}</div>;
  }
  return (
    <svg style={style} viewBox="0 0 96 96" role="img" aria-label={name}>
      <use href={`/sitidrive-avatars-sprite.svg#${avatarId}`} />
    </svg>
  );
}

export function AvatarStack({ users }: { users: { id: string; name: string; avatarId?: string }[] }) {
  const shown = users.slice(0, 3);
  return (
    <div style={{ display: "flex", alignItems: "center" }}>
      {shown.map((u, i) => (
        <div key={u.id} style={{ marginLeft: i ? -10 : 0, border: "2px solid var(--bg)", borderRadius: "50%" }}>
          <Avatar avatarId={u.avatarId} name={u.name} size="md" />
        </div>
      ))}
      {users.length > 3 && <span className="pill-teal">+{users.length - 3}</span>}
    </div>
  );
}
```
Note: inline the sprite once in the document body (or inject at app root), because cross-file `<use>` can fail in some WebViews. For React Native use `react-native-svg` with the individual SVG files; for Flutter use `flutter_svg`.

## 7. Where they go in the screens (map to the UI mockups)
| Screen | Placement |
|---|---|
| Onboarding / welcome | Row of 5 avatars above the headline "Shared city rides. Smarter commutes." |
| Home | Top-right profile avatar (md) |
| Ride search results | Driver avatar (lg) + ride group stack (md) per card |
| Ride details / confirmation | Driver (xl), co-riders as stack, tap to open mini profile |
| In-ride | Co-rider stack in the header |
| Chat | sm avatar beside each message group |
| Ratings / reviews | sm avatar beside each review |
| Empty states | xl avatar cluster with illustration text |
| Profile > Edit avatar | 5x4 selectable grid (20 avatars), selected state = 3px teal ring |

## 8. Tagline
Keep "SHARED CITY RIDES. SMARTER COMMUTES." (uppercase, wide letter-spacing, charcoal) for onboarding and splash only.

## 9. Build checklist for Qwen Coder
- [ ] Add `tokens.json` values to the design system (CSS variables or theme file)
- [ ] Inline sprite at app root
- [ ] Implement `Avatar`, `AvatarStack`, `AvatarPicker`
- [ ] Persist `avatarId` on the user model (string, nullable)
- [ ] Replace placeholder profile images across the screens in section 7
- [ ] Unit test: fallback initials, stack overflow "+N", aria-label
- [ ] Verify at 24px that features stay readable

## 10. Open items for the product owner
- Confirm the target stack (React Native, Flutter, web/PWA) so the component can be finalised.
- Confirm the final logo asset: the avatars here intentionally do not include the logo.
- Optional: a second batch of 10 with male Gen Z styles (fades, tapers, twists, dyed hair) and more variety (uniforms, students, traders).
