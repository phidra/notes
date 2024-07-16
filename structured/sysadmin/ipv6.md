# Désactiver ipv6

## Pourquoi ?

Parce que certains outils (notamment, VPN...) ne gèrent pas ipv6 et nécessitent de rester en ipv4 pour fonctionner.

## Vérifier si ipv6 est activé

```sh
ip addr | grep inet6

# si ipv6 est disabled, sera vide
# si ipv6 est enabled, contiendra notamment :
# inet6 ::1/128 scope host
```

## Désactiver/réactiver ipv6 temporairement

```sh
# Désactiver :
sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1
sudo sysctl -w net.ipv6.conf.lo.disable_ipv6=1
sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1

# Réactiver :
sudo sysctl -w net.ipv6.conf.all.disable_ipv6=0
sudo sysctl -w net.ipv6.conf.lo.disable_ipv6=0
sudo sysctl -w net.ipv6.conf.default.disable_ipv6=0
```

## Pérenniser la désactivation

Éditer `/etc/sysctl.conf` :

```sh
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6=1
net.ipv6.conf.default.disable_ipv6 =
```
