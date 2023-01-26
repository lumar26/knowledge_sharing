# Logging - best practices

## Uvod

***Logger*** je alat čija je uloga da prikuplja i beleži interakcije korisnika sa nekim sistemom koji koristi. Sistem je u ovom slučaju softverski sistem / aplikacija / *web* servis, dok je korisnik svaki entitet koji šalje zahteve sistemu i na taj način komunicira sa njim. To znači da korisnik može da bude korisnik (osoba) ili <u>klijentska aplikacija</u>

***Log file*** je fajl koji sadrži gore navedene interakcije sa sistemom.

### Značaj

**Logovanje** u aplikaciji je bitan izvor podataka u trenucima kad je potreban dublji uvid u neki problem koji se u aplikaciji desio (greška ili neočekivano ponašanje). Pored značaja u slučaju problema logovi su zgodni za generalno praćenje stanja aplikacije. Još jedna moguća upotreba je kao izvor podataka za statistička izračunavanja.

Svaki od komercijalno korišćenih okvira ima u sebi podršku za *logging okvir*, i većina njih podržava logging `out of the box`. 

## Dobre prakse

Kod logovanja je najbitnije odrediti **koji događaji** treba da budu zabeleženi u log fajlovima. To treba da budu događaju koji nam daju uvid u ponašanje naše aplikacije. 

### Događaji

Događaji koje "obavezno" teba logovati su, između ostalih:

1. Greške u aplikaciji (izuzeci)
2. Izuzeci prilikom ulazne i izlazne validacije
3. Autentifikacija - <u>uspešna i neuspešna</u>
4. Autorizacija - <u>neuspešna</u>
5. Greške prilikom upravljanja sesijom (*stateful*)
6. Greške prilikom validacije *access* tokena (*stateless*)
7. Slanje i prijem podataka od externih sistema (npr. sistemi za upravljanje bazom podataka, externi *web* servisi, itd.)
8. *Opt-in* i *Opt-out* preference korisnika u trenutku izmene
9. Događaji vezani za *Security* okvir koji se u aplikaciji koristi

### Struktura log poruke

Kada se definiše struktura log poruke treba imati u vidu koja je upotreba te poruke i log fajlova uopšte. Najčešći slučajevi kada ćemo čitati log fajlove prilikom traženja *bug*-a i nastalog problema, praćenje stanja aplikacije ili praćenje toka događaja koji se dešavaju za neki specifičan zahtev. U skladu sa tim slučajevima treba dizajnirati strukturu log poruke, kako bi se gore navedeni poslovi maksimalno olakšali.

Ono što standardna log poruka treba da sadrži je:

- Akcija koja se desila
- Ko je onaj koji izvršava akciju (korisnik / program / funkcija)
- *timestamp* - tačan trenutak događaja
- tip loga (**INFO**, **WARN**, **ERROR**, **DEBUG**, **TRACE**, ...)
- (*optional* - u slučaju web servisa na nivou kontroler sloja) tip HTTP zahteva
- (*optional* - u slučaju akcija nad bazom podataka) tip naredbe (insert / delete / ...)
- (*optional* - u slučaju izuzetka) cause - razlog zbog koga se izuzetak desio.
- (*optional* - ukoliko ima smisla, npr. prilikom izuzetaka) reference na delove koda u kom se događaj desio

**Napomena**: prilikom definisanja strukture treba obratiti pažnju da log poruke ne sadrže nikakve poverljive informacije, pogotovo one koje nisu u skladu za pravnim ograničenjima ili sa politikom privatnosti korisnika.

Treba se truditi takođe da poruka bude lako čitljiva u tom smislu da ne sadrži previše informacija, odnosno treba da sadrži minimalno podataka koji zajedno čine relevantnu i smislenu informaciju za onoga ko taj log čita / obrađuje.

### Tipovi logovanja i *log level*

Tip logovanja označava kog je tipa događaj koji je upisan u log fajl. Iz više razloga je bitno da se tačno odredi tip događaja: najpre jer to određuje koliko pažnje treba posvetiti tom događaju - koliki je značaj tog događaja, u kom fajlu će log biti sačuvan ili kada će log biti prikazan, itd.

Tipovi logova su sledeći:

1. **FATAL**: događaji koji dovode do: prestanka rada aplikacije, onemogućavaju rad klijenta, dovode sistem u opasno stanje i sl. Zahteva hitno (*odma'!*) rešavanje. Ukratko: jedna ili više biznis funkcionalnosti ne funkcioniše.
   - Primer: neki od eksternih servisa (npr. Stripe) nije dostupan; Nije moguće pristupiti bazi podataka. Greške koje odgovaraju Http statusu 500
2. **ERROR**: događaji koji predstavljaju neočekivano ponašanje sistema i dovode ga u nekonzistentno stanje. Za ove događaje je neophodno ukloniti *bug*. Uglavnom ovi problemi imaju veliki uticaj na korisnika (njegovo korisničko iskustvo i/ili onemogućavaju njegovo nesmetano korišćenje sistema). Zahteva što ranije rešavanje.
   - Primer:  Validan zahtev ne može da prođe validaciju na kontroler sloju.
3. **INFO**: događaji koji su značajni za rad aplikacije a **ne** predstavljaju neko neočekivano ponašanje. **Napomena**: izuzeci koji ne prouzrukuju neočekivano ponašanje (npr. custom izuzeci ili izuzeci prilikom validacije) takođe mogu spadati u INFO log level (ili čak DEBUG ukoliko su manje značajni)
   -  Primer bi bio log koji opisuje korisnički zahtev i da li je taj zahtev uspešno autorizovan i izvršen
4. **WARN**: označava događaj koji može biti potencijalni problem u aplikaciji, ali koji još uvek nema uticaj na korišćenje aplikacije. 
   - Primer: greška prilikom parsiranja dokumenta gde je kao rezultat nastao fajl noji nije čitljiv.
5. **DEBUG**: Kao što i ime kaže, na ovom nivou se beleže događaji koji su potrebni u procesu debagovanja aplikacije.
6. **TRACE**: događaji koji se beleže na ovom nivou su najdetaljniji i prikazuju (skoro) svaki korak u radu programa

Pored navedenih moguće je naići još na neke tipove, ukoliko se mogu uopšte ubrojati u tipove logova. To su još OFF i ALL.

U standardnom radu aplikacije, u <u>produkcijskom okruženju</u>, kada **još** nema neočekivanog ponašanja, relevantni tipovi/nivoi su INFO, ERROR, FATAL. 

Prilikom razvoja aplikacije, u <u>dev okruženju</u>, uobičajeno je uključiti i DEBUG tipove poruka. 

#### Konfigurisanje nivoa logovanja

Okviri koji podržavaju logovanje ostavljaju mogućnost konfiguracije koji nivo logovanja će se koristiti, i tu je bitno napomenuti da <u>svaki *logging level* uključuje i one sa većim prioritetom</u>. Npr: ukoliko podesimo *logging.level.**info*** biće uključeni i ERROR i FATAL tipovi poruka. 



