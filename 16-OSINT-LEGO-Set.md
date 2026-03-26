# OSINT/LEGO — Identify the LEGO Set

## Challenge Information
- **Category**: OSINT
- **Difficulty**: Medium
- **Points**: Unknown
- **Flag**: `zerodays{Coast Guard Head Quarters}`

## Description
An image shows three LEGO pieces. The task is to identify which LEGO set they come from.

Flag format: `zerodays{Name Of Lego Set With Spaces}`

## Challenge Analysis

### Visible LEGO Pieces
1. **Green 2×2 round plate with hole** (part 4032)
2. **Light grey 2×4 brick** (part 3001)
3. **Black 2×2 slope with control panel print** (red/green buttons + radar screen)

The most distinctive piece is the printed slope with the control panel design.

## Solution

### Step 1: Identify the Printed Part
The black slope with red/green lamps and radar screen is the key identifier.

Search BrickLink for slopes with control panel prints:
- Part number: `3039pb048`
- Description: "Slope 45 2×2 with Control Panel with Red and Green Lamps Pattern"

### Step 2: Find Sets Containing This Part
On BrickLink, search for sets containing part `3039pb048`:

**Results show this part appears in:**
- LEGO City set **60167** - "Coast Guard Head Quarters" (2017)
- Possibly other sets, but this is the most prominent

### Step 3: Verify with Other Parts
Cross-reference with the other visible parts:
- Green rotor hub (part 4032) - used in helicopter builds
- Light grey bricks - common in City sets

LEGO City 60167 "Coast Guard Head Quarters" includes:
- Helicopter with green rotor
- Control panel pieces
- Grey building bricks
- Rescue/emergency theme

### Step 4: Confirm the Set Name
Official LEGO set name: **"Coast Guard Head Quarters"**

Format for flag: `zerodays{Coast Guard Head Quarters}`

## Complete Solution

```
Set Number: 60167
Set Name: Coast Guard Head Quarters
Year: 2017
Theme: LEGO City
Pieces: 748
Flag: zerodays{Coast Guard Head Quarters}
```

## Technical Details

### OSINT Methodology
1. **Identify unique features**: The printed slope is most distinctive
2. **Use specialized databases**: BrickLink is the definitive LEGO database
3. **Cross-reference**: Verify with multiple parts from the image
4. **Confirm details**: Check set name, year, theme

### BrickLink Search Strategies
- **By part number**: Most accurate if you know the part
- **By print pattern**: Search for printed pieces
- **By color combination**: Narrow down by specific colors
- **By theme**: Filter by LEGO theme (City, Technic, etc.)

### Part Identification Resources
- **BrickLink**: Comprehensive part database
- **Rebrickable**: Alternative database with set inventories
- **LEGO Pick a Brick**: Official LEGO part search
- **BrickRanker**: Set comparison and ranking

## Alternative Approaches

### 1. Reverse Image Search
Upload the image to Google Images or TinEye to find similar images or the set directly.

### 2. LEGO Community Forums
Post the image on r/lego or LEGO forums - the community is very helpful with identification.

### 3. Color + Theme Analysis
- Green rotor → helicopter
- Control panel → emergency/rescue theme
- City aesthetic → LEGO City line
- Narrow down to Coast Guard or Police sets

### 4. Year Range Estimation
The print style and color scheme can help estimate the release year (2010s based on modern printing).

## Lessons Learned

1. **Specialized databases**: Use domain-specific resources (BrickLink for LEGO)
2. **Unique identifiers**: Focus on the most distinctive elements
3. **Cross-reference**: Verify findings with multiple data points
4. **Community knowledge**: LEGO communities are excellent resources
5. **Exact naming**: Flag format requires exact set name with proper spacing

## Tools Used
- **BrickLink**: Primary LEGO part and set database
- **Rebrickable**: Alternative LEGO database
- **Google Images**: For reverse image search
- **LEGO official website**: For set verification

## OSINT Skills Demonstrated
- Database searching
- Part identification
- Cross-referencing information
- Attention to detail (exact set name)
- Using specialized community resources

## References
- [BrickLink](https://www.bricklink.com/)
- [Rebrickable](https://rebrickable.com/)
- [LEGO Official Site](https://www.lego.com/)
- [r/lego Community](https://www.reddit.com/r/lego/)

---

**Flag**: `zerodays{Coast Guard Head Quarters}`

**Solved**: ✅

**Key Takeaway**: OSINT challenges often require specialized databases and community knowledge. For LEGO identification, BrickLink is the definitive resource. Focus on unique, distinctive elements (like printed pieces) to narrow down the search quickly.
