# pipeline pattern

il y a des moment où nous avons besoin de faire suivre une séquence d’opérations, dans la majorité des cas, vous entendrez parler de pipeline. Même si de manière stricte le pipeline n’est pas un pattern (celui qui lui ressemble le plus et que nous allons implémenter est chain of responsibility pattern) il capture bien l’idée de faire les chose une après l’autre avec une dépendance de l’étape précédente

regardons le pattern chain of responsability [code chain responsibility](https://github.com/JonathanNdambaPro/code_adanced_course/blob/main/design%20patterns/pipeline%20pattern/chain_responsibility.py)

## Chain responsibility

```py
from __future__ import annotations

from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Optional


class Handler(ABC):
    def __init__(self):
        self.next: Optional[Handler] = None

    def set_next(self, handler: Handler) -> None:
        self.next = handler

    def handle_click_event(self) -> None:
        try:
            if self.on_click() and self.next:
                self.next.handle_click_event()
        except AttributeError:
            pass

    @abstractmethod
    def on_click(self) -> bool:
        """Handle a click event."""
```
ici on a une classe abstraite qui servira d’interface qui a plusieurs composante :

- self.next : définie le handler suivant, c’est comme si on avait une liste mais au lieu d’y accéder part l’index on y accède uniquement si on est au précédent qui point vers lui.
- set\_next(self, handler : Handler) : sert à pointer vers le suivant
- handle\_click\_event(self) : cette méthode va faire deux chose :
  - self.on\_click() : vérifier si cette méthode retourne vrai
  - self.next : vérifie si le suivant existe équivalent à ce qu’il ne soit pas à non
  - self.next.handle\_click\_event() : demande au suivant de faire exactement la même chose
- on\_click(self) : méthode abstraite qui impose l’implémentation au autre classe

```py
@dataclass
class Button(Handler):
    name: str = "button"
    activate: bool = True

    def on_click(self) -> bool:
        if self.activate:
            print(f"Button [{self.name}] handling click.")
            return True
        return False


@dataclass
class Panel(Handler):
    name: str = "panel"
    activate: bool = True

    def on_click(self) -> bool:
        if self.activate:
            print(f"Panel [{self.name}] handling click.")
            return True
        return False


@dataclass
class Window(Handler):
    name: str = "window"
    activate: bool = True
    def on_click(self) -> bool:
        if self.activate:
            print(f"Window [{self.name}] handling click.")
            return True
        return False
```

chaque classe à son implémentation de on\_click() qui revoit un booléen ce qui est logique par rapport à la classe mère

```py
def main() -> None:
    button = Button(name="my_button", activate=True)
    panel = Panel(name="my_panel")
    window = Window(name="my_window")

    # setup the chain of responsibility
    button.set_next(panel)
    panel.set_next(window)

    button.handle_click_event()


if __name__ == "__main__":
    main()
```

on initialise chacun des handlers (Button, Panel, Window)

Nous allons voir une façon de coder des fonctions de manière séquentielle en utilisant la composition.

## Compose

[code compose](https://github.com/JonathanNdambaPro/code_adanced_course/blob/main/design%20patterns/pipeline%20pattern/composition.py)
```py
from functools import partial, reduce
from typing import Callable

ComposableFunction = Callable[[float], float]

# Helper function for composing functions
def compose(*functions: ComposableFunction) -> ComposableFunction:
    return reduce(lambda f, g: lambda x: g(f(x)), functions)


def add_three(x: float) -> float:
    return x + 3


def multiply_by_two(x: float) -> float:
    return x * 2


def add_n(x: float, n: float) -> float:
    return x + n


def main():
    x = 12
    # oldres = multiplyByTwo(multiplyByTwo(addThree(addThree(x))))
    myfunc = compose(
        partial(add_n, n=3), partial(add_n, n=2), multiply_by_two, multiply_by_two
    )
    result = myfunc(x)
    print(result)


if __name__ == "__main__":
    main()
```

regardons plus en détail les fonctions

```py
def add_three(x: float) -> float:
    return x + 3


def multiply_by_two(x: float) -> float:
    return x * 2


def add_n(x: float, n: float) -> float:
    return x + n
```
ces fonctions sont assez simples, on a une fonction qui ajoute 3 une qui mutiplie par 2 et une dernière qui additionne n

```py
ComposableFunction = Callable[[float], float]

# Helper function for composing functions
def compose(*functions: ComposableFunction) -> ComposableFunction:
    return reduce(lambda f, g: lambda x: g(f(x)), functions)
```
cette fonction est la plus complexe, elle va faire un reduce avec les sorties d’une série de fonctions

![](Aspose.Words.317399d3-a3a6-4d69-9467-bd1dc1674562.007.png)

- l’ordre du premier lambda va préciser que nous prenons deux fonctions en argument
- le deuxième lambda va prendre les fonctions du premier lambda et va indiquer comment les traiter avec la nouvelle entrée x, la fonction dont on a codé indique que les fonctions partent de gauche à droite
- la fonction compose prends des arguments en positionnel de manière illimitée grâce à \*.

```py
def main():
    x = 12
    # oldres = multiplyByTwo(multiplyByTwo(addThree(addThree(x))))
    myfunc = compose(
        partial(add_n, n=3), partial(add_n, n=2), multiply_by_two, multiply_by_two
    )
    result = myfunc(x)
    print(result)


if __name__ == "__main__":
    main()
```
cette dernière fonction permet d’utiliser toutes les fonctions qu’on a vues précédemment, mais la fonction compose ne permet aux fonctions de prendre qu’un seul paramètre alors que la fonction add, elle en demande deux (x et n) dans ce cas, nous sommes obligés d’utiliser partial pour préremplir le paramètre n.

dans la majorité des cas vous n’aurais pas besoin d’implémenter ces fonction car des librairies (Airflow, Luigi, Pandas..) le feront pour vous, mais si vous vouliez faire quelque chose de vraiment basique sans forcément sortir l’artilleries lourdes, vous pourriez utiliser une des méthodes que l’on vient de voire.
