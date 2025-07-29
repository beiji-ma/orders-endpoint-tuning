# Appendix: Rotlösa Rötter — Arkitektoniska Kōan för MMSA (på Svenska)

> Arkitekten skyndar inte.  
> Långsammare än en quickfix, snabbare än ånger.  
> Griper ej efter teknik, klamrar sig inte fast vid plåster.  
> Ser form, följer åtkomst, bevarar struktur.

---

### 🪷 1. Före struktur, allt är lidande

En utvecklare frågade: Varför är min fråga så långsam?  
Mästaren svarade: För att du aldrig visste vad du verkligen ville ha.  
Du hämtade varje fält men visade bara ett.  
Du fruktade lazy loading, men gav aldrig namn åt din åtkomstväg.  

Utan struktur – hur ska du nå hastighet?

---

### 🪷 2. Caching är ett plåster, struktur är medicinen

Caching döljer skammen av utebliven modellering.  
Du cacheade resultatet, men blundade för vägen.  

Att lita på Redis är som att flyta på drivved –  
Det kan bära dig, eller sjunka.  

Den som ser struktur, fruktar inte cachemissar.

---

### 🪷 3. JPA: Halvsanningar och illusioner

ORM följer dig som en skugga.  
Den mappar data, men visar dig aldrig vägen.  
Den genererar SQL, men döljer din avsikt.  

Att lita helt på ORM är att förblindas av yta.  

Att se dess begränsningar och välja annat – det är första porten till MMSA.

---

### 🪷 4. Stabilitet är att inte behöva fixa

Ett stabilt system saktar inte ner med storlek, brister inte vid tillväxt.  
Det jagar inte mångsidighet, utan tydlighet.  

Dess kärna är tre saker:  
- Struktur  
- Åtkomst  
- Observerbarhet  

Med dessa tre – vad finns att frukta?

---

### 🪷 5. DSL: Mångfald blir enhet

Många händer, många format – entropi följer.  
API:er sväller. Dokumentationen frodas.  

Bara språk kan förena avsikt.  
Bara syntax fångar form.  

Med en DSL består vägen.  
Detta är återgången till källan – hjärtat i MMSA.

---

> Du undrar vad vägen är?  
> Den räddar dig inte från dagens deploy.  
> Men den kan rädda dig från tio år av kaos.

