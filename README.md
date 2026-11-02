# FitFindr

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, every
> command, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> python app.py listings --full -n 6      # read the data (Milestone 1)
> python app.py fields                    # what you can filter on
> python app.py ask 'vintage graphic tee under $30'
> ```
>
> All three tools are stubs, so that last command will do nothing useful yet.
> That's the starting position.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     This is your submission. Fill each section in as you finish the milestone
     it belongs to — don't leave it all to the end.

     Unit 3 asks for the first five sections. Unit 4 adds the five below them.
     Leave the unit 4 sections alone until then; they're here so you know
     what's coming.

     Everything is pasted as TEXT. No screenshots, no images, no video links.
     A typed block of output gets full credit; a picture of the same output
     gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 3: THE BUILD ═══════════════════════ -->

## What This Does

FitFindr is an agent that helps a user search for thrifted clothing and figure out how to style what it finds. A user can describe an item they want and include details such as size and maximum price. The agent searches the available listings, selects a matching item, suggests outfits using pieces from the user's wardrobe, and creates a short fit card caption. If no listing matches the search, the agent stops and tells the user what they can change.

---

## Tool Inventory

### 1. `search_listings`

**What it does:**  
Searches the listings data for items that match the user's description, requested size, and maximum price.

**Inputs:**
- `description` (`str`) - words describing the item the user wants, such as style, category, color, or brand.
- `size` (`str | None`) - the requested clothing size, or `None` if the user does not provide a size.
- `max_price` (`float | None`) - the highest price the user is willing to pay, or `None` if no price limit is given.

**Returns:**  
A list of matching listing dictionaries, ordered by keyword match score. Each listing can contain `id`, `title`, `description`, `category`, `style_tags`, `size`, `condition`, `price`, `colors`, `brand`, and `platform`.

**When it has nothing:**  
Returns an empty list `[]`.

### 2. `suggest_outfit`

**What it does:**  
Takes the selected listing and the user's wardrobe and creates one or two outfit suggestions.

**Inputs:**
- `new_item` (`dict`) - the listing selected from the results returned by `search_listings`.
- `wardrobe` (`dict`) - a wardrobe dictionary with an `"items"` key containing the user's wardrobe items.

**Returns:**  
A non-empty string containing outfit suggestions that combine the selected item with pieces from the user's wardrobe.

**When it has nothing:**  
If the wardrobe contains no items, the tool returns general styling advice for the selected item instead of failing.

### 3. `create_fit_card`

**What it does:**  
Creates a short caption for the selected item and outfit.

**Inputs:**
- `outfit` (`str`) - the outfit suggestion returned by `suggest_outfit`.
- `new_item` (`dict`) - the selected listing returned by `search_listings`.

**Returns:**  
A short two-to-four sentence caption that mentions the selected item, its price, its platform, and the outfit vibe.

**When it has nothing:**  
If the outfit is empty or contains only whitespace, the tool returns a message explaining that there is no outfit suggestion available to create the fit card.

---

## Planning Loop

**Branch rule:**  
If `search_listings` returns an empty list, the agent stores a helpful message in `session["error"]` telling the user to change the description, size, or maximum price. The agent then stops before calling `suggest_outfit` or `create_fit_card`.

If the search returns one or more matches, the agent stores the first result in `session["selected_item"]`. It then uses that item with `suggest_outfit`, stores the response in `session["outfit_suggestion"]`, and finally sends the outfit and selected item to `create_fit_card`.

**Where it lives:**  
`agent.py::run_agent`

**How the query is parsed:**  
The query is parsed using regular expressions and string cleanup. The agent looks for a price in phrases such as `"under $30"` and a size after the word `"size"`. Those values are stored separately, and the remaining words are used as the description for the listing search.

**What moves through the session:**  
The user's original query is stored in `session["query"]`. The parsed description, size, and maximum price are stored in `session["parsed"]`. Search results are stored in `session["search_results"]`, and the first result becomes `session["selected_item"]`. The outfit suggestion is stored in `session["outfit_suggestion"]`, and the final caption is stored in `session["fit_card"]`.

---

## Sample Run

### One Full Query

```text
$ python app.py ask 'vintage graphic tee under $30'

Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   Here are 2 outfit ideas using your new Y2K butterfly baby tee and pieces from your wardrobe:

**Outfit 1: Y2K Streetwear**
*   **Bottoms:** Baggy straight-leg jeans (dark wash)
*   **Shoes:** Chunky white sneakers
*   **Outerwear:** Black cropped zip hoodie (worn open or layered over top)
*   **Accessories:** Black crossbody bag
*   *Why it works:* The fitted baby tee balances the volume of the baggy dark-wash jeans, leaning fully into the Y2K aesthetic with chunky sneakers and a cropped hoodie. 

**Outfit 2: Casual Vintage Contrast**
*   **Bottoms:** Wide-leg khaki trousers
*   **Shoes:** Black combat boots
*   **Outerwear:** Vintage black denim jacket
*   **Accessories:** Brown leather belt, black crossbody bag
*   *Why it works:* Pairing the sweet, pink-and-purple butterfly print with utilitarian khaki trousers and rugged black combat boots creates a cool contrast between girly and grunge styles.

  Fit card: Channeling total 2000s mall-goth energy with this sweet butterfly print Y2K baby tee. Throwing it on with baggy denim and chunky sneakers or styling it grunge with some rugged combat boots. Grab this vintage graphic staple on my Depop right now for just $18!

0 model calls this session, 2 served from cache

```

### Tool 1: `search_listings`

```text
$ python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=30))"

[{'id': 'lst_017', 'title': 'Mesh Long-Sleeve Top — Black', 'description': 'Sheer black mesh long-sleeve. Great for layering under a graphic tee or over a bralette. Stretchy material, fits true to size.', 'category': 'tops', 'style_tags': ['y2k', 'grunge', 'goth', 'layering'], 'size': 'S/M', 'condition': 'excellent', 'price': 15.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_002', 'title': 'Y2K Baby Tee — Butterfly Print', 'description': 'Super cute early 2000s baby tee with butterfly graphic. Fitted crop length. Tag says medium but fits like a small.', 'category': 'tops', 'style_tags': ['y2k', 'vintage', 'graphic tee', 'cottagecore'], 'size': 'S/M', 'condition': 'excellent', 'price': 18.0, 'colors': ['white', 'pink', 'purple'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_033', 'title': 'Vintage Band Tee — Faded Grey', 'description': 'Faded grey band-style tee with distressed graphic. Crew neck. Fits boxy. Well-loved but no holes or major damage.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'band tee', 'graphic tee', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 19.0, 'colors': ['grey', 'charcoal'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_006', 'title': 'Graphic Tee — 2003 Tour Bootleg Style', 'description': 'Vintage-style bootleg tee with faded graphic. Slightly boxy fit. 100% cotton, soft and worn-in.', 'category': 'tops', 'style_tags': ['graphic tee', 'vintage', 'grunge', 'streetwear', 'band tee'], 'size': 'L', 'condition': 'good', 'price': 24.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_012', 'title': 'Oversized Crewneck Sweatshirt — Vintage Navy', 'description': 'Perfectly faded navy crewneck. Genuinely vintage — not manufactured distressed. Ribbed cuffs and hem. No graphics, clean.', 'category': 'tops', 'style_tags': ['vintage', 'basics', 'oversized', 'classic'], 'size': 'XL (fits oversized)', 'condition': 'good', 'price': 20.0, 'colors': ['navy'], 'brand': None, 'platform': 'thredUp'}, {'id': 'lst_015', 'title': 'Vintage Graphic Hoodie — Faded Black', 'description': 'Faded black pullover hoodie with barely-visible vintage graphic on the chest. Cozy interior. Some pilling but adds to the worn-in look.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'graphic', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 26.0, 'colors': ['black', 'charcoal'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_011', 'title': 'Low-Rise Cargo Pants — Khaki', 'description': 'Y2K era low-rise cargo pants. Lots of pockets. Khaki color, slightly distressed at the hems. Great for layering with a long tee.', 'category': 'bottoms', 'style_tags': ['y2k', 'cargo', '2000s', 'streetwear'], 'size': 'W29', 'condition': 'fair', 'price': 27.0, 'colors': ['khaki', 'tan'], 'brand': None, 'platform': 'poshmark'}]
```

### Tool 2: `suggest_outfit`

```text
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"

Here are two outfit suggestions built around your new vintage Levi's 501 jeans, using pieces straight from your wardrobe:

### Outfit 1: Effortless Casual Streetwear
*Vibe: Laid-back, classic, and comfortable for everyday wear.*
* **Top:** White ribbed tank top
* **Outerwear:** Vintage black denim jacket
* **Shoes:** Chunky white sneakers
* **Accessories:** Black crossbody bag
* *Why it works:* The crisp white tank creates a clean base that balances the denim-on-denim combination, while the chunky sneakers and crossbody bag lean into an effortless streetwear aesthetic.

### Outfit 2: Cozy & Edgy Contrast
*Vibe: Balanced proportions with a slightly tougher edge.*
* **Top:** Black cropped zip hoodie
* **Shoes:** Black combat boots
* **Accessories:** Brown leather belt
* *Why it works:* Tucking the medium-wash 501s in with the brown leather belt adds definition, while the cropped black hoodie and combat boots give the vintage jeans a sharper, slightly grungy contrast.
```

### Tool 3: `create_fit_card`

```text
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"

Nothing beats the effortless, lived-in feel of these vintage Levi's 501 jeans in the absolute best medium wash. Just pair them with your favorite crisp white sneakers for that ultimate 90s streetwear aesthetic. Grab this timeless closet staple now on Depop for just $38.00 before someone else does!
```

---

## How I Used AI

### Moment 1

**What I asked for:**  
I used AI to help me work through the implementation of the three tool functions in `tools.py` while keeping the starter function signatures, comments, and docstrings unchanged.

**What came back:**  
The suggested implementation filtered listings by description, size, and price, handled an empty wardrobe in `suggest_outfit`, and handled an empty outfit in `create_fit_card`.

**What I changed:**  
I added only the implementation code inside the existing starter functions and kept the original comments unchanged. I then tested all three tools separately from the terminal and checked the results myself before connecting them to the planning loop.

### Moment 2

**What I asked for:**  
I used AI to help me work through the planning loop in `agent.py`, including parsing the user's query, carrying information through session state, and stopping when no listing was found.

**What came back:**  
The suggested loop parsed the size and maximum price with regular expressions, stored each tool result in the session, and created a branch when `search_listings` returned an empty list.

**What I changed:**  
I tested both possible paths with `python agent.py`. The matching query completed all three tools and returned a fit card. The impossible query stopped early, returned a useful message, and kept `session["fit_card"]` set to `None`.

<!-- ═══════════════════════ UNIT 4 — THE TEST ═══════════════════════

     Don't fill these in during unit 3.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Run Log — Before

<!-- Five criteria, five tries each, in this exact format.

     Five, because your criteria are written out of five. Mark each try PASS
     or FAIL, count the passes, and read that count against your target — a
     row targeting 4 of 5 with three PASS cells is MISSED (3/5).

     `python run_eval.py --label before` runs everything and writes the table
     into results/. Paste it here and fill in the verdicts. -->

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

```

```

---

## Verdicts and Diagnoses

<!-- MET or MISSED per criterion against LAST UNIT's target, plus a sentence on
     how you decided.

     Then, for every miss: which of the four places it happened — a tool, the
     loop's branch, the session, or the model's output — AND the mechanism.

     Not a diagnosis:  "The fit card was bad."
     A diagnosis:      "The fit card criterion missed on 2 of 5 items. Both had
                        an empty brand field. My prompt puts the brand in the
                        first sentence, so the card opened with a blank and read
                        like a fragment. The tool worked; the prompt assumed a
                        field that isn't always there."

     Look for a pattern. Three misses on the same tool is one problem, not
     three. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**



---

## Loop Trace

<!-- One full run, printed step by step, with the MCP call visible in it.

     `python app.py ask '...' --trace` once you've added the trace.step()
     calls in Milestone 2.

     Worth pasting BOTH the happy path and the empty-search path. The empty
     one should be visibly shorter, because it stops. If your two traces are
     the same length, your branch isn't working — and this is the fastest way
     anyone will ever find that out. -->

**Happy path**

```

```

**Empty search**

```

```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->



---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

**Which failure it was meant to fix:**

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 3

       [ ] criteria.md has five numbered criteria, each with a target
       [ ] Each criterion has a reason underneath it
       [ ] All five unit 3 sections above have real content
       [ ] Tool Inventory: all three tools, inputs WITH TYPES, a specific
           return value, and the empty case
       [ ] Planning Loop names the branch rule and agent.py::run_agent
       [ ] Sample Run: one full query plus the three per-tool tests, as text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN, you submit the same one
           next unit

     SUBMISSION CHECKLIST — unit 4

       [ ] mcp_server.py exists with one tool registered
           (or a written record of exactly where the rewire broke)
       [ ] Run Log — Before, five criteria, five tries each
       [ ] Real output pasted underneath, naming file and function
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, naming a place AND a mechanism
       [ ] Loop Trace, with the MCP call visible in it
       [ ] All three failure modes triggered and handled
       [ ] One improvement, with Run Log — After in the same format
       [ ] What's Still Broken
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository. Your commit history is what
     shows your criteria existed before your results did.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
