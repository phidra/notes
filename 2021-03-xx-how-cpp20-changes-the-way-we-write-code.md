# Timur Doumler - How C++20 changes the way we write code - Meeting C++ 2020

- **url** = https://www.youtube.com/watch?v=VK-16tpFQVI
- **type** = vidéo
- **auteur** = [Timur DOUMLER](https://timur.audio/about), dev C++ spécialisé dans l'audio et les tools pour développeurs
- **date de publication** = 2021-02-15
- **source** = [la chaîne youtube des meetings C++](https://www.youtube.com/c/MeetingCPP)
- **tags** = language>cpp ; topics>cpp20


**NOTE** : j'ai pas fini de prendre des notes, mais je pérennise déjà celles que j'ai prises. Reprendre la lecture à **33:00**

**06:00** les 4 avancées majeures du c++ 20 :

- Coroutines
- Concepts
- Range
- Modules

**16:00** bonne explication de comment les coroutines marchent, et de qui fait quoi. En gros, il y a 4 acteurs :

- Code client
- Generator
- Promise (et son promise_type)
- Coroutine_handle (pointant sur une coroutine frame)

Pour que le code client utilise une coroutine, il doit d'abord appeler un Generator, qui créé une Promise (= le conteneur de la valeur yieldée par la coroutine) ainsi qu'une coroutine-frame.

La Promise et le generator vivent sur la stack.

La frame est heap allocated (en effet, elle représente l'état de la coroutine, qui va survivre aux différents yield). Le Generator a un pointeur vers cette frame : le `std::coroutine_handle`

**22:00** L'implémentation de la coroutine-frame (exécutant le code de la coroutine) est faite par le compilo. Pour optimiser, si c'est possible, elle pourra parfois être plutôt stack-allocated ( d'où le fait que les coroutine sont forcément une feature du langage, plutôt que quelque chose proposable par une librairie).


**23:00** seule l'implémentation de la coroutine-frame a été finie à temps pour c++20 : en attendant c++23, la Promise et le Generator doivent être implémentés à la main. Son conseil : ne pas le faire nous même, et se reposer sur une lib, car il y a plein de subtilités et de caveats. Sa suggestion  de lib = [cppcoro](https://github.com/lewissbaker/cppcoro)

Ndm : 3 cas d'usage :

- Générateurs de valeurs
- Pipeline
- AsyncIO
