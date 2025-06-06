---
layout: post
title: "We zijn geen ingenieurs?"
comments: true
categories: [opinie]
hidenav: true
---

Soms zit je met een opinie in je gedachten voor een paar maand, maar kom je er nooit toe om het neer te schrijven of om het idee te laten kristalliseren. En dan lees je een paar artikels die je over de rand duwen en je forceren om achter het toetsenbord te kruipen. Niet omdat je toch beslist om er eens tijd voor te maken, maar omdat de artikels zo bij het haar gegrepen zijn dat een reactie niet kan uitblijven.

Het artikel van [Rudolf De Schipper](http://datanews.knack.be/ict/nieuws/softwarebouwers-mogen-zich-geen-ingenieurs-noemen/article-opinion-1175531.html) in DataNews heeft me over de rand geduwd. Dit artikel is bij velen die ik ken in het verkeerde keelgat geschoten, en niet enkel omwille van het feit dat hij hiermee mogelijks het ego van een paar ontwikkelaars heeft gekrenkt. Nee, het gaat dieper.

Interessant om weten is dat dit artikel geschreven is door iemand die reeds 18 jaar bij Unisys werkt. Hij heeft dus reeds lang genoeg daar gewerkt om te weten hoe het zwaar fout kan lopen. Unisys was immers de leverancier van het Phenix project, het project dat Justitie ging vernieuwen, en diens opvolger, MACH, projecten die heel publiekelijk en heel zwaar de mist in gegaan zijn. Of het bedrijf dat achter Stimer zat, een van de projecten van FOD Financiën die de belastingsbetaler een slordige 30 miljoen euro gekost heeft. Dit is geen opinie, dit zijn feiten. Dat iemand, die tijdens deze periode voor het bedrijf werkte, dit soort artikels durft schrijven, is hemeltergend. 

Het is toch zo gemakkelijk om de schuld op ontwikkelaars en onze technieken te steken. Maar wat te snel wordt vergeten is dat technieken enkel kunnen werken in een omgeving die dit soort technieken durft te voeden. En laat daar nu net het schoentje wringen bij veel van de grote IT bedrijven in België. Elk bedrijf komt wel naar buiten met 'Agile' en geeft daar zijn eigen draai aan. Ikzelf heb de problematiek in de hand gewerkt jaren geleden door in opdracht een soort Agile variant te maken van meer klassieke approaches zoals RUP. Wat veel bedrijven niet snappen is dat klassieke bedrijfsstructuren (verticaal, gelaagd, gedreven door RACI) gewoon totaal niet rijmen met de kern van Agile. En dat ligt aan de kern van de problematiek.

Ik geef Rudolf tot op zekere hoogte wel gelijk dat we geen ingenieurs zijn. Ingenieurs hebben immers een veel hogere graad van zelfbeschikking en verantwoordelijk dan de meeste softwareontwikkelaars ooit zullen krijgen. We mogen dan wel geen bruggen bouwen (die trouwens ook kunnen falen, zie het nieuws), de impact van de zaken die we maken wordt er daarom niet kleiner om. In de VS zijn mensen hun huizen kwijtgeraakt of hebben mensen zelfmoord gepleegd, enkel en alleen door een stuk software. Maar wij worden daar niet verantwoordelijk voor gesteld. Dit zorgt ervoor dat de meerderheid van software ontwikkelaars met een soort gevoel van onschendbaarheid rondlopen. Een bug die een bedrijf een paar miljoen kost? Kan gebeuren. Software ontwikkelaars die moedwillig software schrijven bij Volkswagen om controles te omzeilen en te manipuleren? Tja, bevel is bevel, het is niet dat de ontwikkelaars die dat hebben gedaan persoonlijk aansprakelijk worden gesteld. Wees maar zeker dat ingenieurs die een brug bouwen die later instort het heel moeilijk gaan hebben om dit nog een tweede keer te doen.

Nee, we zijn nog geen ingenieurs. We missen een professionele ethische code. Een code die we aanhouden en die we eisen om aangehouden te worden. En dat laatste is zo belangrijk. In veel bedrijven wordt iets zoals testen schrijven nog steeds gezien als verspilde tijd. Het levert immers geen korte-termijn voordelen op, maar de lange-termijn voordelen zijn ondertussen wel al bewezen. Maar toch hebben vandaag de dag ontwikkelaars het moeilijk om hun management ervan te overtuigen dat dit noodzakelijk is. Maar testen of test-driven development is maar het tipje van de ijsberg. Er zijn een hoop technieken die in de toolkit van developers zitten die moeilijk te slikken zijn voor traditioneel management, neem nu pair programming als voorbeeld. Deze techniek kan ervoor zorgen dat software opgeleverd wordt met een hoger kwaliteitsniveau. Maar sommige bedrijven nemen dan de extreme route en verplichten iedereen om altijd te pairen. Zelfs de bedenkers en cheerleaders van de techniek geven aan dat dit niet ok is. Maar uiteraard faalt de techniek dan en ligt het aan de techniek.

In de VS is er zoiets als de Order of Engineers. Hun code:

```
I am an Engineer. In my profession, I take deep pride. To it, I owe solemn obligations.

As an engineer, I pledge to practice integrity and fair dealing, tolerance and respect, and to uphold devotion to the standards and dignity of my profession. I will always be conscious that my skill carries with it the obligation to serve humanity by making the best use of the Earth's precious wealth.

As an engineer, I shall participate in none but honest enterprises. When needed, my skill and knowledge shall be given, without reservation, for the public good. In the performance of duty, and in fidelity to my profession, I shall give my utmost.
```

Laten we dit eens interpreteren in onze context.

```
I am an Engineer. In my profession, I take deep pride. To it, I owe solemn obligations.
```

Weinig ontwikkelaars hebben een gevoel van trots ten opzichte van de software die ze ontwikkeld hebben. Zonder verantwoordelijkheid hebben ze immers geen reden om zich trots te voelen, ze waren immers een radertje in de machine. Maar net zoals een ingenieur met trots zou kijken naar een gebouw dat hij mee ontworpen heeft, zouden wij ook moeten kijken naar de software die we opleveren. 
Maar bij die verantwoordelijkheden komen ook verplichtingen en als we onszelf ingenieurs willen noemen, mogen we niet bang zijn van die verplichtingen. 

```
As an engineer, I pledge to practice integrity and fair dealing, tolerance and respect, and to uphold devotion to the standards and dignity of my profession. I will always be conscious that my skill carries with it the obligation to serve humanity by making the best use of the Earth's precious wealth.
```

Integriteit wil zeggen dat we bereid zijn om nee te zeggen, nee te roepen, wanneer we aanvoelen dat een oplossing de verkeerde is. Opnieuw, als ingenieurs moeten we bereid zijn om de verantwoordelijkheid en de gevolgen daarvan te dragen. Ondanks dat onze industrie het nogal moeilijk heeft met deftige standaarden op te stellen (ontwikkelaars komen nog niet overeen of ze tabs of spaties moeten gebruiken om code te indenteren), zijn er wel een hoop best practices ontwikkeld die zichzelf bewezen hebben. We schrijven geen software meer zonder testen. We doen aan CI en CD. We automatiseren onze opleveringen. We houden veiligheid in ons achterhoofd. We worden immers ook vertrouwd met een van de belangrijkste rijkdommen van de moderne wereld: data.

```
As an engineer, I shall participate in none but honest enterprises. When needed, my skill and knowledge shall be given, without reservation, for the public good. In the performance of duty, and in fidelity to my profession, I shall give my utmost.
```

Kwaliteitsvolle software maken iets dat ingenieurs doen omdat ze dat intrinsiek willen, niet enkel omdat het van hen verwacht wordt. Wij zien de klant als een eerlijke partner en we verwachten niks anders als tegenprestatie. Wanneer de klant iets vraagt, geven we ons eerlijk antwoord, ook al weten we dat de klant daar misschien niet zo blij mee zal zijn.

Dus ja, een code voor software ontwikkelaars is misschien wel geen slecht gedacht. Maar dan moeten de klassieke software bedrijven bereid zijn om een stap opzij te doen en ons te laten doen waar we zo goed in zijn: kwaliteitsvolle software schrijven die de business vooruit helpt. Velen van ons weten waarmee we bezig zijn. En wees maar zeker dat we echt niet bang zijn om de verantwoordelijkheid op ons te nemen.

Zijn alle software ontwikkelaars ingenieurs? Nee. Maar velen onder ons hebben de mindset van een ingenieur. We weten de grenzen van ons kunnen maar zijn steeds bereid om die grenzen te verplaatsen in functie van wat men ons vraagt. Maar we zijn altijd eerlijk. En ja, we zijn ook bereid om de deur achter ons dicht te slaan als onze meningen en normen in de wind worden geslaan. Misschien ook de reden waarom ik vandaag niet meer voor een consultancy bedrijf of software factory werk.

Waarom bedrijven in zee blijven gaan met de grote dienstverleners blijft me een raadsel. Het ligt ook aan ons, we hebben immers niet het bandbreedte om onze zichtbaarheid te vergroten. Maar moesten CEO's of CTO's eens (incognito) naar de maandelijkse meetups van de software ontwikkelaars met de mindset van een ingenieur komen, dan zouden hun ogen echt wel opengaan. We zijn radicaal vernieuwend bezig. We zijn steeds bezig met onze skillset te vergroten ten voordele van onze klanten. 

Beste Rudolf, die nieuwe technologie waarover je het hebt, daar zijn wij mee bezig. Maar je gaat ons zelden vinden bij de klassieke software dienstverleners. We eisen vrijheid. We eisen verantwoordelijkheid. Wij hanteren een code. Dus de volgende keer dat je ons verwijt geen ingenieurs te zijn, stel jezelf de vraag: waarom hebben wij ze niet in ons bedrijf? Misschien is onze code gewoon niet compatibel met de structuur die jullie opleggen. 