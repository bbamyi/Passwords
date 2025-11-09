CS583 — Password Cracking Report

Generated / last updated: 2025-11-09 (updated after wl1, mask_l6, and split mask_l7 runs)

SUMMARY

I used John the Ripper (Jumbo) locally built in ~/cs583/Passwords/john-src to attack the supplied shadow.txt (20 MD5-crypt hashes). My methodology combined a dictionary + rules pass and targeted brute-force passes. The initial wordlist+rules session (wl1) cracked several dictionary-based passwords; a following lowercase exhaustive length-6 mask (mask_l6) cracked one additional password. To finish remaining hashes I began a length-7 lowercase split (n–z half running, started a..m half in parallel) and also launched several short, targeted strategies (wordlist+best64 rules, suffix masks, capitalization/leet rules) to gather quick wins before the deadline. However, I couldn't make it in time. 

Tools & environment

Tool: John the Ripper (Jumbo) built from source; binary at ~/cs583/Passwords/john-src/run/john.

Session control: tmux sessions used for detached background runs; nice -n 15 used to keep jobs low priority.

Results — cracked passwords (authoritative)

Cracked entries (saved to cracked_list.txt and in john.pot):

user6 → forwarding

user7 → increased

user14 → longitude

user16 → revolutionary

user19 → cincinnati

user18 → ugrknq (cracked by mask_l6)

Total cracked: 6
Remaining: 14

Measured performance (baseline snapshots)

John’s reported checks/sec (c/s) varied across sessions — I recorded representative snapshots from logs:

wl1 (wordlist + rules): 814,211 c/s (log snapshot). 

mask_l6 run: ~732,728 c/s (log snapshot during the run).

mask_l7 (n–z snapshot): 701,868 c/s then later 613,257 c/s observed 
NOTE: I noticed that the rates fluctuate with system load and OpenMP thread counts

I used these measured c/s values to compute time estimates in the next section. For the report include the c/s value most representative of the run you cite (I recommend 814,211 c/s for baseline calculations and 732,728 c/s as the mask snapshot).

Guessing strategies employed

Wordlist + rules (wl1) — --wordlist=wordlist.txt --rules (including simple permutations)

Lowercase exhaustive masks:

mask_l6 — --mask='?l?l?l?l?l?l' (exhaustive length-6 lowercase). This managed to let me crack 1 additional password

mask_l7 — exhaustive length-7 lowercase (initial full run then split into halves to speed results): split into mask_l7_am ([a..m]) and mask_l7_nz ([n..z]) to get results potentially faster

Targeted short jobs (to maximize quick gains before deadline):

Wordlist + compact rules (best64) to try common suffixes and capitalization (comb_wd)

Suffix masks ?l?l?l?l?l?l?d and ?l?l?l?l?l?l?d?d to find word+digits patterns quickly

Capitalization & small l33t rules over the wordlist

All runs were launched in tmux, nice -n 15, and with OpenMP threads tuned to avoid oversubscription

so, concluding the above exhaustive lowercase up to L=7 is practical on Hydra per split/run strategy; full alphanumeric exhaustive search quickly becomes infeasible for L≥8.

CONCLUSION

Tool & why:
I used John the Ripper (Jumbo) because it supports MD5-crypt, flexible wordlist rules, mask attacks, sessions/recovery, and tunable OpenMP threads. 

Average rate (c/s):
Representative measured rates: ~814,211 c/s (wl1 baseline) and ~732,728 c/s (mask_l6); session-specific rates included ~701,868 c/s and ~613,257 c/s during mask_l7 splits

Guessing strategies:
Wordlist+rules, exhaustive lowercase masks (L=6 then L=7 split), targeted suffix masks, combinator rules (word+digits), and small leet/capitalization rule runs

Cracked passwords & strategy:

user6: forwarding — wl1 (wordlist+rules)

user7: increased — wl1

user14: longitude — wl1

user16: revolutionary — wl1

user19: cincinnati — wl1

user18: ugrknq — mask_l6

Time-to-crack calculations:
Examples above use exact combination counts and measured R. For example, a 6-char lowercase password = 308,915,776 combos / 814,211 c/s ≈ 379.4 s.

SHA-512 and salts:

SHA-512 (with many rounds) increases the computational cost of each guess, making offline brute force slower. Although, I noticed that it does improve practical security vs MD5-crypt

Salts force per-hash work and prevent reuse of precomputed tables, but they don't make a hash immune to offline brute force if the attacker has the hashes. They just prevent the mass speed ups
Offline attacks remain important: even though online throttling prevents rapid login attempts, once hashes are stolen an attacker can run offline attacks without throttling — thus strong hashing (slow algorithms + salt) and long random passwords matter

EXTRA QUESTIONS:

1. Formulas: 
keyspace for length L: N = 62^L
Worst case time: Tmax = (62^L)/R
avg time: Tavg = Tmax/2

62^6 = 56,800,235,584
62^8 = 218,340,105,584,896
62^10 = 839,299,365,868,340,224
62^12 = 3,226,266,762,397,899,821,056

result: divide by R which is 2.5 x 10^9

for 6 chars - 56,800,235,584/R -> 22.72 sec for Tmax
for Tavg its about 11.36 second

for 8 chars - 218,340,105,584,896/R = 1 day and approx 15 min 36 sec for Tmax
however Tavg would be around 12 h 7 min

for 10 chars - 839,299,365,868,340,224/R = around 10 years for Tmax
though Tavg would be about half that, 5.32 years

for 12 chars - 3,226,266,762,397,899,821,056/R = about 40,893 years
half that for Tavg would be 20,446 years

2) Using the exact same numbers as above - the results are the same because my machine is actually a gaming PC with a 3080 RTX! (Lucky)

3) I think that meters are hueristic, meaning that they often miss the attacker's actual strategies such as wordlists, rules, and leaked patterns to name a few. Plus, you never know the site's hashing policy or an attacker's hardware. I think that a safe password should be at minimum 12-14 characters because if hashes are fast it wouldn't be impossible to crack 
even the more complex, elongated passwords. 

4) I think that SHA-512 does improve security vs MD5 if it's being used with proper iterations. The crypt (PAM $6$) costs more guesses than the MD5 crypt which would slow most offline attacks. However, because SHA-512 is still a fast hash on most GPUs I think that the best practice would be using a KDF that is tuned to be on the slower end

5) Absolutely they help! Unique salts help in defeating rainbow tables and they also stop user reuse of work across platforms. Even through they don't stop brute forcing hashes, they do make attackers per-account work 

6) I don't think so at all, because throttling may stop online guessing but any hash leak enables unthrottled offline attacks at GPU speeds. You still absolutely need strong hashing, long passwords (12+ characters), and special salts. 