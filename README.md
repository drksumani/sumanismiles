from pathlib import Path
import zipfile, shutil, re

zip_in = Path("/mnt/data/all_smiles_dental_website.zip")
work = Path("/mnt/data/sumani_smiles_website_v2")
if work.exists():
    shutil.rmtree(work)
work.mkdir()

with zipfile.ZipFile(zip_in) as z:
    z.extractall(work)

shutil.copy2("/mnt/data/IMG_4962.jpeg", work / "logo.jpeg")

html_path = work / "index.html"
css_path = work / "styles.css"

html = html_path.read_text(encoding="utf-8")
html = html.replace("<title>All Smiles Dental | By Dr. Khlea Sumani</title>",
                    "<title>Sumani Smiles | All Smiles Dental</title>")
html = html.replace("All Smiles Dental is a modern dental practice",
                    "Sumani Smiles is a modern dental practice")
html = html.replace("All Smiles Dental is currently preparing for opening.",
                    "Sumani Smiles is currently preparing for opening.")
html = html.replace("Columbus, Ohio", "Gahanna, Ohio 43230")

old_brand = '''<a class="brand" href="#top" aria-label="All Smiles Dental home">
      <img src="logo.jpeg" alt="All Smiles Dental by Dr. Khlea Sumani logo">
    </a>'''
new_brand = '''<a class="brand" href="#top" aria-label="Sumani Smiles home">
      <img src="logo.jpeg" alt="All Smiles Dental by Dr. Khlea Sumani logo">
      <span class="brand-name">SUMANI <b>SMILES</b></span>
    </a>'''
html = html.replace(old_brand, new_brand)

start = html.find('<div class="contact-card">')
if start != -1:
    end = html.find('</div>', html.find('</div>', start) + 1)
    # Find the closing contact-card by counting divs.
    pos = start
    depth = 0
    end = None
    for m in re.finditer(r'</?div\b[^>]*>', html[start:]):
        tag = m.group(0)
        if tag.startswith('</div'):
            depth -= 1
            if depth == 0:
                end = start + m.end()
                break
        else:
            depth += 1
    if end:
        card = '''<div class="contact-card">
          <div><span>Practice</span><strong>All Smiles Dental</strong></div>
          <div><span>Brand</span><strong>Sumani Smiles</strong></div>
          <div><span>Doctor</span><strong>Dr. Khlea Sumani</strong></div>
          <div><span>Address</span><strong>1375 Cherry Way Drive<br>Suite 200<br>Gahanna, Ohio 43230</strong></div>
          <div><span>Phone</span><strong>614-383-7253</strong></div>
          <div><span>Status</span><strong>Opening Soon</strong></div>
        </div>'''
        html = html[:start] + card + html[end:]

# Add a visible/clickable phone number before the contact card if not already present.
if "href=\"tel:+16143837253\"" not in html:
    marker = '<div class="contact-card">'
    html = html.replace(marker, '<a class="phone" href="tel:+16143837253">614-383-7253</a>\n        ' + marker, 1)

html = html.replace(
    '<p>© 2026 All Smiles Dental by Dr. Khlea Sumani. All rights reserved.</p>',
    '<p><strong>SUMANI SMILES</strong> · All Smiles Dental by Dr. Khlea Sumani</p><p>1375 Cherry Way Drive, Suite 200 · Gahanna, Ohio 43230 · 614-383-7253</p><p>© 2026 All Smiles Dental. All rights reserved.</p>'
)

html_path.write_text(html, encoding="utf-8")

css = css_path.read_text(encoding="utf-8")
css += """
.brand { display:flex; align-items:center; gap:12px; }
.brand-name { font-family:Cinzel, Georgia, serif; letter-spacing:.13em; font-size:.9rem; color:var(--metal); }
.brand-name b { color:var(--cream); font-weight:600; }
.phone { display:inline-block; margin-top:18px; color:var(--metal); font-family:Cinzel, Georgia, serif; font-size:1.45rem; letter-spacing:.05em; }
footer strong { color:var(--metal); letter-spacing:.12em; }
@media (max-width:800px) { .brand-name { display:none; } }
"""
css_path.write_text(css, encoding="utf-8")

(work / "README.txt").write_text(
"""SUMANI SMILES / ALL SMILES DENTAL WEBSITE — VERSION 2

Brand: SUMANI SMILES
Practice: ALL SMILES DENTAL
Doctor: DR. KHLEA SUMANI

Address:
1375 Cherry Way Drive, Suite 200
Gahanna, Ohio 43230

Phone:
614-383-7253

Before publishing:
- Add a professional email address
- Add opening date when known
- Add online scheduling when ready
- Add insurance/payment information
- Add privacy/accessibility/compliance pages as appropriate
""", encoding="utf-8")

zip_out = Path("/mnt/data/sumani_smiles_website_v2.zip")
with zipfile.ZipFile(zip_out, "w", zipfile.ZIP_DEFLATED) as z:
    for p in work.iterdir():
        z.write(p, p.name)

print("Created:", zip_out)
print("Brand: SUMANI SMILES")
print("Practice: ALL SMILES DENTAL")
print("Address: 1375 Cherry Way Drive, Suite 200, Gahanna, Ohio 43230")
print("Phone: 614-383-7253")
