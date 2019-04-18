# Outline

## Interessepijl mail

Waar vroeger Development en Operations twee aparte afdelingen binnen een IT organisatie waren, zijn die in de laatste jaren behoorlijk naar elkaar toe gegroeid. Developers en beheerders zitten in dezelfde teams en worden samen verantwoordelijk voor het leveren van een IT dienst binnen organisaties. In deze zogenaamde DevOps teams wordt van alle medewerkers verwacht dat ze "T-Shaped" zijn, en dus zowel werk aan de Dev als aan de Ops kant kunnen doen.
Maar, aangezien we niet meer in 2006 leven maar in 2019, gaan we niet meer met 5 beheerders 1 servertje onderhouden en hem tot in de treuren debuggen als er weer wat mee is. Infrastuctuur is vluchtig, als er nog sprake van klassieke infrastructuur (in de vorm van servers) is. Wat belangrijk is, is dat je door de hele otap straat al je infrastructuur consistent configureert. Een computer doet dat altijd op dezelfde manier, een IT'er die slecht heeft geslapen vergeet wel eens iets. Het gevolg daarvan is dat zaken die in acceptatie prima werken het in productie niet doen omdat er een vinkje in de management console is vergeten. En je 3 uur aan het zoeken bent voor je daar achter komt.
In deze bits en bytes wil ik jullie meenemen in de "Ops" kant van 2019. We gaan hands-on aan de slag met het opbouwen, maar ook met het weer snel opruimen, van infrastructuur. Alles geautomatiseerd, zodat we ons kunnen focussen op meerwaarde brengen voor de business in plaats van constant hetzelfde doen. We gaan ons bezig houden met Terraform van Hashicorp, en Ansible van RedHat. Met de eerste configureren we cloud resources, met de tweede gaan we Linux servers configureren.
Het doel is dat we onze infra zo ver kunnen opschalen als we willen zonder dat we in moeten loggen op servers. 
Ik wil graag weten wie er interesse heeft om mee te doen aan de ze hands-on sessie. Waarschijnlijk wil ik dit op 28 mei gaan organiseren.

## Uitnodiging

Beste Collega's,

Een paar weken geleden heb ik interesse gepeilt voor een Bits en Bytes over efficiëntie in operatins. Ik was erg blij met de animo hiervoor, dus ik wil deze sessie definitief gaan organiseren. De sessie zal met name gaan over configuration management.

Wat is configuration management?
In een wereld waar containers, cloud en serverless de norm worden, lijken de  klassieke beheertaken steeds minder belangrijk te worden. Ondanks dat er kant en klare building blocks in de cloud af te nemen zijn om applicaties op te deployen, moeten ook die geconfigureerd worden. Daarnaast zijn er een hoop organisaties die nog met 'ouderwetse' virtuele machines werken.

Ik heb me in de laatste ~4-5 jaar van mijn carriere als beheerder erg veel bezig gehouden met het automatiseren van mijn werkzaamheden. De waarde hiervan is dat ik zeker kan zijn dat zaken altijd op dezelfde manier geïnstalleerd en geconfigureerd worden. Een computer vinkt immers altijd dezelfde installatie opties aan. Hierdoor was het mogelijk om met dezelfde mensen steeds meer installaties te kunnen beheren. Zelf vond ik het leuk dat het werk uitdagend bleef en ik niet constant dezelfde dingen doe maar me bezig hield met het iteratief verbeteren van de omgevingen.

Voor ik omgevormd ben tot een full fledged developer, wil ik graag nog even mijn lessons learned als beheerder met jullie delen.

Wat heb je nodig?

- laptop met wifi

- ondersteunde browser

- een SSH client (Download MobaXTerm als je twijfeld)

- creditcard (nodig om een DigitalOcean-account aan te maken)

We kunnen helaas niet binnen de AWS free tier blijven, want anders konden we de AWS accounts uit de bits 'n bytes van Milco hergebruiken. Daarom heb ik gekozen voor Digital Ocean voor de labs. Een kleinere speler waar ik al een jaar of 5 trouw klant ben. Er zijn genoeg kortingscodes in de omloop waarmee je gratis credits kan krijgen bij DigitalOcean. Voor deze workshop is dat meer dan voldoende. Creditcard gegevens opgeven is alleen ter verificatie van je account. Na de sessie kan je je account eventueel sluiten.

Begin mei zal ik een promo code met jullie delen, waarmee je voldoende gratis credits krijgt om de bits en bytes zonder kosten te doorlopen.

Wat gaan we zien/doen?
De nadruk ligt op het configureren van (Linux) virtual machines op DigitalOcean. Demo is vooral een showcase van wat voor speed en consistency configuration management brengt en nodigt hopelijk uit tot verdere experimenten en verdieping.

Agenda:

16:00 - Inloop
16:30 - Start presentatie
17:15 - Start lab gedeelte
18:30 - Broodjes buffet
19:15 - Verder met de labs
20:30 - Afsluiting

Hopelijk tot dan!

Groeten,
Bas