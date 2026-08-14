Goal
    Build a personal website that matches my style and has all the important stuff that i need to showcase

# V1
Resources
    Screenshot in the project folder 
    Markdown version of my resume 

General instructions
    Don't copy the resume as it is. Only keep the features that would look good on the portfolio website.
    I will update all my socials that at the end. 
    I want to deploy the website using github. So help me with that.
    Content of the website should be exactly how I want it to be.
    Ask me questions to make decisions regarding the website.
    Add a blog section for me to shower my thoughts whenever I feel like
    The website should feel both like a portfolio website and a personal website. 

Verificaiton
    The output website should be similar to the screenshot I have attached.

Output
    A fully working website that I can deploy using GitHub. 

## V1.5
Changes
    - Change the bio to "I like to overanalyze stuff, and watch anime."
    - Use these colors: `#100E13` , `#17131C` , `#EEEAF2` , `#AAA1B5` , `#706778` , `#A78BFA` , `#C4B5FD`  , `#30283A`
    - Use this: 
    Hi, I'm Karthikeshwar — I study Mathematics and Computing at the Indian Institute of Science, Bengaluru. I like anything mathematical and logic related which includes a range of stuff like Probability & Statistics, Discrete Mathematics and Neural Networks. 

    Most recently, I spent a summer at Emergence AI applying formal verification to a 2D physics engine — proving collision resolution and anti-tunneling guarantees using exact rational arithmetic. Outside of that, I've worked on adversarial robustness in vision models and built ML pipelines for real-world problems like traffic congestion in Bengaluru.
    - Remove the tags for projects
    - Add these socials:
        Github github.com/CKarthikeshwar
        LinkedIn linkedin.com/in/karthikeshwar-chimirela-678b33356?
        X x.com/C_Karthikeshwar
    - Add a pic of me it's with the name CK in this folder
    - Change Math & Computing to Mathematics & Computing
    - Remove the education section at the bottom of the page
    - Make the read my writing a button, it's too hidden right now
    - Don't keep the backgroud so blank, just have some small minimal stuff. It looks outirght blank now. I don't want anything to0 fancy though

## Plan after Changes

Clarified first:
    - LinkedIn URL: drop the trailing "?" — use `https://linkedin.com/in/karthikeshwar-chimirela-678b33356`.
    - Instagram: not in the V1.5 social list, so it stays as a placeholder (`#`) in the sidebar for now.
    - Background treatment: soft violet gradient blobs (low-opacity radial gradients in the new accent colors), fixed behind the content — fits "small minimal stuff, nothing too fancy."

1. Sidebar tagline bio (`src/components/Sidebar.astro`, one-liner under the name) → replace with "I like to overanalyze stuff, and watch anime."
2. About section copy (`src/components/Hero.astro`) → replace the whole section with exactly the two paragraphs given above (the current third paragraph about competitive programming gets dropped — the new sidebar bio now covers personality, and this section has exact replacement text).
3. Color palette (`src/styles/global.css` `@theme` block) → remap the 8 hex values onto the existing 7 tokens plus one new one, by lightness/role:
    - `--color-bg: #100E13`
    - `--color-bg-elevated: #17131C`
    - `--color-text-primary: #EEEAF2`
    - `--color-text-secondary: #AAA1B5`
    - `--color-text-muted: #706778`
    - `--color-accent: #A78BFA`
    - `--color-accent-light: #C4B5FD` (new token — hover states, gradient blobs)
    - `--color-border: #30283A`
4. Remove project tags (`src/components/Projects.astro`) → drop the `tags` field and tag-list markup from the Projects component only (Experience tags stay — this only calls out projects).
5. Real social links (`src/components/Sidebar.astro` `socials` array) → GitHub → `https://github.com/CKarthikeshwar`, LinkedIn → `https://linkedin.com/in/karthikeshwar-chimirela-678b33356`, X → `https://x.com/C_Karthikeshwar`, Instagram stays `#` placeholder.
6. Profile photo → copy `CK.jpg` into `public/` (e.g. `public/ck.jpg`), add a circular `<img>` at the top of `src/components/Sidebar.astro`, above the name.
7. Copy fix: "Math & Computing" → "Mathematics & Computing" in the Sidebar tagline (`src/components/Sidebar.astro`).
8. Remove Education section → delete `src/components/Education.astro`, remove its import/usage from `src/pages/index.astro`.
9. "Read my writing" as a button (`src/pages/index.astro` footer) → restyle from a plain text link into a visually distinct button: accent-colored border/fill, padding, rounded corners — no longer a barely-visible mono-text line.
10. Subtle background (`src/layouts/BaseLayout.astro` + `src/styles/global.css`) → 1–2 large, heavily blurred radial-gradient shapes using `--color-accent` / `--color-accent-light` at low opacity (~10–15%), fixed behind the content, `pointer-events: none`, `aria-hidden`. No new dependencies, no animation.



