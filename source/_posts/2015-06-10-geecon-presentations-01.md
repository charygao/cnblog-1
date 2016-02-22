title: Letem Světem s Geekoštětem (1)
date: 2015/06/10 12:00
authorId: JZM
---

Ve článku se dozvíte, co autora na letošním GeeCONu zaujalo (a že jej toho zaujalo hodně).  
_Pokud Vás jakýkoli z výcuců zaujme a přáli byste si vědět více, neváhejte napsat. Vždy se můžeme domluvit na vnitrofiremní INDEED nebo na další, podrobnější článek na blog._
 
 
<!-- more -->

###Dojmy z GeeCON 2015 - Krakow



  {% img center-block /attachments/2015-06-10-geecon-presentations-01/hystrix.png 200 %}
####Hystrix - Jak na chyby ve světě služeb? 
 

> Víte jak postavit takový fault-tolerant systém? Víte co je to fail-fast princip? Pokud ne, čtěte dál. Pokud jste se rozhodli napsat vlastní framework který Vám to umožní, čtete dál.

Chcete systém, který bude schopen počítat s chybami. Řekněme že nějaká z Vašich komponent volá externí službu, která je extrémně nespolehlivá (typicky ta co je napsána v .NET). Co s tím? Je možné tolerovat chybu služby? Ano? Je vyhráno. Uživatel odejde s 80% zážitkem, místo 0% když se z chyby nevzpamatujete. A ještě jste tolerantní. 
No jo, jenomže jak na to? Zajít za projekťákem a poprosit jej o 50 mandays na nový projekt asi nebude to pravé. Ale počkat - ve světě IT se jako v každém jiném problémy opakují. A ano, jedna společnost, říkejme jí třeba Netflix, už do podobné situace zabředla. A protože v IT jsou milí lidé, stává se že uvolní plody své práce i pro ostatní - pod svobodnou licencí. Z nadpisu už víme, že nástroj se volá Hystrix. Jestli je to podle hystericky pobíhajících manažerů na chodbách, autor článku nevěděl. Kdo to zjistí, nechť si přijde pro nálepku. Nebo k nám na pohovor. 
A jak nám Hystrix pomůže? 
Jednoduše obalíme volání externích služeb do HystrixCommandu a o ostatní se už postará on sám. Pokud zavoláme službu, která z nějakého důvodu zlobí, Hystrix nenechá vyčerpat thread pool na volání oné služby dalšími klienty, ale rovnou nahlásí že je služba nedostupná. Spadneme tedy mnohem dřív. A tu se dostáváme k fail-fast principu. Víte-li rychle, že je něco v nepořádku, dokážete se s tím pravděpodobně rychleji poprat. Netflix myslel i na případ, kdy služba po čase znovu zresponzivní. Trochu po elektrikářsku je implementován circuit-breaker, který nám sepne, když externí systém selže. Circuit-breaker ale není definitivní, dokáže se dostat do polopropustného stavu, ve kterém se přeci jen čas od času pokusí o znovuspojení. A pokud se spojení otevře, nechá téct komunikaci dál. 
Samozřejmě je za tím spousta nastavování a svízelů, ale o těch si povíme spíš v samostatném (a odborněji laděném) článku.


-----------------------

{% img center-block /attachments/2015-06-10-geecon-presentations-01/java9.jpg 200 %}
#### Java 9 sneak preview

V minulé verzi jsme se dočkali zrušení mnohými proklínaného PermGenu. Co se s ním stalo, asi víte. A dost možná stále ignorujete, že Vám JVM hlásí neplatný parametr. Tohle se v budoucí verzi změní, JVM nahlásí neplatný parametr a odmítne s Vámi spolupracovat. Zagrepujte si tedy po *-XX:PermSize* a *-XX:MaxPermSize*
Největší změnou ale bude pravděpodobně nové interní řešení modularity systému. Tento krok interně nazýváme "Make peace, not jars". Pro Vás, jako programátory, to ale zas tak mnoho neznamená a dost možná si ani nevšimnete, že Vaše java používá interně úplně jiný systém správy modulů než doposud. 
Pánové se také rozhodli sjednotit logování JVM, takže budeme mít třebas zas o něco veselejší grepová cvičení. 
Další výraznou změnou bude (aspoň pro obchodníky) změna verzování. Java 9 se tedy bude jmenovat... *java 9*  namísto *java 1.9*. 


------------------------

{% img center-block /attachments/2015-06-10-geecon-presentations-01/pitMutation.png 200 %}
#### Želvy Ninja testing

aneb posouváme naše testy ke hvězdám. Kód má 100% coverage ► ship it! Nebo raději ne? Víte jestli Vaše testy dávají smysl? Co třeba zkusit obrátit podmínku? Prošel test, který kód testuje? Nemůže být něco špatně? Ale vždyť máme 100% coverage! Nemůže. 
```spock
def  "Should do cool things without failing " () {
        when: myAwesomeMethodForSubstract(1, 2)
        then: true
}
 ```
Test prochází, pokrytí je. Ještě že jsme podezřívaví a nic jsme zákazníkovi nedodali. Kéž by tak existoval způsob, jak testy co nic netestují a jen suplují coverage také otestovat.. Ale počkat, neexisuje už? I tady už se naši kolegové činili a vytvořili nástroje, které Váš program zmutují a podívají se testům na zoubek. A ještě vygenerují úžasné reporty. Nebude od věci každý večer pustit Vaše testy s nějakým z mutantních frameworků a ráno při první kávě zkontrolovat, neřádil li ve Vašem kódu 100% coverage troll.  


------------------------
{% img center-block /attachments/2015-06-10-geecon-presentations-01/kamni.jpg 250 %}
#### Komunikujte mlčky 


Zamysleme se, jak dobrý posluchač jsme. Děláme někdy něco z následujícího?
* Kolega si nemůže vzpomenout na slovo. Skočíme mu do řeči? 
* Dáváme pozor co kolega říká, nebo přemýšlíme jak navážeme a posuneme se tam, kde debatu chceme?	 
* Dáváme kolegům podmínky aby mohli debatovat a přemýšlet?   

---------------------------

> _a to by bylo pro dnešek vše, príště se můžete těšit na srovnání výkonu streamů z java 8, G1 garbage collector a možná i něčem navíc._





