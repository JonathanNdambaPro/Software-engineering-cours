# Coupling

Le coupling est le concept inverse de cohérence, il définit à quelle point des classes/fonctions/méthodes peuvent avoir besoin l'une de l'autre. Notre objectif et de le baisser au maximum en sachant qu'il ne peut jamais être totalement éliminé (même explication que la cohésion vu précédemment).

[code before](https://github.com/JonathanNdambaPro/code_adanced_course/blob/main/introduction/coupling/order_before.py)

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


def add_item(order: Order, name: str, quantity: int, price: int) -> None:
    order.items.append(name)
    order.quantities.append(quantity)
    order.prices.append(price)


def compute_total_price(order: Order) -> None:
    total = 0
    for i in range(len(order.prices)):
        total += order.quantities[i] * order.prices[i]
    print(f"The total price is: ${(total / 100):.2f}.")


def main() -> None:
    order = Order()
    add_item(order, "Keyboard", 1, 5000)
    add_item(order, "SSD", 1, 15000)
    add_item(order, "USB cable", 2, 500)

    compute_total_price(order)


if __name__ == "__main__":
    main()
```

Ici, on a affaire à un content coupling, la fonction add\_item prends en argument order et le modifie.

Cette fonction ne respecte pas le principe de "Law of Demeter" qui dit que nos fonctions ne devrait pas connaitre la structure précise d'un argument si elles ne sont pas liées.

add\_item doit savoir qu'un objet de la classe Order possède attributs items, quantities et prices.

La solution et d'implémenté une méthode add\_item dans la classe Order. Comme la méthode et les attributs sont dans la même classe la connaissance de ces derniers est justifiable.

On peut faire la même observation pour la fonction compute\_total\_price

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


def add_item(order: Order, name: str, quantity: int, price: int) -> None:
    order.items.append(name)
    order.quantities.append(quantity)
    order.prices.append(price)


def compute_total_price(order: Order) -> None:
    total = 0
    for i in range(len(order.prices)):
        total += order.quantities[i] * order.prices[i]
    print(f"The total price is: ${(total / 100):.2f}.")


def main() -> None:
    order = Order()
    add_item(order, "Keyboard", 1, 5000)
    add_item(order, "SSD", 1, 15000)
    add_item(order, "USB cable", 2, 500)

    compute_total_price(order)


if __name__ == "__main__":
    main()
```

Il existe différent type de coupling :

- Global coupling : quand un variable global intervient dans différentes fonctions, dans le cas quand où on voudrait séparer les fonction dans différent fichier, on serait obliger de prendre cette constantes dans tous les fichiers ce qui mène à une duplication
- External coupling : si on utilise une api externe, on en dépend, si elle changent nous devrons mettre à jour notre code
- Data coupling : c'est quand les fonctions partagent les mêmes data à travers un même paramètre.

Certains coupling sont nécessaires, mais certains sont indispensables pour le bon fonctionnement du code, l'idée est de faire des aller-retour pour voir si certain de ces coupling peuvent être réduit.
