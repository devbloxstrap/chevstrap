Niche Prospector V9
===================

V8 repos dhoondta tha. V9 un repos ko NICHES mein badalta hai — aur usi
rate limit mein 10x zyada candidates laata hai.

Open index.html in a modern browser, add a GitHub token (step 1), enter
seeds, then Auto Discovery → Verify → Niches tab.
The token is never included in exports.


-------------------------------------------------------------------
1. NEW: NICHES TAB   (sabse bara addition)
-------------------------------------------------------------------
Ek repo niche nahi hoti. Niche wo hoti hai jahan demand asli ho,
competition patli ho, aur search results par kisi ka qabza na ho.

Har verified repo apne topics + category par group hota hai. Do ya zyada
repos jo ek topic share karein = ek niche. Har niche par:

  Niche score = Demand 30 + Site gap 22 + Consumer intent 18
              + Growth 14 + Fragmentation 10 + Unmet demand 6

  Demand         niche ke saare repos ka downloads-per-month total
  Site gap       kitne % repos ke paas koi dedicated website nahi
  Consumer       kitne % repos consumer install files bhejte hain
  Growth         median momentum (ek outlier poori niche na uchhale)
  Fragmentation  1 − (leader ka downloads share). Jahan ek project 95%
                 downloads le jata ho wo monopoly hai, mushkil entry.
                 Jahan demand paanch mid-size projects mein bati ho,
                 wahan maidan khula hai. Ye column dekh kar hi faisla
                 karein — bara download number akela kaafi nahi.
  Unmet demand   open issues ÷ stars ka average — feature gaps

Har niche row deti hai: leader repo, wo repos jinki koi site nahi, aur
tayyar "content angles" (best X 2026, <leader> alternative, free X for
windows, <gap-repo> download, X vs Y, X not working fix).
COPY button poora niche brief clipboard par daal deta hai.

Duplicate niches khud merge ho jati hain: agar screen-recorder,
video-capture aur Media Tool bilkul wahi repos rakhte hain to ek hi row
banti hai aur baqi naam "also:" line mein chale jate hain.

"Export niches" button ek alag CSV deta hai — yehi aap ka content plan hai.


-------------------------------------------------------------------
2. DISCOVERY: ~10x bara pool
-------------------------------------------------------------------
Star Banding (naya, default ON)
  V8: stars:>=300 — ek hi query, aur GitHub har query par sirf 1000
  results deta hai, is liye bade repos (scrcpy, yt-dlp) sab kuch dabaa
  dete the.
  V9: har seed 5 star tiers mein tootta hai (300..599, 600..1499,
  1500..3899, 3900..10499, >=10500). Har tier ka apna 1000-result
  ceiling hai — aur asli niche wahi chhote 300–800 star repos mein hoti
  hai jo pehle kabhi surface par nahi aate the.

Pagination (naya)
  V8 har search se sirf page 1 leta tha. V9 "Pages / query" (default 3)
  chalata hai aur khali page milte hi ruk jata hai.

Best-match relevance
  V8 seed searches par sort=updated lagata tha — yani "aaj kis ne push
  kiya", relevance nahi. V9 seeds par koi sort nahi bhejta (best match).
  Rising aur topic expansion abhi bhi sort=stars par hain.

Skip seen (naya, default ON)
  Jo repos aap pehle verify kar chuke hain wo agli discovery se nikal
  jate hain — har run naye candidates deta hai, wahi purane top repos
  nahi. "Reset seen list" se saaf ho jati hai.

Search README too (naya, default OFF)
  in:name,description,readme — wider net, thora noisy.

Behtar filtering
  - Topic expansion ab sirf un topics par chalta hai jo 2+ repos par aaye
    (V8 mein 1 bhi kaafi tha, is liye one-off shor andar aata tha).
  - Topic blacklist ab exact-match Set hai. V8 substring regex tha, is
    liye "linux" ne "linux-app" ko bhi maar diya tha aur "react" ne
    "reactive" ko.
  - Junk penalty barh gayi (-5 → -7) aur list mein cheatsheet, roadmap,
    dotfiles, config repos add hue.
  - Naye ranking signals: open-issues-to-stars (unmet demand ka bonus)
    aur 540 din se push na hone par penalty.

Budget estimate (naya)
  Discover dabane se pehle app bata deta hai kitni search requests
  lagengi aur takreeban kitna waqt (GitHub 30 searches/min deta hai).


-------------------------------------------------------------------
3. RATE LIMIT & SPEED
-------------------------------------------------------------------
ETag cache (naya)
  GitHub har response ke sath ETag deta hai. V9 use store karta hai aur
  agli baar If-None-Match bhejta hai. Agar kuch nahi badla to GitHub 304
  deta hai — aur 304 aap ke rate limit se NAHI kata. Yani wahi list
  dobara scan karna lagbhag muft hai.
  Storage IndexedDB hai. Agar aap file:// se kholte hain aur browser
  IndexedDB block kar de, to app khud memory-only cache par gir jati hai
  (usi session mein kaam karti hai, reload ke baad nahi). Behtar hai koi
  chhota local server chala lein.
  "reuse under N hrs" 0 par ho to hamesha revalidate hota hai (data
  hamesha taaza). 24 daal dein to re-scan turant aur bilkul free — score
  weights tune karte waqt kaam ka.

Release page cap (naya)
  V8 saari release pages ghumta tha — 900 releases wala repo 9 core calls
  kha jata tha. Ab "Release pages max" (default 3 = 300 releases).

Concurrency (naya)
  V8 ek waqt mein ek repo scan karta tha. "Verify threads" default 3.
  6 se upar na jayein — GitHub secondary limits laga sakta hai (app pause
  kar ke sambhal leta hai, magar dhima ho jata hai).

Alag rate-limit counters
  V8 mein search ka remaining core counter ko overwrite kar deta tha, to
  lagta tha aap ke paas sirf 30 calls bachi hain. Ab dono alag dikhte
  hain, plus cache hits.


-------------------------------------------------------------------
4. SCORING
-------------------------------------------------------------------
Tunable weights (naya — step 4)
  Sliders: Demand, Growth, Website gap, Consumer intent, Freshness,
  DL/star, Unmet demand. Score hamesha 0–100 par normalise hota hai,
  chahe weights kuch bhi hon. Badalte hi table dobara score ho jata hai —
  koi re-scan nahi, koi API call nahi.
  Presets: Default · SEO/content hunter · Product clone hunter ·
  Early/rising only. Weights browser mein yaad rehte hain.

FIXED — momentum inflation
  V8: previous release ke 2 downloads aur nayi ke 8000 = 4000x momentum,
  yani growth score jhoot par max. Ab release momentum tabhi ginta hai
  jab pichli release ke kam az kam 100 downloads hon, aur poora momentum
  10x par clamp hai.

FIXED — freshness
  V8 sirf release date dekhta tha, to roz develop hone wala repo jisne
  saal se release nahi ki 1/10 par gir jata tha. Ab pushed_at bhi ginta
  hai (75% weight par).

NEW — unmet demand
  open issues ÷ stars. Bohot saare open issues nisbatan stars ke = log
  cheezein maang rahe hain jo mil nahi rahin. Default weight 0 hai;
  slider se on kar lein. Stars column ke tooltip mein hamesha dikhta hai.

FIXED — sorting
  Error rows ke paas kai fields hote hi nahi the, undefined comparison
  sort ko torh deta tha. Ab har column safe hai.


-------------------------------------------------------------------
5. KEYWORDS
-------------------------------------------------------------------
V8 sirf "name + suffix" deta tha. V9 KW button par:
  - asli search intent: is X safe, how to install X, X not working, X vs
  - README ke H2/H3 headings se asli feature words (boilerplate headings
    jaise Installation/License/Contributing chhoot jate hain)
  - repo ke topics se "best <topic> 2026", "free <topic> for windows"
  - agar repo ke paas site nahi: "X official site", "X portable"
  - us repo ki niche ke leader ka naam: "<leader> alternative" — sab se
    zyada intent wala keyword


-------------------------------------------------------------------
6. CHHOTI CHEEZEIN
-------------------------------------------------------------------
- Heading "V6" likhi thi jabke app V8 tha — theek kar di.
- Repo CSV export mein naye columns: open_issues, issue_ratio,
  days_since_push, topics. Import bhi inhe wapas parh leta hai, is liye
  purani export se bhi niches ban jati hain.
- Import kiya hua data hamesha mojooda weights par dobara score hota hai.
- Discovery checkpoint key np_discovery_v8 → np_discovery_v9, taake
  purana V8 checkpoint naye signature se na takraye.
- 300 repos par har row ke baad poora DOM banane ki jagah render ab
  throttled hai.
- Scan ke doran ETA dikhta hai.


-------------------------------------------------------------------
7. RUK JANE PAR CANDIDATES ZAAYA NAHI HOTE  (V9.1)
-------------------------------------------------------------------
Pehle candidates sirf kaamyab run ke bilkul aakhir mein step 3 ke box
mein likhe jate the. Beech mein ruk jane par 700+ repos sirf localStorage
mein reh jate the aur nazar hi nahi aate the.

Ab:
  - Candidates LIVE box mein likhte rehte hain (har checkpoint par).
    Discovery chalte chalte bhi aap dekh sakte hain kya mil raha hai.
  - Run ruk jaye to jitne mil chuke hain wo box mein maujood hote hain —
    turant Verify kar sakte hain, resume ka intezar zaroori nahi.
  - Page refresh ho jaye to bhi checkpoint ke candidates box mein wapas
    aa jate hain.
  - Resume karne par jo mil chuka tha wo dobara nahi dhoonda jata.

Aur run ab itni aasani se rukti nahi:
  - Ek kharab query poori discovery nahi maarti — skip ho kar aage barh
    jati hai (aur resume par dobara try hoti hai).
  - GitHub ka 500/502/503 temporary hichki samjha jata hai, exponential
    backoff ke sath 5 dafa retry hota hai.
  - Search ka 422 ("sirf pehle 1000 results") ab error nahi, us query ka
    anjaam samjha jata hai.
  - Network blip par 3 retries. Lekin lagataar 3 queries network par
    marein to app maan leti hai internet hi gaya hai aur ruk jati hai —
    har query par 9 second ghisatne ke bajaye.

Agar phir bhi kuch localStorage mein phansa lage, console (F12) mein:
  copy(JSON.parse(localStorage.np_discovery_v9).candidates
    .sort((a,b)=>b.rank-a.rank).map(x=>x.repo).join('\n'))


-------------------------------------------------------------------
8. AUTO MODE — khud discover karta rahe  (V9.2)
-------------------------------------------------------------------
Step 5 mein "Auto mode ON" tick karein. Uske baad app ek timer par khud
discovery (aur chahein to Verify + rank bhi) chalati rehti hai — koi
click zaroori nahi.

  Har kitne ghante mein   agla auto-run kab chale (1–168 hrs)
  Naye seeds / cycle      har run ke baad kitne naye emerging topics
                           khud seed list mein jur jayein (0 = off)
  Auto Verify             discovery ke turant baad Verify + rank bhi
                           khud chala de, taake Niches tab bhi apne aap
                           taaza ho jaye
  Self-expanding seeds    har cycle jo naye topic clusters milte hain
                           unmein se sabse zyada common wale khud seed
                           ban jate hain (aap ki asli seeds hamesha
                           mehfooz rehti hain, list 20 se upar jaye to
                           purane auto-added seeds pehle hatate hain) —
                           is se app hamesha wahi purani jagah nahi
                           dhoondti, khud naye niches ki taraf barhti hai

Zaroori: ye sab kuch browser tab ke andar chalta hai — koi background OS
service ya server nahi hai. Tab band ho ya laptop so jaye to timer ruk
jata hai; wapas tab kholte hi jitna waqt guzar chuka wo hisaab mein aata
hai aur zaroorat ho to turant ek run ho jati hai. Token bhi save hona
zaroori hai (step 1 mein "Remember token"), warna auto mode har baar
token ka intezar kar ke skip karta rahega.

STOP BUTTONS — Discover candidates, Verify + rank niches aur Auto mode
teeno ke apne "■ Stop" button hain. Dabate hi chalu request turant chhorh
di jati hai (rate-limit wait ke beech mein bhi) — poora interval ya poori
list khatam hone ka intezar nahi karna parta. Discovery/Verify ko rok kar
jo candidates ya results ab tak mil chuke the wo box/table mein maujood
rehte hain, "Discover"/"Verify" dobara dabate hi wahin se aage barh jate
hain. Auto mode ka Stop chalu cycle ko bhi turant rok deta hai, sirf agla
scheduled run cancel nahi karta.


-------------------------------------------------------------------
TAJWEEZ KARDA WORKFLOW
-------------------------------------------------------------------
1. Seeds mein 6–10 themes likhein. Min stars 250–400 rakhein — wahin
   asli niches hoti hain.
2. Star Banding ON, Pages/query 3, Skip seen ON. Budget line parh lein.
3. Discover chalayein. Rate limit khatam ho to chhorh dein — khud pause
   aur resume karta hai, refresh ke baad bhi.
4. Max candidates 200–300 rakhein, phir Verify. Threads 3.
5. Niches tab kholein. Sort by Niche score.
6. Wo rows dhoondein jahan: No-site 60%+, Leader share 85% se KAM, aur
   DL/month asli ho. Wahi khula maidan hai.
7. COPY se niche brief lein ya "Export niches" se poora CSV.
8. 30 din baad wahi list dobara scan karein — cache ki wajah se lagbhag
   free hai, aur Total DL par measured growth aa jayegi.
