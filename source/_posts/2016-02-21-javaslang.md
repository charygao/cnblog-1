title: Javaslang - Functional-Boost for Java
date: 2016/02/21 12:00
authorId: JZM
tags: [java, funkcionalní programování]
---

Java nám ve verzi 8 dala částečně přivonět k funkcionálnímu programování - nadšeně jsme tedy přešli z features které nám poskytovala Guava na nativní možnosti. Ale na jak dlouho nám budou stačit? Na poli funkcionálních knihoven se objevil nový, silný hráč - [Javaslang](http://www.javaslang.io/). 

<!-- more -->


Na co nás autoři knihovny můžou nalákat? Je to další z knihoven která upadne v zapomnění, nebo se jednou stane součástí javy?

#### Tuple 
Kolikrát jste potřebovali do libovolné struktury vmáčknout více informací a kolikrát jste si pro tuhle situaci vytvořili speciální sdružovací třídu která neměla žádný jiný účel a nehodilo se pro ni žádné doménově specifické jméno? No jo, a jak pak takovou třídu, nesoucí kolem třeba String a Long pojmenovat? Uncle Bob by Vás asi praštil za název `StringAndLongBecauseSimplyThisPieceOfCodeIsUsedForCarryingStringAndLongAndHasNoSpecialMeaning`? Javaslang se inspiroval Scalou a dalšími, a přidal koncept Tuple hodnot.  

Název Tuple je odvozen ze série násobných číslovek (single, double, triple, quadruple, quin_tuple_) a označuje sekvenci hodnot. A jak byste určitě od funkcionální knihovny čekali, takové Tuple bude immutable struktura.   

Kurzovní lístek ve směnárenské aplikaci pak může vypadat třeba takto:

```java 
        HashMap<Tuple2, Double> exchangeOffice = HashMap.of(
            Tuple.of("CZK", "USD"), 0.04119,
            Tuple.of("EUR", "CZK"), 27.0271911,
            Tuple.of("DKK", "CZK"), 3.62102452
        );
```

Jako klíč v mapě se tentokrát nabízela dvojice konvertibilních měn a jako hodnota kurz pro dnešní den. Mapa pochází také z Javaslangu, krom jiného přepracovává i kolekce, ale o tom potom.

Javaslang přidává Tuple pro 0 až 8 hodnot, což je oproti Scale trochu méně, tam se tvůrci rozhodli pro limitování Tuple na 22 hodnot.

Proč bych měl používat Tupláky místo vlastních výtvorů?
 - Immutable 
 - Comparable
 - Serializable
 - Mapovací funkce umožnující transformaci jednotlivých elementů tuple tak i celého tuple 
 
#### Value

Druhým pilířem javaslang knihovny je `Value`, interface se spoustou zajímavých default metod, implementující Iterable a umožňující vcelku dobrou zpětnou kompatibilitu s Javou.

Například naši HashMapu můžeme lehce přehodit zpátky do Java Mapy a to pomocí:
`Map<Tuple2, Double> backToJavaMap = exchangeOffice.toJavaMap(entry -> Tuple.of(entry._1(), entry._2()));`

Na Value můžeme nahlížet jako na abstrakci immutable struktur, usnaďnujících práci programátora. 

Jednou z nich je `Option`, obdoba monády `Optional` z javy. Stejně jako v Javě se jedná o kontejner, který může a nemusí nést hodnotu a který náš kód odpleveluje od null checků.

Deklarace vypadá identicky jako java 8 `Option<String> string = Option.of("String");` a dá se lehce převést.
Ať už na `Optional` pomocí `toJavaOptional()` nebo třeba na list :) `toJavaList()`.

za zmínku stojí i monadické zpracování vyjímek, za pomoci `Try`. Příklad: 

```java
    Try.of(JavaSlang::mayThrowRuntimeException)
        .andThenTry(JavaSlang::mayThrowDifferentException)
        .onFailure(JavaSlang::logError)
        .onSuccess(JavaSlang::persist);
```

Nahrazuje try-catch blok v javě a podobně jako se Option skládá z nějaké nebo žádné hodnoty, výsledek try se skládá ze Success nebo z Failure. Jak je vidno z příkladu, v několika málo řádcích můžeme vyvolat několik příkazů v chráněném bloku, definovat, co se stane pokud nastane vyjímka (lze i reagovat různě na různé druhy vyjímek) a v neposlední řadě použít návratovou hodnotu volání dál.

Další Values jsou `Future` a `Lazy`, ale o těch už se tu nebudu rozepisovat. Pokud stále nechcete vyzkoušet Javaslang, nejspíš Vás ani ony nepřesvědčí.

#### Funkce

Javaslang o sobě prohlašuje, že jeho λ je functional interface v javě na steroidech krát 10. A něco na tom bude. V Javě končíme s `BiFunction`, neboli funkcí se dvěma argumenty. Javaslang má kompatibilní `Function2`. Jméno se mi napřed nezdálo, ale když jsem si přečetl, že javaslang končí u funkce s 8 parametry, neboli Function8, pochopil jsem.  

Jako malou ochutnávku si můžeme dát příklad, kdy v obecné funkci f2 zafixujeme první parametr. Jedná se fci použitou v sekci později, respektive o její obecnou podobu a poté o převod této obecné podoby do stavu, který je použit pro přepočet kurzu.

```java
    Function2<Double, Double, Double>  obecnaFceADelenoB = (a, b) -> a / b;
    Function1<Double, Double> prevodKurzu = obecnaFceADelenoB.curried().apply(1d);
```

Pro ty, co si nejsou jisti co se stalo přidávám [vysvětlení](https://en.wikipedia.org/wiki/Currying)

No jo, ale co když nám nějaký dobrák napíše `prevodKurzu.apply(0)`? Vyjímka, rekonvalescence nebo smrt. Pro tyhle případy můžeme použít lifting, který z parciální fce vyrobí fci o něco kompletnější. 

```
    Function1<Double, Option<Double>> safeFce = Function1.lift(prevodKurzu);
    safeFce.apply(0d); // pryc s neocekavanyma vyjimkama
```

Co se týká funkcí, vydaly by na samostatný článek, ale snad Vás aspoň pár příkladů přesvedčilo, že je javaslang víc než použitelný.

 
#### Vlastní mapy a kolekce
Tvůrci nebyli spokojeni s nabídkou kolekcí nabízenou v javě, v dokumentaci zmiňují jako jeden z důvodů pro vytvoření vlastní řady kolekcí fakt, že kolekce v javě byly navrženy pro objektově orientované programování, nikoli pro funkcionální a jít cestou wrapperů ( `Collections.unmodifiableList(otherList);`) jim nepřišlo správné.

Knihovna definuje `Traversable` jako společného předka pro vše, co se dá procházet. Najdeme zde `Seq`, `Set` a `Map` interfejsy s vlasními implementacemi Listu, HashMapy i HashSetu. 

Javaslang přidavá také poměrně bohatou sbírku immutable kolekcí a map a činí některé z úkonů méně ukecanými. 
Pro příklad si zkusíme udělat opačný kurzovní lístek ze sekce o Tuples. Jak by takovýto úkon vypadal v Javě bez Javaslangu?

```java 
    // HashMap(((CZK, EUR), 0.03699977538546357), ((USD, CZK), 24.277737314882256), ((CZK, DKK), 0.2761649346688213))
    exchangeOffice.map((t, d) -> Tuple.of(Tuple.of(t._2(), t._1()), 1d / d));
```


Samozřejmostí je pak streamové zpracování, ale o tom snad potom.

_to by bylo prozatím vše, javaslang skýtá mnohé libůsty a budu rád, když článek rozšíří, opraví, doplní, nebo mě dokope napsat pokračování_

