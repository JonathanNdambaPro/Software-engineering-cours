# Cohésion

La cohésion est un concept informatique qui définit à quel point une fonction ou une classe peuvent interagir ou non d'autre partie du code.

Une fonction avec un forte cohésion peut être réutilisée beaucoup de fois, c'est que l'on cherche à faire dans la majorité des cas. Pour ce faire, une fonction doit avoir le moins de responsabilité (idéalement une seule). Une responsabilité est le nombre de choses qu'une fonction/méthode/classe prend en charge.

[code_before](https://github.com/JonathanNdambaPro/code_adanced_course/blob/main/introduction/cohesion/order_before.py)

```py
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

    def pay(self, payment_type: str, security_code: str) -> None:
        if payment_type == "debit":
            print("Processing debit payment type")
            print(f"Verifying security code: {security_code}")
            self.status = "paid"
        elif payment_type == "credit":
            print("Processing credit payment type")
            print(f"Verifying security code: {security_code}")
            self.status = "paid"
        else:
            raise ValueError(f"Unknown payment type: {payment_type}")


def main() -> None:
    order = Order()
    order.add_item("Keyboard", 1, 5000)
    order.add_item("SSD", 1, 15000)
    order.add_item("USB cable", 2, 500)

    order.pay("debit", "0372846")


if __name__ == "__main__":
    main()
```

On voit ci-dessus la classe Order a beaucoup trop de responsabilité 
- Elle contient la responsabilité de tout ce qui entoure un ordre de paiement. 
- Elle contient la responsabilité de tout ce qui entoure le paiement, comme on peut le voir la classe contient deux responsabilité, à long terme sa capacité à être réutilisé sera limité. Il faut donc créer une classe pour chaque
responsabilité


[code_after](https://github.com/JonathanNdambaPro/code_adanced_course/blob/main/introduction/cohesion/order_before.py)

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
        self.status: PaymentStatus = PaymentStatus.OPEN

    def add_item(self, name: str, quantity: int, price: int) -> None:
        self.items.append(name)
        self.quantities.append(quantity)
        self.prices.append(price)


class PaymentProcessor:
    def pay_debit(self, order: Order, security_code: str) -> None:
        print("Processing debit payment type")
        print(f"Verifying security code: {security_code}")
        order.status = PaymentStatus.PAID

    def pay_credit(self, order: Order, security_code: str) -> None:
        print("Processing credit payment type")
        print(f"Verifying security code: {security_code}")
        order.status = PaymentStatus.PAID


def main():
    order = Order()
    order.add_item("Keyboard", 1, 5000)
    order.add_item("SSD", 1, 15000)
    order.add_item("USB cable", 2, 500)

    processor = PaymentProcessor()
    processor.pay_debit(order, "0372846")


if __name__ == "__main__":
    main()
```

plusieurs chose se sont passée ici : 
- Order et Payment sont séparés, chaque classe est plus simple à comprendre, car elle a un objectif bien prédéfinit.
- Les classes Order et Payement interagissent entre elles, comme elles ne peuvent plus prendre beaucoup de responsabilité, on utilise ce qu'on appelle la composition 
- Au niveau de la méthode pay de la classe Order avant la refactorisation, elle devait gérer la partie crédit et débit alors que ces opération sont distincts, on les a également séparés 
- On utilise Enum pour les choix limités 
    - PaymentStatus.OPEN
    - PaymentStatus.PAID

Regardons un autre exemple.

[code_before](https://github.com/JonathanNdambaPro/code_adanced_course/blob/main/introduction/cohesion/vehicle_rental_before.py)

```py
def main():
    print("Vehicle Rental before")

    price_per_km_vw = 30
    price_per_km_bmw = 35
    price_per_km_ford = 25
    price_per_day_vw = 6000
    price_per_day_bmw = 8500
    price_per_day_ford = 12000

    vehicle_type = ""
    while vehicle_type != "vw" and vehicle_type != "bmw" and vehicle_type != "ford":
        vehicle_type = input(
            "What type of vehicle would you like to rent (choose vw, bmw, or ford)? "
        )

    days = 0
    while days < 1:
        days_str = input(
            "How many days would you like to rent the vehicle? (enter a positive number) "
        )
        try:
            days = int(days_str)
        except ValueError:
            print("Invalid input. Please enter a number.")

    km = 0
    while km < 1:
        km_str = input(
            "How many kilometers would you like to drive (enter a positive number)? "
        )
        try:
            km = int(km_str)
        except ValueError:
            print("Invalid input. Please enter a number.")

    # subtract the base number of kms
    km = max(km - 100, 0)

    # compute the final rental price
    rental_price = 0
    if vehicle_type == "vw":
        rental_price = price_per_km_vw * km + price_per_day_vw * days
    elif vehicle_type == "bmw":
        rental_price = price_per_km_bmw * km + price_per_day_bmw * days
    elif vehicle_type == "ford":
        rental_price = price_per_km_ford * km + price_per_day_ford * days
    else:
        print("Error")

    # print the result
    print(f"The total price of the rental is ${(rental_price / 100):.2f}")


if __name__ == "__main__":
    main()
```


Ici la fonction main à beaucoup trop de responsabilité 
- Le type de véhicule 
- Le nombre de jours de location 
- Le nombre de kilomètres 
- Le calcule de coût

Comme on l'a dit que ce soit une classe/methode/fonction on doit faire tout notre possible pour avoir le minimum de responsabilité

[code_after](https://github.com/JonathanNdambaPro/code_adanced_course/blob/main/introduction/cohesion/vehicle_rental_after.py)

```py
EHICLE_DATA = {
    "vw": {"price_per_km": 30, "price_per_day": 6000},
    "bmw": {"price_per_km": 35, "price_per_day": 8500},
    "ford": {"price_per_km": 25, "price_per_day": 12000},
}


def read_vehicle_type() -> str:
    """Reads the vehicle type from the user."""
    vehicle_type = ""
    while vehicle_type not in VEHICLE_DATA:
        vehicle_type = input(
            f"What type of vehicle would you like to rent ({', '.join(VEHICLE_DATA)})? "
        )
    return vehicle_type


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


def compute_rental_cost(vehicle_type: str, days: int, km: int) -> int:
    """Computes the rental cost for a vehicle."""
    vehicle_data = VEHICLE_DATA[vehicle_type]
    price_per_km = vehicle_data["price_per_km"]
    price_per_day = vehicle_data["price_per_day"]
    paid_kms = max(km - 100, 0)
    return price_per_km * paid_kms + price_per_day * days


def main():
    print("Vehicle Rental before")

    vehicle_type = read_vehicle_type()

    days = read_rent_days()

    km = read_kms_to_drive()

    # compute the final rental price
    rental_price = compute_rental_cost(vehicle_type, days, km)

    # print the result
    print(f"The total price of the rental is ${(rental_price / 100):.2f}")


if __name__ == "__main__":
    main()
```
- On a juste décomposé la fonction en sous fonction pour chaque responsabilité observée. 
- On a rendu plus générique la partie qui gère le calcule
- On a donné la responsabilité du prix par kilomètre et prix par jour à la constante VEHICULE_DATA

La cohésion est un des deux objectifs les plus importants de la programmation propre, tous les principes de software design ont pour but de maximiser la cohésion entre les fonctions et de minimiser le coupling (prochain cours). Cependant, il est important de préciser que c’est deux principes ne sont pas forcément faciles voir possible, par exemple la partie main() souffre de manque de cohésion, car elle utilise les autres fonctions, et donc la réutiliser pour un autre but est très peu probable.

La méthode que nous vous recommandons est de d’abord coder vite de manière que le code soit fonctionnelle et dans un second temps faire une repasse de code, c’est la phase de refactoring, vous pouvez faire plusieurs repasse petit à petit plutôt que d’essayer de faire tout en une fois. 


