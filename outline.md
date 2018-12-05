# Outline
Je moet deze aankondiging een beetje zien als een pitch op basis waarvan je collega’s snel enthousiast moeten worden.

Het moet denk ik niet te veel ‘beheer’ uitstralen want dat is misschien niet helpzaam.

Waar vroeger Development en Operations twee aparte afdelingen binnen een IT organisatie waren, zijn die in de laatste jaren behoorlijk naar elkaar toe gegroeid. Developers en beheerders zitten in dezelfde teams en worden samen verantwoordelijk voor het leveren van een IT dienst binnen organisaties. In deze zogenaamde DevOps teams wordt van alle medewerkers verwacht dat ze "T-Shaped" zijn, en dus zowel werk aan de Dev als aan de Ops kant kunnen doen.

Maar, aangezien we niet meer in 2006 leven maar in 2018, gaan we niet meer met 5 beheerders 1 servertje onderhouden en hem tot in de treuren debuggen als er weer wat mee is. Infrastuctuur is vluchtig, als er nog sprake van klassieke infrastructuur (in de vorm van servers) is. Wat belangrijk is, is dat je door de hele otap straat al je infrastructuur consistent configureerd. Een computer doet dat altijd op dezelfde manier, een IT'er die slecht heeft geslapen vergeet wel eens iets. Het gevolg daarvan is dat zaken die in acceptatie prima werken het in productie niet doen omdat er een vinkje in de management console is vergeten. En je 3 uur aan het zoeken bent voor je daar achter komt.

In deze bits en bytes wil ik jullie meenemen in de "Ops" kant van 2018. We gaan hands-on aan de slag met het opbouwen, maar ook met het weer snel opruimen, van infrastructuur. Alles geautomatiseerd, zodat we ons kunnen focussen op meerwaarde brengen voor de business in plaats van constant hetzelfde doen. We gaan ons bezig houden met Terraform van Hashicorp, en Ansible van RedHat. Met de eerste configureren we cloud resources, met de tweede gaan we Linux servers configureren.

Het doel is dat we onze infra zo ver kunnen opschalen als we willen zonder dat we in moeten loggen op servers.

Ik denk dat het goed is om de aankondiging iets toe te spitsen op bijvoorbeeld DevOps. Steeds meer collega’s komen dagelijks namelijk in aanraking met ops-gerelateerde werkzaamheden of willen er graag wat handiger in worden of verbeteren.

Misschien kun je in deze mail nog wat meer focus leggen op het Wat gaan we zien/doen, dan het Hoe (benodigdheden etc.). De ‘Hoe’ komt meestal te pas in een vervolg uitnodiging. Dat het over DigitalOcean gaat is wellicht in deze intro ook nog niet relevant.
Kun je nog iets meer uitweiden over Wat we gaan zien/doen? Wat zijn bijvoorbeeld de herkenbare/voorkomende problemen van slecht geconfigureerde servers? Ga je met bepaalde scripttalen aan de slag? Welke? Wat hebben ze na afloop geleerd? Is een specifieke Linux distributie wel relevant hier?


# Uitnodiging
Wat is configuration management?
In een wereld waar containers, cloud en serverless de norm worden, lijken de  klassieke beheertaken steeds minder belangrijk te worden. Ondanks dat er kant en klare building blocks in de cloud af te nemen zijn om applicaties op te deployen, moeten ook die geconfigureerd worden.

Ik heb me in de laatste ~4 jaar van mijn carriere als beheerder erg veel bezig gehouden met het automatiseren van mijn beheer werkzaamheden. De waarde hiervan is dat ik vrij zeker kan zijn dat zaken altijd op dezelfde manier opgebouwd worden. Een computer vinkt immers altijd dezelfde installatie opties aan. Hierdoor was het mogelijk om met dezelfde mensen steeds meer installaties te kunnen beheren. Zelf vond ik het leuk dat het werk uitdagend bleef en ik niet constant dezelfde dingen doe.

Voor ik omgevormd ben tot een full fledged developer, wil ik graag nog even mijn lessons learned als beheerder met jullie delen.
 
Wat heb je nodig?
•	laptop met wifi
•	ondersteunde browser
•	een SSH client (Download MobaXTerm als je twijfeld)
•	creditcard (nodig om een DigitalOcean-account aan te maken)

We kunnen helaas niet binnen de AWS free tier blijven, want anders konden we de AWS accounts uit de bits 'n bytes van Milco hergebruiken. Er zijn genoeg kortingscodes in de omloop waarmee je gratis credits kan krijgen bij DigitalOcean. Voor deze workshop is dat meer dan voldoende. Creditcard gegevens opgeven is alleen ter verificatie van je account. 
 
Wat gaan we zien/doen?
De nadruk ligt op het configureren van (Linux) virtual machines op DigitalOcean. Demo is vooral een showcase van wat voor speed en consistency configuration management brengt en nodigt hopelijk uit tot verdere experimenten en verdieping.