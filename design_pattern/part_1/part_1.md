# Strategy pattern

Un design pattern est une solution éprouvée à un problème courant de conception de logiciel, présentée sous forme d'un modèle générique qui peut être adapté et réutilisé dans divers contextes. Les design patterns sont souvent considérés comme des « recettes » ou des « modes d'emploi » pour résoudre des problèmes courants en programmation. Les développeurs qui connaissent les design patterns peuvent les utiliser pour résoudre rapidement les problèmes de conception et améliorer la qualité et la maintenabilité du code qu'ils écrivent.

Le strategy pattern permet d’adapter le comportement d’une fonction en fonction de sont contexte.

## Sans pattern

[code before](https://github.com/JonathanNdambaPro/code_adanced_course/blob/main/design%20patterns/strategy%20pattern/strategy_before.py)

```py
from dataclasses import dataclass


@dataclass
class Order:
    price: int
    quantity: int

    def compute_total(self, discount_type: str) -> int:
        if discount_type == "percentage":
            discount = int(self.price * self.quantity * 0.20)
        elif discount_type == "fixed":
            discount = 10_00
        else:
            discount = 0
        return self.price * self.quantity - discount


def main() -> None:
    order = Order(price=100_00, quantity=2)
    print(order)
    print(f"Total: ${order.compute_total('percentage')/100:.2f}")


if __name__ == "__main__":
    main()
```
Ici, on voit qu’on a beaucoup d'if-else quand ce genre de cas ce produit c'est un signe qu’un refactoring peut être envisager. Si de nouvelle promotion (discount) s’ajoute alors la fonction grandira sans cesse augmentera la complexité cyclomatique (le nombre de if else et tout autre conditions, plus il y en a plus notre code est complexe et dure à maintenir).

## Avec pattern

[code after](https://github.com/JonathanNdambaPro/code_adanced_course/blob/main/design%20patterns/strategy%20pattern/strategy_classic.py)

```py
from abc import ABC, abstractmethod
from dataclasses import dataclass


class DiscountStrategy(ABC):
    @abstractmethod
    def compute(self, price: int) -> int:
        """Computes the discount for the given price."""


class PercentageDiscount(DiscountStrategy):
    def compute(self, price: int) -> int:
        return int(price * 0.20)


class FixedDiscount(DiscountStrategy):
    def compute(self, _: int) -> int:
        return 10_00


@dataclass
class Order:
    price: int
    quantity: int
    discount: DiscountStrategy

    def compute_total(self) -> int:
        discount = self.discount.compute(self.price * self.quantity)
        return self.price * self.quantity - discount


def main() -> None:
    order = Order(price=100_00, quantity=2, discount=PercentageDiscount())
    print(order)
    print(f"Total: ${order.compute_total()/100:.2f}")


if __name__ == "__main__":
    main()
```


Qu’avons-nous fait ?

- Chaque if else conditions à une classe qui lui est propre
- On a fait une méthode compute pour toutes les fonctions grâce à la classe Abstraite DiscountStrategy qui l’impose à toutes ces classes filles
- La méthode main utilise compute\_total() qui est implémenté dans toutes les classe elles sont dnc interchangeable en fonction de la “stratégie” que l’on veut implémenter (FixedDiscount et PercentageDiscount)

Avec ce pattern, on peut augmenter autant que l’on veut le type de promotion sans que le code ait à se complexifier.

Les design patterns sont souvent implémentés en objet, mais avec le temps la programmation fonctionnelle a pris une grande place, regardons l’équivalent de l’exemple ci-dessus avec des fonction

[code fonctionnel](https://github.com/JonathanNdambaPro/code_adanced_course/blob/main/design%20patterns/strategy%20pattern/strategy_fn.py)

## Avec pattern sous forme fonctionnelle


```py
from dataclasses import dataclass
from typing import Callable

DiscountFunction = Callable[[int], int]


def percentage_discount(price: int) -> int:
    return int(price * 0.20)


def fixed_discount(_: int) -> int:
    return 10_00


@dataclass
class Order:
    price: int
    quantity: int
    discount: DiscountFunction

    def compute_total(self) -> int:
        discount = self.discount(self.price * self.quantity)
        return self.price * self.quantity - discount


def main() -> None:
    order = Order(price=100_00, quantity=2, discount=percentage_discount)
    print(order)
    print(f"Total: ${order.compute_total()/100:.2f}")


if __name__ == "__main__":
    main()
```