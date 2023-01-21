---
title: "Programování bez frameworku"
date: 2017-08-06T15:40:35+02:00
draft: false
---

# Všichni jedou na goučku

Dočetli jste se o novém super jazyce od Googlu. Tvrdili vám, že spousta firem v něm píše své core, production critical aplikace - jako třeba:  

- Google (Kubernetes, YouTube, dl.google.com)
- Docker
- Dropbox
- DigitalOcean
- Twitter (docela zajímavé čtení - [link](https://blog.twitter.com/engineering/en_us/a/2015/handling-five-billion-sessions-a-day-in-real-time.html))

[a další](https://github.com/golang/go/wiki/GoUsers)

Stejní lidé vám řekli, že nejlepší je používat _go_ bez těžkopadných frameworků, jichž jsou ostatní jazyky plné a že je mnohem 
lepší použít využít bohatou standardní knihovnu, kterou udržují samotní tvůrci. 

Po napsání pár převážně webových aplikací jim musím dát za pravdu. Pokaždé si vystačím s všehovšudy se 3-10 externími závislostmi. To je asi stokrát míň, než u nodejs. Většinou jde o malé knihovny řešící specifický problém (např. `github.com/pkg/errors` nebo `github.com/gorilla/mux`). 

# Svazující framework 

Pakliže ke go přicházíte ze světa nodejs, pak vás tento fakt nevyvede z míry: tam, kde byste v node použili express, použijete `net/http` ze standardní knihovny a tam, kde byste hledali v npm balíčcích, ten nejvhodnější pro šifrování hesla, podívate se přímo do dokumentace [crypto/bcrypt](https://godoc.org/golang.org/x/crypto/bcrypt) package. Mimochodem, u `golang.org/x/crypto/bcrypt`, to _x_ znamená, že jde o knihovnu od golang týmu, ale nejsou na ní tak vysoké nároky na kompatibilitu, jako u package ze standardní knihovny.

Pokud přícházíte z jazyka jako je php nebo java, pak to pro vás bude větší šok. Jste zvyklí používat jeden framework, který řeší všechny světové problémy. Jedná se o úplně jiný přístup k programování. Jazyk tady slouží spíše jako garant syntaxe, sémantiku určuje konkrétní framework. To vám přineslo hodně úspory času, protože nemusíte psát už jednou napsané - v tom dobrém případě. V tom horším je váš usecase netradiční, pak hodně štěstí s ohýbáním funkcionalit. Vaše programování sestává spíše z lepení kusů kódu dohromady, než z vytváření něčeho nového.

# Shrnutí

Go je nový (né tak docela) programovací jazyk napsaný programátory, kteří se podíleli na tvorbě jazyka C, zaštítený Googlem, který [nabírá na popularitě](https://www.tiobe.com/tiobe-index/). Jedná se o kompilovaný, silně typový jazyk, který si zakládá na vysoké paralelizovatelnosti a přehledné syntaxi. 

Tady na tom blogu se budu snažit ukázat, proč si myslím, že způsob programování v _golang_ je daleko příjemnější - proč je výsledný kód lépe čitelný udržovatelný.