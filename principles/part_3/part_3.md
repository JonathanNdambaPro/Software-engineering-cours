# Simplicité

Le but de tout développeur est de faire en sorte que son code soit simple possible,

la réduction de coupling, l’augmentation de cohésion, tout ce que nous avons vu jusqu’à présent nous dirige vers ce paradigme.

Ici, nous allons voir quelques principes simples qui vont vous aider à écrire du code simple, compréhensible et maintenable.

## DRY (Don’t repeat yourself)

Ce concept parle de lui-même, si vous code présent beaucoup de duplication/répétition on doit trouver un moyen de le réduire.

[code before](https://github.com/JonathanNdambaPro/code_adanced_course/blob/main/introduction/simplicity/DRY/dry_before.py)

```py
def read_vehicle_type() -> str:
    vehicle_types = ["vw", "bmw", "tesla"]
    vehicle_type = ""
    while vehicle_type not in vehicle_types:
        vehicle_type = input(
            f"What type of vehicle would you like to rent ({', '.join(vehicle_types)})? "
        )
    return vehicle_type


def read_vehicle_color() -> str:
    vehicle_colors = ["black", "red", "blue"]
    vehicle_color = ""
    while vehicle_color not in vehicle_colors:
        vehicle_color = input(
            f"What color vehicle would you like to rent ({', '.join(vehicle_colors)})? "
        )
    return vehicle_color


def read_rent_days() -> int:
    """Reads the number of days from the user."""
    days = 0
    while days < 1:
        days_str = input(
            "How many days would you like to rent the vehicle? (enter a positive number) "
        )
        try:
            days = int(days_str)
        except ValueError:
            print("Invalid input. Please enter a number.")
    return days


def read_kms_to_drive() -> int:
    """Reads the number of kilometers to drive from the user."""
    km = 0
    while km < 1:
        km_str = input(
            "How many kilometers would you like to drive (enter a positive number)? "
        )
        try:
            km = int(km_str)
        except ValueError:
            print("Invalid input. Please enter a number.")
    return km


def main():

    vehicle_type = read_vehicle_type()

    vehicle_color = read_vehicle_color()

    days = read_rent_days()

    kms = read_kms_to_drive()

    print(
        f"You rented a {vehicle_type} vehicle, color {vehicle_color}, for {days} days",
        f"and you're allowed to drive {kms} kilometers.",
    )


if __name__ == "__main__":
    main()
``` 

Globalement ces fonction sont très redondantes, les fonction font quasiment toute la même chose, mais avec des valeur qui diffère

[Code after](https://github.com/JonathanNdambaPro/code_adanced_course/blob/main/introduction/coupling/order_after.py)

```py
from enum import Enum, auto


class PaymentStatus(Enum):
    """Payment status"""

    OPEN = auto()
    PAID = auto()


class Order:
    def __init__(self):
        self.items: list[str] = []
        self.quantities: list[int] = []
        self.prices: list[int] = []
        self.status: str = "open"

    def add_item(self, name: str, quantity: int, price: int) -> None:
        self.items.append(name)
        self.quantities.append(quantity)
        self.prices.append(price)

    @property
    def total_price(self) -> int:
        total = 0
        for i in range(len(self.prices)):
            total += self.quantities[i] * self.prices[i]
        return total


def main() -> None:
    order = Order()
    order.add_item("Keyboard", 1, 5000)
    order.add_item("SSD", 1, 15000)
    order.add_item("USB cable", 2, 500)

    print(f"The total price is: ${(order.total_price / 100):.2f}.")


if __name__ == "__main__":
    main()
```
Étapes :

- les fonctions read\_vehicule\_type() et read\_vehicule\_color() sont des fonctions qui font la même chose à part le message dans input et leur choix, en mettant le message et les choix dans des paramètres, on a la fonction read\_choice() qui est assez générique pour traiter les deux.
- C’est exactement la même chose les fonctions read\_rent\_days() et read\_kms\_drive() il suffit de stocker le message dans input et les valeurs numériques (days, kms) dans une variable numérique et la fonction read\_int() est suffisamment générique.

## KISS (Keep It simple stupid)

Le nom est choc est, c'est volontaire, le principe KISS vous exige de coder de la manière la plus simple possible.

[code before](https://github.com/JonathanNdambaPro/code_adanced_course/blob/main/introduction/simplicity/KISS/kiss_before.py)

```py
from dataclasses import dataclass
from enum import Enum, auto
from typing import Protocol


class PaymentStatus(Enum):
    """Payment status"""

    OPEN = auto()
    PAID = auto()


@dataclass
class LineItem:
    name: str
    quantity: int
    price: int

    @property
    def total_price(self) -> int:
        return self.quantity * self.price
```
cette data classe prend toutes les commandes et calcule le prix total

```py
class Discount(Protocol):
    def compute_discount(self, price: int) -> int:
        """Computes the discount, given a price."""


@dataclass
class FixedDiscount:
    discount: int = 0

    def compute_discount(self, price: int) -> int:
        return self.discount


@dataclass
class VariableDiscount:
    discount_percentage: float = 0

    def compute_discount(self, price: int) -> int:
        return int(self.discount_percentage * price)
```

Plusieurs classes qui permettent de calculer les promotion, pour information un protocol est dans son fonctionnement similaire à une classe abstraite (vous pourriez d’ailleurs en utiliser une à la place) en moins restrictif.
```py
class Order:
    def __init__(self):
        self.items: list[LineItem] = []
        self.status: PaymentStatus = PaymentStatus.OPEN
        self.discounts: list[Discount] = []

    def add_item(self, name: str, quantity: int, price: int) -> None:
        self.items.append(LineItem(name, quantity, price))

    def add_discount(self, discount: Discount) -> None:
        self.discounts.append(discount)

    @property
    def total_price(self) -> int:
        total = 0
        for item in self.items:
            total += item.total_price
        total_discount = 0
        for discount in self.discounts:
            total_discount += discount.compute_discount(total)
        return total - total_discount


def create_order(items: list[LineItem], discounts: list[Discount]):
    order = Order()
    for item in items:
        order.add_item(item.name, item.quantity, item.price)
    for discount in discounts:
        order.add_discount(discount)
    return order
```

La classe Order qui va calculer le total.

il y a beaucoup de classe qui sont générés par le fait que nous devions ajouter des réductions, ce qui rend le code beaucoup plus difficile à lire et comprendre,

```py
def main() -> None:
    order = create_order(
        [
            LineItem("Keyboard", 1, 5000),
            LineItem("SSD", 1, 15000),
            LineItem("USB cable", 2, 500),
        ],
        [VariableDiscount(0.1), FixedDiscount(1000), VariableDiscount(0.05)],
    )
    print(f"The total price is: ${(order.total_price / 100):.2f}.")


if __name__ == "__main__":
    main()
```

Et finalement la fonction main qui assemble tout, personnellement, je trouve ce design très dur à comprendre, rien que la manière dont sont calculées les promotions n’est pas triviale à comprendre. Voyons comment nous pouvons le rendre plus simple.

Les classes discounts sont dans notre code ne font que renvoyer des entiers à partir d’un autre entier, est-ce que cela est bien nécessaire de créer une classe pour quelque chose d’aussi simple ? Bien que nous ayons et allons apprendre plusieurs méthodes, leur but est le même, rendre notre code le plus clair et lisible possible et là ce n’est pas le cas.

[code after](https://github.com/JonathanNdambaPro/code_adanced_course/blob/main/introduction/simplicity/KISS/kiss_after.py)

```py
from dataclasses import dataclass
from enum import Enum, auto


class PaymentStatus(Enum):
    """Payment status"""

    OPEN = auto()
    PAID = auto()


@dataclass
class LineItem:
    name: str
    quantity: int
    price: int

    @property
    def total_price(self) -> int:
        return self.quantity * self.price


class Order:
    def __init__(self):
        self.items: list[LineItem] = []
        self.status: PaymentStatus = PaymentStatus.OPEN
        self.fixed_discount: int = 0
        self.variable_discount: float = 0.0

    def add_item(self, item: LineItem) -> None:
        self.items.append(item)

    @property
    def total_price(self) -> int:
        sub_total = sum(item.total_price for item in self.items)
        discount = int(self.fixed_discount + self.variable_discount * sub_total)
        return sub_total - discount


def main() -> None:
    order = Order()
    order.add_item(LineItem("Keyboard", 1, 5000))
    order.add_item(LineItem("SSD", 1, 15000))
    order.add_item(LineItem("USB cable", 2, 500))
    order.variable_discount = 0.15
    order.fixed_discount = 1000

    print(f"The total price is: ${(order.total_price / 100):.2f}.")


if __name__ == "__main__":
    main()
```

Notre code ici est beaucoup plus simple est aussi beaucoup plus court :

- Order prend la responsabilité qu’avaient les classes discounts avec les attributs self.fixed\_discount et self.variable\_discount
- Le calcule de discount est aussi pris en compte ce qui rend le code beaucoup plus compréhensible.

Notre classe à plus de responsabilités, mais est beaucoup plus clair et simple à communiquer entre développeur. Lors du choix de vos optimisation, il peut arriver qu’une approche soutenue par une méthode ne soit pas celle qui rende le code le plus élégant, c’est à vous de déterminer quel solution est la “moins pire”, ce concept de “moins pire” est important, car dans le code en général la meilleure solution n’existe pas ce sont souvent des choix avec des avantages et inconvénient.

Tout ça peu sembler compliqué, mais garder en tête que vous pouvez le faire par incrémentation, d’abord une tentative, puis une autre et ainsi de suite jusqu’à ce que vous ayez un refactoring suffisamment simple pour être maintenu et compris par n’importe qui.

## YAGNI (You ain’t gonna need it)

On dit souvent “qu’on ne peut pas prédire le futur”, c’est faux. Prédire c’est émettre des probabilité sur des possibilités futures, la météo le fait tous les jour par exemple.

Ce qu’on ne peut pas par contre, c'est “savoir le futur”, avoir une certitude sur ce qui va se passer, et en software engineering, c'est un problème.

Plus vous essayerez de prédire des choses plus vous aller rendre votre code complexe car plus de chose à tester plus de fonctionnalité à ajouter et un code plus dur à maintenir.

L’implémentation code doit être justifiée, nous devons faire le minimum de chose possible pour gagner du temps, les différentes méthodes que nous avons vues sont justifiées parce qu'elles rendent le code plus simple et dans le futur les développeurs qui verront votre code le comprendrons quasiment instantanément et ne perdons pas de temps. L’ajout de features qui ne sont pas demandés et qui peut être ne servirons à rien est elle par contre une perte de temps.
