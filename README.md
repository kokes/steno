# Stenoprotokoly z PSP ČR
Účelem tohodle repa je ukázat si, jak rychle a snadno překlopit stenoprotokoly ze Sněmovny do hledacího enginu. Z nuly můžete mít plně funkční fulltextové hledání během 10 minut.

Zatím to funguje jen pro nejnovější data ze současné Sněmovny a jen na jednom vyhledávacím enginu, ale obojí jde snadno změnit.

## Instalace
Předpokládám určitou technickou znalost, kdybyste měli nějaké konkrétní dotazy, piště mi [e-maily](ondrej.kokes@gmail.com) nebo [tweety](http://twitter.com/kondrej). Ideálně větší záludnosti než *"co je Solr?"*, ale zas ne nějaká zvěrstva, jsem přeci jen ekonom :-)

Jak jsem psal, je to postavené na konkrétní hledací technologii, konkrétně jde o [Solr](http://lucene.apache.org/solr/), což je engine predatující Elastic, dnešní hip hledadlo, funguje pěkně a rychle, ale já si ho vybral, protože se mi jméno líbilo víc jak Elastic. Ukážeme si ale oba přístupy.

Na pozadí si tedy dejte instalovat Solr. Já jsem dítě Macintoshů, takže pro mě to je `brew install solr`, pro ostatní systémy dohledejte informace na webu projektu. Předpokládám, že Python máte, bude stačit základní sada knihoven + [pyquery](https://pypi.python.org/pypi/pyquery). Puristé mohou přepsat do `lxml`, mně to vyhovuje takto.

Celý proces jak se dostat od zdrojových dat až po zpracované projevy, je popsaný [zvlášť v notebooku](Priprava_dat.ipynb).

## Hledání
Rád si nechám poradit, sám hledám teprv pár dní. Nasypal jsem data do Solru i Elastiku, obojí funguje královsky. Solr jsem naplnil přes dva příkazy (napřed jsem mazal původní data):

	$ solr delete -c steno
	$ solr create_core -c steno
	$ post json -c steno

Hotovo, v základu máte i prohlížedlo, takže hurá na [localhost:8983/solr/steno/browse](http://localhost:8983/solr/steno/browse) a můžete hledat. Můžete i snadno přidat facety, tedy agregace podle jednotlivých položek. Přes [facet.field](http://localhost:8983/solr/steno/browse?q=hitler*&facet.field=autor&facet.field=schuze) můžete snadno filtrovat výlevy o Hitlerovi podle řečníka a/nebo schůze.

![nahled hledani](https://dl.dropboxusercontent.com/u/5758323/Screenshot%202016-05-31%2023.14.46.png "nahled hledani")

Paráda, máte v zásadě hotovo. Solr funguje přes HTTP API, takže ho můžete napojit na svoji aplikaci bez nějakých větších potíží.

## Elasticsearch
Pokud byste chtěli data nahodit do Elastiku, tak je to samozřejmě možné. Já udělal následující pomocí [oficiálního balíku](https://www.elastic.co/guide/en/elasticsearch/client/python-api/current/index.html).

```python
import json
import glob
from elasticsearch import Elasticsearch

fns = glob.glob('json/*.json')

es = Elasticsearch()

for fn in fns:
    print(fn)
    with open(fn) as f:
        docs = json.load(f)
    for d in docs:
        res = es.index(index="steno", doc_type='projev', id=d['id'], body=d)

res = es.search(index='steno', q='Bohuslav Sobotka')
res['hits']['total']
```

Asi to bude chtít použít bulk mód v subbalíků `helpers`, ale to už si poradíte.

---

Hotovo, tolik asi k tomu.