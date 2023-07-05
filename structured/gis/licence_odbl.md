**Contexte** = juillet 2023, je m'intéresse à l'utilisation conjointe de données OSM et non-OSM.

* [Synthèse](#synthèse)
* [Ressources et notes](#ressources-et-notes)
   * [<a href="https://opendatacommons.org/licenses/odbl/1-0/" rel="nofollow">https://opendatacommons.org/licenses/odbl/1-0/</a>](#httpsopendatacommonsorglicensesodbl1-0)
   * [<a href="https://www.openstreetmap.fr/open-data/" rel="nofollow">https://www.openstreetmap.fr/open-data/</a>](#httpswwwopenstreetmapfropen-data)
   * [<a href="https://wiki.osmfoundation.org/wiki/Licence/Licence_and_Legal_FAQ" rel="nofollow">https://wiki.osmfoundation.org/wiki/Licence/Licence_and_Legal_FAQ</a>](#httpswikiosmfoundationorgwikilicencelicence_and_legal_faq)
   * [<a href="https://wiki.osmfoundation.org/wiki/Licence/Community_Guidelines" rel="nofollow">https://wiki.osmfoundation.org/wiki/Licence/Community_Guidelines</a>](#httpswikiosmfoundationorgwikilicencecommunity_guidelines)
      * [<a href="https://wiki.osmfoundation.org/wiki/Licence/Attribution_Guidelines" rel="nofollow">https://wiki.osmfoundation.org/wiki/Licence/Attribution_Guidelines</a>](#httpswikiosmfoundationorgwikilicenceattribution_guidelines)
   * [<a href="https://learn.microsoft.com/en-us/bingmaps/articles/open-maps-understanding-odbl" rel="nofollow">https://learn.microsoft.com/en-us/bingmaps/articles/open-maps-understanding-odbl</a>](#httpslearnmicrosoftcomen-usbingmapsarticlesopen-maps-understanding-odbl)
   * [<a href="https://doc.transport.data.gouv.fr/presentation-et-mode-demploi-du-pan/conditions-dutilisation-des-donnees" rel="nofollow">https://doc.transport.data.gouv.fr/presentation-et-mode-demploi-du-pan/conditions-dutilisation-des-donnees</a>](#httpsdoctransportdatagouvfrpresentation-et-mode-demploi-du-panconditions-dutilisation-des-donnees)

# Synthèse

Je me risque à un résumé :

- l'ensemble de la donnée OSM est sous [license ODbL = Open Database License](https://opendatacommons.org/licenses/odbl/1-0/)
- la donnée OSM est librement accessible, mais moyennant deux clauses :
    - **attribution** = on doit indiquer la provenance des données de façon explicite
    - **share-alike** = la license ODbL est contaminante : si mon projet utilise de la donnée OSM modifiée ou enrichie avec d'autres données, je dois distribuer publiquement toutes les données de mon projet, y compris les données non-OSM
- la clause d'attribution est TOUJOURS applicable
- la clause de repartage (= share-alike) est applicable dans beaucoup de cas, mon on distingue grosso-modo 3 situations :
    - **Collective Database** = la donnée OSM et la donnée non-OSM sont utilisées ensemble, mais n'interagissent pas ; elles restent indépendantes
    - **Derivative Database** = la donnée OSM et la donnée non-OSM sont utilisées ensemble et interagissent pour former une database résultante, qu'on distribue
    - **Produced Work** = grosso-modo, on distribue un service qui utilise une derivative database
- seules les Collective Database ne déclenchent pas la clause share-alike ; dans les autres cas, toutes les données de mon projet doivent être publiées ou publiables à la demande.
- à noter que ces clauses ne s'appliquent que si on distribue publiquement un service ou un jeu de données : en interne, on peut faire ce qui nous chante.
- le fait de décider si la clause share-alike s'applique ou non n'est pas simple, il y a [toute une FAQ](https://wiki.osmfoundation.org/wiki/Licence/Licence_and_Legal_FAQ) très intéressante sur le sujet, ainsi que [des guidelines utiles pour des cas spécifiques](https://wiki.osmfoundation.org/wiki/Licence/Community_Guidelines) ; un extrait qui montre que ça peut être contre-intuitif :
    > For all features of a given type ("Feature Types") (e.g. streets, restaurants, parks, cemeteries), if all data for that Feature Type is from non-OpenStreetMap sources, then the ODbl share-alike conditions do not apply to that Feature Type, even if OpenStreetMap data is used for other Feature Types

# Ressources et notes

## https://opendatacommons.org/licenses/odbl/1-0/

Le texte officiel de la license.

## https://www.openstreetmap.fr/open-data/

> Les données du projet OpenStreetMap sont disponibles sous licence Open Data Commons Open Database License (ODbL) auprès de la Fondation OpenStreetMap (OSMF) (...)
>
> Vous êtes libre de copier et d’adapter nos données, à condition de créditer OpenStreetMap et ses contributeurs (avec la mention « © les contributeurs OpenStreetMap, sous licence ODbL »).
>
> Par ailleurs, si vous modifiez ou utilisez nos données dans d’autres œuvres dérivées, vous ne pouvez distribuer celles-ci que sous la même licence.

## https://wiki.osmfoundation.org/wiki/Licence/Licence_and_Legal_FAQ

C'est la ressource que j'ai le plus utilisée pour construire ma compréhension ; je donne quelques extraits pertinents et mes notes :

> What license applies to the use of the OSM Data? OSM data is made available under the Open Database Licence version 1.0 (the "ODbL"), a copy of which is available here.
>
> What can I do with the OSM Data? The ODbL allows you to use the OSM data for any purpose you like. This includes personal, community, educational, commercial or governmental.

^ si on respecte les clauses (essentiellement attribution et share-alike), on peut utiliser la donnée OSM pour ce qu'on veut, comme on le veut

> Is attribution required? Yes! Please see our Attribution Guideline here.

^ ils ont des guidelines très bien foutues qui expliquent ce qu'on peut faire.

> There are a number of key conditions that you need to be aware of if you want to use our data. (...)
>
> **Public Use** This is where you distribute the OSM data or any Derivative Database (see below), or a Produced Work built from either of these, outside of your organisation or, if you are not an organisation, you make it available to third parties. If you only use OSM data and any Derivative Databases privately for yourself or within your school, organisation or company then it is not public use.

^ si on ne rend rien accessible à l'extérieur, alors ce n'est pas un usage public et on fait ce qu'on veut.

> **Produced Work** Most uses of OSM are as Produced Works. A produced work is where you take the OSM data and turn it into a finished work (as opposed to it being made available as a database). (...)

^ un calculateur d'itinéraire qui utilise de la donnée OSM est certainement un Produced Work

Plus subtil, un calculateur d'itinéraires public qui utiliser de la donnée non-OSM, mais qui a utilisé de la donnée OSM pour catégoriser la donnée non-OSM pourrait être un Produced Work public ?

> **Derivative Database** This is one of the most complex parts of the ODbL. You should read the exact wording in the licence along with the meaning of a Collective Database below. However, at a high level, a Derivative Database is created where you adapt, modify, enhance, correct or extend our data.

^ l'idée semble être de publier une BDD basée sur OSM, mais où celle-ci est modifiée.

> **Collective Database** A Collective Database is where the OSM Data is used as part of a collection of otherwise independent databases which are assembled into a collective whole. A Collective Database is not therefore considered a Derivative Database.

^ pas limpide, mais par opposition à une Derivative Database, l'idée est ici d'intégrer la donnée OSM à une database plus large, comme jeu de données indépendant du reste, et sans la modifier ?

> The concepts of Collective Databases and Derivative Databases are particularly relevant where you want to use your data or third party data in conjunction with our data and is a critical point to get right due to the Share- Alike condition we explain below.
>
> To help you understand the difference between a Collective Database vs a Derivative Database for OSM data, we have published a number of guidelines to help you here. We recommend you read through these Guidelines and the examples contained in them.

Effectivement, [cette guideline sur les Collective Database](https://wiki.osmfoundation.org/wiki/Licence/Community_Guidelines/Collective_Database_Guideline_Guideline) est intéressante, la règle est plus souple que ce que je pensais au premier abord.

> With the above core concepts explained, what are the key conditions you need to be aware of? Well, they are:
>
> - Where you make our data or any Derivative Database available to others, it must continue to be licensed under the ODbL. This is often referred to as Share-Alike.
> - If you create a Produced Work, you can apply whatever terms you like to the Produced Work, but you must upon request offer recipients either a copy of your data and any Derivative Databases under the terms of the ODbL or the means of creating the Derivative Databases upon request.
> - You must attribute the use of our data in both of the above cases.

^ le point concernant le Produced Work est très important à comprendre : en gros, dès qu'on utilise de la donnée OSM dans un projet, il faut pouvoir publier l'ensemble des données utilisée par le projet !

> What does Share-Alike mean and what do I need to do for Produced Works? If you distribute our data onwards to third parties, you must do so under the ODbL license. This means they can then in turn use that data for any purpose as well as themselves provide it onwards under the terms of the ODbL. The same applies for any Derivative Database you create.

^ c'est assez clair = l'idée est que le caractère ouvert de la licence ODbL est contaminante = si on utilise de la donnée OSM pour publier des données enrichies, on doit publier le résultat final sous ODbL.

> However, it will not apply to any other data forming part of a Collective Database. This data you can license under any terms you like.

^ de ce que j'en comprends, si on intègre la donnée OSM sans la modifier dans une collection de databases, les autres databases peuvent être distribuées sous licence plus restrictive : le caractère contaminant ne s'applique pas.

Du coup, savoir ce qui est une Collective Database vs. ce qui est une Derivative Database est très important !

> For Produced Works, the situation is a little different. You can license a Produced work under any terms you like. However, any recipient to which you make the Produced Work available can ask for a copy of both our data and any Derivative Database you use in connection with that Produced Work. You are required to provide these if requested. If you haven’t made changes to the OSM data, you can simply refer users back to openstreetmap.org as the data source.

^ du coup, si je builde un service qui utilise de la donnée OSM, il faut que je mette la donnée à disposition. Là où ça devient touchy, c'est que la donnée OSM est "contaminante" (le terme est de moi) = si le service utilise à la fois la donnée OSM et UNE AUTRE donnée (e.g. la donnée non-OSM), alors je dois publier TOUTES les données que j'utilise, y compris la donnée non-OSM.

> Do you have to share changes you make to OSM data back to the community? (...) you do not have to as long as you comply with the above.

^ on n'est jamais obligé de contribuer directement à OSM (mais comme on doit publier les données, quelqu'un pourra le faire à notre place).

> What if you are only using the OSM data and any Derivative Database internally? If you do not make Public Use of the data then you do not have to share anything with anybody.

^ si on utilise la donnée OSM en interne, la licence ODbL ne s'applique pas ; mais le point critique est à quel point l'utilisation de l'OSM Data est interne/publique ?

> Attribution is also a specific requirement of ODbL.

^ l'attribution est très importante, et il y a [des guidelines spécifiques assez détaillées sur le sujet](https://wiki.osmfoundation.org/wiki/Licence/Attribution_Guidelines).

> Can I charge for distributing OSM data or data derived from OSM data? Yes. You can charge any amount of money you want for any service or data you provide. However, since the data (or service) that is derived from OSM data must be licensed as above, other people may then redistribute this without payment.

^ Cette formulation montre que la licence est contaminante.

> Can I copy data from Google? No! There are no services provided by Google that have terms of service or licenses that allow them to be used. Further Google services are not a reliable source of geographic data, they are vehicles for transporting advertising and gathering behavioural data and tend to be biased in many ways. Do not forget that OpenStreetMap is not a project to explore legal grey areas of copyright and contract law, it is a global project to provide free to use, open and legally untainted geodata. Please do not endanger it by trying to take shortcuts.

^ Ha ha, c'te réponse :-D

> Can i use osm data and openstreetmap-derived maps to verify my own data without triggering share-alike ?
>
> Yes, provided that you are only comparing and do not copy any OpenStreetMap data. If you make any changes to your data after making the comparison, you should be able to reasonably demonstrate that any such change was made either from your own physical observation or comes from a non-OpenStreetMap source accessed directly by you. I.e you can compare but not take!
>
> Example 1: You notice that a street is called one name on your map and another in OpenStreetMap. You should visit the street and check the name, then you are free to put that name in your data as it is your own observation.
>
> Example 2: You notice that a boundary is different in your data and OpenStreetMap. You should check back to original authoritative sources and make any correction required.

[Le lien direct](https://wiki.osmfoundation.org/wiki/Licence/Licence_and_Legal_FAQ#4._CAN_I_USE_OSM_DATA_AND_OPENSTREETMAP-DERIVED_MAPS_TO_VERIFY_MY_OWN_DATA_WITHOUT_TRIGGERING_SHARE-ALIKE?)

^ le premier exemple montre que la donnée OSM doit servir en interne à déclencher le fait de récupérer de la donnée **par un autre moyen** (ici, la donnée OSM pointe du doigt les endroits où il y a des soucis sur les noms des rues, mais on va agréger les noms des rues corrects **par un autre moyen**).

Dit autrement, même en interne, on ne peut pas incorporer la donnée OSM, juste "choisir d'incorporer une autre donnée sur la base d'information OSM".


## https://wiki.osmfoundation.org/wiki/Licence/Community_Guidelines

Pas mal de guidelines spécifiques sur certains cas, très intéressantes, notamment :

- [guideline sur geocoder (qui s'appliquera sans doute à un routing engine)](https://wiki.osmfoundation.org/wiki/Licence/Community_Guidelines/Geocoding_-_Guideline)
- [quand un projet est-il un Produced Work](https://wiki.osmfoundation.org/wiki/Licence/Community_Guidelines/Produced_Work_-_Guideline)
- [quand un projet est-il une Collective Database ?](https://wiki.osmfoundation.org/wiki/Licence/Community_Guidelines/Collective_Database_Guideline_Guideline), c'est plus subtil et moins restrictif que je ne pensais

### https://wiki.osmfoundation.org/wiki/Licence/Attribution_Guidelines

Guideline sur comment correctement respecter la clause d'attribution.

> While attribution is required, the way it must be done depends on the type of map, database, or other creation involved, and how it is presented.

^ la façon dont on s'y conforme dépend du produit dont il est question.

> Note that attribution is only necessary when a Produced Work is used Publicly (as defined by the ODbL). This guideline does not concern internal uses.

^ pas d'obligation pour une utilisation interne.

> The base requirement in ODbL is attribution that is “…reasonably calculated to make any Person that uses, views, accesses, interacts with, or is otherwise exposed to the Produced Work aware that Content was obtained from [OpenStreetMap] …”.

^ en gros, l'idée c'est d'être fair-play et de s'assurer que l'utilisateur saura qu'il utilise de la donnée OSM.

> Requirements to fit within OSMF’s safe harbour

^ détails de comment attribuer.

> Attribution must also make it clear that the data is available under the Open Database License. This may be done by making the text “OpenStreetMap” a link to openstreetmap.org/copyright,

^ le plus safe est d'avoir un lien.

> Safe harbour requirements for specific scenarios

^ Il y a des paragraphes particuliers pour certains cas :

- Databases
- Interactive maps
- Static images
- Geocoding (search)
- Routing engines

## https://learn.microsoft.com/en-us/bingmaps/articles/open-maps-understanding-odbl

> Do I have to share my data if I use ODbL data?
>
> You will not have to automatically share your data if you use data under the ODbL license in your application.
>
> If you simply use the data as-is, without modifying the database or creating a hybrid database that combines multiple datasets, it is unlikely that you will have to share your contributed data.
>
> If you segregate your data from ODbL-licensed data or just use ODbL-licensed data to create a map, it is unlikely that you would have to share your segregated data or the output map.
>
> However, if you combine a number of sources into a single, unified database or modify an existing ODbL database using other information, it is likely that you would need to release the resulting database under the ODbL license, which means that other people could see your data and use it.

## https://doc.transport.data.gouv.fr/presentation-et-mode-demploi-du-pan/conditions-dutilisation-des-donnees

> Nous recommandons toutefois aux producteurs de favoriser l’emploi de la licence ouverte Etalab, puisqu’elle permet au mieux de favoriser la réutilisation des données.

Le [PAN (= Point d’Accès National aux données de transport)](https://transport.data.gouv.fr/) trouve que le caractère contaminant de l'ODbL est trop contraignant.
