# classifier for doc direct


De Hanzehogeschool heeft de mogelijkheden onderzocht voor een document classifier. De achterliggende analyse is te vinden in `DocDirect Machine Learning.html`. Vervolgens is de analyse omgezet in werkbare scripts dat een model trained en documenten klassificeert. Om een document te klassificeren zijn een aantal preparatie handelingen nodig (5 stappen). De aanroep via een shell script wordt als eerste beschreven en daarna worden de verschillende stappen uitgelegd. 

De mappen structuur is als volgt

- `ocr` # bevat de ocr gescande documenten (input)
- `pdf` # bevat de naar pdf omgezette documenten (resultaat stap 1)
- `txt` # bevat de naar txt format omgezette documenten (resultaat stap 2)
- `formatted` # bevat de tabular format geformateerde documenten (resultaat stap 3)
- `normalized` # bevat de genormaliseerde tabular format geformateerde documenten (resultaat stap 4)
- `cleaned` # bevat de gecleande tabular format geformateerde documenten (resultaat stap 5)
- `scripts` # bevat de geprogrammeerde scripts, configuratiefiles en taaldictionaries
- `models` # bevat de getrainde modellen
- `trainingsdata` # bevat het trainingscorpus


## Gebruik via shell script

Om een ocr document te klassificeren kan het volgende shell script gebruikt worden:

    python3 ocrpdf ../ocr/{sample} ../pdf/{sample}
    python3 readpdf.py ../pdf/{sample} ../txt/{sample} 
    python3      /txt/{sample} ../formatted/{sample} 
    python3 normalize.py ../formatted/{sample} ../normalized/{sample} 
    python3 clean_doc.py ../normalized/{sample} ../cleaned/{sample}

    # aanroepen classifier
    python3 use_model.py ../cleaned/{sample}
    
Om meerdere gecleande bestanden samen te voegen in een trainingscorpus is het volgende commando nodig (waarbij file1, file2 etc fictieve namen zijn)

    cat file1 file2 file3 file4 > ../trainingsdata/doc2wordbiza_cleaned.txt


## Preparatie handelingen

Een gescand document moet eerst omgezet worden naar een text file in een bepaald format. Dat gaat volgens de volgende stappen

1. OCRpdf -> pdf
2. pdf -> txt
3. txt -> formatted txt (doc, line, word)
4. formatted txt -> normalized txt (doc, line, word, normalized)
5. normalized txt -> spell corrected txt (doc, line, word, normalized, corrected)

Na het preprocessen kan het document geclassificeerd worden


## Preparatie handeling stap 1: van ocr naar pdf

Om de OCR om te zetten naar pdf kan gebruik gemaakt worden van een programma dat aangeroepen kan worden via de commandline:

    python3 ocrpdf {input} {output}
    
In een pipeline kan dat als volgt geprogrammeerd worden

    input:
        '../ocr/{sample}'
    output:
        '../pdf/{sample}'
    shell:
        'python3 normalize.py {input} {output}'


## Preparatie handeling stap 2: van pdf naar txt

Om de pdf om te zetten naar tekst kan gebruik gemaakt worden van een pythonscript dat aangeroepen kan worden via de commandline: 

    python3 readpdf.py {input} {output}


In een pipeline kan dat als volgt geprogrammeerd worden

    input:
        '../pdf/{sample}'
    output:
        '../txt/{sample}'
    shell:
        'python3 normalize.py {input} {output}'

Het programma genereerd een document met dezelfde bestandsnaam maar dan met .txt extentie
benodigde packages voor dit programma:

    PyPDF2

## Preparatie handeling stap 3: van txt naar formatted

De txt wordt vervolgens omgezet naar een tekst document met elk woord op een regel. Het document is tabular seperated en bevat de kollommen doc (documentnaam), line (regel van het document), word (woord van de regel). Het programma kan worden aangeroepen via de commandline

    ......... 
    
In een pipeline kan dat als volgt geprogrammeerd worden

    input:
        '../txt/{sample}'
    output:
        '../formatted/{sample}'
    shell:
        'python3 normalize.py {input} {output}'
        

## Preparatie handeling stap 4: normalizeren

De woorden uit het document van stap 3 worden genormaliseerd. Rare tekens worden eruit gehaald. Het programma kan gebruikt worden door een pythonscript dat aangeroepen kan worden via de commandline: 

    python3 normalize.py {input} {output}
    
In een pipeline kan dat als volgt geprogrammeerd worden

    input:
        '../formatted/{sample}'
    output:
        '../normalized/{sample}'
    shell:
        'python3 normalize.py {input} {output}'
   
Het programma maakt gebruik van de volgende libraries:
    
    pandas
    
Het genormalizeerde woord komt in een kolom achter het woord    


## Preparatie handeling stap 5: corrigeren (cleanen)

De genormaliseerde woorden worden vervolgens gecontroleerd op spelling en eventueel gecorrigeerd. Deze stap duurt lang bij grote bestanden die niet goed ingelezen zijn. Het programma kan gebruikt worden door een pythonscript dat aangeroepen kan worden via de commandline: 

    python3 clean_doc.py {input} {output}
    
In een pipeline kan dat als volgt geprogrammeerd worden

    input:
        '../normalized/{sample}'
    output:
        '../cleaned/{sample}'
    shell:
        'python3 clean_doc.py {input} {output}'
        
Het programma heeft libraries nodig maar ook een aantal bestanden. 

    spell.py    (spell corrector)
    pandas      (python library)
    yaml.       (python library)
    woorddict   (bevat NL en veel voorkomende woorden)
    config.yaml (naam input file)

Het gecorrigeerde woord komt in een kolom achter het genormalizeerde woord. 


## Gebruik classifier

De classifier kan gebruikt worden door een pythonscript dat aangeroepen kan worden via de commandline: 

    use_model.py {input}

De input is dus een resultaatdocument `../cleaned/{sample}` van stap 5. 
Het programma voorspelt de kans dat het document bewaard moet worden.

Het programma maakt gebruik van de volgende libraries en files

    numpy (python library)
    pandas (python library)
    pickle (python library)
    yaml (python library)
    nltk (python library)
    
    config.yaml (configuratiefile voor directories en bestandsnamen)
    
## Aanpassen Threshold

De classifier maakt gebruik van een optimale threshold. Alle bestanden die een kans van bewaren boven de threshold hebben worden voorspelt als 'bewaren'. De threshold kan aangepast worden in de configuratiefile config.yaml (te openen met bijvoorbeeld kladblok)
    



## Optimaliseren classifier

Het model kan bijleren met nieuwe files. Het programma `train_model.py` kan daarvoor gebruikt worden. Het programma gebruikt als input een bestand dat een verzameling is van gepreproccessed bestanden. Default is deze naam 

    traincorpus : 'doc2wordbiza_cleaned.txt'

Verder heeft het een mappings document nodig van de namen van het document

    docidmap : 'docidmapping.txt'
    
Het gebruikt de opentaal library 
    
    dictionary : 'OpenTaal_200G.txt

en slaat het getrainde model op in drie bestanden:

    modelpath: 'models/biza_model.sav'
    bestwords: 'models/bestwords.p'
    vector: 'models/vector.p'
    
Alle namen en paden kunnen aangepast worden in config_train.yaml met behulp van een teksteditor, bijvoorbeeld kladblok

De `train_model.py` kan aangeroepen worden met

    python3 train_model.py
    
Het gebruikt naast de bovenstaande programmas ook de volgende libraries

    numpy
    pandas
    pickle
    scipy
    sklearn
    yaml

## Aanpassen label

Het label is op dit moment gebaseerd op de titel van het document. Als de bestandsnaam begint met `INV` wordt het als een te bewaren document gelabeled. Om dit aan te passen moet in `train_model.py` de code regel aangepast worden:

    labels = docnames['name'].str.startswith('I') * 1  # "Inv.nr.*" should be archived, otherwise discarded
    
## Aanpassen model

Het best presterende model is een `Naive Bayes BernoulliNB` model met `N = 8` keywoorden. Om dit aan te passen moet de code in train_model aangepast worden. 



    

