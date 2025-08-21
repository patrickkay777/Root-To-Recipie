[Recipe Unlocks Prototype – Project Plan (draft V0.docx](https://github.com/user-attachments/files/21916234/Recipe.Unlocks.Prototype.Project.Plan.draft.V0.docx)
Project: Recipe Unlocks at Point of Sale (Prototype)
Owner: Patrick Kay (@patrickkay)
Org: Riverford Organic
Context: Build a demo-quality tool that shows the value of adding recipe recommendations (and a curated set of add‑on items) at the point of sale, increasing the number of recipes a customer can make with their basket while clearly communicating added basket value and unlocked recipes.
________________________________________
1) Goals & Success Criteria
Primary Goals
•	Increase average order value (AOV) by suggesting high‑leverage add‑on products.
•	Minimise waste by showing more uses for the veg box contents via unlocked recipes.
•	Demonstrate customer value clearly with a large, prominent recipe‑count indicator and a transparent basket total.
Success Criteria (Prototype)
•	A single‑page demo runs locally or as a static web page.
•	For a given veg box, the tool:
o	Calculates base unlocked recipes (veg box contents + ticked pantry).
o	Identifies the top 10 add‑on products that maximise additional unique recipe unlocks.
o	Allows toggling pantry items with the exact basket rules described (see §5.3).
o	Updates basket total and unlocked recipes list in real time.
o	Presents a clean Riverford‑styled UI with the supplied font.
________________________________________
2) Data Inputs
•	Veg boxes: /mnt/data/veg_boxes.csv
Contains box products and their contents.
•	Recipes (raw/normalised): /mnt/data/recipes_ALL_raw.json, /mnt/data/recipes_normalized.json
Ingredient lists per recipe (normalised set preferred).
•	Pantry: /mnt/data/pantry_STRICT_v3_full.json
Canonical pantry item list; which items are Riverford‑purchasable.
•	Products catalogue: /mnt/data/products_raw.json
Names, prices, IDs, product types; used to price add‑ons & pantry items.
•	Font: /mnt/data/Riverford2020-Regular.otf
Use for headings/body to match brand.
Assumption: All purchasable pantry items and candidate add‑ons exist in products_raw.json with prices inc. VAT.
________________________________________
3) Data Normalisation & Mapping
3.1 Ingredient Normalisation
•	Lowercase, strip whitespace, unify plurals and UK/metric variants (e.g., “tomatoes”→“tomato”).
•	Map common synonyms (e.g., “courgette”↔“zucchini”, “scallion”↔“spring onion”).
•	Remove descriptors (“chopped”, “finely sliced”) during match logic.
3.2 Product–Ingredient Linking
•	Build a dictionary: ingredient → {product_ids[], is_pantry_default, purchasable}.
•	For veg boxes, map contents to same ingredient tokens.
•	Units: convert recipe requirements to approximate whole‑item presence (prototype assumes presence/absence, not precise grams). See §9 risks.
________________________________________
4) Core Computations
4.1 What counts as a recipe being “unlocked”?
A recipe is unlocked only if every required ingredient is present in the user’s current selection with strict matching (no fuzzy unlocks): 1) The selected veg box contents; 2) The currently ticked pantry items; and 3) Any explicitly added add‑ons (from the Top‑10 list for the selected box and pantry state).
Strict matching rules: normalise synonyms, plurals, and descriptors, but do not substitute or infer near‑equivalents (e.g., “red onion” maps to “onion”, but “lemon” does not unlock “lime”). Compound products contribute only their exact mapped ingredients.
4.2 Base Unlock Set
•	Compute AvailableSet = VegBoxIngredients ∪ TickedPantryIngredients.
•	BaseUnlockedRecipes = { r ∈ Recipes | r.ingredients ⊆ AvailableSet }.
4.3 Dynamic Candidate Add‑Ons (per veg box & pantry state)
•	Candidate pool: all purchasable Riverford items (excluding the selected veg box).
•	For each candidate product p, derive its exact IngredientContribution(p).
•	Recompute recommendations dynamically when the user changes veg box or pantry ticks.
4.4 Marginal Unlocks per Product
For each candidate product p: - NewAvailableSet(p) = AvailableSet ∪ IngredientContribution(p) - UnlockedWith(p) = { r | r.ingredients ⊆ NewAvailableSet(p) } − BaseUnlockedRecipes - Gain(p) = |UnlockedWith(p)|
4.5 Selecting the Top‑10 Add‑Ons (Set‑Coverage Greedy)
Because recipes often share missing items, use greedy maximum coverage with strict matching: 1) Covered = BaseUnlockedRecipes
2) Repeat until 10 items selected or no gains: - For each remaining candidate p, compute marginal gain under strict matching.
- Tie‑break on £ per additional recipe unlocked (price(p)/gain). - If still tied, prefer lower price, then wider recipe category coverage. - Add argmax p* to Selected; update Covered.
This greedy approach yields strong results for the demo and stays faithful to strict ingredient presence.
________________________________________
5) Basket & Pantry Interaction Rules
5.1 Initial State
•	All pantry items ticked (selected) by default.
•	Basket total starts at veg box price only (no pantry costs included).
5.2 Add‑Ons
•	The curated Top‑10 add‑ons each show price and an Add button.
•	Clicking Add increases basket total and updates AvailableSet → more recipes unlocked.
•	Clicking Remove (or toggling off) subtracts its price and recomputes unlocks.
5.3 Pantry Toggle Logic (per spec)
•	Unticking a pantry item simply removes that ingredient from AvailableSet, potentially reducing current unlocks; no price change.
•	If the user subsequently ticks a pantry item that is also a Riverford product, treat this as adding that product to the basket:
o	Increase basket total by that item’s price.
o	The item becomes part of ExplicitAdds until unticked again; unticking then removes its price from basket.
•	If the pantry item is not purchasable, ticking/unticking only affects unlocks, never price.
5.4 Basket Total
BasketTotal = VegBoxPrice + Σ AddedAddOnPrices + Σ AddedPantryPurchases
5.5 Large Circle Indicator
•	Prominent circular badge shows TotalUnlockedRecipes under current state.
•	Secondary text: (+N from add‑ons) vs base, and BasketTotal nearby.
________________________________________
6) UI/UX Design (Riverford‑styled)
6.1 Layout
•	Header: Veg box selector (if multiple), price, short description.
•	Hero Row:
o	Large circle with unlocked recipe count.
o	Basket card with itemised list and total.
•	Two Columns:
o	Left: Pantry list (search, category filters, “Untick All” + “Tick All”). Purchasable pantry items labelled (price pill). Sticky sub‑header for quick actions.
o	Right: “Top 10 Add‑Ons” grid/cards with images, short text, price, and Add/Remove.
•	Footer Section: “Unlocked recipes” dynamic list with recipe names and key ingredients.
6.2 Visual Style
•	Use Riverford2020-Regular.otf for headings & body.
•	Colour palette: off‑white background, muted greens, earthy browns; soft shadows; rounded corners (12–16px).
•	Buttons: clear hierarchy (primary: Add/Remove; secondary: filters). Large hit targets on mobile.
•	Accessibility: WCAG AA contrast; focus states; keyboard navigable.
6.3 Micro‑interactions
•	Smooth count‑up animation on the circle when the number changes.
•	Toasts/snackbars confirming Add/Remove.
•	Tooltips explaining why an add‑on is recommended (e.g., “Unlocks 7 more recipes: soups, stir‑fries”).
6.4 Recommendations presentation (Honest mode + Perfect Combos)
•	Honest mode default: Show only add‑ons/combos that add value now (marginal unlocks > 0 under strict matching).
•	Cap at 10 recommendations total per box/pantry state.
o	Priority order: (1) Single add‑ons (sorted by unlocks ↓, then recipes‑per‑£ ↑, then price ↑), (2) Perfect Combos (pairs/trios) to fill any remaining slots up to 10.
•	Section labels & microcopy:
o	Singles section heading: “Recommended add‑ons”. Badge: “+X recipes”.
o	Combos section heading: “Perfect Combos”. Badge: “Perfect Combo: +X together”. CTA: “Add combo”. Show combined price.
o	Optional link: “See more combos” (collapsible) if we hide extras beyond the cap.
•	Behaviour: Selecting a combo adds all items as a grouped action; basket chip shows +£Total and the circle shows +X together.
•	Transparency: If we ever display non‑unlocking items (e.g., in a secondary view), they are clearly marked “+0 alone (sets up Y recipes to one short)”.
________________________________________
7) Implementation Plan
7.1 Tech Stack
•	Frontend: React + Vite, TypeScript, TailwindCSS.
•	State: Zustand or Redux Toolkit (small store); URL params for shareable state.
•	Data: Load local JSON/CSV at build time; no backend required.
•	Packaging: Single static bundle that runs offline (for demos).
7.2 Key Modules
•	data/normalise.ts – tokenise & normalise ingredients; synonym map.
•	data/load.ts – parse veg boxes CSV, products & pantry JSON, recipes JSON.
•	engine/match.ts – recipe vs available ingredients; base unlocked set.
•	engine/coverage.ts – greedy selection for Top‑10 add‑ons.
•	engine/combos.ts – compute Perfect Combos (pair/trio synergy), rank by combined unlocks then recipes‑per‑£.
•	engine/basket.ts – pricing model following §5.
•	ui/components/* – CircleCounter, BasketCard, PantryList, AddOnCard, ComboCard, RecipesList.
•	theme/fonts.css – embed Riverford font with @font-face.
7.3 Performance
•	Precompute tokenised recipe ingredient sets once at load.
•	Efficient set operations with bitsets or hashed string IDs.
•	Memoise coverage gains across small state changes.
7.4 Analytics (Prototype‑local)
•	Capture counts of: base unlocks, per add‑on gains, £ per unlock, interactions.
•	Export anonymised JSON snapshot for each session.
________________________________________
8) Algorithm Details & Pseudocode
// Build dictionaries
ING_MAP = normalise_all(ingredient_synonyms)
RECIPES = load_recipes_normalised().map(r => Set(normalise(r.ingredients)))
PANTRY = load_pantry()
PRODUCTS = load_products()
VEGBOX = load_vegbox(selected_box_id)

// Initial availability
Available = Set(normalise(VEGBOX.contents)) ∪ Set(PANTRY.default_ticked_items)
BaseUnlocked = { r in RECIPES | r ⊆ Available }

// Candidate pool
CANDIDATES = { p in PRODUCTS | p.purchasable && contributes_ingredients(p) }

function marginal_gain(selected):
  SelectedIngredients = union(IngredientContribution(p) for p in selected)
  Covered = recipes_unlocked(Available ∪ SelectedIngredients)
  best = []
  for p in CANDIDATES − selected:
    gain = |recipes_unlocked(Available ∪ SelectedIngredients ∪ IngredientContribution(p)) − Covered|
    if gain > 0:
      score = gain / price(p) // recipes per £
      best.push({p, gain, score})
  return top by gain, tie‑break by score then price

// Greedy Top‑10
Selected = []
repeat 10 times:
  option = argmax(marginal_gain(Selected))
  if option.gain == 0: break
  Selected.push(option.p)

return Selected
________________________________________
9) Edge Cases, Constraints & Risks
•	Ingredient granularity: Recipes may specify quantities; prototype assumes binary presence. Mitigation: only use presence to unlock; later add a stock/quantity model.
•	Synonyms/variants: Incomplete synonym mapping could undercount matches. Mitigation: grow a curated map based on top recipes.
•	Compound products: Some products map to multiple ingredients (e.g., “stir‑fry pack”). Ensure IngredientContribution covers all.
•	Seasonality: Veg box contents change; ensure the tool reads from the CSV for a chosen week/box.
•	Pricing: Confirm price source and VAT status; cache within products_raw.json for offline demo.
•	Performance: Recipe set size may be large; use hashed IDs and memoisation.
________________________________________
10) QA & Acceptance Criteria
Functional - [ ] Base unlocked count equals recipes available from veg box + default pantry (strict matching). - [ ] Untick pantry item reduces availability; no price change. - [ ] Retick purchasable pantry item adds to basket and price increases. - [ ] Add/Remove add‑on items updates basket and unlock count. - [ ] Top‑10 add‑ons recompute dynamically per veg box & pantry state with strict matching. - [ ] Honest mode: Only show items/combos with marginal unlocks > 0 in the primary list. - [ ] Cap of 10 recommendations enforced; singles prioritised; Perfect Combos fill remainder. - [ ] Selecting a Perfect Combo adds all items as a group; basket and circle reflect combined price and combined unlocks. - [ ] Unlocked recipe list refreshes deterministically for any state change.
UX - [ ] Riverford font loads fast with fallback. - [ ] Large count circle is readable on mobile/desktop with diff badges (e.g., “+12 recipes”, “+£5.90”). - [ ] Buttons have clear focus states and labels. - [ ] Before vs After toggle shows base vs current (recipes & basket) for storytelling. - [ ] Combos section titled “Perfect Combos”; badges read “Perfect Combo: +X together”.
Performance - [ ] Initial load < 2s on mid‑range laptop; interaction < 100ms updates.
________________________________________
11) Demo & Distribution
•	Option A (simplest): Static build (Vite) served from any HTTP server; share a single index.html + assets folder.
•	Option B (standalone): Electron or Tauri wrapper for offline demo (optional).
•	Option C (internal preview): Host on a Riverford staging subdomain.
Provide a sample state preset (chosen veg box + fixed Top‑10) for reproducible demos.
________________________________________
12) Timeline (Indicative)
Phase	Deliverables	Effort
0. Prep	Confirm data formats, price fields, veg box mapping; agree UI wireframes	0.5–1d
1. Engine	Normalisation, base unlocks, greedy Top‑10, basket rules	2–3d
2. UI	Circle counter, pantry, add‑ons grid, basket, recipes list	2–3d
3. Polish	Styling, animations, accessibility, analytics	1–2d
4. QA + Demo	Test matrix, packaging, sample presets	1–2d
Total: ~6–11 days for a polished prototype.
________________________________________
13) Open Questions (Targeted)
1)	Prices: Which field in products_raw.json is authoritative for customer‑visible price? Are all prices inc. VAT? Any items to exclude?
2)	Veg box choice: Should we support multiple boxes on the demo, or fix one example?
3)	Recipe scope: Use recipes_normalized.json only, or include ALL and normalise on load? Any categories to exclude (e.g., desserts)?
4)	Synonyms: Do we have an existing ingredient synonym list to import, or shall we curate one during dev (seeded from top 200 ingredients)?
5)	Pantry defaults: Are all items in pantry_STRICT_v3_full.json ticked by default, or a subset? (Spec says all ticked; confirm.)
6)	Purchasable pantry: Is there a definitive mapping pantry_item → product_id (1:1 always?), or do we need a ruleset (e.g., brand/size variants)?
7)	Images: Do we have product and recipe imagery sources for cards? If not, should we use placeholders?
8)	Allergens/diet: Any filters required in the demo (vegetarian/vegan already assumed)?
9)	Units & quantities: OK to keep prototype as presence/absence? (No grams/ml modelling.)
10)	Copy & tone: Any brand copy for labels (e.g., “Unlocks 7 recipes”) we should match?
11)	A/B framing: Do we need a simple “before vs after” snapshot in the UI for storytelling?
12)	Analytics: Which metrics are most useful to capture for stakeholder review (e.g., recipes per £, average add‑ons added)?
________________________________________
14) Next Steps Checklist
☐	Approve this plan and resolve open questions (esp. 1, 3, 5, 6).
☐	I’ll implement the normalisation + coverage engine.
☐	Hook up UI with real data and basket logic.
☐	Produce a packaged static demo and a preset walkthrough.
________________________________________
Draft v0.1 – intended to be iterated after Q&A.
