# On Cache Invalidation - Yihui Xie | 谢益辉

- **url** = https://yihui.org/en/2018/06/cache-invalidation/
- **type** = post
- **auteur** = Yihui Xie, dev bilingue (chinois+américain) dont [le centre d'intérêt semble être les stats / R](https://yihui.org/).
- **date de publication** = 2018-06-22
- **source** = https://yihui.org/
- **tags** = language-agnostic ; topic>caching


**TL;DR** : son point de vue sur l'invalidation de cache. Son post est surtout intéressant pour son exemple concret :
- sa stratégie de caching de ce qui semble être du code = md5 du contenu ; peut paraître bonne au premier abord, mais en fait pas toujours adaptée
- cache parfois regénéré à tort : (e.g. va regénérer le cache même pour un changement du contenu qui n'est en fait pas significatif, comme un ajout de commentaire de code)
- cache parfois non-regénéré à tort : (e.g. ne va pas regénérer le cache si un fichier externe utilisé par le code est modifié)
- en plus du coût en storage, le fait d'utiliser un cache vient donc avec le coût de "devoir comprendre comment fonctionne le cache pour être sûr de l'invalider quand on le souhaite"

Quote célèbre, de Phil Karlson (apparemment [dans le cadre du dev netscape](https://skeptics.stackexchange.com/questions/19836/has-phil-karlton-ever-said-there-are-only-two-hard-things-in-computer-science)) :

> There are only two hard things in Computer Science: cache invalidation and naming thing.

À quoi sert le caching :

> the main purpose of caching is speed. [...]
>
> if you know you are going to compute the same thing, you may just load the result saved from the previous run, and skip the computing this time. There are two keywords here: “the same thing”, and “the saved result”
>
> The latter means you are essentially trading (more) storage space for (less) time. That is the price to pay for caching, and also an important fact to be aware of when you use caching (i.e., caching is definitely not free, and sometimes the price can be fairly high).

La façon dont il faut comprendre la quote, c'est "comment savoir si tu essayes de calculer exactement ce qui a déjà été calculé (et a été caché), ou bien si la réponse a changé ?" :

> The tricky thing is “the same thing”. How do you know that you are computing the same thing? That is all “cache invalidation” is about.
>
> When things become different, you have to invalidate the cache, and do the (presumably time-consuming) computing again.

Il donne un exemple, résumé par :

> The toy example shows the basic idea of implementing caching: turn your input into a key, use this key to retrieve the output in a cache database if it exists, otherwise do the computing and save the output to the database with the key.

Dans son exemple, la stratégie de caching repose sur le hash du contenu :

> The key of a code chunk is pretty much an MD5 hash (via digest::digest()) of the chunk options and the chunk content (code). Whenever you modify chunk options or the code, the hash will change, and the cache will be invalidated.

**Problème n°1** = parfois, cette stratégie regénère le cache trop souvent :

> Some thought it was too sensitive, and some thought it was dumb. For example, when you add a space in an R comment in your code chunk, should knitr invalidate the cache? Modifying a comment certainly won’t affect the computing at all (but the text output may change if you show the code in the output via echo = TRUE)

**Problème n°2** = parfois, cette stratégie ne regénère pas le cache assez souvent :

> Then an example to explain why people thought knitr’s caching was dumb: if you read an external CSV file in a code chunk, knitr does not know whether you have modified the data file. If you happen to have updated the data file, knitr won’t re-read it by default if you didn’t modify chunk options or the code.

**En résumé :**

> The tricky thing is, it is hard to find the balance. Either direction can offend users.

Ce qu'il en déduit, c'est que le "prix" du caching (en plus du storage supplémentaire), c'est de devoir se taper la compréhension de comment fonctionne la stratégie de cache :

> I said above that the obvious price to pay for caching is storage (either in memory or on disk). However, to make caching work perfectly for you, there is a hidden cost. That is, the cost to understand caching.
> 
> If you don’t fully understand how caching works and the conditions for its invalidation, caching could be too sensitive or dumb, and may not serve you well.

Side-node : un trade-off qu'il suggère à ses utilisateurs, pour avoir une stratégie de caching certes basique, mais très simple à comprendre :

> The ultimate suggestion I often give to users is that if you feel knitr’s caching is too complicated, it is totally fine to use a much simpler caching mechanism like this:

```ruby
if (file.exists('results.rds')) {
  res = readRDS('results.rds')
} else {
  res = compute_it()  # a time-consuming function
  saveRDS(res, 'results.rds')
}
```

> In this case, you clearly understand how your caching works. The one and only way to invalidate the cache is to delete results.rds,
