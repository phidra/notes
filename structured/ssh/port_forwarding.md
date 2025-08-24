**TL;DR** :

USECASE 1 = pour rendre accessible le port distant `8877` sur le port `7000` de ma machine locale :

```sh
ssh -i ~/.ssh/luke.dsa -N -L 7000:localhost:8877 luke@coruscant
```

^ avec cette commande, à chaque fois que je consulterai le port `7000` de ma machine locale, je consulterai en fait le port `8877` de coruscant.

USECASE 2 = si ma machine n'a pas accès à une ressource (e.g. `192.168.1.2`) , mais que coruscant y a accès, je peux "passer par" coruscant pour y accéder sur mon poste :

```sh
ssh -i ~/.ssh/luke.dsa -N -L 7000:192.168.1.2:8877 luke@coruscant
```

^ avec cette commadne, à chaque fois que je consulterai le port `7000` de ma machine locale, je consulterai en fait le port `8877` de `192.168.1.2`, mais en passant par coruscant.

Références :

- http://www.linux-france.org/prj/edu/archinet/systeme/ch13s04.html
- https://openclassrooms.com/courses/mise-en-place-d-un-tunnel-tcp-ip-via-ssh

Il y a deux besoins différents pour le port-forwarding, à partir d'une machine à laquelle on a accès (ici, la machine s'appellerait `coruscant`) :

- `ssh -L` = "pull" = _rapatrier un port de coruscant_ = ouvrir un port sur ma propre machine qui reflète en fait un port de `coruscant`
- `ssh -R` = "push" = _exposer un de mes ports_ = ouvrir un port sur `coruscant` qui reflète un port de ma propre machine


# valable pour les deux commandes

```sh
ssh -i ~/.ssh/luke.dsa luke@coruscant
```

Cette partie ne change pas : on peut se connecter à la machine remote (=coruscant) en utilisant le compte `luke` et l'identité `~/.ssh/luke.dsa`

Option `-N` : par défaut, le port forwarding se fait EN PLUS d'une connexion classique ; `-N` indique de se contenter de faire le port forwarding (je trouve qu'on y voit plus clair).

# ssh -L

> Specifies that connections to the given TCP port on the local (client) host are to be forwarded to the given host and port on the remote side.

Ça sert à quoi : `ssh -L` permet d'ouvrir un port sur SA PROPRE MACHINE qui reflète un port d'une machine distante.

Après avoir fait `ssh -L`, je peux faire un curl sur MA PROPRE MACHINE pour accéder en fait à la machine distante

Exemple de situation où ça sert :

- depuis ma machine locale, je veux accéder au port 8877 sur coruscant (à laquelle j'ai un accès ssh)
- iptables bloque l'accès au port 8877 de coruscant depuis le monde extérieur, mais l'autorise depuis coruscant elle-même
- grâce à `ssh -L`, j'accède au port 8877 de coruscant sur ma machine locale, via un tunnel SSH

Commande de base :

```sh
ssh -L LOCALPORT:FORWARDEDHOST:FORWARDEDPORT USER@REMOTEHOST

# exemple :
ssh -L 7000:localhost:8877 luke@coruscant
```

**CAVEAT** : avec la commande ci-dessus, le port 7000 sur ma machine redirige vers coruscant:8877. En effet, c'est contre-intuitif, mais le `localhost` au milieu de la commande est résolu SUR CORUSCANT (c'est "localhost sur coruscant").

Les deux commandes suivantes sont donc (presque) équivalentes :

```sh
ssh -L 7000:localhost:8877 luke@coruscant
ssh -L 7000:coruscant:8877 luke@coruscant
```

Équivalents conceptuel pour mieux comprendre :

```sh
# commande de base :
ssh -L   7000:localhost:8877   luke@coruscant

# équivalent conceptuel :
ssh -L   [PORT LOCAL]   [OÙ DOIT REDIRIGER MON PORT LOCAL SUR LE REMOTE HOST]   [REMOTE HOST]
ssh -L       7000:                    localhost:8877                            luke@coruscant
```


# ssh -R

> Specifies that connections to the given TCP port on the local (client) host are to be forwarded to the given host and port on the remote side.

Ça sert à quoi : `ssh -R` permet d'ouvrir un port sur UNE MACHINE DISTANTE qui reflète un port de ma propre machine.

Après avoir fait `ssh -R`, quelqu'un peut faire un curl sur LA MACHINE DISTANTE pour accéder en fait à ma machine.

Exemple de situation où ça sert :

- je veux faire tester un code local à un utilisateur externe qui n'a pas accès à ma machine, sur le port 80 de ma machine locale
- mais il n'a pas accès à ma machine locale
- en revanche, nous avons tous deux accès à un serveur intermédiaire, sur lequel j'ai moi-même un accès ssh
- grâce à `ssh -R`, l'utilisateur accède à mon port 80 sur le serveur intermédiaire, via un tunnel SSH

