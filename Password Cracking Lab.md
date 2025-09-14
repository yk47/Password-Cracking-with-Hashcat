# Password Cracking with Hashcat and John the Ripper
## Crack password hashes using dictionary and brute-force attacks with tools like Hashcat and John the Ripper.

### 1. Objective

Crack password hashes using dictionary and brute-force attacks with tools like Hashcat and John the Ripper. Demonstrate how weak passwords are cracked and provide defensive recommendations.



### 2. Environment

OS: Kali Linux (VM)

Hashcat version: v6.2.6

John the Ripper (Jumbo): specify version used

CPU/GPU: Intel Core i5-7300HQ (CPU-only used in lab) — replace with GPU if available

Wordlists used: /usr/share/wordlists/rockyou.txt, custom_wordlist.txt



### 3. Files in this repo

hashlist.txt — input hashes

custom_wordlist.txt — small candidate list (lab-only)

cracked_all.txt — final merged cracked results

commands.txt — exact commands run

methodology.md — steps & strategy

screenshots/ — terminal evidence 


### 4. Hashes tested
5f4dcc3b5aa765d61d8327deb882cf99

d8578edf8458ce06fbc5bb76a58c5ca4

5ebe2294ecd0e0f08eab7690d2a6ee69

e99a18c428cb38d5f260853678922e03

606613ee02c515729dacd89c24cc93b8

<img width="822" height="642" alt="hashlist" src="https://github.com/user-attachments/assets/147850de-9da8-46ea-8cf6-db7f0d305de1" />


### 5. Steps & commands (detailed)
Start here — this section documents the exact commands you ran, the screenshots that show each step, and the outputs used in the report.

#### 5.1 Clean & prepare
#remove empty lines and trailing whitespace from the hash file

sed -i '/^$/d; s/[[:space:]]\+$//' hashlist.txt


#verify hashes (view file)

cat hashlist.txt

#### 5.2 Wordlists available
#list available wordlists (rockyou is expected in /usr/share/wordlists)

ls -la /usr/share/wordlists

Screenshot — wordlists folder
<img width="968" height="680" alt="wordlist" src="https://github.com/user-attachments/assets/6c1f6892-170b-4e5d-88d5-811fd807e759" />



#### 5.3 Hashcat — start (merged / rules run)

Run Hashcat with rockyou.txt and rules (example):
hashcat -m 0 -a 0 hashlist.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule \
--outfile=cracked_hashcat_rules.txt --outfile-format=2 --status --potfile-path=./hashcat.pot

Screenshot — Hashcat starting, done and status 
<img width="1088" height="716" alt="hashing_start" src="https://github.com/user-attachments/assets/12229061-1094-4a97-895d-00693737c9f8" />
<img width="1920" height="1051" alt="hashing_done" src="https://github.com/user-attachments/assets/b7100270-11bd-4c92-b090-12bc29d8a5f4" />



#### 5.4 Hashcat — status and output

During the run Hashcat may report that some hashes were already found in the potfile. If so, use hashcat --show or run with a fresh --potfile-path.

#if Hashcat reports "All hashes found as potfile...", show them

hashcat --show hashlist.txt > cracked_hashcat_rules.txt || true


#inspect cracked file

cat cracked_hashcat_rules.txt
Screenshot — Hashcat status & progress

 Hashcat progress output. The session summary shows what was recovered and the speed.
<img width="1920" height="1051" alt="hashing_status" src="https://github.com/user-attachments/assets/ff1f60ec-65ea-4723-83f4-804f7c4f74d4" />

Screenshot — final Hashcat output (cracked list)
cracked_hashcat_rules.txt contents — the hash:plaintext pairs recovered by Hashcat.
<img width="822" height="642" alt="cracked_hashes" src="https://github.com/user-attachments/assets/df88c63a-c241-48d7-b346-6bb95453e6a8" />



#### 5.5 Hashcat — combination with custom list

We used a small custom_wordlist.txt and combined it with rockyou.txt using combination mode (-a 1) to generate candidates of the form custom+rockyou.
#preview combinations (first 50)

hashcat --stdout custom_wordlist.txt /usr/share/wordlists/rockyou.txt | head -n 50


#run combination mode

hashcat -m 0 -a 1 hashlist.txt custom_wordlist.txt /usr/share/wordlists/rockyou.txt \
--outfile=cracked_hashcat_combo.txt --outfile-format=2 --potfile-path=./hashcat_combo.pot

Screenshot — Hashcat completed combination run and recovered remaining hash
<img width="1920" height="1051" alt="combo_hashing_start" src="https://github.com/user-attachments/assets/0eb7d085-d996-4e7b-9254-6cf1bd8af998" />

Summary after combination run. Note the Candidates.#1 line showing sample generated candidates and Recovered......: 5/5 indicating all five hashes were recovered.
<img width="1920" height="1051" alt="All_hashes_found" src="https://github.com/user-attachments/assets/5baea3ad-5eb9-4ad4-a588-4c594e4a5d95" />


#### 5.6 John the Ripper — dictionary run and show
Run John with a fresh potfile and the custom wordlist (or merged list):
#run john with custom wordlist

john --pot=./john_custom.pot --wordlist=custom_wordlist.txt --format=raw-md5 hashlist.txt


#show cracked results and save

john --show --format=raw-md5 hashlist.txt | tee cracked_john.txt

<img width="1920" height="1051" alt="John1" src="https://github.com/user-attachments/assets/1c4878c9-17f6-408b-9505-bced183cd6d2" />
<img width="1920" height="1051" alt="john2" src="https://github.com/user-attachments/assets/a69cecdb-f830-4c7a-bcae-e5075c4acf00" />
<img width="1920" height="1051" alt="John3" src="https://github.com/user-attachments/assets/ff56c282-b689-4dcf-8fc2-424a82de5f7e" />

#### 5.7 Merge results & final cracked file
#merge hashcat and john outputs (dedupe)

cat cracked_hashcat_rules.txt cracked_john.txt 2>/dev/null | sort -u > cracked_all.txt


#view final merged file

cat cracked_all.txt

Final cracked list (example)

5f4dcc3b5aa765d61d8327deb882cf99:password

d8578edf8458ce06fbc5bb76a58c5ca4:qwerty

5ebe2294ecd0e0f08eab7690d2a6ee69:secret

e99a18c428cb38d5f260853678922e03:abc123

606613ee02c515729dacd89c24cc93b8:Hi, I am Yash Karnik and this is Task 2 of broskiesHub internship

<img width="1920" height="1051" alt="All_hashes_found" src="https://github.com/user-attachments/assets/5238d16f-aaec-49d4-b017-6d584624f059" />


