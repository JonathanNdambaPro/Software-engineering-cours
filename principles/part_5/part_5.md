Composition over inherence

L'héritage en orienté objet sert à ce qu'une classe fille soit une spécialisation d'une classe mère, beaucoup de gens l'oublie et utilise pour créer de nouvelles responsabilités, ce qui est une très mauvaise à long terme en réduisant la cohérence et augmentant le coupling.

[code before](https://github.com/JonathanNdambaPro/code_adanced_course/blob/main/introduction/composition_over_inherence/before.py)

```py
"""
Very advanced Employee management system.
"""

from dataclasses import dataclass


@dataclass
class HourlyEmployee:
    name: str
    id: int
    commission: int = 10000
    contracts_landed: float = 0
    pay_rate: int = 0
    hours_worked: float = 0
    employer_cost: int = 100000

    def compute_pay(self) -> int:
        return int(
            self.pay_rate * self.hours_worked
            + self.employer_cost
            + self.commission * self.contracts_landed
        )


@dataclass
class SalariedEmployee:

    name: str
    id: int
    commission: int = 10000
    contracts_landed: float = 0
    monthly_salary: int = 0
    percentage: float = 1

    def compute_pay(self) -> int:
        return int(
            self.monthly_salary * self.percentage
            + self.commission * self.contracts_landed
        )


@dataclass
class Freelancer:

    name: str
    id: int
    commission: int = 10000
    contracts_landed: float = 0
    pay_rate: int = 0
    hours_worked: float = 0
    vat_number: str = ""

    def compute_pay(self) -> int:
        return int(
            self.pay_rate * self.hours_worked + self.commission * self.contracts_landed
        )


def main() -> None:

    henry = HourlyEmployee(name="Henry", id=12346, pay_rate=5000, hours_worked=100)
    print(f"{henry.name} earned ${(henry.compute_pay() / 100):.2f}.")

    sarah = SalariedEmployee(
        name="Sarah", id=47832, monthly_salary=500000, contracts_landed=10
    )
    print(f"{sarah.name} earned ${(sarah.compute_pay() / 100):.2f}.")


if __name__ == "__main__":
    main()
```

comme on le voit il y a beaucoup de point commun entre chaque classe, notre code est quelque peu redondant, une bonne méthode serait d'utiliser l'héritage

[code héritage](https://github.com/JonathanNdambaPro/code_adanced_course/blob/main/introduction/composition_over_inherence/with_inheritance.py)

```py
"""
Very advanced Employee management system.
"""

from dataclasses import dataclass


@dataclass
class HourlyEmployee:
    name: str
    id: int
    pay_rate: int
    hours_worked: float = 0
    employer_cost: int = 100000

    def compute_pay(self) -> int:
        return int(self.pay_rate * self.hours_worked + self.employer_cost)


@dataclass
class SalariedEmployee:
    name: str
    id: int
    monthly_salary: int
    percentage: float = 1

    def compute_pay(self) -> int:
        return int(self.monthly_salary * self.percentage)


@dataclass
class Freelancer:
    name: str
    id: int
    pay_rate: int
    hours_worked: float = 0
    vat_number: str = ""

    def compute_pay(self) -> int:
        return int(self.pay_rate * self.hours_worked)


@dataclass
class SalariedEmployeeWithCommission(SalariedEmployee):
    commission: int = 10000
    contracts_landed: float = 0

    def compute_pay(self) -> int:
        return super().compute_pay() + int(self.commission * self.contracts_landed)


@dataclass
class HourlyEmployeeWithCommission(HourlyEmployee):
    commission: int = 10000
    contracts_landed: float = 0

    def compute_pay(self) -> int:
        return super().compute_pay() + int(self.commission * self.contracts_landed)


@dataclass
class FreelancerWithCommission(Freelancer):
    commission: int = 10000
    contracts_landed: float = 0

    def compute_pay(self) -> int:
        return super().compute_pay() + int(self.commission * self.contracts_landed)


def main() -> None:

    henry = HourlyEmployee(name="Henry", id=12346, pay_rate=5000, hours_worked=100)
    print(f"{henry.name} earned ${(henry.compute_pay() / 100):.2f}.")

    sarah = SalariedEmployeeWithCommission(
        name="Sarah", id=47832, monthly_salary=500000, contracts_landed=10
    )
    print(f"{sarah.name} earned ${(sarah.compute_pay() / 100):.2f}.")


if __name__ == "__main__":
    main()
```

Cette solution est optimale n'est pas ? Eh bien pas vraiment, il y a toujours autant de classe et également la redondance n'a toujours pas disparu, la raison pour laquelle le gain est si faible, c'est parce que nous essayons par l'héritage de séparer des chose pas vraiment de les spécialiser. La différence est subtile, mais on peut s'en rendre compte de plusieurs façon.

- La classe devient trop grande.
- La classe a trop de responsabilité dans, c'est moment là, il vaut mieux essayer de faire une refactorisation et utiliser la composition plutôt que l'héritage

[code composition](https://github.com/JonathanNdambaPro/code_adanced_course/blob/main/composition_over_inherence/with_composition.py)

```py
from abc import ABC, abstractmethod
from dataclasses import dataclass, field

class PaymentSource(ABC):
  @abstractmethod
  def compute_pay(self) -> int:
        pass


@dataclass
class DealBasedCommission(PaymentSource):

    commission: int = 10000
    deals_landed: int = 0

    def compute_pay(self) -> int:
        return self.commission * self.deals_landed


@dataclass
class HourlyContract(PaymentSource):

    hourly_rate: int
    hours_worked: float = 0.0
    employer_cost: int = 100000

    def compute_pay(self) -> int:
        return int(self.hourly_rate * self.hours_worked + self.employer_cost)


@dataclass
class SalariedContract(PaymentSource):

    monthly_salary: int
    percentage: float = 1

    def compute_pay(self) -> int:
        return int(self.monthly_salary * self.percentage)
```

Cette partie prend la responsabilité du paiement ce qui implique que les classe n’ont plus à le faire, on voit que ses classe sont distinctes (pas du tout les mêmes attributs)

```py
@dataclass
class Employee:

    name: str
    id: int
    payment_sources: list[PaymentSource] = field(default_factory=list)

    def add_payment_source(self, payment_source: PaymentSource):
        self.payment_sources.append(payment_source)

    def compute_pay(self) -> int:
        return sum(source.compute_pay() for source in self.payment_sources)


def main() -> None:

    henry_contract = HourlyContract(hourly_rate=5000, hours_worked=100)
    henry = Employee(name="Henry", id=12346, payment_sources=[henry_contract])
    print(f"{henry.name} earned ${(henry.compute_pay() / 100):.2f}.")

    sarah_contract = SalariedContract(monthly_salary=500000)
    sarah_commission = DealBasedCommission(deals_landed=10)
    sarah = Employee(
        name="Sarah", id=47832, payment_sources=[sarah_contract, sarah_commission]
    )
    print(f"{sarah.name} earned ${(sarah.compute_pay() / 100):.2f}.")


if __name__ == "__main__":
    main()
```

Dans cette solution, nous avons séparé notre classe en deux classe distincte, mais qui interagissent entre elles, cette interaction s'appelle une composition. Les classes filles de PaymentSource n'utilisent pas les mêmes attributs, elles sont donc valables, car elles sont "spécialisé"
